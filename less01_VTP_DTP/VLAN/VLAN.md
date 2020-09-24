# Configure Router-on-a-Stick Inter-VLAN Routing



![Топология](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_1.jpg)




### 1. Базовые настройки на сетевых устройствах и хостах

#### 1.1 Базовые настройки на роутере R1



```
!
service password-encryption
!
hostname R1
!
no ip domain lookup
!
enable secret 5 $1$NJ/w$Mxc5Tyci8SCrpkExhgbCR0
!
banner motd ^CDostup tolko dlya avtorizovannykh sotrudnikov^C
!
line con 0
 password 7 121A0C041104
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 0822455D0A16
 login
 transport input none
!
```

Настроим время

```
R1#clock set 20:53:00 24 September 2020

R1#show clock
20:56:52.519 EET Thu Sep 24 2020
```

#### 1.2 Базовые настройки на свитчах на примере SW1

```
service password-encryption
!
hostname SW1
!
enable secret 5 $1$Ef8B$DwfNRwZJlxP5iC4fIlXRV1
!
no ip domain-lookup
!
banner motd ^CDostup tolko dlya avtorizovannykh sotrudnikov^C
!
line con 0
 password 7 01100F175804
 logging synchronous
 login
line aux 0
line vty 0 4
 password 7 05080F1C2243
 login
!
```

Установим время

```
SW1#clock set 21:15:00 24 September 2020
SW1# show clock
21:17:55.002 EET Thu Sep 24 2020
```

#### 1.3 Базовые настроки на хостах на примере PC-A

VPCS> set pcname PC-A

```
PC-A> ip 192.168.3.3/24 192.168.3.1
Checking for duplicate address...
PC1 : 192.168.3.3 255.255.255.0 gateway 192.168.3.1
```

### 2. Настройка VLAN и маршрутизации

#### 2.1 Создаем VLAN

```
SW1#conf t
SW1(config)#vlan 3
SW1(config-vlan)#name Managment
SW1(config-vlan)#exit
SW1(config)#vlan 7
SW1(config-vlan)#name ParkingLot
SW1(config-vlan)#exit
SW1(config)#vlan 8
SW1(config-vlan)#name Native
SW1(config-vlan)#exit
SW1(config)#end
SW1#
```

#### 2.2 Делаем Managment-интерфейс и создаем маршурт по умолчанию



```
!
interface Vlan3
 description Managment
 ip address 192.168.3.11 255.255.255.0
!
ip route 0.0.0.0 0.0.0.0 192.168.3.1
!
```

#### 2.3 Не используемые порты загоняем в access, присваеваем VLAN 7 и деактивируем.

```
!
interface Ethernet0/3
 switchport access vlan 7
 switchport mode access
 shutdown
! 
```

#### 2.4 Аналогичные настройки согласно таблице адресации в начале статьи делаем для SW2

#### 2.5 Настроим порты под хосты на каждом из свитче в режие access 

##### SW1

```
SW1#show running-config interface ethernet 0/0
!
interface Ethernet0/0
 description PC-A
 switchport access vlan 3
 switchport mode access
end
```

##### SW2

```
SW2#show running-config interface ethernet 0/0
!
interface Ethernet0/0
 description PC-B
 switchport access vlan 4
 switchport mode access
end
```

Проверим настройки влан с помощью команды *show vlan brief*:

![show vlan brief](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_2.jpg)

### 3. Настройка транков dot1q между свитчами

#### 3.1 Настроим транки между свичтами

```
SW1#show running-config interface ethernet 0/2

interface Ethernet0/2
 description SW2_TRUNK
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
end


```

```
SW2#show running-config interface ethernet 0/2

interface Ethernet0/2
 description SW1_TRUNK
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
end
```

Проверим настройки транка командой *show interfaces trunk*:
![show trunk](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_3.jpg)

#### 3.2 Настроим на SW1 транк в сторон роутера R1

Настроим его так же как и транки между свитчами:

```
SW1#show running-config interface ethernet 0/1

interface Ethernet0/1
 description R1_TRUNK
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
end
```

Порта eth0/1 не будет в списках портов в транке, т.к. не сделаны настройки на стороне R1

### 4. Настроим маршрутизацию и саб-интерфейсы на R1:

4.1 Настроим саб-интерфейсы под каждый из VLAN.

```
!
interface Ethernet0/1
 no ip address
!
interface Ethernet0/1.3
 description Managment
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface Ethernet0/1.4
 description Operations
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface Ethernet0/1.8
 description Native
 encapsulation dot1Q 8
!
```

#### 4.2  Проверим настройки командой *show ip interface brief*

![show ip interface brief](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_4.jpg)

5. Проверим маршрутизацию

Пинг с PC-A до маршрута по умолчанию

![тут скриншот 5](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_5.jpg)


Пинг PC-A до PC-B

![тут скриншот 6](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_6.jpg)

Пинг PC-A до SW2

![тут скриншот 7](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_7.jpg)

Трассировка с PC-B до PC-A

![тут скриншот 8](https://github.com/degreekeeper/otus-network/blob/master/less01_VTP_DTP/VLAN/screen/Screenshot_8.jpg)



#### Конец.