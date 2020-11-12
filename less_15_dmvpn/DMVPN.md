# DMVPN



### Топология



![](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_1.jpg)



#### Техническое задание:



1. Настроить GRE между офисами Москва и С.-Петербург
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги
3.  Все узлы в офисах в лабораторной работе должны иметь IP связность
4. План работы и изменения зафиксированы в документации



### 1. Настройка GRE между офисами Москва и С.-Петербург



#### Описание технических решений для задачи:

1.1 Для решения задачи, при условии резервирования, будет настроено два GRE-туннеля:

-  Первый между R14 и R18 до стыка с R24 будет называться Tun1418
-  Второй между R15 и R18 до стыка с R26 будет называться Tun1518

1.2 Для решения задачи проблем с MTU на всех точках туннулелей настроим команды  ***ip mtu 1400*** и ***ip tcp adjust-mss 1360***.

1.3 Для туннеля Tu1418 будет выбрана подсеть 10.14.18/24. Для туннеля Tu1518 будет выбрана подсеть 10.15.18/24



### 1.4 Настройка туннеля Tun1418



#### Настройка R14



```
!
interface Tunnel1418
 description GRE R14-R18
 ip address 10.14.18.14 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 101.0.0.6
 tunnel destination 52.0.0.26
!
```



#### Настройка R18



```
!
interface Tunnel1418
 description GRE R14-R18
 ip address 10.14.18.18 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 52.0.0.26
 tunnel destination 101.0.0.6
!
```



#### Проверим маршрутизацию и связность с R14 до R18



![тут скрин 2](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_2.jpg)



#### Проверим маршрутизацию и связность с R18 до R14



![тут скрин 3](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_3.jpg)



Как видим проблем с mtu нет. Пакеты фрагментируются нормально.





### 1.5 Настройка туннеля Tun1518



#### Настройка R15



```
!
interface Tunnel1518
 description GRE R15-R18
 ip address 10.15.18.15 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 30.1.0.2
 tunnel destination 52.0.0.30
!
```



#### Настройка R18



```
!
interface Tunnel1518
 description GRE R15-R18
 ip address 10.15.18.18 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 52.0.0.30
 tunnel destination 30.1.0.2
!
```



#### Проверим маршрутизацию и связность с R15 до R18




![тут скрин-4](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_4.jpg)



#### Проверим маршрутизацию и связность с R18 до R15


![тут скрин-12](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_12.jpg)





### 2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги



#### Описание технических решений для задачи:

2.1 Т.к. у нас присутсвует более двух точек по заданию в DMVPN выберем режим работы ***Phase 2,*** чтобы была возможность связности напрямую между Spoke в обход Hub в виду теоретического большого географического разнесения точек(Москва - Чокурдах, Москва - Лабытнанги).

2.2 Т.к. на большинстве точек DMVPN у нас есть какое-либо резервирование выберем вариант архитектуры DMVPN - ***Dual Hub-Dual Cloud***. Для второго Hub дополнительно сделаем Hub'ом офис в СПБ. Данная архитектура позволит избежать узкого места в виде одного Hub в Москве, в случае выхода которого, Spoke очень скоро потеряют связность. Это минимизурует недостатки схемы Phase 2.

2.3 Hub в Москве выберем R15, Hub в СПБ будет R18. Network-id DMVPN и Tunnel-интерфейсы будет иметь значения по номерам Hub-роутеров с обоих сторон. Т.е. будут равны 15 и 18.

2.4 Для решения задачи проблем с MTU на всех точках туннулелей настроим команды  ***ip mtu 1400*** и ***ip tcp adjust-mss 1360***.

2.5 Для туннельных сетей первого облака через Офис Москва выберем сеть 192.168.15.0/24. Для второго облака выберем сеть 192.168.18.0/24



### 3. Настройка первого облака DMVPN через Hub в офисе Москва.



#### 3.1 Настройки на Hub -  R15



```
!
interface Tunnel15
 description HUB-NHRP
 ip address 192.168.15.15 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 15
 ip tcp adjust-mss 1360
 tunnel source 30.1.0.2
 tunnel mode gre multipoint
!
```



#### 3.2 Настройка на Spoke R27 в Лабытнанги



```
!
interface Tunnel15
 description SPOKE-NHRP-MSW-R15
 ip address 192.168.15.27 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 30.1.0.2
 ip nhrp map 192.168.15.15 30.1.0.2
 ip nhrp network-id 15
 ip nhrp nhs 192.168.15.15
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
!
```



#### 3.3 Настройки на Spoke R28  в Чокурдах



```
!
interface Tunnel15
 description SPOKE-NHRP-MSW-R15
 ip address 192.168.15.28 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 30.1.0.2
 ip nhrp map 192.168.15.15 30.1.0.2
 ip nhrp network-id 15
 ip nhrp nhs 192.168.15.15
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
!
```



#### Проверим маршрутизацию, соседства по NHRP и доступность Spoke с Hub'a



![тут скрин-5](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_5.jpg)




#### Проверим маршрутизация, соседства по NHRP и доступность между Spoke.



Обращаем внимания что изначально соседства между Spoke нет и первая трассировка идёт через Hub. Последующая трассировка идёт уже напрямую и появляется соседство.



![тут скрин-6](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_6.jpg)



Делаем аналогичную проверку со второго Spoke в Чокурдах



![тут скрин-7](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_7.jpg)





### 4. Настройки второго облака DMVPN через офис СПБ



#### 4.1 Настройки на Hub - R18



```
!
interface Tunnel18
 description HUB-NHRP
 ip address 192.168.18.18 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp network-id 18
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/3
 tunnel mode gre multipoint
!
```



#### 4.2 Настройки на Spoke R27 в Лабытнанги



```
!
interface Tunnel18
 description SPOKE-NHRP-SPB-R18
 ip address 192.168.18.27 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 52.0.0.30
 ip nhrp map 192.168.18.18 52.0.0.30
 ip nhrp network-id 18
 ip nhrp nhs 192.168.18.18
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
!
```



#### 4.3 Настройки на Spoke R28 в Чокурдах



```
!
interface Tunnel18
 description SPOKE-NHRP-SPB-R18
 ip address 192.168.18.28 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 52.0.0.30
 ip nhrp map 192.168.18.18 52.0.0.30
 ip nhrp network-id 18
 ip nhrp nhs 192.168.18.18
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
!
```



#### Проверим маршрутизацию, соседства по NHRP и доступность Spoke с Hub'a



![тут скрин-8](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_8.jpg)





#### Проверим маршрутизация, соседства по NHRP и доступность между Spoke на R27



Видим что в соседствах на R27 ещё нет R28 и трассировка идёт через Hub на R18



![тут скрин-9](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_9.jpg)



Повторная трассировка уже идёт напрямую на R28 и появляется соседство с R28



![тут скрин-10](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_10.jpg)



Проведем аналогичную проверку на другому Spoke на R28





![тут скрин-11](https://github.com/degreekeeper/otus-network/blob/master/less_15_dmvpn/screens/Screenshot_11.jpg)







### Конец.