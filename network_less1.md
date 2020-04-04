OTUS LAB LESSON 1

"VTP AND DTP"

1. VTP

1.1. Был настроен протокол VTP на коммутаторе S2 в режиме server. Именно с этого коммутатора
будут создаваться обновления по имеющимся VLAN. Так же был настроен VTP-Domian "CCNA" 
и vtp password "cisco"

использовались следующие команды:

conf t
 vtp mode server
 vtp domain CCNA
 vtp password cisco
end

1.2. Коммутаторы S1 и S3 были настроены в VTP режиме client. Эти коммутаторы не будут рассылать
обновления по имеющимся у них VLAN, а только получать обновления от сервера и конфигурировать
своим транковые порты, соглано эти обновлениям

использовались команды:

conf t
 vtp mode client
 vtp domain CCNA
 vtp password cisco
end

Посмотреть информацию по текущему состояния протокола VTP на всех коммутаторах можно командой:

show vtp status

1.3. После данных настроек обновления не получили коммутаторы S1 и S3, т.к. между ними и S2
не было связности. Нужно было настроить транковые порты.

2. DTP

Для настройки транковых портов использовался протокол DTP, так и статический режим транка.

2.1. Настройка S1

2.1.1. На коммутаторе S1 был использован режим DTP desirable в сторону S2, в этом режиме противоположный
порт коммутатора должен находится в режиме авто, trunk или тоже desirable. Так же потребовалось применить
команду "switchport trunk encapsulation dot1q" т.к. по умолчанию на данном образе коммутаторов
по умолчанию была выставлена энкапсуляция isl и требовалось явно указать энкапсуляцию dot1q.
Эту комманду в последствии потребовалось указать на всех транковых портах. Более того в режиме
статического транка, она необходима, т.к. без неё порт не переходил в режим транка, выдавая ошибку.

использовались команды:

interface Ethernet0/2
 description to_S2_eth0/2
 switchport trunk encapsulation dot1q
 switchport mode dynamic desirable
end

2.1.2. На S1 порт в сторону S3 был настроен в режиме постоянного(статического) транка.

использовались команды:

interface Ethernet0/0
 description to_S3
 switchport trunk encapsulation dot1q
 switchport mode trunk
end

2.1.3. В сторону хоста на коммутаторе S1 порт был настроен в режим access с вланом 10

использовались команды:

interface Ethernet0/3
 description to_PC1
 switchport access vlan 10
 switchport mode access
end

Создать vlan 10 на коммутаторе S1 не получилось, согласно методичке, потому что он уже находился в режиме
VTP client и в таком режиме создание вланов на коммтутаторе не предполагается.

2.2. Настройка S2

2.2.1. На коммутаторе S2 в сторону S1 не потребовалось никаких настроек, т.к. он находился в режиме auto
и принял автоматически все необходимые настройки с соседнего порта S1, который находится в режиме desirable

использовались команды:

interface Ethernet0/2
 description to_S1
end

2.2.2. На S2 порт в сторону S3 был настроен в режиме постоянного(статического) транка.

использовались команды:

interface Ethernet0/1
 description to_S3
 switchport trunk encapsulation dot1q
 switchport mode trunk
end

2.2.3. В сторону хоста на коммутаторе S1 порт был настроен в режим access с вланом 20

использовались команды:

interface Ethernet0/3
 description to_PC2
 switchport access vlan 20
 switchport mode access
end

2.2.4. На S2 как на коммутаторе VTP server были настроены все необходимые vlan, чтобы в последвствии
они разлетелись в обновлениях по соседним коммутатором в этом же VTP домене

комнады:

S2(config)# vlan 10
S2(config-vlan)# name Red
S2(config-vlan)# vlan 20
S2(config-vlan)# name Blue
S2(config-vlan)# vlan 30
S2(config-vlan)# name Yellow
S2(config-vlan)# vlan 99
S2(config-vlan)# name Management
S2(config-vlan)# end

таблица влан стала выглядеть следующим образом:

S2#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0
10   Red                              active
20   Blue                             active    Et0/3
30   Yellow                           active
99   Management                       active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

2.3. Настройка S3

2.3.1. На коммутаторе S3 в сторону S1 и S2 порты были настроены в режиме постоянного транка.

использовались команды:

interface Ethernet0/0
 description to_S1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 description to_S2
 switchport trunk encapsulation dot1q
 switchport mode trunk


2.3.2. В сторону хоста на коммутаторе S1 порт был настроен в режим access с вланом 10

использовались команды:

interface Ethernet0/3
 description to_PC2
 switchport access vlan 20
 switchport mode access
end

2.4.

Были настроены ip адреса и маска на коммутаторах на SVI интерфейсах привязанных к vlan 99 
согласно следующей схеме:

1. S1 - 192.168.99.1 маска 255.255.255.0
2. S2 - 192.168.99.2 маска 255.255.255.0
3. S3 - 192.168.99.3 маска 255.255.255.0

сделано это было следующими командами, менялся только ip адрес:

S1(config)# interface vlan 99
S1(config-if)# ip address 192.168.99.1 255.255.255.0
S1(config-fi)# no shutdown

2.5.

были настроен хостовые виртуальные PC, согласно следующей схеме

1. PC1 - 192.168.10.1/24 подключен к S1 находится во vlan 10
2. PC2 - 192.168.20.1/24 подключен к S2 находитcя во vlan 20
3. PC3 - 192.168.10.2/24 подключен к S3 находится во vlan 10

2.6. Были проверены обновления VTP на коммутаторах, следующей командой(обновления пришли от S2 с версией
конфигурации 4)

S1#show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : aabb.cc00.1000
Configuration last modified by 0.0.0.0 at 4-3-20 13:14:05

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 4
MD5 digest                        : 0xB2 0x9A 0x11 0x5B 0xBF 0x2E 0xBF 0xAA
                                    0x31 0x18 0xFF 0x2C 0x5E 0x54 0x0A 0xB7

									
2.7.

По вопросам из методички:

a.	Отправьте ping-запрос с компьютера PC2 на PC1 и проверьте результат. Поясните ответ.

Пинг отсутствует, т.к. данные хосты находятся в разных vlan 10 и 20, 
поэтому широковещательный домен этих хостов разный и выяснить. Они не смогу выяснить, соответсвие
ip-адреса мак-адресу, т.к. их arp-запросы будут ходить в разных vlan.

Так же на этих хостах настроены разные подсети и даже если бы они были в одном влане, то необходимо
было бы настраивать маршрут по умолчанию для каждого хоста, таким шлюзом для них могли бы стать
SVI интерфейсы  vlan 10 и vlan 20 к примеры на свитче S2, тогда была маршрутизация между vlan
происходила бына этом свитче.

b.	Отправьте ping-запрос с компьютера PC1 на PC3 и проверьте результат. Поясните ответ.

Пинг есть, т.к. они находятся в одном vlan, следовательно нашли друг друга по протоколу arp.

пример с PC1:

PC1> show ip all

NAME   IP/MASK              GATEWAY           MAC                DNS
PC1    192.168.10.1/24      0.0.0.0           00:50:79:66:68:04

PC1> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=0.466 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=0.620 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=0.649 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=0.639 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=64 time=0.656 ms

c.	Отправьте ping-запрос с коммутатора S1 на компьютер PC1. Была ли проверка успешной? Поясните ответ.

Пинга не будет. Не смотря на то что PC1 включен в S1 и vlan 10 настроен акссессом. Пинг с S1 запускается с 
SVI интерфейса который находится во vlan 99, а не vlan 10. Так же у них разные подсети.
Пинг появится если настроить акссес с vlan 99 и сделать одинаковую подсеть. Либо прописать дополнительно ещё
один SVI интерфейс во влан 10 на S1 c ip из той же подсети.

d.	Отправьте ping-запрос с коммутатора S2 на коммутатор S1. Была ли проверка успешной? Поясните ответ.

Пинг будет, т.к. находятся их SVI интерфейсы в одном влан и в одной подсети.

S1#ping 192.168.99.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

S1#show running-config interface vlan 99

interface Vlan99
 ip address 192.168.99.1 255.255.255.0
end



3.1

Переводим S1 в режим transparent и создаём влан из расширенного диапазона. В режиме transparent обновления
VTP пропускаются, но сам коммутатор их не воспринимает, так же сам их не рассылает. Поэтому на других
коммутаторах не появится vlan 2000. Так же он не появится, потому что расширенный диапазон не рассылается
в версии 1 протокола VTP, которая у нас включена.


S1(config)#vtp mode transparent
Setting device to VTP Transparent mode for VLANS.
S1(config)#end
S1#sh
S1#show
*Apr  4 21:08:42.122: %SYS-5-CONFIG_I: Configured from console by console
S1#show vtp
S1#show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : aabb.cc00.1000
Configuration last modified by 0.0.0.0 at 4-3-20 13:14:05

Feature VLAN:
--------------
VTP Operating Mode                : Transparent
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 0
MD5 digest                        : 0xB2 0x9A 0x11 0x5B 0xBF 0x2E 0xBF 0xAA
                                    0x31 0x18 0xFF 0x2C 0x5E 0x54 0x0A 0xB7

Создание влана из расширенного диапазона:

S1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/1
10   Red                              active    Et0/3
20   Blue                             active
30   Yellow                           active
99   Management                       active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
2000 VLAN_2000                        active











