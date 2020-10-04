#  OSPF. Основные концепции, зоны, фильтрация.



##### Техническое Задание

Цель: *Настроить OSPF офисе Москва*
*Разделить сеть на зоны*
*Настроить фильтрацию между зонами*



1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
3.  Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
4.  Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101
5. План работы и изменения зафиксированы в документации 



Топология офиса в Москве

![тут скриншот 1](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_1.jpg)



### 1. Настройка пограничных с провайдерами роутеров R14 и R15

1.1 Согласно нашей концепции выбираем процессом ospf 1

1.2 Не настраиваем сети которые будут передаваться по OSPF через команду network. Все сети добавляются через комануды на интерфейсах *ip ospf 1 area XXX*.

1.3 Пока не настроено подключение к Провайдеру шлюзом по умолчанию в виде теста настраивается статический маршрут на каждом из роутеров с next-hop интерфейсом смотряющим в сторону Провайдера. 

1.4 Данные роутеры должны анонсировать маршрут по умолчанию всем другим роутерам офиса. Поэтому на них настраивается команда *default-information originate*.

1.5 Интерфейс-loopback 1000 и интерфейс в сторону Провайдера настраиваются как passive-interface, т.к. с них и на них не должны приходить пакеты OSPF.

1.6 Между собой роутеры буду объединены областью area 0, которая будет являться backbone-областью для OSPF

1.7 Так же для подключение внешних роутеров из других областей OSPF на R14 будет настроена area 101 на одном из интерфейсов, на R15 настроена area 102. Соответственно R14 и R15 будут считаться ABR роутерами.

1.8 На R12 и R13 в аноносы OSPF будет добавлен каждый саб-интерфейс настроенный под хосты и vlan-управления с их подсетями. Данные интерфейсы будут в *area 10*. Данная область будет standart area, т.е. получать все виды LSA.

1.9 R19 будет находится в area 101. Данная область будет настроена в режиме tottaly stub, т.к. должна получать анонс только маршрута по умолчанию

1.10 R20 будет находится в area 102. Данная область будет настроена в режиме standart. Но выше на R15 будет применен фильтр-OSPF через ip prefix-list, чтобы убрать анонсы сетей area 101, согласно ТЗ.

### 2. Приведем примеры настроек для R14

###### Настройка статического маршурт по шлюз по умолчанию:



```
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/2 101.0.0.5 name default
!
```



###### Настройка процесса OSPF



```
!
router ospf 1
 area 101 stub no-summary
 passive-interface Ethernet0/2
 passive-interface Loopback1000
 default-information originate
!
```



###### Настройка интерфейсов



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.14 255.255.255.255
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:FFFF::14/128
 ipv6 enable
!
interface Ethernet0/0
 description R12
 ip address 10.201.10.1 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:1::1/64
 ipv6 enable
!
interface Ethernet0/1
 description R13
 ip address 10.201.10.5 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:2::1/64
 ipv6 enable
!
interface Ethernet0/2
 description R22_AS101_EGP
 ip address 101.0.0.6 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::14 link-local
 ipv6 address 2001:A:A:1::2/64
 ipv6 enable
!
interface Ethernet0/3
 description R19
 ip address 10.201.10.21 255.255.255.252
 ip ospf 1 area 101
 ipv6 address FE80::14 link-local
 ipv6 address 2001:F:F1:6::1/64
 ipv6 enable
!
```



###### Настройка роутера R15 идентичны, кроме маршрута по умолчанию, интерфейса в сторону area 102 и фильтры(о фильтре подробно будет написано в пункте 4.)



```
!
interface Ethernet0/3
 description R20
 ip address 10.201.10.17 255.255.255.252
 ip ospf 1 area 102
 ipv6 address FE80::15 link-local
 ipv6 address 2001:F:F1:5::1/64
 ipv6 enable
!
router ospf 1
 area 102 filter-list prefix DENY_PREFIX_AREA_101 in
 passive-interface Ethernet0/2
 passive-interface Loopback1000
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/2 30.1.0.1 name default
!
```

###### Пример таблицы маршрутизации на R14 и R15 с маршрутами только от OSPF. Эти роутеры магистральные и буду знать обо всех сетях.



![тут скриншоты 2 и 3](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_2.jpg)



![](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_3.jpg)



### 3. Настройка роутеров R12 и R13



#### Примере настроек R12 



###### настройка процесса OSPF



```
!
router ospf 1
 passive-interface Loopback1000
!
```



###### Настройки интерфейсов



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.12 255.255.255.255
 ip ospf 1 area 0
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:FFFF::12/128
 ipv6 enable
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
!
interface Ethernet0/2
 description R14
 ip address 10.201.10.2 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:1::2/64
 ipv6 enable
!
interface Ethernet0/3
 description R15
 ip address 10.201.10.10 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:3::2/64
 ipv6 enable
!
```

###### Таблица маршутизации R12 с маршрутами только от OSPF. Это вывод команды show ip route ospf. Согласно ТЗ, данный роутер получил маршрут по умолчанию от R14 и R15. Так же маршруты о сетях R13 получены видны через сабинтерфейс из area 10 которой и принадлежат интерфейсы R12 и R13.



![тут скриншот 4](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_4.jpg)



#### R13 настроен аналогично R12.



###### Приведем таблицу маршрутизации R13



![тут скриншот 5](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_5.jpg)



### 4. Настройка R19 и area 101



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
!
interface Ethernet0/0
 description R14
 ip address 10.201.10.22 255.255.255.252
 ip ospf 1 area 101
 ipv6 address FE80::19 link-local
 ipv6 address 2001:F:F1:6::2/64
 ipv6 enable
!
```



###### настройки процесса OSPF



```
!
router ospf 1
 area 101 stub
 passive-interface Loopback1000
!
```



###### настройка на R14 для area 101



```
!
router ospf 1
 area 101 stub no-summary
!
```



###### Пример общей таблицы маршрутизации на R19, обращаем внимание, что как задано в ТЗ, по OSPF получен только маршрут по умолчанию:



![тут скриншот 6](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_6.jpg) 



### 5. Настрйоки R20  и area 102



###### настройки интерфейсов



```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.20 255.255.255.255
 ip ospf 1 area 102
 ipv6 address FE80::20 link-local
 ipv6 address 2001:F:F1:FFFF::20/128
 ipv6 enable
!
interface Ethernet0/0
 description R15
 ip address 10.201.10.18 255.255.255.252
 ip ospf 1 area 102
 ipv6 address FE80::20 link-local
 ipv6 address 2001:F:F1:5::2/64
 ipv6 enable
!
```



###### настройка процесса OSPF



```
!
router ospf 1
 passive-interface Loopback1000
!
```



###### Настройки prefix-list на вышестоящем  R15 для фильтрации сетей из area 101. Обращаем внимание на необходимость в конце добавлять префикс под все сети, иначе буду отфильтрованы все сети.



```
!
ip prefix-list DENY_PREFIX_AREA_101 seq 5 deny 10.201.77.19/32
ip prefix-list DENY_PREFIX_AREA_101 seq 10 deny 10.201.10.20/30
ip prefix-list DENY_PREFIX_AREA_101 seq 15 permit 0.0.0.0/0 ge 1
!
```



###### настройка процесса OSPF на R15. Обращаем внимание, что добавлен фильтр по prefix-list для area 102.



```
!
router ospf 1
 area 102 filter-list prefix DENY_PREFIX_AREA_101 in
 passive-interface Ethernet0/2
 passive-interface Loopback1000
 default-information originate
!
```



###### Таблица маршрутизации на R20. Обращаем внимание на отсутствие сетей 10.201.10.20/30 и 10.201.77.19/32 из area 101 При этом есть все остальные подсети.



![тут скриншот 7](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_7.jpg)



###### При этом эти сети доступны по пингу, т.к. есть дефолтный маршрут и маршрутизация происходит через него.



![тут скриншот 8](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/new/screenshots/Screenshot_8.jpg)



#### Конец.



