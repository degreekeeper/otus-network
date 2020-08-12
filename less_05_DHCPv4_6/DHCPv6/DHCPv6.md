# DHCPv6

![1](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv6/screen/Screenshot_1.jpg)

### Часть 1. Базовая настройка оборудования

Произведем базовую настройку оборудования: роутеров, свитчей, хостов. Проверим их связанность пингами.

R1

```
Router(config)#hostname R1
R1(config)#no ip domain-lookup

R1(config)#ipv6 unicast-routing

interface Ethernet0/0
 description to_R2
 no ip address
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
!
interface Ethernet0/1
 description to_cliets_R1
 no ip address
 shutdown
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
exit
ipv6 route ::/0 2001:DB8:ACAD:2::2
```

R2

```
Router>ena
Router#conf t
Router(config)#no ip domain-lookup
Router(config)#hostname R2
R2(config)#ipv6 unicast-routing
R2(config)#interface ethernet 0/0
R2(config-if)#description to_R1
R2(config-if)#ipv6 address 2001:db8:acad:2::2/64
R2(config-if)#ipv6 address fe80::2 link-local
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#interface ethernet 0/1
R2(config-if)#description to_clients_R2
R2(config-if)#ipv6 address 2001:db8:acad:3::1/64
R2(config-if)#ipv6 address fe80::2 link-local
R2(config-if)#no shu
R2(config-if)#exit

R2(config)#ipv6 route ::/0 2001:db8:acad:2::1
R2(config)#exit
```

Проверим связность через пинг с R1 до R2

![2](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv6/screen/Screenshot_2.jpg)

### Часть 2. Настроим SLAAC на R2

Включим свитчи и хосты. Видим, что PC-A и PC-B получили ipv6 адреса через SLAAC автоматически:

![3](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv6/screen/Screenshot_3.jpg)

![4](https://github.com/degreekeeper/otus-network/blob/master/less_05_DHCPv4_6/DHCPv6/screen/Screenshot_4.jpg)

### Часть 3. Настроим Stateless DHCP-сервер на R1

R1:

```
R1#
R1#conf t
R1(config)#ipv6 dhcp pool R1-STATELESS
R1(config-dhcpv6)#dns-server 2001:db8:acad::254
R1(config-dhcpv6)#domain-name STATELESS.com
R1(config-dhcpv6)#exit
R1(config)#interface ethernet 0/1
R1(config-if)#ipv6 nd other-config-flag
R1(config-if)#ipv6 dhcp server R1-STATELESS
R1(config-if)#end
```

### Часть 4. Настроим Stateful DHCP-сервер на R1

R1

```
R1#conf t
R1(config)#ipv6 dhcp pool R2-STATEFUL
R1(config-dhcpv6)#address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcpv6)#dns-server 2001:db8:acad::254
R1(config-dhcpv6)#domain-name STATEFUL.com
R1(config-dhcpv6)#exit
R1(config)#interface eth0/0
R1(config-if)#ipv6 dhcp server R2-STATEFUL
R1(config-if)#end
```

Продемонстировать получение хостом по DHCPv6 - DNS и домена возможности нет, т.к. виртуальный образ хоста VPC не поддерживает DHCPv6, а только SLAAC.

### Часть 5. Настроим DHCP-relay на R2

R2

```
R2#conf t
R2(config)#interface eth0/1
R2(config-if)#ipv6 nd managed-config-flag
R2(config-if)#ipv6 dhcp relay destination 2001:db8:acad:2::1 ethernet 0/0
R2(config-if)#end
```

В данном примере мы настроили dhcp-relay через адрес R1 на его интерфейсе eth0/0. Запросы DHCP будут перенаправляться на него уже unicastом, а не broadcastом. Продемонстировать получение хостом по DHCPv6 - DNS и домена возможности нет, т.к. виртуальный образ хоста VPC не поддерживает DHCPv6, а только SLAAC.



### Конец.