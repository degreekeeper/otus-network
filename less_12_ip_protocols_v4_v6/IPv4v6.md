# IPv4/v6

### ![](https://github.com/degreekeeper/otus-network/blob/master/less_12_ip_protocols_v4_v6/screen/TOPOLOGY.jpg)

### Задание:

1. Разработать и задокументировать адресное пространство;

2. Настроить IP адреса на каждом активном порту;

3. Использовать IPv4 и IPv6;

4. Настроить каждый VPC в любом офисе в своем VLAN;

5. Настроить VLAN управления для сетевых устройств;

6. Настроить сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано;

7. Задокументировать все изменения.

   

   ##### Комментарии и пояснения к пунктам задания:

   ###### Пункт 1: 

   1.1.1 Для сетей Провайдеров выделяется сеть ipv6 вида  2001:A:**X**::/48, из которой дополнительно выделяется подсеть 2001:A:**X**:FFFF::/64 для loopback-интерфейсов. Значение  **X** для провайдеров распределяется так: 

   - Киторн - А 

   - Триадан - B

   - Ламас - С

     

    1.1.2 Для сетей Офисов выделяется сеть ipv6 вида 2001:F:F**X**::/48, из которой дополнительно выделяется подсети 2001:F:F**X**:FFFF::/64 для loopback-интерфейсов, 2001:F:F**X**:2000::/64 для менеджмент-интерфейсов сетевого оборудования, 2001:F:F**X**:1000::/64 для хостов офиса. Значение **X** для офисов распределяется так:

   - Москва - 1

   - С.-Петербург - 2

   - Чокурдах - 3

   - Лабытнанги - 4

     

   1.1.3 Link-local ipv6 адреса выбираются из сеть FE80::**XX**/10, где **XX** это порядковый номер роутера, который берется из hostname

   

   2.1.1 для ipv4 сеть распределяются следющим образом - первый октет совпадает с номером AS:

   - Киторн - 101.0.0.0/16, где 101.0.101.0/24 подсеть для loopback, 101.0.10.0/24 для EGP в AS1001

   - Триада - 52.0.0.0/16, где 52.0.52.0/24 подсеть для loopback

   - Ламас -   30.1.0.0/16, где   30.1.30.0/16 подсеть для loopback, 30.1.10.0./24 для EGP в AS1001

     

   2.1.2 Для офисов выделяются серые подсети вида 10.20**X**.0.0/16, где **X** порядоковый номер офиса. Из этой подсети выделяются дополнительные 10.20**X**.10.0/24 для линков между роутерами, 10.20**X**.77.0/24 для loopback-интерфейсов, 10.20**X**.66.0/24 для менеджмент-интерфейсов сетевого оборудования, 10.20**X**.100.0/24 для хостов офиса.

   - Офис Москва - 10.201.0.0/16, 10.201.10.0/24, 10.201.77.0/24, 10.201.66.0/24, 10.201.100.0/24

   - Офис С.-Петербург - 10.202.0.0/16, 10.202.10.0/24, 10.202.77.0/24, 10.202.66.0/24, 10.202.100.0/24

   - Офис Чокурдах - 10.203.0.0/16, 10.203.10.0/24, 10.203.77.0/24, 10.203.66.0/24, 10.203.100.0/24

   - Офис Лабытнанги - 10.204.0.0/16, 10.204.10.0/24, 10.204.77.0/24, 10.204.66.0/24, 10.204.100.0/24

     

   ###### Пункт 3

   Ipv4 маршрутизация уже включена на оборудовании по умолчанию. Ipv6 включается глобально (*ipv6 unicast-routing*) и на каждом из интерфейсов (*ipv6 enable*)

   

   ###### Пункт 4

   Для VPC в каждом офисе выбирается VLAN 200**X** и 20**XX**, где **Х** равен:

   - Москва - 1

   - С.-Петербург - 2

   - Чокурдах - 3

   - Лабытнанги - 4

     

   ###### Пункт 5

   Вланом управления для сетевых устройств выбирается VLAN 1000. Для роутеров loopback-интерфейсом выбирается loopback1000.

   

   ###### Пункт 6

   1. Для избежания петель и шторомов в офисе Москва на всех свитчах настраивается протокол STP для всех VLAN. Root-свитчом принудительно назначается коммутатор SW4 для VLAN2001  с помощью понижения значения bridge ID Priority до 4096, резервным root-свитчом для VLAN 2001 принудительно назначается коммутаторы SW5 с помощью понижения значения bridge ID Priority до 8192. Для VLAN2011 наоборот, root с помощью  priority 4096 назначается SW5, резервным с priority 8192 назначается SW4. Таким образом балансируется нагрузка между коммутаторами.

   2. Для оптимального использования линков  между SW4 и SW5 в Москве и между SW9 и SW10 в С.-Петербурге настраивается LACP.

   3. Для резервирования шлюза в офисах Москвы и С.-Петербурга на роутерах R12 и R13 и R17 и R16 настраивается протокол HSRP. Основными роутерами назначаются, сооответственно R12 и R17 для VLAN 2001 и 2002, для VLAN 2011 и 2022 основными будут R13 и R16 с помощью повышения значения *priority* до 150, так же настраивается приоритетное вытеснение с помощью настройки *standby 1 preempt*.

      

   ### 1.Задокументируем сети

   

   ##### ISP Триада AS 520

   | Network ipv4  | Summary Nerwork ipv4 | Network ipv6          | Summary Network ipv6 | Description                     |
   | ------------- | -------------------- | --------------------- | -------------------- | ------------------------------- |
   |               | 52.0.0.0/16          |                       | 2001:A:B::/48        | Summary Networks                |
   |               | 52.0.0.0/24          |                       |                      | ipv4/ipv6 IGP                   |
   | 52.0.0.0/30   |                      | 2001:A:B:1::/64       |                      | IGP R23-R25                     |
   | 52.0.0.4/30   |                      | 2001:A:B:2::/64       |                      | IGP R25-R26                     |
   | 52.0.0.8/30   |                      | 2001:A:B:3::/64       |                      | IGP R24-R26                     |
   | 52.0.0.12/30  |                      | 2001:A:B:4::/64       |                      | IGP R23-R24                     |
   | 52.0.0.16/30  |                      | 2001:A:B:5::/64       |                      | EGP to AS101(R22)               |
   | 52.0.0.20/30  |                      | 2001:A:B:6::/64       |                      | EGP to AS301(R21)               |
   | 52.0.0.24/30  |                      | 2001:A:B:7::/64       |                      | EGP to AS2042 LINK#1(R24-R18)   |
   | 52.0.0.28/30  |                      | 2001:A:B:8::/64       |                      | EGP to AS2042 LINK#2(R25-R18)   |
   | 52.0.0.32/30  |                      | 2001:A:B:A::/64       |                      | EGP to Чокурдах LINK#1(R25-R28) |
   | 52.0.0.36/30  |                      | 2001:A:B:B::/64       |                      | EGP to Чокурдах LINK#2(R26-R28) |
   | 52.0.0.40/30  |                      | 2001:A:B:9::/64       |                      | EGP to Лабытнанги LINK#1(К25)   |
   |               |                      |                       |                      |                                 |
   |               |                      |                       |                      |                                 |
   |               |                      |                       |                      |                                 |
   |               | 52.0.52.0/24         |                       | 2001:A:B:FFFF::/64   | Summary ipv4/ipv6 loopbacks     |
   | 52.0.52.23/32 |                      | 2001:A:B:FFFF::23/128 |                      | R23 loopback                    |
   | 52.0.52.24/32 |                      | 2001:A:B:FFFF::24/128 |                      | R24 loopback                    |
   | 52.0.52.25/32 |                      | 2001:A:B:FFFF::25/128 |                      | R25 loopback                    |
   | 52.0.52.26/32 |                      | 2001:A:B:FFFF::26/128 |                      | R26 loopback                    |
   | 52.0.10.0/24  |                      | 2001:A:B:C::/64       |                      | for AS2042 Spb(PA) NAT          |
   |               |                      |                       |                      |                                 |

##### ISP Киторн AS 101

| Network ipv4    | Summary Nerwork ipv4 | Network ipv6          | Summary Network ipv6 | Description                   |
| --------------- | -------------------- | --------------------- | -------------------- | ----------------------------- |
|                 | 101.0.0.0/16         |                       | 2001:A:A::/48        | Summary Networks              |
|                 | 101.0.0.0/24         |                       | 2001:A:A::/48        | Summary IGP&EGP               |
| 101.0.0.0/30    |                      | 2001:A:A:2::/64       |                      | eth 0/1 EGP to AS301 R22-R21  |
| 101.0.0.4/30    |                      | 2001:A:A:1::/64       |                      | eth 0/0 EGP to AS1001 R22-R14 |
|                 | 101.0.101.0/24       |                       | 2001:A:A:FFFF::/64   | Summary loopback              |
| 101.0.101.22/32 |                      | 2001:A:A:FFFF::22/128 |                      | R22 loopback                  |
| 101.0.10.0/24   | 101.0.10.0/24        | 2001:A:A:1::/64       |                      | for AS1001 Moscow(PA) NAT     |

##### ISP Ламас AS 301

| Network ipv4  | Summary Nerwork ipv4 | Network ipv6          | Summary Network ipv6 | Description           |
| ------------- | -------------------- | --------------------- | -------------------- | --------------------- |
|               | 30.1.0.0/16          |                       | 2001:A:С::/48        | Summary Networks      |
|               | 30.1.0.0/24          |                       | 2001:A:С::/48        | Summary IGP&EGP       |
| 30.1.0.0/30   |                      | 2001:A:С:1::/64       |                      | EGP to AS1001 R21-R15 |
|               | 30.1.30.0/24         |                       | 2001:A:C:FFFF::/64   | Summary loopback      |
| 30.1.30.21/32 |                      | 2001:A:C:FFFF::21/128 |                      | R21 loopback          |
| 30.1.10.0./24 | 30.1.10.0./24        | 2001:A:С:1::/64       |                      | for AS1001 Moscow(PA) |
|               |                      |                       |                      |                       |
|               |                      |                       |                      |                       |

##### 	Офис Москва AS 1001

| Network ipv4    | Summary Nerwork ipv4 | Network ipv6            | Summary Network ipv6 | Description                   |
| --------------- | -------------------- | ----------------------- | -------------------- | ----------------------------- |
|                 | 10.201.0.0/16        |                         | 2001:F:F1::/48       | Summary Network fo AS 1001    |
|                 | 10.201.10.0/24       |                         |                      | Summary Link Network          |
| 10.201.10.0/30  |                      | 2001:F:F1:1::/64        |                      | R14-R12                       |
| 10.201.10.4/30  |                      | 2001:F:F1:2::/64        |                      | R14-R13                       |
| 10.201.10.8/30  |                      | 2001:F:F1:3::/64        |                      | R15-R12                       |
| 10.201.10.12/30 |                      | 2001:F:F1:4::/64        |                      | R15-R13                       |
| 10.201.10.16/30 |                      | 2001:F:F1:5::/64        |                      | R15-R20                       |
| 10.201.10.20/30 |                      | 2001:F:F1:6::/64        |                      | R14-R19                       |
|                 | 10.201.77.0/24       |                         | 2001:F:F1:FFFF::/64  | ipv4v6 loopback for routers   |
| 10.201.77.14/32 |                      | 2001:F:F1:FFFF::14/128  |                      | loop R14                      |
| 10.201.77.15/32 |                      | 2001:F:F1:FFFF::15/128  |                      | loop R15                      |
| 10.201.77.12/32 |                      | 2001:F:F1:FFFF::12/128  |                      | loop R12                      |
| 10.201.77.13/32 |                      | 2001:F:F1:FFFF::13/128  |                      | loop R13                      |
| 10.201.77.19/32 |                      | 2001:F:F1:FFFF::19/128  |                      | loop R19                      |
| 10.201.77.20/32 |                      | 2001:F:F1:FFFF::20/128  |                      | loop R20                      |
|                 | 10.201.66.0/24       |                         | 2001:F:F1:2000::/64  | loopback for MNG switch       |
| 10.201.66.4/24  |                      | 2001:F:F1:2000::2004/64 |                      | MNG for SW4                   |
| 10.201.66.5/24  |                      | 2001:F:F1:2000::2005/64 |                      | MNG for SW5                   |
| 10.201.66.3/24  |                      | 2001:F:F1:2000::2003/64 |                      | MNG for SW3                   |
| 10.201.66.2/24  |                      | 2001:F:F1:2000::2002/64 |                      | MNG for SW2                   |
| 10.201.66.1/24  |                      | 2001:F:F1:2000::2012/64 |                      | R12 gateway for switch        |
| 10.201.66.13/24 |                      | 2001:F:F1:2000::2013/64 |                      | R13 gateway for switch backup |
|                 | 10.201.100.0/24      |                         | 2001:F:F1:1000::/64  | summary network for host      |
| 10.201.100.1/24 |                      | 2001:F:F1:1000::1/64    |                      | HSRP virtual address VLAN2001 |
| 10.201.100.2/24 |                      | 2001:F:F1:1000::2/64    |                      | VPC1 VLAN2001                 |
| 10.201.101.1/24 |                      | 2001:F:F1:1001::1/64    |                      | HSRP virtual address VLAN2011 |
| 10.201.101.2/24 |                      | 2001:F:F1:1001::2/64    |                      | VPC7 VLAN2011                 |
|                 |                      |                         |                      |                               |

##### Офис Санкт-Петербург AS 2042

| Network ipv4    | Summary Nerwork ipv4 | Network ipv6            | Summary Network ipv6 | Description                   |
| --------------- | -------------------- | ----------------------- | -------------------- | ----------------------------- |
|                 | 10.202.0.0/16        |                         | 2001:F:F2::/48       | Summary Network fo AS 2042    |
|                 | 10.202.10.0/24       |                         |                      | Summary Link Network          |
| 10.202.10.0/30  |                      | 2001:F:F2:1::/64        |                      | R18-R17                       |
| 10.202.10.4/30  |                      | 2001:F:F2:2::/64        |                      | R18-R16                       |
| 10.202.10.8/30  |                      | 2001:F:F2:3::/64        |                      | R17-R16                       |
| 10.202.10.12/30 |                      | 2001:F:F2:4::/64        |                      | R16-R32                       |
|                 | 10.202.77.0/24       |                         | 2001:F:F2:FFFF::/64  | Summary for loopback router   |
| 10.202.77.18/32 |                      | 2001:F:F2:FFFF::18/128  |                      | loop R18                      |
| 10.202.77.17/32 |                      | 2001:F:F2:FFFF::17/128  |                      | loop R17                      |
| 10.202.77.16/32 |                      | 2001:F:F2:FFFF::16/128  |                      | loop R16                      |
| 10.202.77.32/32 |                      | 2001:F:F2:FFFF::32/128  |                      | loop R32                      |
|                 | 10.202.66.0/24       |                         | 2001:F:F2:2000::/64  | loopback for MNG switch       |
| 10.202.66.1/24  |                      | 2001:F:F2:2000::2017/64 |                      | R17 gateway for switch        |
| 10.202.66.2/24  |                      | 2001:F:F2:2000::2016/64 |                      | R16 gateway for switch        |
| 10.202.66.9/24  |                      | 2001:F:F2:2000::2009/64 |                      | MNG for SW9                   |
| 10.202.66.10/24 |                      | 2001:F:F2:2000::2010/64 |                      | MNG for SW10                  |
|                 | 10.202.100.0/24      |                         | 2001:F:F2:1000::/64  | summary network for host      |
| 10.202.100.1/24 |                      | 2001:F:F2:1000::1/64    |                      | HSRP virtual address VLAN2002 |
| 10.202.100.2/24 |                      | 2001:F:F2:1000::2/64    |                      | VPC8 VLAN 2002                |
| 10.202.101.1/24 |                      | 2001:F:F2:1001::1/64    |                      | HSRP virtual address VLAN2022 |
| 10.202.101.2/24 |                      | 2001:F:F2:1001::2/64    |                      | VPC9 VLAN 2022                |

##### Офис Чокурдах

| Network ipv4    | Summary Nerwork ipv4 | Network ipv6            | Summary Network ipv6 | Description                   |
| --------------- | -------------------- | ----------------------- | -------------------- | ----------------------------- |
|                 | 10.203.2.0/16        |                         | 2001:F:F3::/48       | Summary Network Чокурдах      |
|                 | 10.203.77.0/24       |                         | 2001:F:F3:FFFF::/64  | Summary for loopback router   |
| 10.203.77.28/24 |                      | 2001:F:F3:FFFF::28/128  |                      | loopback R28                  |
|                 | 10.203.66.0/24       |                         | 2001:F:F3:2000::/64  | loopback for MNG switch       |
| 10.203.66.29/24 |                      | 2001:F:F3:2000::2029/64 |                      | MNG SW29 VLAN1000             |
| 10.203.66.1/24  |                      | 2001:F:F3:2000::2028/64 |                      | R28 gateway for switch        |
|                 | 10.203.100.0/24      |                         | 2001:F:F3:1000::/64  | summary network for host      |
| 10.203.100.1/24 |                      | 2001:F:F3:1000::1/64    |                      | R28 gateway for host VLAN2003 |
| 10.203.100.2/24 |                      | 2001:F:F3:1000::2/64    |                      | VPC30 VLAN2003                |
|                 | 10.203.101.0/24      |                         | 2001:F:F3:1001::/64  |                               |
| 10.203.101.1/24 |                      | 2001:F:F3:1001::1/64    |                      | R28 gateway for host VLAN2033 |
| 10.203.101.2/24 |                      | 2001:F:F3:1001::2/64    |                      | VPC31 VLAN2033                |

##### Офис Лабытнанги

| Network ipv4    | Summary Nerwork ipv4 | Network ipv6           | Summary Network ipv6 | Description                |
| --------------- | -------------------- | ---------------------- | -------------------- | -------------------------- |
|                 | 10.204.0.0/16        |                        | 2001:F:F4::/48       | Summary Network Лабытнанги |
|                 | 10.204.77.0/24       |                        | 2001:F:F4:FFFF::/48  | loopback for routers       |
| 10.204.77.27/24 |                      | 2001:F:F4:FFFF::27/128 |                      | loopback R27               |

##### Link-local IPV6 адреса

| Router | Link-Local address | Description                            |
| ------ | ------------------ | -------------------------------------- |
| R12    | FE80::12           | Link-Local addresses for R12 all ports |
| R13    | FE80::13           | Link-Local addresses for R13 all ports |
| R14    | FE80::14           | Link-Local addresses for R14 all ports |
| R15    | FE80::15           | Link-Local addresses for R15 all ports |
| R16    | FE80::16           | Link-Local addresses for R16 all ports |
| R17    | FE80::17           | Link-Local addresses for R17 all ports |
| R18    | FE80::18           | Link-Local addresses for R18 all ports |
| R19    | FE80::19           | Link-Local addresses for R19 all ports |
| R20    | FE80::20           | Link-Local addresses for R20 all ports |
| R21    | FE80::21           | Link-Local addresses for R21 all ports |
| R22    | FE80::22           | Link-Local addresses for R22 all ports |
| R23    | FE80::23           | Link-Local addresses for R23 all ports |
| R24    | FE80::24           | Link-Local addresses for R24 all ports |
| R25    | FE80::25           | Link-Local addresses for R25 all ports |
| R26    | FE80::26           | Link-Local addresses for R26 all ports |
| R27    | FE80::27           | Link-Local addresses for R27 all ports |
| R28    | FE80::28           | Link-Local addresses for R28 all ports |
|        |                    |                                        |

### Примеры конфигураций протоколов на оборудовании:



#### 1. Настройки на свитчах



##### 1.1 RSTP на примере SW5

```
spanning-tree mode rapid-pvst
spanning-tree extend system-id
spanning-tree vlan 1000,2001 priority 8192
spanning-tree vlan 2011 priority 4096
```



##### 1.2 LACP на SW5

```
!
interface Port-channel1
 description LACP SW4-SW5
 switchport trunk allowed vlan 1000,2001,2011
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 description SW4_Port-channel1
 switchport trunk allowed vlan 1000,2001,2011
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode passive
!
interface Ethernet0/3
 description SW4_Port-channel1
 switchport trunk allowed vlan 1000,2001,2011
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode passive
!
```



##### 1.3 Маршрутизация на SW5

```
ipv6 unicast-routing
!
interface Vlan1000
 description MNG
 ip address 10.201.66.5 255.255.255.0
 ipv6 address 2001:F:F1:2000::2005/64
 ipv6 enable
!
ip route 0.0.0.0 0.0.0.0 10.201.66.1
!
!
ipv6 route ::/0 2001:F:F1:2000::2012
!
```

##### 1.4 Access-порт на SW3

```
!
interface Ethernet0/2
 description VPC7
 switchport access vlan 2001
 switchport mode access
!


```

##### 1.5 Trunk-порт на SW3

```
!
interface Ethernet0/0
 description SW4
 switchport trunk allowed vlan 1000,2001,2011
 switchport trunk encapsulation dot1q
 switchport mode trunk
!


```





#### 2. Настройки на роутерах



##### 2.1 Настройки loopback-интерфейса на R12

```
!
interface Loopback1000
 description LOOPBACK
 ip address 10.201.77.12 255.255.255.255
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:FFFF::12/128
 ipv6 enable
!
```

##### 2.2 Настройки порта между роутерами на R12

```
!
interface Ethernet0/2
 description R14
 ip address 10.201.10.2 255.255.255.252
 ipv6 address FE80::12 link-local
 ipv6 address 2001:F:F1:1::2/64
 ipv6 enable
!
```

##### 2.3 Настройки гейтвей под VLAN-управления свитчами на R12

```
!
interface Ethernet0/0.1000
 description MNG_SW
 encapsulation dot1Q 1000
 ip address 10.201.66.1 255.255.255.0
 ipv6 address 2001:F:F1:2000::2012/64
 ipv6 enable
!
```

##### 2.4 Настройки HSRP на R12 - primary для VLAN2001, stanby для VLAN2011

```
!
interface Ethernet0/0.2001
 description HSRP_VLAN2001_GATE
 encapsulation dot1Q 2001
 ip address 10.201.100.100 255.255.255.0
 standby version 2
 standby 1 ip 10.201.100.1
 standby 1 priority 150
 standby 1 preempt
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
 ipv6 address 2001:F:F1:1001::1/64
 ipv6 enable
!
```

#### 3. Настройки на хостах

##### 3.1. Настройки TCP/IP на VPC1

```
ip 10.201.100.2 10.201.100.1 24
ip 2001:f:f1:1000::2/64
```



Резюме: ip-связность p2p между оборудованием проверена для ipv4 и для ipv6. Так же проверена работа протоколов STP, LACP, HSRP, которые используются для резервирования и равномерного распределния трафика по линкам и оборудованию.



### Конец.