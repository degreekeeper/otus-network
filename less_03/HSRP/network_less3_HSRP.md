#  HSRP

**Настроим устройства согласно следующей таблице:**

![Table](https://github.com/degreekeeper/otus-network/blob/master/less_03/HSRP/HSRP_3.jpg "Таблица") 


**ПК друг друга пингуют:**

PC-A> ping 192.168.1.33  

84 bytes from 192.168.1.33 icmp_seq=1 ttl=64 time=0.557 ms  
84 bytes from 192.168.1.33 icmp_seq=2 ttl=64 time=0.548 ms  


PC-C> ping 192.168.1.3  

84 bytes from 192.168.1.3 icmp_seq=1 ttl=255 time=0.278 ms  
84 bytes from 192.168.1.3 icmp_seq=2 ttl=255 time=0.466 ms  

**Был настроен протокол RIPv2 на всех маршрутизаторах, а так же дефолтный маршрут и его распространение через RIPv2 на R2:**

**R1:**

router rip  
 version 2  
 network 10.0.0.0  
 network 192.168.1.0  


**R2:**

router rip  
 version 2  
 network 10.0.0.0  
 default-information originate  
!  
ip route 0.0.0.0 0.0.0.0 Loopback0  

**R3:**

router rip  
 version 2  
 network 10.0.0.0  
 network 192.168.1.0  

**Удачный пинг с PC-A на все интерфейсы R1, R2, R3, PC-C, а так же аналогичный тест с PC-C:**

![Ping PC-A](https://github.com/degreekeeper/otus-network/blob/master/less_03/HSRP/HSRP_1.jpg "Пинг с PC-A")

**Определяем путь от PC-A до 209.165.200.225**

PC-A> trace  209.165.200.225 -P 1  
trace to 209.165.200.225, 8 hops max (ICMP), press Ctrl+C to stop  
 1   192.168.1.1   0.334 ms  0.310 ms  0.283 ms  
 2   209.165.200.225   8.754 ms  8.695 ms  9.236 ms  
 
**Определяем путь от PC-С до 209.165.200.225**

PC-C> trace  209.165.200.225 -P 1  
trace to 209.165.200.225, 8 hops max (ICMP), press Ctrl+C to stop  
 1   192.168.1.3   0.378 ms  0.332 ms  0.270 ms  
 2   209.165.200.225   8.845 ms  8.738 ms  8.789 ms  

**Пробуем погасить интерфейс в сторону шлюза по умолчанию.**

S1#conf t  
S1(config)#interface ethernet 0/0  
S1(config-if)#shutdown  

**Пинг пропадает:**  

84 bytes from 209.165.200.225 icmp_seq=96 ttl=254 time=9.185 ms    
84 bytes from 209.165.200.225 icmp_seq=97 ttl=254 time=9.060 ms   
209.165.200.225 icmp_seq=98 timeout  
209.165.200.225 icmp_seq=99 timeout

**Обратно поднимаем интерфейс:**

S1(config-if)#no shutdown  
S1(config-if)#end  

**Пинг через некоторое время появляется:**

209.165.200.225 icmp_seq=150 timeout  
209.165.200.225 icmp_seq=151 timeout  
84 bytes from 209.165.200.225 icmp_seq=152 ttl=254 time=8.952 ms  
84 bytes from 209.165.200.225 icmp_seq=153 ttl=254 time=7.059 ms  


**На PC-C при отключении на S3 интерфейса eth0/0 результат получился такой же.**


**Настраиваем протокол HSRP:**

**на R1 и делаем его активным с помощью повышения дефолтного приоритета:**  

R1(config)#interface Ethernet0/0  
R1(config-if)#standby version 2  
R1(config-if)#standby 1 ip 192.168.1.254  
R1(config-if)#standby 1 priority 150  
R1(config-if)#standby 1 preempt  
R1(config-if)#end  

**на R3:**

interface Ethernet0/0  
 description S3  
 ip address 192.168.1.3 255.255.255.0  
 standby version 2  
 standby 1 ip 192.168.1.254  
end  


**Проверяем настройки на R1 и R3:**

R1#show standby  
Ethernet0/0 - Group 1 (version 2)  
  State is Active  
    2 state changes, last state change 00:02:56  
  Virtual IP address is 192.168.1.254  
  Active virtual MAC address is 0000.0c9f.f001  
    Local virtual MAC address is 0000.0c9f.f001 (v2 default)  
  Hello time 3 sec, hold time 10 sec  
    Next hello sent in 2.288 secs  
  Preemption enabled  
  Active router is local  
  Standby router is unknown  
  Priority 150 (configured 150)  
  Group name is "hsrp-Et0/0-1" (default)  


R3#show standby  
Ethernet0/0 - Group 1 (version 2)  
  State is Standby  
    1 state change, last state change 00:01:16  
  Virtual IP address is 192.168.1.254  
  Active virtual MAC address is 0000.0c9f.f001  
    Local virtual MAC address is 0000.0c9f.f001 (v2 default)  
  Hello time 3 sec, hold time 10 sec  
    Next hello sent in 0.576 secs  
  Preemption disabled  
  Active router is 192.168.1.1, priority 150 (expires in 9.792 sec)  
    MAC address is aabb.cc00.1000  
  Standby router is local  
  Priority 100 (default 100)  
  Group name is "hsrp-Et0/0-1" (default)  

**Исходя из данных команд:  
Активным является R1  
Для виртуального интерфейса используется мак-адрес 0000.0c9f.f001  
Для резервного роутера используется приоритет 100 и ip 192.168.1.254**  

**Выводы команд *show stanby brief*:**  

R1#show standby brief  
                     P indicates configured to preempt.  
                     |  
Interface   Grp  Pri P State   Active          Standby         Virtual IP  
Et0/0       1    150 P Active  local           192.168.1.3     192.168.1.254  


R3#show standby brief  
                     P indicates configured to preempt.  
                     |  
Interface   Grp  Pri P State   Active          Standby         Virtual IP  
Et0/0       1    100   Standby 192.168.1.1     local           192.168.1.254  

**Соответственно как шлюз на PC-A, PC-C, S1, S3 следует настраивать виртуальный 192.168.1.254
На всех данных хостах перенастроен шлюз по умолчанию.**


**Пинги после перенастройки до loopback на R2 проходят успешно.**

**При отключении интерфейса на S1 в сторону R1 ни пропало ни одного пинг-запроса с PC-A. Отработал HSRP:**

84 bytes from 209.165.200.225 icmp_seq=164 ttl=254 time=8.992 ms
84 bytes from 209.165.200.225 icmp_seq=165 ttl=254 time=9.127 ms
84 bytes from 209.165.200.225 icmp_seq=166 ttl=254 time=9.066 ms
84 bytes from 209.165.200.225 icmp_seq=167 ttl=254 time=6.253 ms

**В данный момент активным считается R3:**

R3#show standby brief  
                     P indicates configured to preempt.  
                     |  
Interface   Grp  Pri P State   Active          Standby         Virtual IP  
Et0/0       1    100   Active  local           unknown         192.168.1.254  

**При поднятии интерфейса на R1 обратно, он сразу же станет активным благодаря команде** 

standby 1 preempt

**Меняем приоритет HSRP на R3 на 200:**  

R3(config-if)#standby 1 priority 200  

**R3 все ещё в standby не смотря на то, что его приоритет сейчас выше чем у R1:**

R3#show standby brief  
                     P indicates configured to preempt.  
                     |  
Interface   Grp  Pri P State   Active          Standby         Virtual IP  
Et0/0       1    200   Standby 192.168.1.1     local           192.168.1.254  

**Применим команду приоритетного вытеснения и R1 сразу станет активным роутером:**  

R3#show standby brief  
                     P indicates configured to preempt.  
                     |  
Interface   Grp  Pri P State   Active          Standby         Virtual IP  
Et0/0       1    200 P Active  local           192.168.1.1     192.168.1.254  

**Избытычность локальной сети требуется для исключения влияния на сервисы выхода из строя шлюза по умолчанию или линка в сторону него.**

Конец.



