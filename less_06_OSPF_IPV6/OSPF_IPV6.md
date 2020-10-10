# OSPF для IPv6. Основные концепции, зоны, фильтрация



#### Техническое задание

Цель: *Настроить OSPF для IPv6 поверх работающего протокола OSPF, сохранив ту же логику работы*



1. Настроите OSPF для IPv6 поверх OSPF. сохранив ту же логику работы(метрики, таймеры, фильтры)

2. План работы и изменения зафиксированы в документации



##### Топология офиса в Москве





![тут скриншот 1](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_1.jpg)



------

[Полное описание выбранных технических решений можно прочитать тут. Для IPV6 все настройки аналогичны, отличия будут описаны ниже.](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/OSPF_ipv4.md)

------



### 1. Отличия



1.1 Единственным отличием будет настройка фильтрации анонсов **area_101(R19)** **в** **area_102(R20)**. В настройках *ipv6 router ospf* нет фильтрации вида  *area <area number> filter-list prefix <prefix-list-name> [in | out]*. Поэтому было решено применить фильтрацию не на R15, а ниже на самом R20 с помощью фильтра на входящие анонсы в ап-линка eth0/0 вида - *distribute-list prefix-list IPV6_DENY_PREFIX_AREA_101 in Ethernet0/0*. Для этого был создан префикс-лист *IPV6_DENY_PREFIX_AREA_101*.



### 2. Настройки на пограничных маршрутизаторах R14 и R15



##### Настройки на R14



###### Настройки интерфейсов



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.14 255.255.255.255
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:FFFF::14/128
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/0
 description R12
 ip address 10.201.10.1 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:1::1/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 description R13
 ip address 10.201.10.5 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:2::1/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 description R22_AS101_EGP
 ip address 101.0.0.6 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:A:A:1::2/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/3
 description R19
 ip address 10.201.10.21 255.255.255.252
 ip ospf 1 area 101
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:6::1/64
 ipv6 enable
 ipv6 ospf 1 area 101
!
```



###### Настройки OSPF IPV6 и маршрута по умолчанию для IPV6



```
!
ipv6 route ::/0 Ethernet0/2 2001:A:A:1::1 name defualt
!
ipv6 router ospf 1
 area 101 stub no-summary
 default-information originate
 passive-interface Ethernet0/2
 passive-interface Loopback1000
!
```



###### Пример таблицы маршрутизации IPV6 на R14  с маршрутами только от OSPF. Это магистральный роутер и он будет знать обо всех сетях.



![тут скрин 2](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_2.jpg)



##### Настройки на R15 



###### Настройки на интерфейсах



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.15 255.255.255.255
 ip ospf 1 area 0
 ipv6 address FE80::15 link-local
 ipv6 address 2001:F:F1:FFFF::15/128
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/0
 description R13
 ip address 10.201.10.13 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::15 link-local
 ipv6 address 2001:F:F1:4::1/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 description R12
 ip address 10.201.10.9 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::15 link-local
 ipv6 address 2001:F:F1:3::1/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 description R21_AS301_EGP
 ip address 30.1.0.2 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::15 link-local
 ipv6 address 2001:A:C:1::2/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/3
 description R20
 ip address 10.201.10.17 255.255.255.252
 ip ospf 1 area 102
 ipv6 address FE80::15 link-local
 ipv6 address 2001:F:F1:5::1/64
 ipv6 enable
 ipv6 ospf 1 area 102
!
```



###### Настройки OSPF IPV6 и маршрута по умолчанию для IPV6



```
!
ipv6 route ::/0 Ethernet0/2 2001:A:C:1::1 name default
!
ipv6 router ospf 1
 default-information originate
 passive-interface Ethernet0/2
 passive-interface Loopback1000
!
```



###### Пример таблицы маршрутизации IPV6 на R14  с маршрутами только от OSPF. Это магистральный роутер и он будет знать обо всех сетях.



![тут скрин 3](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_3.jpg)



### 3. Настройки на маршрутизаторах уровня distribution R12 и R13



##### Настройки на R12



###### Настройки на интерфейсах



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.12 255.255.255.255
 ip ospf 1 area 0
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:FFFF::12/128
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/0
 no ip address
 ipv6 address FE80::12 link-local
 ipv6 enable
!
interface Ethernet0/0.1000
 description MNG_SW
 encapsulation dot1Q 1000
 ip address 10.201.66.1 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:F:F1:2000::2012/64
 ipv6 enable
 ipv6 ospf 1 area 10
!
interface Ethernet0/0.2001
 description HSRP_VLAN2001_GATE
 encapsulation dot1Q 2001
 ip address 10.201.100.100 255.255.255.0
 standby version 2
 standby 1 ip 10.201.100.1
 standby 1 priority 150
 standby 1 preempt
 ip ospf 1 area 10
 ipv6 address 2001:F:F1:1000::1/64
 ipv6 enable
 ipv6 ospf 1 area 10
!
interface Ethernet0/0.2011
 description HSRP_VLAN2011_GATE
 encapsulation dot1Q 2011
 ip address 10.201.101.201 255.255.255.0
 standby version 2
 standby 2 ip 10.201.101.1
 standby 2 preempt
 ip ospf 1 area 10
 ipv6 address 2001:F:F1:1001::1/64
 ipv6 enable
 ipv6 ospf 1 area 10
!
interface Ethernet0/2
 description R14
 ip address 10.201.10.2 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:1::2/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/3
 description R15
 ip address 10.201.10.10 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:3::2/64
 ipv6 enable
 ipv6 ospf 1 area 0
!
```



###### Настройки OSPF



```
!
ipv6 router ospf 1
 passive-interface Loopback1000
!
```



###### Таблица маршрутизации на R12. Виден маршрут по умолчанию полученный от R14 и R15  согласно техническому заданию.



![тут скриншот 4](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_4.jpg)



##### Настройки на R13 аналогичны R12



###### Таблица маршрутизации на R13. Виден маршрут по умолчанию полученный от R14 и R15  согласно техническому заданию.



![тут скрин 5](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_5.jpg)



### 4. Настройки на R19 и area 101



###### настройки интерфейсов



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.19 255.255.255.255
 ip ospf 1 area 101
 ipv6 address FE80::19 link-local
 ipv6 address 2001:F:F1:FFFF::19/128
 ipv6 enable
 ipv6 ospf 1 area 101
!
interface Ethernet0/0
 description R14
 ip address 10.201.10.22 255.255.255.252
 ip ospf 1 area 101
 ipv6 address FE80::19 link-local
 ipv6 address 2001:F:F1:6::2/64
 ipv6 enable
 ipv6 ospf 1 area 101
!
```



###### Настройки OSPF IPV6



```
!
ipv6 router ospf 1
 area 101 stub
 passive-interface Loopback1000
!
```



###### Настройки area 101 на R14



```
!
ipv6 router ospf 1
 area 101 stub no-summary
!
interface Ethernet0/3
 description R19
 ip address 10.201.10.21 255.255.255.252
 ip ospf 1 area 101
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:6::1/64
 ipv6 enable
 ipv6 ospf 1 area 101
!
```



###### Пример общей таблицы маршрутизации IPV6 на R19, обращаем внимание, что как задано в ТЗ, по OSPF IPV6 получен только маршрут по умолчанию:



![тут скрин 6](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_6.jpg)



### 5. Настройки на R20 и area 102



###### Настройки на интерфейсах



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.20 255.255.255.255
 ip ospf 1 area 102
 ipv6 address FE80::20 link-local
 ipv6 address 2001:F:F1:FFFF::20/128
 ipv6 enable
 ipv6 ospf 1 area 102
!
interface Ethernet0/0
 description R15
 ip address 10.201.10.18 255.255.255.252
 ip ospf 1 area 102
 ipv6 address FE80::20 link-local
 ipv6 address 2001:F:F1:5::2/64
 ipv6 enable
 ipv6 ospf 1 area 102
!
```



###### Настройка OSPF IPV6



```
!
ipv6 router ospf 1
 distribute-list prefix-list IPV6_DENY_PREFIX_AREA_101 in Ethernet0/0
 passive-interface Loopback1000
!
```



###### Настройка prefix-list на R20 для фильтрации анонсов с area 101



```
!
ipv6 prefix-list IPV6_DENY_PREFIX_AREA_101 seq 5 deny 2001:F:F1:FFFF::19/128
ipv6 prefix-list IPV6_DENY_PREFIX_AREA_101 seq 10 deny 2001:F:F1:6::/64
ipv6 prefix-list IPV6_DENY_PREFIX_AREA_101 seq 15 permit ::/0
ipv6 prefix-list IPV6_DENY_PREFIX_AREA_101 seq 20 permit ::/0 ge 1
!
```



###### Таблица маршрутизации на R20 для OSPF IPV6. Обращаем внимание что в ней, согласно техническому заданию, отсутствуют подсети из area 101 - 2001:F:F1:FFFF::19/128 и 2001:F:F1:6::0/64



![тут скриншот 7](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_7.jpg)



###### Сами сети area 101 доступны с R20, т.к. есть маршрут до шлюза по умолчанию и явного указания о их блокировке не было в техническом задании.



![тут скриншот 8](https://github.com/degreekeeper/otus-network/blob/master/less_06_OSPF_IPV6/screenshots/Screenshot_8.jpg)



#### Конец.