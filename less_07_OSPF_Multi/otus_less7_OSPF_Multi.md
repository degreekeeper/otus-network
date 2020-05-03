**Настройка OSPFv2 для нескольких областей**  

Топология  

![скрин1](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/screen_1_topology.jpg)  

**Часть 1**  

**Создание сети и настройка основных параметров устройства**  

Делаем базовые настройке согласно топологии и таблице адресации на всех роутерах и их интерфейсах.  

Проверяем настройки интерфейсов, приводим пример с роутера R2:  

R2#show ip interface brief  
Interface              IP-Address      OK? Method Status                Protocol  
Serial0/0              192.168.12.2    YES manual up                    up  
Serial0/1              192.168.23.1    YES manual up                    up  
Serial0/2              unassigned      YES unset  administratively down down  
Serial0/3              unassigned      YES unset  administratively down down  
Loopback6              192.168.6.1     YES manual up                    up  

Проверяем доступность соседних, как пример пинги с R2:  

![скрин2](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_2_ping.jpg)  

**Часть 2**  	

**Настройка сети OSPFv2 для нескольких областей**  

Настроим OSPF.  

**Шаг 1** 

**Определим роли маршрутизаторов в топологии**  

Определите магистральные маршрутизаторы: R1 и R2   
Определите граничные маршрутизаторы автономной системы (ASBR): R1    
Определите граничные маршрутизаторы области (ABR): R1 и R2    
Определите внутренние маршрутизаторы: R3    
  
**Шаг 2**  

**Настроим OSPF на R1 под разные области и запишем какие команды использовали:**



R1#show running-config | section ospf  

router ospf 1  
 router-id 1.1.1.1  
 passive-interface Loopback1  
 passive-interface Loopback2  
 network 192.168.1.0 0.0.0.255 area 1  
 network 192.168.2.0 0.0.0.255 area 1  
 network 192.168.12.0 0.0.0.3 area 0  
 default-information originate  
 
R1#show running-config interface loopback 1  
Building configuration...  


interface Loopback1   
 description 192.168.1.0/24  
 ip address 192.168.1.1 255.255.255.0  
 ip ospf 1 area 1  
end  

R1#show running-config interface loopback 2  
Building configuration...  


interface Loopback2  
 description 192.168.2.0/24  
 ip address 192.168.2.1 255.255.255.0  
 ip ospf 1 area 1  
end  

R1#show running-config interface serial 0/0  
Building configuration...  


interface Serial0/0  
 description R2  
 ip address 192.168.12.1 255.255.255.252  
 ip ospf 1 area 0  
 serial restart-delay 0  
end  

**Шаг 3**  

**Настроим OSPF на R2 под разные области:**  

R2#show running-config | section ospf  

router ospf 1  
 router-id 2.2.2.2  
 passive-interface Loopback6  
 network 192.168.6.0 0.0.0.255 area 3  
 network 192.168.12.0 0.0.0.3 area 0  
 network 192.168.23.0 0.0.0.3 area 3  

R2#show running-config  


interface Loopback6  
 description 192.168.6.0/24  
 ip address 192.168.6.1 255.255.255.0  
 ip ospf 1 area 3  
!  
interface Serial0/0  
 description R1  
 ip address 192.168.12.2 255.255.255.252  
 ip ospf 1 area 0  
 serial restart-delay 0  
!  
interface Serial0/1   
 description R3  
 ip address 192.168.23.1 255.255.255.252  
 ip ospf 1 area 3  
 serial restart-delay 0  

**Шаг 4**  

**Настроим OSPF на R3 под разные области:**  
 
 
 R3#show running-config | section ospf  

router ospf 1  
 router-id 3.3.3.3  
 passive-interface Loopback4  
 passive-interface Loopback5  
 network 192.168.4.0 0.0.0.255 area 3  
 network 192.168.5.0 0.0.0.255 area 3  
 network 192.168.23.0 0.0.0.3 area 3  

 
 R3#show running-config  

 
 interface Loopback4  
 description 192.168.4.0./24  
 ip address 192.168.4.1 255.255.255.0  
 ip ospf 1 area 3  
!  
interface Loopback5  
 description 192.168.5.0/24  
 ip address 192.168.5.1 255.255.255.0  
 ip ospf 1 area 3  
  
interface Serial0/1  
 description R2  
 ip address 192.168.23.2 255.255.255.252  
 ip ospf 1 area 3  
 serial restart-delay 0  
 
**Шаг 5**
 
 **Проверка настроек OSPF**
 
Проверяем на всех роутерах правильность настроек OSPF, как пример скриншоты вывода команд проверки с R1:

![скрин3](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_3_ip_protocol.jpg)  


*К какому типу маршрутизаторов OSPF относится каждый маршрутизатор?*  
R1: ASBR и ABR  
R2: ABR  
R3: внутризонный маршрутизатор  

Проверяем дальше настройки OSPF, как пример вывод команды на R2:  

![скрин4](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_4_neigbor.jpg)  

Проверяем стоимости маршрутов на всех роутерах, как пример скриншот с R3:  

![скрин5](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_5_cost_ospf.jpg)

**Шаг 6**  

Включаем аутентификацию на уровне интерфейсов на всех роутерах на всех их серийных интерфейсах участвующих в процеесе OSPG 
следующими командами:  

interface Serial0/0  

 ip ospf authentication message-digest  
 ip ospf authentication-key Cisco123  

*Почему перед настройкой аутентификации OSPF полезно проверить правильность работы OSPF?*  

Потому что можно потерять доступ на оборудование  

**Шаг 7**  

Проверяем, что все соседства после настройки аутентификации восстановились:  

R2#show  ip ospf neighbor  
  
Neighbor ID     Pri   State           Dead Time   Address         Interface  
1.1.1.1           0   FULL/  -        00:00:39    192.168.12.1    Serial0/0  
3.3.3.3           0   FULL/  -        00:00:34    192.168.23.2    Serial0/1  

**Часть 3**  


**Часть 3:	Настройка межобластных суммарных маршрутов**  

Пример таблицы маршрутизации роутеров на примере R1:

![скрин6](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_6_routes_no_summ.jpg)  


Пример базы данных LSDB для R1:

![скрин7](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_7_lsdb.jpg)  

Настроим межобластные суммарные маршруты:

Для области 1 суммарным маршрутом будет сеть 192.168.0.0/22  

Настроим суммарный маршрут:  

R1(config)#router ospf 1  
R1(config-router)#area 1 range 192.168.0.0 255.255.252.0  

Для области 3 суммарным маршрутом будет сеть 192.168.4.0/22

Настроим суммарный маршрут:

router ospf 1  
 area 3 range 192.168.4.0 255.255.252.0  

Смотрим как уменьшились таблицы маршрутизации на всех роутерах:  

![скрин8](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_8_R1.jpg)  

![скрин9](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_9_R2.jpg)  

![скрин10](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_10_R3.jpg)  

Запишем все идентификаторы всех суммарных маршрутов на всех роутерах:  

**R1**  

                Summary Net Link States (Area 0)

Link ID         
192.168.0.0     
192.168.4.0     
192.168.23.0    

                Summary Net Link States (Area 1)  

Link ID           
192.168.4.0       
192.168.12.0      
192.168.23.0      

**R2**  

                Summary Net Link States (Area 0)    

Link ID  
192.168.0.0       
192.168.4.0       
192.168.23.0      

                Summary Net Link States (Area 3)  

Link ID           
192.168.0.0      
192.168.12.0      

**R3**    

                Summary Net Link States (Area 3)  

Link ID          
192.168.0.0  
192.168.12.0  

*Пакет LSA какого типа передается в магистраль маршрутизатором ABR, когда включено объединение межобластных маршрутов?*  

Используются LSA 3-его типа  

Проверяем сквозные пинги со всех роутеров до всех удаленных сетей. Пинги есть. Пример с R3:  

![скрин11](https://github.com/degreekeeper/otus-network/blob/master/less_07_OSPF_Multi/screen/Screenshot_11_final_ping.jpg)  

*Какие три преимущества при проектировании сети предоставляет OSPF для нескольких областей?*  

1. Меньшая таблица маршрутизации  
2. Уменьшение базы LSDB, следовательно меньшая нагрузка на оборудование  
3. Уменьшение частоты расчет SPF, меньше лавинных рассылок    

**Конец.**  










