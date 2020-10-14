# EIGRP

Цель: Построение сети и проверка соединения
Настройка маршрутизации EIGRP

Создание сети и настройка основных параметров устройства
Настройка EIGRP и проверка подключения
Настройка EIGRP для автоматического суммирования
Настройка и распространение статического маршрута по умолчанию



#### Топология офиса СПБ:

![тут скрин 1](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_1.jpg)





В этой лабораторной работе необходимо настроить EIGRP:


1. EIGRP в части схемы СПБ
2. EIGRP использует суммарные маршруты, для оптимизации протокола
3. Используется распределение нагрузки по всем возможным линкам
4. Аналогичную работу реализовать для IPv6





### 1. Настройка пограничного роутера R18



1.1. Данный роутер подключен к Провайдеру. Поэтом на нем будут прописаны два статических маршрута с дефолт-гействеем с одинаковой метрикой для балансировки нагрузки. Анонсироваться данные маршрут буду по EIGRP с помощью команды  *redistribute static*

1.2 Будет включена суммаризация через команду auto-summary

1.3 Ipv4 сети будут анонсироваться через команду *network*, ipv6 сети через настройку на интерефейсах *ipv6 eigrp 1*.

1.4 Будет настроена балансировка трафика между маршрутами с неравной дистанцией с помощью команды *variance*. Математически высчитано что будет достаточно коэффициента 2 - *variance 2*.

1.5 Интерфейс looback и интерфейсы в сторону Провайдера будут настроен как пассивные через команду *passive-interface*. Т.к. с них и на них анонсы сетей по eigrp попадать не должны

1.6. Для ipv6 все команды будут аналогичные



##### Примеры настроек R18



###### Настройка интерфейсов

```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.202.77.18 255.255.255.255
 ipv6 address FE80::18 link-local
 ipv6 address 2001:F:F2:FFFF:18::/128
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0
 description R16
 ip address 10.202.10.5 255.255.255.252
 duplex auto
 ipv6 address FE80::18 link-local
 ipv6 address 2001:F:F2:2::1/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/1
 description R17
 ip address 10.202.10.1 255.255.255.252
 duplex auto
 ipv6 address FE80::18 link-local
 ipv6 address 2001:F:F2:1::1/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/2
 description R24_AS520_EGP
 ip address 52.0.0.26 255.255.255.252
 duplex auto
 ipv6 address FE80::18 link-local
 ipv6 address 2001:A:B:7::2/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/3
 description R26_AS520_EGP
 ip address 52.0.0.30 255.255.255.252
 duplex auto
 ipv6 address FE80::18 link-local
 ipv6 address 2001:A:B:8::2/64
 ipv6 enable
 ipv6 eigrp 1
!
```



###### Настройка EIGRP IPV4

```
!
router eigrp 1
 variance 2
 network 10.202.10.0 0.0.0.3
 network 10.202.10.4 0.0.0.3
 network 10.202.77.18 0.0.0.0
 network 52.0.0.24 0.0.0.3
 network 52.0.0.28 0.0.0.3
 redistribute static
 auto-summary
 passive-interface Loopback1000
 passive-interface Ethernet0/2
 passive-interface Ethernet0/3
!
```



###### Настройка статического маршрута для дефолт-гейтвея IPV4

```
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/3 52.0.0.29 name default_R26
ip route 0.0.0.0 0.0.0.0 Ethernet0/2 52.0.0.25 name default_R24
!
```



###### Настройка EIGRP IPV6

```
!
ipv6 router eigrp 1
 passive-interface Loopback1000
 variance 2
 redistribute static
!
```



###### Настройка статического маршрута для дефолт-гейтвея IPV6

```
!
ipv6 route ::/0 Ethernet0/2 2001:A:B:7::1 name default_ipv6_R24
ipv6 route ::/0 Ethernet0/3 2001:A:B:8::1 name default_ipv6_R26
!
```



###### Таблицы топологии EIGRP R18 

![тут скрин 2 и 3](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_2.jpg)



![](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_3.jpg)



###### Таблица маршрутизации EIGRP R18

![тут скрин 4](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_4.jpg)

##### Пример настройки R16 и R17



1.1 Будет включена суммаризация через команду auto-summary

1.2 Ipv4 сети будут анонсироваться через команду *network*, ipv6 сети через настройку на интерефейсах *ipv6 eigrp 1*.

1.3 Будет настроена балансировка трафика между маршрутами с неравной дистанцией с помощью команды *variance*. Математически высчитано что будет достаточно коэффициента 2 - *variance 2*.

1.4 Интерфейс looback и будет настроен как пассивный через команду *passive-interface*. Т.к. с него и на него анонсы сетей по eigrp попадать не должны

1.5 Для ipv6 все команды будут аналогичные



##### Настройки R16

###### Настройка интерфейсов R16

```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.202.77.16 255.255.255.255
 ipv6 address FE80::16 link-local
 ipv6 address 2001:F:F2:FFFF::16/128
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0
 no ip address
 duplex auto
 ipv6 address FE80::16 link-local
 ipv6 enable
!
interface Ethernet0/0.1000
 encapsulation dot1Q 1000
 ip address 10.202.66.2 255.255.255.0
 ipv6 address FE80::16 link-local
 ipv6 address 2001:F:F2:2000::2016/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0.2002
 description HSRP_VLAN2002_GATE
 encapsulation dot1Q 2002
 ip address 10.202.100.200 255.255.255.0
 standby version 2
 standby 1 ip 10.202.100.1
 standby 1 preempt
 ipv6 address FE80::16 link-local
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0.2022
 description HSRP_VLAN2022_GATE
 encapsulation dot1Q 2022
 ip address 10.202.101.101 255.255.255.0
 standby version 2
 standby 2 ip 10.202.101.1
 standby 2 priority 150
 standby 2 preempt
 ipv6 address FE80::16 link-local
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/1
 description R18
 ip address 10.202.10.6 255.255.255.252
 duplex auto
 ipv6 address FE80::16 link-local
 ipv6 address 2001:F:F2:2::2/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/2
 no ip address
 shutdown
 duplex auto
!
interface Ethernet0/3
 description R32
 ip address 10.202.10.13 255.255.255.252
 duplex auto
 ipv6 address FE80::16 link-local
 ipv6 address 2001:F:F2:4::1/64
 ipv6 enable
 ipv6 eigrp 1
!
```



###### Настройка EIGRP IPV4 R16

```
!
router eigrp 1
 variance 2
 network 10.202.10.4 0.0.0.3
 network 10.202.10.12 0.0.0.3
 network 10.202.66.0 0.0.0.255
 network 10.202.77.16 0.0.0.0
 network 10.202.100.0 0.0.0.255
 network 10.202.101.0 0.0.0.255
 auto-summary
 passive-interface Loopback1000
!
```



###### Настройка EIGRP IPV6 R16

```
!
ipv6 router eigrp 1
 passive-interface Loopback1000
 variance 2
!
```



###### Таблицы топологии EIGRP R16 

![тут скрин 5 и 6](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_5.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_6.jpg)



###### Таблица маршрутизации EIGRP R16

![тут скрин 7](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_7.jpg)



##### Настройка R17



###### Настройка интерфейсов R17

```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.202.77.17 255.255.255.255
 ipv6 address FE80::17 link-local
 ipv6 address 2001:F:F2:FFFF::17/128
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0
 no ip address
 duplex auto
 ipv6 address FE80::17 link-local
 ipv6 enable
!
interface Ethernet0/0.1000
 encapsulation dot1Q 1000
 ip address 10.202.66.1 255.255.255.0
 ipv6 address FE80::17 link-local
 ipv6 address 2001:F:F2:2000::2017/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0.2002
 description HSRP_VLAN2002_GATE
 encapsulation dot1Q 2002
 ip address 10.202.100.100 255.255.255.0
 standby version 2
 standby 1 ip 10.202.100.1
 standby 1 priority 150
 standby 1 preempt
 ipv6 address FE80::17 link-local
 ipv6 address 2001:F:F2:1000::1/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0.2022
 description HSRP_VLAN2022_GATE
 encapsulation dot1Q 2022
 ip address 10.202.101.201 255.255.255.0
 standby version 2
 standby 2 ip 10.202.101.1
 standby 2 preempt
 ipv6 address FE80::17 link-local
 ipv6 address 2001:F:F2:1001::1/64
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/1
 description R18
 ip address 10.202.10.2 255.255.255.252
 duplex auto
 ipv6 address FE80::17 link-local
 ipv6 address 2001:F:F2:1::2/64
 ipv6 enable
 ipv6 eigrp 1
!
```



###### Настройки EIGRP для IPV4

```
!
router eigrp 1
 variance 2
 network 10.202.10.0 0.0.0.3
 network 10.202.66.0 0.0.0.255
 network 10.202.77.17 0.0.0.0
 network 10.202.100.0 0.0.0.255
 network 10.202.101.0 0.0.0.255
 auto-summary
 passive-interface Loopback1000
!
```



###### Настройки для EIGRP IPV6 будут идентичны с R17



###### Таблицы топологии EIGRP R17

![](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_8.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_9.jpg)



###### Таблица маршрутизации EIGRP R17

![тут скрин 10](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_10.jpg)

##### Настройка R32



1.1 Будет включена суммаризация через команду auto-summary

1.2 Ipv4 сети будут анонсироваться через команду *network*, ipv6 сети через настройку на интерефейсах *ipv6 eigrp 1*.

1.3 Интерфейс looback и будет настроен как пассивный через команду *passive-interface*. Т.к. с него и на него анонсы сетей по eigrp попадать не должны

1.4 Для ipv6 все команды будут аналогичные



###### Настройка интерфейсов R32

```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.202.77.32 255.255.255.255
 ipv6 address FE80::32 link-local
 ipv6 address 2001:F:F2:FFFF:32::/128
 ipv6 enable
 ipv6 eigrp 1
!
interface Ethernet0/0
 description R16_UPLINK
 ip address 10.202.10.14 255.255.255.252
 duplex auto
 ipv6 address FE80::32 link-local
 ipv6 address 2001:F:F2:4::2/64
 ipv6 enable
 ipv6 eigrp 1
!
```



###### Настройка EIGRP IPV4 R32

```
!
router eigrp 1
 network 10.202.10.12 0.0.0.3
 network 10.202.77.32 0.0.0.0
 auto-summary
 passive-interface Loopback1000
!
```



###### Настройка EIGRP IPV6 R32

```
!
ipv6 router eigrp 1
 passive-interface Loopback1000
!
```



###### Таблицы топологии EIGRP R32

![](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_11.jpg)

![](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_12.jpg)



###### Таблица маршрутизации EIGRP R32



![тут скрин 13](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/new/screenshots/Screenshot_13.jpg)



#### Конец.