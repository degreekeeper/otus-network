**OSPF Single Area**

**Часть 1**

**Тополгия и базовая настройка**

![Топология](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/screen_1_topology.jpg)

Маршрутизаторы и хосты настроены в соответсвии с методичкой. Маршрутизаторы пингуют друг друга. Хосты пингуют свои шлюзы.

![Пинг](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_2_ping.jpg)

![пинг](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_3_ping.jpg)

![Ещё пинг](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_4_ping.jpg)


**Часть 2**

**Приступаем к настройке протокола OSPF на маршрутизаторах**

настройка на R1:

R1#conf t  
R1(config)#router ospf 1  
R1(config-router)#network 192.168.1.0 0.0.0.255 area 0  
R1(config-router)#network 192.168.12.0 0.0.0.3 area 0  
R1(config-router)#network 192.168.13.0 0.0.0.3 area 0  

настройка на R2 и R3 аналогичная с соответствующими сетями

после настройках на всех роутерах устанавливается ospf-соседство, как пример скриншот с R1:

![OSPF соседство](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_5_ospf_adj.jpg)

а так же видим что все сети появились в таблице маршрутизации

![OSPF соседи](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_6_neigbr.jpg)

Проверить только маршруты распространенные по OSPF можно командой:

show ip route ospf

Информация по настройкам OSPF:

![Протокол ip](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_7_ip_protocol.jpg)

![show ip ospf](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_8.jpg)

так можно проверить настройки OSPF на интерфейсах, команда show ip ospf interface даст более полный вывод:

![show ip ospf brief](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_9.jpg)

Связанность между всем хостами есть:

![Пинги](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_10.jpg)



**Часть 3**	

**Изменение назначенных идентификаторов маршрутизаторов**  

Идентификатор OSPF-маршрутизатора используется для уникальной идентификации домена маршрутизации OSPF. 
Маршрутизаторам компании Cisco идентификатор назначается одним из трех способов и в следующем порядке:

1)	IP-адрес, настроенный с помощью команды OSPF router-id (при наличии)
2)	Наибольший IP-адрес любого из loopback-адресов маршрутизатора (при наличии)
3)	Наибольший активный IP-адрес любого из физических интерфейсов маршрутизатора

На наших маршрутизаторах ни настроены loopback интерфейсы и router-id поэтому id назначаются на основе самого большого ip-адреса на физ. интерфейсе:

**3.1 Изменим идентификатор маршрутизатора с помощью loopback-адреса**

R1#conf t  
R1(config)#interface loopback 0  
*Apr 25 19:47:00.503: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up  
R1(config-if)#ip address 1.1.1.1 255.255.255.255  
R1(config-if)#end  


На R2 и R3 настроим соответственно 2.2.2.2 и 3.3.3.3.  
После чего потребуется перезагрузка роутеров, для того чтобы роутер-id стал ip-адрес loopback интерфейса.

![interface loopback](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_11_loop.jpg)

И мы увидим что теперь именно ip-адреса loopback видны как router-id

R1#show ip ospf neighbor  

Neighbor ID     Pri   State           Dead Time   Address         Interface  
3.3.3.3           0   FULL/  -        00:00:33    192.168.13.2    Serial1/1  
2.2.2.2           0   FULL/  -        00:00:35    192.168.12.2    Serial1/0   

**3.2 Изменение идентификатора маршрутизатора R1 с помощью команды router-id**

Предпочтительным методом настройки идентификатора маршрутизатора является команда router-id.

Настроем его поочерёдно на всех роутерах:


R1(config)#router ospf 1  
R1(config-router)#router-id 11.11.11.11  
% OSPF: Reload or use "clear ip ospf process" command, for this to take effect  
R1(config-router)#end  

После чего требуется либо перезагрузить роутер, либо применить команду clear ip ospf process, что предпочтительней, т.к. будет сброшены
соседства ospf.  

R1#clear ip ospf process  
Reset ALL OSPF processes? [no]: yes  
R1#  
*Apr 25 20:13:02.437: %OSPF-5-ADJCHG: Process 1, Nbr 3.3.3.3 on Serial1/1 from FULL to DOWN, Neighbor Down: Interface down or detached  
*Apr 25 20:13:02.437: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on Serial1/0 from FULL to DOWN, Neighbor Down: Interface down or detached  
*Apr 25 20:13:02.470: %OSPF-5-ADJCHG: Process 1, Nbr 3.3.3.3 on Serial1/1 from LOADING to FULL, Loading Done  
*Apr 25 20:13:02.470: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on Serial1/0 from LOADING to FULL, Loading Done  

Проверяем, что router-id теперь соответствуют измененым нами настройками:

R1#show ip ospf neighbor  

Neighbor ID     Pri   State           Dead Time   Address         Interface  
33.33.33.33       0   FULL/  -        00:00:33    192.168.13.2    Serial1/1  
22.22.22.22       0   FULL/  -        00:00:37    192.168.12.2    Serial1/0  

**Часть 4**

**4.1 Настройка пассивных интерфейсов**

Команда passive-interface запрещает отправку обновлений маршрутов через указанный интерфейс маршрутизатора. 
В большинстве случаев команда используется для уменьшения трафика в локальных сетях, поскольку им не нужно получать сообщения 
протокола динамической маршрутизации.

Настроим интерфейсы в сторону хостов в пассивный режим:  

R1(config)#router ospf 1  
R1(config-router)#passive-interface ethernet 0/0  
R1(config-router)#end  

После этого в выводе команды show ip ospf interface ethernet 0/0 пропадёт таймер хеллоу пакетов:

  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5  
    oob-resync timeout 40  
    No Hellos (Passive interface)  

Маршрут не пропал после этого на соседних маршрутизаторах:

R2#show ip route  
  
   2.0.0.0/32 is subnetted, 1 subnets  
C        2.2.2.2 is directly connected, Loopback0  
O     192.168.1.0/24 [110/74] via 192.168.12.1, 00:43:09, Serial1/0  


**4.2	Настроим на маршрутизаторе пассивный интерфейс в качестве интерфейса по умолчанию**

Данная настройка сделает все интерфейсы пассивными по умолчанию.


R2#conf t  
Enter configuration commands, one per line.  End with CNTL/Z.  
R2(config)#router ospf 1  
R2(config-router)#passive-interface default  
R2(config-router)#end  

После этого у нас упадут все ospf-соседства, т.к. интерфейсы перестанут посылать хеллоу-пакеты:  

*Apr 25 21:03:33.217: %OSPF-5-ADJCHG: Process 1, Nbr 11.11.11.11 on Serial1/0 from FULL to DOWN, Neighbor Down: Interface down or detached  
*Apr 25 21:03:33.217: %OSPF-5-ADJCHG: Process 1, Nbr 33.33.33.33 on Serial1/1 from FULL to DOWN, Neighbor Down: Interface down or detached  

Т.к. R2 больше не распространяет свои сети по OSPF на соседних роутерах пропал маршрут до его сетей:

R1#sh ip route 192.168.2.0  
% Network not in table  

Выполним команду отключающую режим пассивного интерфейса на портах:

R2(config)#router ospf 1  
R2(config-router)#no passive-interface serial 1/0  
*Apr 25 21:09:26.164: %OSPF-5-ADJCHG: Process 1, Nbr 11.11.11.11 on Serial1/0 from LOADING to FULL, Loading Done  
 

Маршрут до сетей R2 снова появился на соседних роутерах:

![no passive interface](https://github.com/degreekeeper/otus-network/blob/master/less_04_ospf_single/screen/Screenshot_12_no_pass.jpg)

*Какой интерфейс использует R3 для маршрута к сети 192.168.2.0/24?*

Используется интерфейс Serial1/0 к которому присоединен R1

*Чему равна суммарная метрика стоимости для сети 192.168.2.0/24 на R3?*

Суммарная метрика равна 138

Routing entry for 192.168.2.0/24  
  Known via "ospf 1", distance 110, metric 138, type intra area  
  Last update from 192.168.13.1 on Serial1/0, 00:00:39 ago  


*Отображается ли маршрутизатор R2 как соседнее устройство OSPF на маршрутизаторе R1?* 

Да  

*Отображается ли маршрутизатор R2 как соседнее устройство OSPF на маршрутизаторе R3?*

Нет  

*Что дает вам эта информация?*

R2#show ip ospf neighbor  

Neighbor ID     Pri   State           Dead Time   Address         Interface  
11.11.11.11       0   FULL/  -        00:00:31    192.168.12.1    Serial1/0  


Т.к. соседства между R2 и R3 нет. О сетях включенных в R2 роутер R3 узнаёт только от R1 и маршрут идёт через единственный 
длинный путь R3-R1-R2.

Включим объявления маршрутов на интерфесе Serial 1/1 на R2:

R2#conf t  
R2(config)#router ospf 1  
R2(config-router)#no passive-interface Serial 1/1  
R2(config-router)#end  

Проверяем маршрут до сетей роутера R2 на роутере R3:  

R3#show ip route 192.168.2.0  
Routing entry for 192.168.2.0/24  
  Known via "ospf 1", distance 110, metric 74, type intra area  
  Last update from 192.168.23.1 on Serial1/1, 00:00:53 ago  
  Routing Descriptor Blocks:  
  * 192.168.23.1, from 22.22.22.22, 00:00:53 ago, via Serial1/1  

*Какой интерфейс использует R3 для маршрута к сети 192.168.2.0/24?*  

Используется интерфейс Serial 1/1  

*Чему равна суммарная метрика стоимости для сети 192.168.2.0/24 на маршрутизаторе R3? Как она была рассчитана?*  

Метрика равна 74

*Как она была рассчитана?*  

Рассчитана из суммы метрик интерфейса eth1/1+Serial1/1 - 10+64=74  

*Отображается ли маршрутизатор R2 как сосед OSPF для маршрутизатора R3?*   

Да, отображается:

Neighbor ID     Pri   State           Dead Time   Address         Interface  
22.22.22.22       0   FULL/  -        00:00:38    192.168.23.1    Serial1/1  
11.11.11.11       0   FULL/  -        00:00:31    192.168.13.1    Serial1/0  

**Часть 5**	

**Изменение метрик OSPF**  

Метрики OSPF можно менять с помощью команд 

1. auto-cost reference-bandwidth   
2. bandwidth  
3. ip ospf cost  

**5.1	Изменим заданную пропускную способность для маршрутизаторов**

Сейчас Serial-интерфейсы имеют цену 64, а ethernet-интерфейсы имеют цену 10  

Меня значение auto-cost reference-bandwidth  на всех роутерах:  

R1#conf t    
R1(config)#router ospf 1  
R1(config-router)#auto-cost reference-bandwidth 10000  
R1(config-router)#end  

Теперь Serial-интерфейсы имеют цены 6476, а ethernet-интерфейсы 1000

R2#show ip ospf interface ethernet 0/0  
Ethernet0/0 is up, line protocol is up  
  Internet Address 192.168.2.1/24, Area 0, Attached via Network Statement  
  Process ID 1, Router ID 22.22.22.22, Network Type BROADCAST, Cost: 1000  

R2#show ip ospf interface serial 1/0  
Serial1/0 is up, line protocol is up  
  Internet Address 192.168.12.2/30, Area 0, Attached via Network Statement  
  Process ID 1, Router ID 22.22.22.22, Network Type POINT_TO_POINT, Cost: 6476  

Новая стоимость маршрута до 192.168.3.0/24 на R1 равна 7476  

O     192.168.3.0/24 [110/7476] via 192.168.13.2, 00:05:39, Serial1/1  

Возвращаем стандартное значение командой:  

R3(config-router)#auto-cost reference-bandwidth 100  

*Почему может понадобиться изменить эталонную пропускную способность OSPF по умолчанию?*  

При наличии в сети высокоскоростных интерфейсов таких ка 10гб/с OSPF будет назначать им стоимость 1 по умолчанию, т.е. такую же как и 1Гб/с.
Это будет означать, что при рассчете маршрута они будут считаться равнозначными, хотя разницу в пропускной способности в 10 раз.

**5.2	Измените пропускную способность для интерфейса**


Пропускная способность серийного интерфейса показана как 1544кб/с. Фактически физическая пропускная способность 128кб/с.

R1#show interfaces serial 1/0
Serial1/0 is up, line protocol is up
  Hardware is M4T
  Internet address is 192.168.12.1/30
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,

В маршрутной информации метрика 128:

O        192.168.23.0 [110/128] via 192.168.13.2, 00:26:13, Serial1/1   
                      [110/128] via 192.168.12.2, 00:27:45, Serial1/0  
					  
Изменим настройку bandwidth

R1(config)#interface serial 1/0  
R1(config-if)#bandwidth 128  

Маршрут через Serial 1/0 пропал из таблицы маршрутизации, т.к. его стоимость теперь хуже альтернативного  

O        192.168.23.0 [110/128] via 192.168.13.2, 00:29:58, Serial1/1  

Это видно в настройках.

R1#show ip ospf interface brief  
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C  
Et0/0        1     0               192.168.1.1/24     10    DR    0/0  
Se1/1        1     0               192.168.13.1/30    64    P2P   1/1  
Se1/0        1     0               192.168.12.1/30    781   P2P   1/1  

После изменения bandwith на Serial 1/0, снова будет два маршрута до сети, т.к. их стоимость снова будет равной, только теперь 845:  

O        192.168.23.0 [110/845] via 192.168.13.2, 00:00:05, Serial1/1  
                      [110/845] via 192.168.12.2, 00:00:05, Serial1/0 

*Объясните, как были рассчитаны стоимости маршрутов от маршрутизатора R1 для сетей 192.168.3.0/24 и 192.168.23.0/30?*  

Рассчитывается из стоимости линков: 781+64=845  

На R3 цена серийного линка не изменится, команду bandwidth надо вводить на каждом интерфейсе отдельно, она не согласуется автоматом:

O     192.168.1.0/24 [110/74] via 192.168.13.1, 00:36:14, Serial1/0  
O     192.168.2.0/24 [110/74] via 192.168.23.1, 00:36:14, Serial1/1  

Выставляем на все остальныех серийных интерфейсах в топологии 128. Изменится цена маршрута на R1 до 192.168.23.0/30

O        192.168.23.0 [110/1562] via 192.168.13.2, 00:01:04, Serial1/1  
                      [110/1562] via 192.168.12.2, 00:01:04, Serial1/0  

Это связано с тем что теперь суммарная стоимость линков до это сети равна 781+781=1562  


**5.3	Измените стоимость маршрута**


Для расчёта стоимости канала по умолчанию OSPF использует значение пропускной способности. 
Но этот расчёт можно изменить, вручную задав стоимость канала с помощью команды ip ospf cost. 
Подобно команде bandwidth, команда ip ospf cost влияет только на ту сторону канала, где она была применена.  

Выполняем команду ip ospf cost 1565 на R1 Serial 1/1:

R1(config)#interface serial 1/1  
R1(config-if)#ip ospf cost 1565  
R1(config-if)#end  

Теперь все маршруты с R1 идут через R2:

O     192.168.2.0/24 [110/791] via 192.168.12.2, 00:16:46, Serial1/0  
O     192.168.3.0/24 [110/1572] via 192.168.12.2, 00:03:57, Serial1/0  
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/1562] via 192.168.12.2, 00:09:32, Serial1/0  

*Почему маршрут к сети 192.168.3.0/24 от маршрутизатора R1 теперь проходит через R2?*  
					  
Т.к. стоимость маршрута через R2 теперь меньше. 1572 против 1575 через R3.  

**Вопросы на повторение:**


*1.	Почему так важно управлять назначением идентификатора маршрутизатора при использовании протокола OSPF?*

Потому что иначе DR-роутером может стать какой-то слабый роутер или неудобный в плане топологии. Так же в случае если, DR выбирается
на основании самого большого ip-адреса физ.интерфейса частый падения данного интерфейса будут вызывать частые перевыборы DR, что 
повлечет за собой частую потерю сходимости сети на момент постоянных выборов.

*2.	Почему в этой лабораторной работе не рассматривается процесс выбора DR/BDR?*

Для трех роутеров в топологии это фактически не требуется, т.к. один бы был DR, другой BDR, другой DROTHER и фактически все бы слали
анонсы всем.

*3.	Почему рекомендуется настраивать интерфейс OSPF как пассивный?*

Чтобы не рассылать лишнюю информацию в интерфейсы которые не участвуют в работе OSPF.  

**Конец**