# DHCPv4

![1](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_1.jpg)



### Часть 1. Базовая настройка оборудования

Произведем базовую настройку оборудования: роутеров, свитчей, хостов. Проверим их связанность пингами.

###### шаг 1

Разобьем сеть 192.168.1.0/24 на подсети

А: 192.168.1.0/26 на 58 хостов для клиенсткого VLAN на R1

B: 192.168.1.64/27 на 28 хостов для менеджмент VLAN для R1

C: 192.168.1.96/28 на 12 хостов клиентская сеть на R2

###### шаг 2

сделаем базовые настройки ip и роутинга и VLAN на свитчах.

R1

```
R1(config)#interface ethernet 0/1
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#interface ethernet 0/1.100
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#no shutdown
R1(config-subif)#exit
R1(config)#interface ethernet 0/1.200
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#no shutdown
R1(config-subif)#exit
R1(config)#interface ethernet 0/1.1000
R1(config-subif)#no shutdown
R1(config-subif)#exit
R1(config)#interface ethernet 0/1.100
R1(config-subif)#description clients
R1(config-subif)#exit
R1(config)#interface ethernet 0/1.200
R1(config-subif)#description mng
R1(config-subif)#exit
R1(config)#interface ethernet 0/1.1000
R1(config-subif)#description native
R1(config-subif)#end
R1#conf t
R1(config)#ip route 0.0.0.0 0.0.0.0 ethernet 0/0 10.0.0.2
R1(config)#interface ethernet 0/0
R1(config-if)#description to R2
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#end


```

R2

```
R2#conf t
R2(config)#interface ethernet 0/1
R2(config-if)#description clients
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#exit
R2(config-if)#interface ethernet 0/1
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#interface ethernet 0/0
R2(config-if)#description to R1
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#ip route 0.0.0.0 0.0.0.0 ethernet 0/0 10.0.0.1
R2(config)#end
```

Проверяем связность, пинг успешный:

![2](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_2.jpg)

S1

```
S1#show running-config

no ip domain-lookup

interface Ethernet0/0
 description to PC
 switchport access vlan 100
 switchport mode access
!
interface Ethernet0/1
 description to R1
 switchport trunk allowed vlan 100,200,1000
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 1000
 switchport mode trunk
!
interface Ethernet0/2
 switchport access vlan 1000
 switchport mode access
 shutdown
!
interface Ethernet0/3
 switchport access vlan 1000
 switchport mode access
 shutdown
!
interface Vlan200
 description mng
 ip address 192.168.1.66 255.255.255.224

ip route 0.0.0.0 0.0.0.0 192.168.1.65

end
```

S2

```
S2#show running-config

no ip domain-lookup

interface Ethernet0/0
 description to PC
 switchport mode access
!
interface Ethernet0/1
 description to R2
 switchport mode access
!
interface Ethernet0/2
 shutdown
!
interface Ethernet0/3
 shutdown
!
interface Vlan1
 description mng
 ip address 192.168.1.98 255.255.255.240

ip route 0.0.0.0 0.0.0.0 192.168.1.97

end
```

Проверим пингуют ли свитчи друг друга

![3](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_3.jpg)

### Часть 2. Настройка DHCP

Настроим DHCP на R1

Зададим адрес исключения которые не будут выдаваться клиентам из пула. Сделаем два пула для клиентов R1 и R2

```
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.102

R1(config)#ip dhcp pool R1_clients

R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#domain-name ccna-lab.com
R1(dhcp-config)#default-router 192.168.1.1

R1(dhcp-config)#lease 2 12 30
R1(dhcp-config)#exit
R1(config)#ip dhcp pool R2_clients
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#domain-name ccna-lab.com
R1(dhcp-config)#lease 2 12 30
R1(dhcp-config)#end
```

![4](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_4.jpg)

![5](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_5.jpg)

![6](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_6.jpg)



Зайдем на PC-A и сделаем ipconfig/renew после чего пропингуем шлюз на R1

![7](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_7.jpg)

### Шаг 3. Настройка DHCP-Relay на R2

Настроим R2



```
R2(config)#interface ethernet 0/1
R2(config-if)#ip helper-address 10.0.0.1
R2(config-if)#end
```

Проверим настройки на PC-B и проверим пинг до R1

![8](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_8.jpg)

Проверим состояние DHCP на R1 и R2

![9](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_9.jpg)

![10](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv4/screen/Screenshot_10.jpg)

### Конец.