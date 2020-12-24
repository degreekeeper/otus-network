# NAT.DHCP.NTP



### Топология



![тут скрин 1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_1.jpg)



#### Техническое задание

Цель: Настроить DHCP в офисе Москва
Настроить синхронизацию времени в офисе Москва
Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах

​                                    

1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адреса автономной системы AS1001

2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042
3. Настроите статический NAT для R20
4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления
5. Настроите статический NAT(PAT) для офиса Чокурдах
6. Настроите DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP
7. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13
8. Все офисы в лабораторной работе должны иметь IP связность
9. План работы и изменения зафиксированы в документации





### 1. Настройка NAT(PAT) в офисе Москва

#### Описание технических решений для задачи:

1. Уходим от анонса серой подсети офиса 10.201.0.0/16. Вместо неё анонсится будут белые подсети предоставленные провайдером это 30.1.10.0/24 и 101.0.10.0/24. Посмотреть их можно в описании в лабе по распределнию адресного пространства по ссылке:

   

[ссылка на описание лабораторной работы по распределению адресного пространства](https://github.com/degreekeeper/otus-network/blob/master/less_12_ip_protocols_v4_v6/IPv4v6.md)

2. NAT будет настроен на пограничных роутерах R14 и R15, провайдерские сети будут настроены на линках как secondary, чтобы не перенастраивать сесии. Эти сети дополнительно будут включены в анонсы BGP с помощью команды ***network x.x.x.x***. 



#### R14

```
!
interface Ethernet0/2
 description R22_AS101_EGP
 ip address 101.0.10.1 255.255.255.0 secondary
!
router bgp 1001

 address-family ipv4

  network 101.0.10.0 mask 255.255.255.0
!

```



#### R15

```
!
interface Ethernet0/2
 description R21_AS301_EGP
 ip address 30.1.10.1 255.255.255.0 secondary
!
router bgp 1001

 address-family ipv4

  network 30.1.10.0 mask 255.255.255.0
!

```



3. NAT outside интерфейсами буду интерфейсы в сторону провайдера - это eth 0/2. Настроены будут с помощью команды ***ip nat outside***. Через эти интерфейсы буду уходить пакеты с уже изменеными адресами источника. Меняться они буду как раз из пула адресов, которые прописаны на этих интерфейсах как secondary.  Команда  ***ip virtual-reassembly*** ***in*** включается на роутере автоматически при включении NAT.

#### R14

```
interface Ethernet0/2

 ip nat outside
 ip virtual-reassembly in
```



#### R15

```
interface Ethernet0/2

 ip nat outside
 ip virtual-reassembly in


```



4. NAT inside интерфейсами будут интерфейсы в сторону R12 и R13, т.к. именно с них будут приходить трафик с серой подсети 10.201.0.0/16. Настроены будут с помощью команды ***ip nat inside***.

#### R14

```
!
interface Ethernet0/0
 description R12
 ip address 10.201.10.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 description R13
 ip address 10.201.10.5 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
```



#### R15

```
!
interface Ethernet0/0
 description R13
 ip address 10.201.10.13 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 description R12
 ip address 10.201.10.9 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
```



5. NAT pool будет настроен из диапазона провайдерских адресов с помощью команды ***ip nat pool POOL_NAME***.

#### R14

```
!
ip nat pool FOR-NAT 101.0.10.2 101.0.10.252 netmask 255.255.255.0
!
```



#### R15

```
!
ip nat pool FOR-NAT 30.1.10.2 30.1.10.253 netmask 255.255.255.0
!
```

6. NAT настраиваем типа overload, т.е. трансляция в один ip-адрес с распределением по tcp/udp портам. В нашем случае это будет не один ip-адрес, а диапазон от провайдера. Как только закончатся все порты на одном внешнем(глобальном) адресе провайдера, трансляция будет происходить в следующий. Общее максимальное количество трансляций равно формуле 

   ```
   (кол-во портов*кол-во глобальных адресов)
   ```

   настраивается это командами

   ***access-list X permit network wild_card***

     ***ip nat inside source list ACL_NAME_OR_NUMBER pool POOL_NAME overload***

команда означает, что будут изменятся адреса на интерфейсах с настройкой inside, будут меняться source адреса попадающие на эти интерфейсы, только если они подпадают под указанный ACL, изменяться будут на глобальные адреса из указанного POOL, по правилу overload (это PAT).

R14

```
!
access-list 1 permit 10.201.0.0 0.0.255.255
!
ip nat pool FOR-NAT 101.0.10.2 101.0.10.252 netmask 255.255.255.0
ip nat inside source list 1 pool FOR-NAT overload
!
```

R15

```
!
access-list 1 permit 10.201.0.0 0.0.255.255
!
ip nat pool FOR-NAT 30.1.10.2 30.1.10.253 netmask 255.255.255.0
ip nat inside source list 1 pool FOR-NAT overload
!
```



#### Проверяем работоспособность резервирования совместно с работой NAT.

Будем пинговать и делать трассировка до адреса в офисе СПБ в AS 2042 на роутере R18 - 52.0.0.30.



Трассировка с VPC7 имеющего серый адрес 10.201.101.3. Пинг успешный. Маршрут идет через R15 и уходит в провайдера AS301 Киторн и дальше до R18.

![тут скрин 2](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_2.jpg)

На R15 видим трансляцию адреса источника в белый адрес из пула - 30.1.10.5. Обратная трассировка пойдет именно с этим адресом назначения.

![тут скрин 3](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_3.jpg)

Изобразим аварию у провайдера Киторн и погасим интерфейс до него на R15.  Увидим как упала BGP-сессия.

![тут скрин 4](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_4.jpg)

сделаем трассировку ещё раз. Видим что маршрут перестроился не сразу и часть пингов пропала, но потом связность восстановилас. Трассировка успешная, но теперь идёт другим маршрутом и через R14, значит транслироваться адреса должны в другой пул

![тут скрин 6](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_6.jpg)

проверяем трансляцию на R14. Видим её и видим что адресе VPC7 теперь транслировался в адрес из пула на R14 - 101.0.10.5

![тут скрин 5](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_5.jpg)



Резервирование успешно работает



### 2. Настройка NAT(PAT) в СПБ.



##### Описание технических решений для задачи:



2.1. Уходим от анонса серой подсети офиса 10.202.0.0/16 по EBGP. Вместо неё анонсится будут белая подсеть предоставленная провайдером это 52.0.10.0/24. Настроена они будут на специально созданом Loopback18. Анонсироваться эта сеть будет по EBGP через команду ***network x.x.x.x***. Посмотреть эти сети в описании в лабе по распределнию адресного пространства по ссылке:

[нажать тут для перехода по ссылке]: https://github.com/degreekeeper/otus-network/blob/master/less_12_ip_protocols_v4_v6/IPv4v6.md	"нажать тут для перехода по ссылке"



#### R18



```
!
interface Loopback18
 description NAT
 ip address 52.0.10.1 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
!

router bgp 2042
 !
 address-family ipv4
  network 52.0.10.0 mask 255.255.255.0
 exit-address-family
 !
```



2.2 Настраиваем интерфейсы на R18 в сторону R16 и R17 как **ip nat inside**. Интерфейсы в сторону провайдера как **ip nat outside**.



```
!
interface Ethernet0/0
 description R16
 ip address 10.202.10.5 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 address 2001:F:F2:2::1/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/1
 description R17
 ip address 10.202.10.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 address 2001:F:F2:1::1/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/2
 description R24_AS520_EGP
 ip address 52.0.0.26 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 address 2001:A:B:7::2/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/3
 description R26_AS520_EGP
 ip address 52.0.0.30 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::18 link-local
 ipv6 address 2001:A:B:8::2/64
 ipv6 enable
 ipv6 eigrp 1
!

```





2.3  Сам NAT настраиваем в режиме overload, т.к. dynamic настроить через интерфейс невозможно, трансляция будет происходить в подсеть интерфейса Loopback18. Так же настраиваем ACL в котором будет весь диапазон внуртренних адресов.



```
!
ip nat inside source list 101 interface Loopback18 overload
!
access-list 101 permit ip 10.202.0.0 0.0.255.255 any
!
```



#### 





#### Проверяем работоспособность резервирования совместно с работой NAT.



##### Проверим резервирвоание на  случай аварии на R18 до провайдера или у провайдера.



Делаем трассировку с VPC 10.202.101.2 до 101.0.10.253. Видим что хост доступен и трассировка идёт через R16-R18-R26.



![тут скрин 7 - 1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_7_VPC8_1.jpg)



Смотрим трансляцию на R16.  Адрес назначения 10.202.101.2 транслировался в адрес 52.0.11.7



Изображаем аварию на R18, погасив интерфейс eth0/3 в сторону R26.



![тут скрин 8 VPC when fail](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_8_VPC8_when_fail.jpg)



Делаем трассировку и видим, что теперь маршрут выглядит как R16-R18-R24. На участке до R18 ничего не изменилось, трансляция идёт по прежнему через R16(так и должно быть), а вот активный линк до провайдера изменился.



![тут скрин 11 R18 fail 3](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_11_R18_fail_3.jpg)



### 3. Настройка static-NAT на R20 (реализация двойного NAT)



##### Описание технических решений для задачи:



3.1  На R20 будет настроен статический NAT один в один. Внутренний ip Loopback1000 10.201.77.20 будет транслироваться в серый внешний ip 10.201.10.18. Это NAT, но фактически тут нет внешнего ip как такового не будет.  Inside-интерфейсом будет Loopback1000, outside-интерфейсом будет порт в сторону R15.



### R20



#### Настройки на интерфейсах

```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.20 255.255.255.255
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/0
 description R15
 ip address 10.201.10.18 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
```



#### Настройки static NAT



```
!
ip nat inside source static 10.201.77.20 10.201.10.18
!
```



3.2 Т.к. в пункте 1 по ТЗ мы убрали анонс серой подсети 10.201.0.0/16 внешним соседям из други AS - чтобы работал наш статический NAT на R20 на R15 нужно организовать дополнительное преобразование в внешнюю сеть. Иначе с R20 не будет доступа во внешние подсети. Для этого мы интерфейс на R15 в сторону R20 настраиваем как inside, после чего трафик с него будет ещё раз транслировать адреса отправителя уже в белую подсеть 30.0.10.0/24 по NAT-overload, который мы настроили в пункте 1.  Это будет видного в его таблице трансляций



### R15



#### Настройки на интерфейсе



```
!
interface Ethernet0/3
 description R20
 ip address 10.201.10.17 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
```



##### Демонстрация доступности внешних интерфейсов и таблиц NAT на R20  и R15



Пропингуем и пустим трассировку до R16 адрес 52.0.11.1 и 52.0.10.1 на R17. Видим что адреса доступны. Трассировка уходит через R20-R15-R21 через провайдера Ламас



![тут скрин 12-1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_1.jpg)



Смотрим трансляции на R20. Видим что трансляция статическая до обоих адресов назначения офиса СПБ.



![тут скрин 12-2](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_2.jpg)



Смотрим вторую трансляцию на R15. Видим что внешний адрес первой трансляции на R20 - 10.201.10.18 выступает здесь внутренним адресом и транслируется уже во внешний 30.1.10.5.



![тут скрин 12-3](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_3.jpg)



##### Демонстрация доступности при аварии на линке в сторону провайдера и изменении этого провайдера с Ламас на Киторн.



Изобразим аварию до провайдера Ламас. Погасим интерфейс на R15 в сторону Ламас.



![тут скрин 12-4](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_4_fail.jpg)



Делаем снова пинг и трассировку до R16 адреса 52.0.11.1. Видим что маршрутизация перестроилась не сразу и адрес был недоступен, но через некоторое время трассировка перестроилась и адрес запинговался. Трассировка идёт через Киторн R20-R15-R12-R14-R22



![тут скрин 12-6](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_6.jpg)



Смотрим трассировку на R15 и видим что трансляции нет, т.к. маршрут не уходит в Ламас.



![тут скрин 12-5-1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_5_1.jpg)



А вот на R14 появилась трансляция 10.201.10.18 во внешний адрес Киторна 101.0.10.6



![тут скрин 12-5](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_12_r20_nat_5.jpg)



### 4. Настройка NAT для удаленного доступа на R19.



##### Описание технических решений для задачи:



4.1 Для удаленного доступа на R19 настроим статический NAT на R14. Выберем один адрес из диапазона выбранных Провайдером Киторн. Так же этот адреас исключим из диапазона адресов, которые могут выдаваться случайным образом из даипазона для PAT, что мы настроили в пункте 1. Это будет 101.0.10.253.  Удаленный доступ будет осуществляться с помощью протокола telnet. Для этого будет организован проброс порта в статическом NAT с tcp 23 в tcp 23. Внутренним адресом на R19 выберем адрес loopback 1000. Интерфейс на R14 в сторону R19 будет настроен как inside nat.



### R14



#### настройки static NAT проброса



```
!
ip nat inside source static tcp 10.201.77.19 23 101.0.10.253 23
!
```



4.2 Для сохранениея условий резервирования на R15 так же настроим аналогичный static NAT, на случай если произойдет авария у провайдера Киторн (стык с R14). Адресация будет аналогичной, порты так же.



### R15



#### настройки static NAT проброса



```
!
ip nat inside source static tcp 10.201.77.19 23 101.0.10.253 23
!
```







### 5. Статический NAT в офисе Чокурдах



##### Описание технических решений для задачи:

5.1 Т.к. у нас в NAT должны попадать два диапазона адресов, а так же должно продолжать работать резервирование провайдера по средствам PBR+ IP SLA - NAT будет настроен через два правила, где будут использоваться два route-map.



5.2. Настройка интерфейсов на R28. Саб-интерфейсы в сторону хостов будут настроены как **ip nat inside**. Интерфейсы в сторону провайдера будут настроены как **ip nat outside**.



```
!
interface Ethernet0/0
 description R26_AS520_EGP
 ip address 52.0.0.38 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 duplex auto
 ipv6 address FE80::28 link-local
 ipv6 address 2001:A:B:B::2/64
 ipv6 enable
!
interface Ethernet0/1
 description R25_AS520_EGP
 ip address 52.0.0.34 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 duplex auto
 ipv6 address FE80::28 link-local
 ipv6 address 2001:A:B:A::2/64
 ipv6 enable
!
interface Ethernet0/2.2003
 description HOST_VLAN2003
 encapsulation dot1Q 2003
 ip address 10.203.100.1 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map R25
 ipv6 address FE80::28 link-local
 ipv6 address 2001:F:F3:1000::1/64
 ipv6 enable
!
interface Ethernet0/2.2033
 description HOST_VLAN2033
 encapsulation dot1Q 2033
 ip address 10.203.101.1 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
 ip policy route-map R26
 ipv6 address FE80::28 link-local
 ipv6 address 2001:F:F3:1001::1/64
 ipv6 enable
!
```



5.3 Настройка NAT, а так же ACL под NAT и route-map под NAT



```
!
ip nat inside source route-map R25_FOR_NAT interface Ethernet0/1 overload
ip nat inside source route-map R26_FOR_NAT interface Ethernet0/0 overload
!
route-map R25_FOR_NAT permit 10
 match ip address VPC30_VLAN2003_2
 match interface Ethernet0/1
!
route-map R26_FOR_NAT permit 10
 match ip address VPC30_VLAN2003_2
 match interface Ethernet0/0
!
ip access-list standard VPC30_VLAN2003_2
 permit 10.203.101.0 0.0.0.255
 permit 10.203.100.0 0.0.0.255
 deny   any
!
```



##### Проверка резервирования в случае аварии у провайдера и работоспособности NAT:



Делаем пинги и трейсы с VPC30 и VPC31 до адреса под проброс порта на R19 - 101.0.10.253. Видим что адрес доступен, а так же видим что трассировка маршрута для VPC30 идёт через R25, а для VPC31 идёт через R26. Это работает балансировка нагрузки через PBR, настроенная в прошлой лабораторной



![тут скрин2-1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_1.jpg)



Проверяем трансляции NAT на R28. Видим трансляции icmp в диапазон интерфейсов eth0/0 и eth0/1 в соответствии с нашими настройками. Так же видим трансляции udp, это трансляции тестов udp-echo настроенных в IP SLA.



![тут скрин2-2](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_2.jpg)



Изобразим аварию у провайдера на линке до R25. Погасим на нем интерфейс в сторону Чокурдах



![тут скрин2-3](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_3.jpg)



Видим что на R28 упал тест и сработал IP SLA убрав маршрут по умолчанию до этого интерфейса



![тут скрин2-4](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_4.jpg)



Ещё раз делаем пинг с VPC30 и VPC31 и видим что резервирование работает. Удалённый адрес по прежнему доступен с обоих хостов. Так же видим что на VPC30 маршрут перелег с R25 на R26. На VPC31 - R26 и так был приоритетным маршрутом.



![тут скрин2-5](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_5.jpg)



Проверяем трансляции на R28 и видим, что сейчас они происходят только в диапазон оставшегося интерфейса до R26.



![тут скрин2-6](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_6.jpg)



Теперь изобразим аварию на другом линке до R26. Только в этот раз изобразим не падение линка, а недоступность ip-адреса. Для этого удалим ip-адрес с интерфейса на R26.



![тут скрин2-7](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_7.jpg)



Снова на R28 фиксируем ошибку теста и недоступный IP SLA для R26



![тут скрин2-8](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_8.jpg)



Ещё раз проверяем доступность удаленного ip-адреса с VPC30 и VPC31. Адрес доступен. Так же видим, что трассировка маршрут теперь проходит, только через R25. На VPC30 это и так был основной маршрут, а вот на VPC31 маршрут перешёл с R26 на R25



![тут скрин2-9](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_9.jpg)



Ну и последнее убедимся, что на R28 сейчас трансляции NAT идут только через диапазон интерфейса до R25



![тут скрин2-10](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot2_10.jpg)



### 6. Настройка DHCP в офисе Москва на R12 и R13



##### Описание технических решений для задачи:



6.1 Адреса будут выдаваться из пула сети преназначенной для хостов это 10.201.100.0/24 и 10.201.101.0/24. Исключаем из пула адрес шлюза HSRP и адреса на саб-интерфейсах. Так же будем выдавать по DHCP маршрут по умолчанию и адрес DNS-сервера.



### R12



#### Настройки DHCP



```
!
ip dhcp excluded-address 10.201.100.1
ip dhcp excluded-address 10.201.100.100
ip dhcp excluded-address 10.201.101.1
ip dhcp excluded-address 10.201.101.201
!
ip dhcp pool VLAN2001
 network 10.201.100.0 255.255.255.0
 default-router 10.201.100.1
 dns-server 10.201.100.1
!
ip dhcp pool VLAN 2011
 network 10.201.101.0 255.255.255.0
 default-router 10.201.101.1
 dns-server 10.201.101.1
!
```



#### R13



```
!
ip dhcp excluded-address 10.201.100.200
ip dhcp excluded-address 10.201.100.1
ip dhcp excluded-address 10.201.101.101
ip dhcp excluded-address 10.201.101.1
!
ip dhcp pool VLAN2001
 network 10.201.100.0 255.255.255.0
 default-router 10.201.100.1
 dns-server 10.201.100.1
!
ip dhcp pool VLAN2011
 network 10.201.101.0 255.255.255.0
 default-router 10.201.101.1
 dns-server 10.201.101.1
!

```



6.2 На  VPC1 и VPC7 выставим получать адрес автоматически



Видим что адреса получены автоматически для VPC1, а так же маршрут по умолчанию и DNS-сервер.



![тут скрин 13_1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_1.jpg)




На VPC7 аналогично, только адрес выдаётся из другой подсети:



![тут скрин 13_2](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_2.jpg)



Видим привязку этих адресов на R12, т.к. именно он сейчас является активным по HSRP:



![тут скрин 13_3](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_3.jpg)



Изобразим аварию на линке до R12



![тут скрин 13_4](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_4.jpg)



После обновления аренды адресов на VPC1 и VPC7 удаленные хосты так же доступны



![тут скрин 13_5 и 13_6](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_5.jpg)



![13-6](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_6.jpg)



Видим привязку адресов уже на активном R13



![тут скрин 13_7](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_13_7.jpg)





### 7. Настройка NTP в офисе Москва



##### Описание технических решений для задачи:





7.1 Настроим на R12 и R13 NTP-сервера. Тайм-зону выставим UTC+3. Броадкаст рассылка будет осуществляться с loopback интерфейсов



R12



```
!
clock timezone MSK 3 0
!
ntp master 1
ntp update-calendar
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.12 255.255.255.255
 ntp broadcast
!
```



R13



```
!
clock timezone MSK 3 0
!
ntp master 1
ntp update-calendar
!
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.13 255.255.255.255
 ntp broadcast
!
```



7.2 Остальные роутеры настроим на синхронизацию NTP-серверами на R12 и R13.



Для пример R14. На всем сетевом оборудовании настройки будут аналогичные



```
!
ntp update-calendar
ntp server 10.201.77.12
ntp server 10.201.77.13
!
```





Проверим статус синхронизации на R14 c помощью команд ***show ntp status*** и ***show ntp associations***. Видим что синхронизирован с R13, но так же в списке серверов есть и адрес R12.



![тут скрин 14_1](https://github.com/degreekeeper/otus-network/blob/master/less_14_nat_dhcp_ntp/screenshots/Screenshot_14_1.jpg)







#### Конец.