# BGP. Базовые настройки.



![тут скрин 1 топология](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_1.jpg)



##### Техническое задание:

Цель: Настроить BGP между автономными системами
Организовать доступность между офисами Москва и С.-Петербург



1. eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
2. Настроите eBGP между провайдерами Киторн и Ламас
3. Настроите eBGP между Ламас и Триада
4. eBGP между офисом С.-Петербург и провайдером Триада
5. Организуете IP доступность между офисами Москва и С.-Петербург
6. Настроите отслеживание линка через технологию IP SLA
7. План работы и изменения зафиксированы в документации 





### 1. Концепция проекта:

1.  IPV6 настраивается так же как IPV4 где это возможно.

2.  Вместо настройки IP SLA в офисах применяется команда принудительной разрыва BGP-сессии  ***neighbor x.x.x.x fall-over***. При проверки отрабатывает корректно и позволяет разорвать BGP-соседство сразу же после падения лика.

3.  В офисах Москва и СПБ провайдерам будет анонсироваться суммарный маршрут. Для этого мы выбирали изначально в проекте такую адресацию, чтобы уменьшить таблицу маршрутизации. Поэтому будут анонсироваться только сеть 10.201.0.0/16 и 10.202.0.0/16 с помощью команды ***network***.  Т.к. BGP анонсирует только сети которые есть в таблице маршрутизации, а этих сетей на пограничных роутерах нет - будет применяться специальная настройка в виде статического маршрута в интерфейс Null 0 вида:
   

   ```
   ip route 10.201.0.0 255.255.0.0 Null0
   ipv6 route 2001:F:F1::/48 Null0
   ip route 10.202.0.0 255.255.0.0 Null0
   ipv6 route 2001:F:F2::/48 Null0
   ```

4. Будет перенастроена логика работы EIGRP в СПБ. Т.к. в EIGRP нет такого удобного способа анонсировать маршрут по умолчанию соседям по IGP, как в OSPF с помощью ***default-information originate*** - будет выбран способ настройки шлюза через статические маршруты на R17 и R16.  Это костыль, но при этом рабочий, отказоустойчивость обеспечивается. Иначе не получается принять дефолтный маршрут от провайдеров и передать его другим соседям по EIGRP. В офисе Москва где, как IGP настроен OSPF, таких проблем нет.

5. Все провайдеры (Киторн, Ламас, Триада) будут анонсировать в офисы маршрут по умолчанию через BGP с помощью команды ***neighbor x.x.x.x default-originate***.

6. Провайдеры Киторн и Ламас, так же как и офисы и будут анонсировать только свой суммарный маршрут сетей 52.0.0.0/16 и 30.1.0.0/16, чтобы уменьшить таблицу маршрутизации у себя, у офисов, у соседних провайдеров.

7. Провайдер Триада на данный момент будет анонсировать все свои внунтренние сети, т.к. внутри AS требуется анонс всех сетей, а в офисы только суммарного маршрута. Для этого нужно настраивать фильтрацию, что будет сделано в последующих лабораторных работах.

8. Провайдер Триада на данном этапе не будет иметь полную связанность между всеми своими роутерами, т.к. не настроен iBGP. Из-за этого к примеру R26 не видит сети AS101, т.к. не знает маршрута до next-hop к AS101. R25 в свою очередь не будет видеть сети AS301 по той же причине. Однако техническое задание при этом будет выполняться, связнность офисов Моска и СПБ будет обеспечена.

9. В провайдере Триада будет настроена редистрибуция connected и статических сетей, для обеспечения внутренней связности между inter-AS роутерами с помощью команд - ***redistribute connected*** и ***redistribute static***.



### 2. Настройка роутеров в офисах Москва и СПБ



#### Офис Москва

##### R14

```
!
router bgp 1001
 bgp log-neighbor-changes
 neighbor 2001:A:A:1::1 remote-as 101
 neighbor 2001:A:A:1::1 fall-over
 neighbor 101.0.0.5 remote-as 101
 neighbor 101.0.0.5 fall-over
 !
 address-family ipv4
  network 10.201.0.0 mask 255.255.0.0
  no neighbor 2001:A:A:1::1 activate
  neighbor 101.0.0.5 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:F:F1::/48
  neighbor 2001:A:A:1::1 activate
 exit-address-family
!
ip route 10.201.0.0 255.255.0.0 Null0
!
ipv6 route 2001:F:F1::/48 Null0
!
```

##### R15

```
!
router bgp 1001
 bgp log-neighbor-changes
 neighbor 30.1.0.1 remote-as 301
 neighbor 30.1.0.1 fall-over
 neighbor 2001:A:C:1::1 remote-as 301
 neighbor 2001:A:C:1::1 fall-over
 !
 address-family ipv4
  network 10.201.0.0 mask 255.255.0.0
  neighbor 30.1.0.1 activate
  no neighbor 2001:A:C:1::1 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:F:F1::/48
  neighbor 2001:A:C:1::1 activate
  neighbor 2001:A:C:1::1 default-originate
 exit-address-family
!
ip route 10.201.0.0 255.255.0.0 Null0
!
ipv6 route 2001:F:F1::/48 Null0
!
```



##### BGP-соседства и таблицы BGP на R14 и R15

![тут скриншоты 2 и 3 и 7](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_2.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_3.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_7.jpg)

#### Офис СПБ

##### R18

```
!
router bgp 2042
 bgp log-neighbor-changes
 neighbor 2001:A:B:7::1 remote-as 520
 neighbor 2001:A:B:7::1 fall-over
 neighbor 2001:A:B:8::1 remote-as 520
 neighbor 2001:A:B:8::1 fall-over
 neighbor 52.0.0.25 remote-as 520
 neighbor 52.0.0.25 fall-over
 neighbor 52.0.0.29 remote-as 520
 neighbor 52.0.0.29 fall-over
 !
 address-family ipv4
  network 10.202.0.0 mask 255.255.0.0
  no neighbor 2001:A:B:7::1 activate
  no neighbor 2001:A:B:8::1 activate
  neighbor 52.0.0.25 activate
  neighbor 52.0.0.29 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:F:F2::/48
  neighbor 2001:A:B:7::1 activate
  neighbor 2001:A:B:8::1 activate
 exit-address-family
!
ip route 10.202.0.0 255.255.0.0 Null0
!
ipv6 route 2001:F:F2::/48 Null0
!
```



##### BGP-соседства и таблицы BGP на R18

![тут скриншот 4 и 5 и 8](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_4.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_5.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_8.jpg)

#### Настройки перенастроенного EIGRP на R17  и R16 для корректного анонсирования дефолта. Настроена статика и редистрибуция статики

#### R17

```
!
router eigrp 1
 variance 2
 network 10.202.10.0 0.0.0.3
 network 10.202.66.0 0.0.0.255
 network 10.202.77.17 0.0.0.0
 network 10.202.100.0 0.0.0.255
 network 10.202.101.0 0.0.0.255
 redistribute static
 auto-summary
 passive-interface Loopback1000
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/1 10.202.10.1 name DEFAULT
!
ipv6 route ::/0 Ethernet0/1 2001:F:F2:1::1 name DEAFAULT_IPV6
!
```

#### R16

```
router eigrp 1
 variance 2
 network 10.202.10.4 0.0.0.3
 network 10.202.10.12 0.0.0.3
 network 10.202.66.0 0.0.0.255
 network 10.202.77.16 0.0.0.0
 network 10.202.100.0 0.0.0.255
 network 10.202.101.0 0.0.0.255
 redistribute static
 auto-summary
 passive-interface Loopback1000
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/1 10.202.10.5 name DEFAULT
!
ipv6 route ::/0 Ethernet0/1 2001:F:F2:2::1 name DEFAULT_IPV6
!
```



#### Демонстрация работы новой схемы EIGRP, на примере R17 и отключения его ап-линк интерфейса. 



##### Как видим изначальная трассировка до loopback 52.0.52.24 провайдерского R24  изменилась и пошла через оставшийся путь 10.202.10.5 линка R16-R18. При этом трассировка все равно успешная.



![тут скриншот 6](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_6.jpg)



### 3. Настройка провайдеров Киторн и Ламас

#### Настройка Киторн

```
!
router bgp 101
 bgp log-neighbor-changes
 neighbor 2001:A:A:1::2 remote-as 1001
 neighbor 2001:A:A:2::2 remote-as 301
 neighbor 2001:A:B:5::1 remote-as 520
 neighbor 52.0.0.17 remote-as 520
 neighbor 101.0.0.2 remote-as 301
 neighbor 101.0.0.6 remote-as 1001
 !
 address-family ipv4
  network 101.0.0.0 mask 255.255.0.0
  no neighbor 2001:A:A:1::2 activate
  no neighbor 2001:A:A:2::2 activate
  no neighbor 2001:A:B:5::1 activate
  neighbor 52.0.0.17 activate
  neighbor 101.0.0.2 activate
  neighbor 101.0.0.6 activate
  neighbor 101.0.0.6 default-originate
 exit-address-family
 !
 address-family ipv6
  network 2001:A:A::/48
  neighbor 2001:A:A:1::2 activate
  neighbor 2001:A:A:1::2 default-originate
  neighbor 2001:A:A:2::2 activate
  neighbor 2001:A:B:5::1 activate
 exit-address-family
!
ip route 101.0.0.0 255.255.0.0 Null0
!
ipv6 route 2001:A:A::/48 Null0
!
```




##### Таблица соседств на R22

![тут скрин 9](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_9.jpg)



##### Анонсы которые получаем от офиса Москва и которые отдаем в офис

![тут скрин 10](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_10.jpg)



#### Настройки Ламас



```
!
router bgp 301
 bgp log-neighbor-changes
 neighbor 30.1.0.2 remote-as 1001
 neighbor 2001:A:A:2::1 remote-as 101
 neighbor 2001:A:B:6::1 remote-as 520
 neighbor 2001:A:C:1::2 remote-as 1001
 neighbor 52.0.0.21 remote-as 520
 neighbor 101.0.0.1 remote-as 101
 !
 address-family ipv4
  network 30.1.0.0 mask 255.255.0.0
  neighbor 30.1.0.2 activate
  neighbor 30.1.0.2 default-originate
  no neighbor 2001:A:A:2::1 activate
  no neighbor 2001:A:B:6::1 activate
  no neighbor 2001:A:C:1::2 activate
  neighbor 52.0.0.21 activate
  neighbor 101.0.0.1 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:A:C::/48
  neighbor 2001:A:A:2::1 activate
  neighbor 2001:A:B:6::1 activate
  neighbor 2001:A:C:1::2 activate
  neighbor 2001:A:C:1::2 default-originate
 exit-address-family
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 30.1.0.0 255.255.0.0 Null0
!
ipv6 route 2001:A:C::/48 Null0
!
```

##### Покажем IPV6  BGP-соседства и анонсы в/из офис Моcква

![тут скрин 11 и 12](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_11.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_12.jpg)

### 4. Настройки Триада

#### R23 

```
!
router bgp 520
 bgp log-neighbor-changes
 neighbor 2001:A:B:1::2 remote-as 520
 neighbor 2001:A:B:4::2 remote-as 520
 neighbor 2001:A:B:5::2 remote-as 101
 neighbor 52.0.0.2 remote-as 520
 neighbor 52.0.0.14 remote-as 520
 neighbor 52.0.0.18 remote-as 101
 !
 address-family ipv4
  network 52.0.0.0 mask 255.255.0.0
  redistribute connected
  redistribute static
  no neighbor 2001:A:B:1::2 activate
  no neighbor 2001:A:B:4::2 activate
  no neighbor 2001:A:B:5::2 activate
  neighbor 52.0.0.2 activate
  neighbor 52.0.0.14 activate
  neighbor 52.0.0.18 activate
 exit-address-family
 !
 address-family ipv6
  redistribute connected
  redistribute static
  network 2001:A:B::/48
  neighbor 2001:A:B:1::2 activate
  neighbor 2001:A:B:4::2 activate
  neighbor 2001:A:B:5::2 activate
 exit-address-family
!
ip route 52.0.0.0 255.255.0.0 Null0
!
ipv6 route 2001:A:B::/48 Null0
!
```

##### Таблица маршрутизации R23

![тут скрины 13 и 14](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_13.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_14.jpg)

#### R24

```
!
router bgp 520
 bgp log-neighbor-changes
 neighbor 2001:A:B:3::2 remote-as 520
 neighbor 2001:A:B:4::1 remote-as 520
 neighbor 2001:A:B:6::2 remote-as 301
 neighbor 2001:A:B:7::2 remote-as 2042
 neighbor 52.0.0.10 remote-as 520
 neighbor 52.0.0.13 remote-as 520
 neighbor 52.0.0.22 remote-as 301
 neighbor 52.0.0.26 remote-as 2042
 !
 address-family ipv4
  network 52.0.0.0 mask 255.255.0.0
  redistribute connected
  redistribute static
  no neighbor 2001:A:B:3::2 activate
  no neighbor 2001:A:B:4::1 activate
  no neighbor 2001:A:B:6::2 activate
  no neighbor 2001:A:B:7::2 activate
  neighbor 52.0.0.10 activate
  neighbor 52.0.0.13 activate
  neighbor 52.0.0.22 activate
  neighbor 52.0.0.26 activate
  neighbor 52.0.0.26 default-originate
 exit-address-family
 !
 address-family ipv6
  redistribute connected
  redistribute static
  network 2001:A:B::/48
  neighbor 2001:A:B:3::2 activate
  neighbor 2001:A:B:4::1 activate
  neighbor 2001:A:B:6::2 activate
  neighbor 2001:A:B:7::2 activate
  neighbor 2001:A:B:7::2 default-originate
 exit-address-family
!
ip route 52.0.0.0 255.255.0.0 Null0
!
ipv6 route 2001:A:B::/48 Null0
!
```

##### Посмотрим анонсы в/из офиса СПБ.

![тут скриншот 15](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_15.jpg)



#### R25. В данной лабораторной чисто транзитный роутер.

```
!
router bgp 520
 bgp log-neighbor-changes
 neighbor 2001:A:B:1::1 remote-as 520
 neighbor 2001:A:B:2::2 remote-as 520
 neighbor 52.0.0.1 remote-as 520
 neighbor 52.0.0.6 remote-as 520
 !
 address-family ipv4
  network 52.0.0.0 mask 255.255.0.0
  redistribute connected
  redistribute static
  no neighbor 2001:A:B:1::1 activate
  no neighbor 2001:A:B:2::2 activate
  neighbor 52.0.0.1 activate
  neighbor 52.0.0.6 activate
 exit-address-family
 !
 address-family ipv6
  redistribute connected
  redistribute static
  network 2001:A:B::/48
  neighbor 2001:A:B:1::1 activate
  neighbor 2001:A:B:2::2 activate
 exit-address-family
!
ip route 52.0.0.0 255.255.0.0 Null0
!
ipv6 route 2001:A:B::/48 Null0
!
```



##### Смотрим таблицу маршрутизации. Как и говорилось выше, пока не настроен iBGP у Триады будет не полная маршрутная информация на всех  роутерах. На данном нет маршрутов в сеть 30.1.0.0/16 AS301.

![тут скриншот 16](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_16.jpg)



#### R26

```
!
router bgp 520
 bgp log-neighbor-changes
 neighbor 2001:A:B:2::1 remote-as 520
 neighbor 2001:A:B:3::1 remote-as 520
 neighbor 2001:A:B:8::2 remote-as 2042
 neighbor 52.0.0.5 remote-as 520
 neighbor 52.0.0.9 remote-as 520
 neighbor 52.0.0.30 remote-as 2042
 !
 address-family ipv4
  network 52.0.0.0 mask 255.255.0.0
  redistribute connected
  redistribute static
  no neighbor 2001:A:B:2::1 activate
  no neighbor 2001:A:B:3::1 activate
  no neighbor 2001:A:B:8::2 activate
  neighbor 52.0.0.5 activate
  neighbor 52.0.0.9 activate
  neighbor 52.0.0.30 activate
  neighbor 52.0.0.30 default-originate
 exit-address-family
 !
 address-family ipv6
  redistribute connected
  redistribute static
  network 2001:A:B::/48
  neighbor 2001:A:B:2::1 activate
  neighbor 2001:A:B:3::1 activate
  neighbor 2001:A:B:8::2 activate
  neighbor 2001:A:B:8::2 default-originate
 exit-address-family
!
ip route 52.0.0.0 255.255.0.0 Null0
!
ipv6 route 2001:A:B::/48 Null0
!
```

##### Посмотрим таблицу маршрутизации на этом роутере. Видим что нет сети  101.0.0.0/16 из AS101, из-за того что нет маршрута до next-hop на R23.



![тут скрин 17](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_17.jpg)



### 5. Проверим доступность ПК с офиса СПБ с ПК офиса Москва. И наоборот.

##### пинг из МСК до СПБ

![тут скрин 18](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_18.jpg)

##### пинг из СПБ до МСК

![тут скрин 19](https://github.com/degreekeeper/otus-network/blob/master/less_10_bgp_basic/screenshots/Screenshot_19.jpg)





#### Конец.