# Часть 1

Настраиваем свитчи и хосты согласно следующей схеме:

S1	VLAN 99	192.168.99.11	255.255.255.0  
S2	VLAN 99	192.168.99.12	255.255.255.0  
S3	VLAN 99	192.168.99.13	255.255.255.0  

PC-A	NIC	192.168.10.1	255.255.255.0  
PC-B	NIC	192.168.10.2	255.255.255.0  
PC-C	NIC	192.168.10.3	255.255.255.0  

на всех свитчах для подключения хостов будет использоваться интерфейс eth0/0 с vlan 10 со следующей конфигурацией:

interface Ethernet0/0  
 description PC-A  
 switchport access vlan 10  
 switchport mode access  
!  

все остальные интерфейсы на всех свитчах были погашены

# Часть 2



S1(config)#interface ethernet 0/3  
S1(config-if)#channel-group 1 mode desirable  
S1(config-if)#no shutdown  
S1(config-if)#exi  
S1(config)#interface ethernet 1/0  
S1(config-if)#channel-group 1 mode desirable  
S1(config-if)#no shutdown  

сначал в логах появился лог о создании Port-channel group:

Creating a port-channel interface Port-channel 1

после открытия физических интерфейсов появился лог, что аггрегационный порт поднялся:  

*Apr  7 16:45:49.242: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up

S1#show interfaces ethernet 0/3 switchport  
Name: Et0/3  
Switchport: Enabled  
Administrative Mode: dynamic auto  
Operational Mode: static access (member of bundle Po1)  
Administrative Trunking Encapsulation: negotiate 
Operational Trunking Encapsulation: native  
Negotiation of Trunking: On 
Access Mode VLAN: 1 (default)  
Trunking Native Mode VLAN: 1 (default)  
Administrative Native VLAN tagging: enabled  
Voice VLAN: none  

Так же проверим другой командой на S1 и S3 что порты объединены:

S3#show etherchannel summary  
  
Number of channel-groups in use: 1  
Number of aggregators:           1  

Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
1      Po1(SU)         PAgP      Et0/3(P)    Et1/0(P)  

*Из методички: Что означают флаги «SU» и «P» в сводных данных по Ethernet?*

S - означает, что аггрегационный интерфейс работает в режиме L2  
U - означает, что аггрегационный порт в работе  
P - означает, что физические порты привязаны к аггрегационному  

Переведём порты в транки+пропишем native-vlan 99

*a.	Выполните команды show run interface идентификатор-интерфейса на S1 и S3. Какие команды включены в список для интерфейсов F0/3 и F0/4 на обоих коммутаторах? Сравните результаты с текущей конфигурацией для интерфейса Po1. Запишите наблюдения.*

конфигурация физических интерфейсов и логического аггрегационного индентичны за исключением
команд привязки к channel-group на физических интерфейсах:

channel-group 1 mode desirable  
channel-group 1 mode auto

*b.	Выполните команды show interfaces trunk и show spanning-tree на S1 и S3. Какой транковый порт включен в список? Какая используется сеть native VLAN? Какой вывод можно сделать на основе выходных данных?*

1. В список включен только логический транковый порт Port-channel1    
2. Используется сеть native-vlan 99  
3. Из выходных данных можно сделать вывод, что после включения в port-channel1 физические порты не участвуют в работе протокола STP и так же в работе транка как самостоятельные порты. Участвует исключительно Port-channel1.  

*Какие значения стоимости и приоритета порта для агрегированного канала отображены в выходных данных команды show spanning-tree?*  

Для всех трех созданных экземплярах spanning-tree на обоих свитчах стоимость 56

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Root FWD 56        128.65   Shr



#  Часть 3

Настраиваем LACP меожу S1 и S2. После настройки на S1 и поднятия физических интерфейсов LACP не поднимается, т.к. на удалённом коммутаторе протокол ещё не настроен. И выдаст следующие логи:

*Apr  7 19:20:52.283: %EC-5-L3DONTBNDL2: Et0/1 suspended: LACP currently not enabled on the remote port.
*Apr  7 19:20:52.544: %EC-5-L3DONTBNDL2: Et0/2 suspended: LACP currently not enabled on the remote port.

по выводу командц проверки состояния etherchannel так же аггрегационный порт будет в состоянии SD, D - означает down. Физические порты будут в состояние S - suspended(приостановлены)

S1#show etherchannel summary  

Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
1      Po1(SU)         PAgP      Et0/3(P)    Et1/0(P)  
2      Po2(SD)         LACP      Et0/1(s)    Et0/2(s)  

После включения со стороны S2 изменится состояния, а так же поднимется порт Port-channel2.

*Apr  7 19:29:41.911: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up  

S1#show etherchannel summary

2      Po2(SU)         LACP      Et0/1(P)    Et0/2(P)

*Какой протокол использует Po2 для агрегирования каналов? Какие порты агрегируются для образования Po2? Запишите команду, используемую для проверки.*

1. Используется протокол LACP это видно в выводе команды *show etherchannel summary*
2. Аггрегируются порты eth0/1 и eth0/2
3. show etherchannel summary

etherchannel между S2 и S3 настраиваем аналогично и проверяем что они поднялись:

S3#show etherchannel summary  

Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
1      Po1(SU)         PAgP      Et0/3(P)    Et1/0(P)  
3      Po3(SU)         LACP      Et0/1(P)    Et0/2(P)  

S3#show etherchannel summary   
Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
2      Po2(SU)         LACP      Et0/1(P)    Et0/2(P)  
3      Po3(SU)         LACP      Et0/3(P)    Et1/0(P)  

Теперь проверяем пинг между всеми коммутаторами:

![Ping](https://github.com/degreekeeper/otus-network/blob/master/less_03/LACP_PAGP/Screenshot_5.jpg "Пинг с PC-A")


![Ping](https://github.com/degreekeeper/otus-network/blob/master/less_03/LACP_PAGP/Screenshot_6.jpg "Пинг с PC-A")

# Часть 3.2  

Заливаем на все свитчи конфиг из методички, где есть какая-либо ошибка:

Диагностика S1:

*Отображаются ли агрегированные каналы 1 и 2, как транковые порты?*

Нет. Аггрегированных каналов Po1 и Po2 нет в транке. Отображаются только физические интерфейсы 
т.к. они в транке.

Port        Mode             Encapsulation  Status        Native vlan  
Et0/1       on               802.1q         trunking      99  
Et0/2       on               802.1q         trunking      99  
Et0/3       auto             802.1q         trunking      99  
Et1/0       auto             802.1q         trunking      99  


*Есть ли в выходных данных сведения о неполадках в работе EtherChannel? В случае обнаружения неполадок запишите их в отведённом ниже месте.*  

Да, есть. Po1 и Po2 в состоянии down, как и физические интерфейсы в состоянии suspended(приостановлен).

Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
1      Po1(SD)         LACP      Et0/1(s)    Et0/2(s)  
2      Po2(SD)         PAgP      Et0/3(s)    Et1/0(s)  


*c.	Используйте команду show run | begin interface Port-channel для просмотра текущей конфигурации, начиная с первого интерфейса агрегированного канала.*


interface Port-channel2  
 description S3  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode access  
!  
interface Port-channel1   
 description S2  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
!  
interface Ethernet0/0  
 description PC-A  
 switchport access vlan 10  
 switchport mode access  
!  
interface Ethernet0/1  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 1 mode active  
!   
interface Ethernet0/2  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 1 mode active  
!  
interface Ethernet0/3  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 channel-group 2 mode desirable  
!  
interface Ethernet1/0  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 channel-group 2 mode desirable  
!  

Были найдены несоответствия:

**1. Po2 находился в режим access, когда как его физические интерфейсы в режим trunk.** 
 
**2. На Po1 настройки верные, но он неактивен, т.к. настроен в режиме LACP, а удаленный коммутатор S2 в режиме PAgP**


Проблемы с Po2 были исправлены следующими командами:

S1#conf t
S1(config)#interface Port-channel 2
S1(config-if)#switchport mode trunk
S1(config-if)#end

после чего выскочил лог о том, что теперь режим Po и его физических интерфейсов совпадают:

*Apr  8 13:30:03.796: %EC-5-COMPATIBLE: Et0/3 is compatible with port-channel members  
*Apr  8 13:30:03.796: %EC-5-COMPATIBLE: Et1/0 is compatible with port-channel members  

Так же этот режим автоматически выставился на физических интерфейсах данного Port-channel:

interface Ethernet0/3  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 **switchport mode trunk**  
 channel-group 2 mode desirable  
!  
interface Ethernet1/0  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 **switchport mode trunk**  
 channel-group 2 mode desirable  
 
Данный Po2 по состоянию появился как транк, когда его members исчезли:

S1#show interfaces trunk  

Port        Mode             Encapsulation  Status        Native vlan  
Et0/1       on               802.1q         trunking      99  
Et0/2       on               802.1q         trunking      99  
**Po2         on               802.1q         trunking      99**  


Так же он поднялся по выводу команды:

S1#**show etherchannel summary**  
   
2      **Po2(SU)**         PAgP      Et0/3(P)    Et1/0(P)

Дальше переходим к проверки и устранению ошибок конфигурации на S2:

По команде *show interfaces trunk* аггрегационный интерфейсы не являются транками:

S2#**show interfaces trunk**  
  
Port        Mode             Encapsulation  Status        Native vlan  
Et0/1       on               802.1q         trunking      99  
Et0/2       on               802.1q         trunking      99  
Et0/3       on               802.1q         trunking      99  
Et1/0       on               802.1q         trunking      99  

По нижеследующей команде видим, что Port-channel неактивны, а их физические интерфейсы в режиме I - stand alone(т.е. не включены в аггрегационный интерфейс)
S2#show etherchannel summary

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)         PAgP      Et0/1(I)    Et0/2(I)
3      Po3(SD)         PAgP      Et0/3(I)    Et1/0(I)

Вывод команды *show run*:  

!  
interface Port-channel1  
 description S1  
 switchport trunk allowed vlan 1,99  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
!  
interface Port-channel3  
 description S3  
 switchport trunk allowed vlan 1,10,99  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
!  
interface Ethernet0/0  
 description PC-B  
 switchport access vlan 10  
 switchport mode access  
!  
interface Ethernet0/1  
 switchport trunk allowed vlan 1,99  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 1 mode desirable  
!  
interface Ethernet0/2  
 switchport trunk allowed vlan 1,99  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 1 mode desirable  
!  
interface Ethernet0/3  
 switchport trunk allowed vlan 1,10,99  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 3 mode desirable  
!  
interface Ethernet1/0  
 switchport trunk allowed vlan 1,10,99  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 3 mode desirable  
!  

Были найдены следующие ошибки в конфигурации:

**1. Port-channel1 находился в режиме PAgP, когда на S1 он находился в режиме LACP active. Было решено перевести его в режим LACP passive.**

**2. Port-channel3 настроен верно. Нужно искать проблемы с конфигурацией на стороне S3.**  




S2#conf t  
Enter configuration commands, one per line.  End with CNTL/Z.  
S2(config)#interface range ethernet 0/1-2  
S2(config-if-range)#no channel-group  
S2(config-if-range)#channel-group 1 mode passive  
S2(config-if-range)#end   

После этого Port-channel1 появился как транк, а так же поднялся по состоянию:

S2#**show interfaces trunk**  
  
Port        Mode             Encapsulation  Status        Native vlan  
Et0/3       on               802.1q         trunking      99  
Et1/0       on               802.1q         trunking      99  
**Po1         on               802.1q         trunking      99**  

S2#**show etherchannel summary**

Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
**1      Po1(SU)         LACP      Et0/1(P)    Et0/2(P)**  
3      Po3(SD)         PAgP      Et0/3(I)    Et1/0(I)  

Проверка конфигурации на S3:

S3#**show interfaces trunk**  
  
Port        Mode             Encapsulation  Status        Native vlan  
Po3         on               802.1q         trunking      99  

Только Po3 находится в транке, Po2 отсутствует, как и его физические интерфейсы.

S3#show etherchannel summary    
  
Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
3      Po3(SU)         PAgP      Et0/3(P)    Et1/0(P)  

Согласно выводу данной команды Port-channel1 не настроен вообще, а Port-channel3 находится в поднятом состоянии, когда со стороны S2 он находится в down.

Смотрим команду **show run**:  

!  
interface Port-channel3  
 description S2  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
!  
interface Ethernet0/0  
 description PC-C  
 switchport access vlan 10  
 switchport mode access  
!  
interface Ethernet0/1  
 shutdown  
!
interface Ethernet0/2  
 shutdown  
!  
interface Ethernet0/3  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 3 mode desirable  
!
interface Ethernet1/0  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 99  
 switchport mode trunk  
 channel-group 3 mode desirable  
!  

По выводу данной команды найдены следующие проблемы с конфигурацией:

**1. Физические интерфейсы под Port-channel3 несконфигурены и погашены.**  
**2. Физические интерфейсы под Port-channel2 привязаны к Port-channel3.**  
**3. Port-channel2 вообще не создан.**  

Конфигурация были исправлена следующими командами:  

S3#conf t  
S3(config)#interface ethernet 0/3  
S3(config-if)#channel-group 2 mode desirable  
S3(config-if)#exit  
S3(config)#interface ethernet 1/0  
S3(config-if)#channel-group 2 mode desirable  
S3(config-if)#end  

S3#conf t
S3(config)#interface range ethernet 0/1-2
S3(config-if-range)# switchport trunk encapsulation dot1q
S3(config-if-range)# switchport trunk native vlan 99
S3(config-if-range)# switchport mode trunk
S3(config-if-range)#channel-group 3 mode desirable
S3(config-if-range)#no shutdown
S3(config-if-range)#end

После исправления аггрегационные интерфейсы появились в транке и так же поднялись по состоянию:

S3#show interfaces trunk  
  
Port        Mode             Encapsulation  Status        Native vlan  
Po2         on               802.1q         trunking      99  
Po3         on               802.1q         trunking      99  

S3#show etherchannel summary  
  
Group  Port-channel  Protocol    Ports  
------+-------------+-----------+-----------------------------------------------  
2      Po2(SU)         PAgP      Et0/3(P)    Et1/0(P)  
3      Po3(SU)         PAgP      Et0/1(P)    Et0/2(P)  


Пинг между коммутаторами есть:

S3#ping 192.168.1.12  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 192.168.1.12, timeout is 2 seconds:  
.!!!!  
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms  
S3#ping 192.168.1.11  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 192.168.1.11, timeout is 2 seconds:  
.!!!!  
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms  
S1#ping 192.168.1.12  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 192.168.1.12, timeout is 2 seconds:  
!!!!!  
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms  


Пинг между компьтерами так же есть:

(скриншоты пингов с ПК)