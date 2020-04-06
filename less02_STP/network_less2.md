# Часть 1

Свитчи были соединены согласно схеме в методичке. C исключением - eth0/4 используется порт eth1/0.

Были настроены SVI интерфейсы на vlan1 согласно списку:

Устройство	Интерфейс	IP-адрес	Маска подсети

+ S1	VLAN 1	192.168.1.1	255.255.255.0

+ S2	VLAN 1	192.168.1.2	255.255.255.0

+ S3	VLAN 1	192.168.1.3	255.255.255.0

После этого все коммтутаторы могли успешно пинговать друг друга.

# Часть 2

Порты были перенастроены идентично в транк, с пропуском всех vlan:

использовались команды:

S1#conf t
S1(config-if)#interface Ethernet0/1
S1(config-if)# description S2
S1(config-if)# switchport trunk encapsulation dot1q
S1(config-if)# switchport mode trunk

Согласно методичке были погашены все порты на всех коммутаторах

использовалась команда:

S1#conf t
S1(config-if)#interface Ethernet0/1
S1(config-if)# shutdown
S1(config-if)#!


Затем этого статус на всех свитчах выглядел следующим образом:

S1#show interfaces status

Port      Name               Status       Vlan       Duplex  Speed Type
Et0/0                        disabled     1            auto   auto unknown
Et0/1     S2                 disabled     1            auto   auto unknown
Et0/2     S2                 disabled     1            auto   auto unknown
Et0/3     S3                 disabled     1            auto   auto unknown
Et1/0     S3                 disabled     1            auto   auto unknown
Et1/1                        disabled     1            auto   auto unknown
Et1/2                        disabled     1            auto   auto unknown
Et1/3                        disabled     1            auto   auto unknown

Были открыты порты eth0/2 и eth1/0 на всех коммутаторах.

После чего состояние протокола STP на всех коммутаторах выглядело так:

S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Desg FWD 100       128.3    Shr
Et1/0               Desg FWD 100       128.5    Shr



S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Root FWD 100       128.3    Shr
Et1/0               Desg FWD 100       128.5    Shr



S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        5 (Ethernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Altn BLK 100       128.3    Shr
Et1/0               Root FWD 100       128.5    Shr

root выбран свитч с самым низким BID, т.к. приоритет одинаковый у всех 32769,
выбрана S1 т.к.его BID будет ниже за счёт меньшего мак-адреса свитча.

на двух портах свитчей S2 и S3 будет одинаковая стоимость до root(0+100):

S3#show spanning-tree interface ethernet 0/2 rootcost
VLAN0001            100

S2#show spanning-tree interface ethernet 1/0 rootcost
VLAN0001            100


Но заблокируется порт на S3 т.к. его BID больше за счёт большего мак-адреса при равных приоритетах 32769

# Часть 3

Настраиваем STP по приоритетам.

На свитче S3 где у нас заблокированный порт, уменьшаем приоритет другого активного интерфейса eth1/0

S3#conf t
S3(config)#interface ethernet 1/0
S3(config-if)#spanning-tree cost 18
S3(config-if)#end

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        18
             Port        5 (Ethernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Desg FWD 100       128.3    Shr
Et1/0               Root FWD 18        128.5    Shr


S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Root FWD 100       128.3    Shr
Et1/0               Altn BLK 100       128.5    Shr

Заблокированный порт на S3 стал назначенным, т.к. его стоимость до рута стала 18(0+18). Заблокировался
порт S2 eth0/2 т.к. его стоимость осталась 100(0+100) и сейчас это самая большая стоимость до root.

отменяем приоритет 18 на порту S3:

S3(config)#interface ethernet 1/0
S3(config-if)#no spanning-tree cost 18
S3(config-if)#end

Все возвращается обратно, теперь заблокированный порт это снова назначенны порт на S3:

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        5 (Ethernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Altn BLK 100       128.3    Shr
Et1/0               Root FWD 100       128.5    Shr


# Часть 4.

Открываем оставшиеся погашенными порты 0/1 и 0/3 на всех коммутатора:

S3(config)#interface range ethernet 0/1, ethernet 0/3
S3(config-if-range)#no shutdown
S3(config-if-range)#end

На S3 порт root изменился с 1/0 на 0/3, т.к его BID меньше, за счёт меньшего номера порта - 4 вместо
5 у прежнего порта 1/0

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Altn BLK 100       128.3    Shr
Et0/3               Root FWD 100       128.4    Shr
Et1/0               Altn BLK 100       128.5    Shr

на S2 аналогично порт 0/1 стал root портом


S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Altn BLK 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr

Отдельно будут приложены конфиги свитчей в github.

Конец.





