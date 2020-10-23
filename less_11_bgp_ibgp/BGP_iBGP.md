# BGP. iBGP.

### Топология

![тут скрин 4](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_4.jpg)



### Техническое задание

Цель: Настроить iBGP в офисе Москва
Настроить iBGP в сети провайдера Триада
Организовать полную IP связанность всех сетей 

​                                 

1. iBGP в офисом Москва между маршрутизаторами R14 и R15
2. Настроите iBGP в провайдере Триада
3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас. 
4. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно
5. Все сети в лабораторной работе должны иметь IP связность
6. План работы и изменения зафиксированы в документации 



### 1. Описание технических решений проекта:

1.1 IPV6 настраивается так же как и IPV4 где это возможно.

1.2 В офисе Москва делаем приоритетным провайдера Ламас с помощью атрибута BGP - local-preference. Для этого через inbound route-map на bgp-сессии на R15 изменяем этот атрибут у всех входящих анонсах от провайдера Ламас на значение 200 с помощью команды ***set local-preference 200***. Этот же route-map используем для IPV6.

1.3 В офисе Москва включаем iBGP между R14 и R15

1.4 В офисе СПБ в BGP включаем настройку  ***maximum-paths 2*** для попактной балансировки трафика между двумя линками(двумя сессиями) до провайдера Триада.

1.5 В провайдере Триада включаем в качестве IGP OSPF с одной зоной. Анонсировать через OSPF будем только линковые сети и loopback для полной связи между всеми роутерами. Делать это будем через включение OSPF на интерфейсах и увода необходимых интерфейсов в passive-режим. Такая архитектура позволит убрать анонсы внешним AS множества мелких внутренних сетей при этом оставив связность внутри AS. Хотя в реальности провайдеры и так бы отфильтрвали все меньше /24, а вот клиенты нет.

1.6 В провайдере Триада после включения IGP, отключим в BGP redistribute connected, что избавивт от необходимости дополнительной фильтрации всех мелких сетей.

1.7 В провайдере Триада в BGP оставим redistribute static для включения в анонсы EGP и IGP соседям агрегированной сети 52.0.0.0/16.

1.8 В провайдере Триада для ухода от full-mesh настроим R23 как router-reflector сервер. Остальные роутеры будут клиентами RR. На RR нейборов до клиентов объединим в peer-group.

1.9 В провайдере Триада в качестве update-source будем использовать loopback 1000.

1.10 В провайдере Триада все iBGP-нейборы будут заменять next-hop на себя с помощью команды ***neighbor x.x.x.x next-hop-self***.

1.11 В провайдере Триада уберем все лишние bgp-сесии. На RR останутся только сессии до клиентов и сессии EBGP. На клиентах только одна сессия до RR и сессии EBGP.



### 2. Настройка IBGP в офисе Москва и настройка приоритетным провайдера Ламас.



#### R14

#### Настройки IBGP

```
!
router bgp 1001
 neighbor 10.201.77.15 remote-as 1001
 neighbor 10.201.77.15 update-source Loopback1000
 neighbor 2001:F:F1:FFFF::15 remote-as 1001
 neighbor 2001:F:F1:FFFF::15 update-source Loopback1000
 !
 address-family ipv4
  neighbor 10.201.77.15 activate
  neighbor 10.201.77.15 next-hop-self
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:F:F1:FFFF::15 activate
  neighbor 2001:F:F1:FFFF::15 next-hop-self
 exit-address-family
!
```

##### Таблица соседств-bgp, таблица bgp и таблица маршрутизации bgp. Как видим таблицы сильно уменьшились и в таблице bgp почти везде лучшим маршрутом выбран Ламас через  R14 за счет LP 200.

![тут скриншот 5](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_5.jpg)

#### R15

#### Настройки IBGP

```
!
router bgp 1001
 bgp log-neighbor-changes
 neighbor 10.201.77.14 remote-as 1001
 neighbor 10.201.77.14 update-source Loopback1000
 neighbor 2001:F:F1:FFFF::14 remote-as 1001
 neighbor 2001:F:F1:FFFF::14 update-source Loopback1000
 !
 address-family ipv4
  neighbor 10.201.77.14 activate
  neighbor 10.201.77.14 next-hop-self
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:F:F1:FFFF::14 activate
  neighbor 2001:F:F1:FFFF::14 next-hop-self
 exit-address-family
!
```



##### Настройки приоритезации провайдера Ламас с помощью local-preference

```
!
route-map LP200 permit 10
 set local-preference 200
!
router bgp 1001
 !
 address-family ipv4
  neighbor 30.1.0.1 route-map LP200 in
 !
 address-family ipv6
  neighbor 2001:A:C:1::1 route-map LP200 in
!
```

##### Таблицы IPV6 соседств-bgp, таблица IPV6 bgp и таблица маршрутизации bgp IPV6. Как видим почти везде выигрывает путь через Ламас за счет LP 200.

![тут скрин 6](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_6.jpg)

##### Трассировка подтверждает маршрут. Путь с VPC1 до VPC8 идёт через  хоп Ламас 30.1.0.1

![тут скрин 7](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_7.jpg)

### 3.Настройка балансировки трафика между двумя линками до провайдера в офисе СПБ на R18



#### R18

##### Трассировка с R18 до VPC1 до настройки балансировки.

![тут скрин 1](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_1.jpg)



##### Настройка балансировка по двум путям до провайдера Триады.

```
!
router bgp 2042 
 address-family ipv4
  maximum-paths 2
 exit-address-family
 !
 address-family ipv6
  maximum-paths 2
 exit-address-family
!
```

##### Трассировка после настройки балансировки

![тут скрин 2](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_2.jpg)

![123](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_3.jpg)

В таблице BGP теперь можно увидеть букву ***m*** напротив префиксов, что означает multipath. Надо отметить что данная балансировка работает, только если as-path и другие атрибуты одинаковые. К примеру в последствии as-path до офиса Москва 10.201.100.0/16 изменился и и балансировка до этой сети больше не работает. Это можно увидеть на скриншоте ниже, буквы ***m*** там нет.

![тут скрин 8](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_8.jpg)



### 4. Настройка IGP OSPF на примере R23 в провайедер Триада

```
!
router ospf 1
 passive-interface Ethernet0/0
 passive-interface Loopback1000
!
!
interface Loopback1000
 ip ospf 1 area 0
 ipv6 ospf 1 area 0
!
interface Ethernet0/0
 ip ospf 1 area 0
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 ip ospf 1 area 0
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 ip ospf 1 area 0
 ipv6 ospf 1 area 0
!
```

##### Таблица маршрутизации только с маршрутами от OSPF и таблица базы OSPF

![тут скрин 9](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_9.jpg)

#### 5.Настройка ROUTE-REFLECTOR у провайдера Триада



#### R23

##### Настройка RR

```
!
router bgp 520
 bgp log-neighbor-changes
 neighbor RR-CLIENTS peer-group
 neighbor RR-CLIENTS remote-as 520
 neighbor RR-CLIENTS update-source Loopback1000
 neighbor IPV6-RR-CLIENTS peer-group
 neighbor IPV6-RR-CLIENTS remote-as 520
 neighbor IPV6-RR-CLIENTS update-source Loopback1000
 neighbor 2001:A:B:FFFF::24 peer-group IPV6-RR-CLIENTS
 neighbor 2001:A:B:FFFF::25 peer-group IPV6-RR-CLIENTS
 neighbor 2001:A:B:FFFF::26 peer-group IPV6-RR-CLIENTS
 neighbor 52.0.52.24 peer-group RR-CLIENTS
 neighbor 52.0.52.25 peer-group RR-CLIENTS
 neighbor 52.0.52.26 peer-group RR-CLIENTS
 !
 address-family ipv4
  redistribute static
  neighbor RR-CLIENTS route-reflector-client
  neighbor RR-CLIENTS next-hop-self
  no neighbor 2001:A:B:FFFF::24 activate
  no neighbor 2001:A:B:FFFF::25 activate
  no neighbor 2001:A:B:FFFF::26 activate
  neighbor 52.0.52.24 activate
  neighbor 52.0.52.25 activate
  neighbor 52.0.52.26 activate
 exit-address-family
 !
 address-family ipv6
  redistribute static
  neighbor IPV6-RR-CLIENTS route-reflector-client
  neighbor IPV6-RR-CLIENTS next-hop-self
  neighbor 2001:A:B:FFFF::24 activate
  neighbor 2001:A:B:FFFF::25 activate
  neighbor 2001:A:B:FFFF::26 activate
 exit-address-family
!
```

Вывод команды ***show ip bgp update-group*** c информацией по роут-рефлектору. А так же информация по соседствам на роут-рефлекторе.

![тут скрин 10](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_10.jpg)



### 6. Настройка клиентов ROUTE-REFLECTOR



#### R24

##### Настройка IBGP

```
!
router bgp 520
 neighbor 2001:A:B:FFFF::23 remote-as 520
 neighbor 2001:A:B:FFFF::23 update-source Loopback1000
 neighbor 52.0.52.23 remote-as 520
 neighbor 52.0.52.23 update-source Loopback1000
 !
 address-family ipv4
  redistribute static
  no neighbor 2001:A:B:FFFF::23 activate
  neighbor 52.0.52.23 activate
  neighbor 52.0.52.23 next-hop-self
 exit-address-family
 !
 address-family ipv6
  redistribute static
  neighbor 2001:A:B:FFFF::23 activate
  neighbor 2001:A:B:FFFF::23 next-hop-self
 exit-address-family
!
```

##### Таблица соседств и таблица маршрутизации BGP

![тут скрин 11](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_11.jpg)



#### R25

##### Настройка IBGP

```
!
router bgp 520
 neighbor 2001:A:B:FFFF::23 remote-as 520
 neighbor 2001:A:B:FFFF::23 update-source Loopback1000
 neighbor 52.0.52.23 remote-as 520
 neighbor 52.0.52.23 update-source Loopback1000
 !
 address-family ipv4
  redistribute static
  neighbor 52.0.52.23 activate
  neighbor 52.0.52.23 next-hop-self
 exit-address-family
 !
 address-family ipv6
  redistribute static
  neighbor 2001:A:B:FFFF::23 activate
  neighbor 2001:A:B:FFFF::23 next-hop-self
 exit-address-family
!
```

##### Таблица соседств IPV6 и таблица маршрутизации BGP IPV6

![тут скрин 12](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_12.jpg)



#### R26

##### Настройка IBGP

```
!
router bgp 520
 neighbor 2001:A:B:FFFF::23 remote-as 520
 neighbor 2001:A:B:FFFF::23 update-source Loopback1000
 neighbor 52.0.52.23 remote-as 520
 neighbor 52.0.52.23 update-source Loopback1000
 !
 address-family ipv4
  redistribute static
  neighbor 52.0.52.23 activate
  neighbor 52.0.52.23 next-hop-self
 exit-address-family
 !
 address-family ipv6
  redistribute static
  neighbor 2001:A:B:FFFF::23 activate
  neighbor 2001:A:B:FFFF::23 next-hop-self
 exit-address-family
!
```

##### Таблица соседств и таблица маршрутизации BGP

![тут скрин 13](https://github.com/degreekeeper/otus-network/blob/master/less_11_bgp_ibgp/screenshots/Screenshot_13.jpg)





#### Конец.