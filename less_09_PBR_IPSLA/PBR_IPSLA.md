# Маршрутизация на основе политик (PBR)

Цель: Настроить политику маршрутизации в офисе Чокурдах
Распределить трафик между 2 линками

![тут скриншот 1 топология](https://github.com/degreekeeper/otus-network/blob/master/less_09_PBR_IPSLA/screenshots/Screenshot_1.jpg)

1. Настроить отслеживание линков провайдера через технологию IP SLA
2. Настроить политику маршрутизации для сетей офиса
3.  Распределить трафик между двумя линками с провайдером
4. Настройть для офиса Лабытнанги маршрут по-умолчанию
5. План работы и изменения зафиксировать в документации

### 1. Настроим отслеживание линков провайдера через технологию IP SLA на R28

Настроим два SLA с номерами 33 и 37, что соответствует последним октетам ап-линк адресов провайдера. 

- **SLA 33** будет SLA для линка к **R25**
- **SLA 37** будет SLA для линка к **R26**

Для track даем соответствующие номер 33 и 37, проверяем ими доступность - *reachability* 

#### Берутся следующие параметры для SLA:

- Проверяться доступность будет с помощью *icmp-echo*
- Часто запросов icmp будет 10 секунд - *frequency 10*
- Провайдер будет считаться недоступным после 3 промежутков *frequency*. Т.е. *threshold* выставляем *30000* миллисекунд(30 секунд).
- Запуск IP SLA выставляем немедленный, после загрузки маршрутизатора - *start-time now*
- Время работы неограниченное (беспконечный) - *life forever* 

#### Примеры конфигурации IP SLA и TRACK на R28

```
!
track 33 ip sla 33 reachability
!
track 37 ip sla 37 reachability
!
ip sla auto discovery
ip sla 33
 icmp-echo 52.0.0.33 source-interface Ethernet0/1
 threshold 30000
 frequency 10
ip sla schedule 33 life forever start-time now
ip sla 37
 icmp-echo 52.0.0.37 source-interface Ethernet0/0
 threshold 30000
 frequency 10
ip sla schedule 37 life forever start-time now
!
```

С помощью IP SLA для начала зарезервируем маршуты по умолчанию и будем следить за их доступностью. Для этого применим floating static default-route с помощью ручного изменения AD(метрики) до 50 по основному маршруту и 100 по резервному.

```
!
ip route 0.0.0.0 0.0.0.0 52.0.0.33 10 track 33
ip route 0.0.0.0 0.0.0.0 52.0.0.37 20 track 37
!
```

Видим в таблице маршрутизации только один маршрут по умолчанию

![тут скриншот 2](https://github.com/degreekeeper/otus-network/blob/master/less_09_PBR_IPSLA/screenshots/Screenshot_2.jpg)



### 2. Настроим PBR для офиса Чокурдах.

##### 2.1 для начала настроим два access-list'а 



**ACL VPC30_VLAN2003** - будет обрабатывать только сеть **VLAN 2003 10.203.100.0/24**

```
ip access-list standard VPC30_VLAN2003
 permit 10.203.100.0 0.0.0.255
 deny   any
```

**ACL VPC31_VLAN2033** - будет обрабатывать только сеть **VLAN 2033 10.203.101.0/24**

```
ip access-list standard VPC31_VLAN2033
 permit 10.203.101.0 0.0.0.255
 deny   any
```

Оба листа **не будут** обрабатывать в PBR все остальные сети. Для этого прописывается правило *deny any*

##### 2.2 теперь настроим route-map

Они должны удовлетворять следующим параметрам:

1. Распределять нагрузкую сетей Чокурдах 10.203.100.0/24 и 10.203.101..0/24 между линками до провайдера
2. В случае аварии на одном из линков автоматически переводить трафик на резервный

Настроим два route-map



##### R25

```
!
route-map R25 permit 10
 match ip address VPC30_VLAN2003
 set ip next-hop verify-availability 52.0.0.33 10 track 33
!
route-map R25 permit 20
!
```

##### R26

```
!
route-map R26 permit 10
 match ip address VPC31_VLAN2033
 set ip next-hop verify-availability 52.0.0.37 10 track 37
!
route-map R26 permit 20
!
```



##### 2.3 Для проверки работы route-map  и ip-sla временно настроим статические маршруты на R25 до R26 



**R25**

```
ip route 0.0.0.0 0.0.0.0 52.0.0.34
```

**R26**

```
ip route 10.203.101.0 255.255.255.0 52.0.0.38
ip route 52.0.0.40 255.255.255.252 52.0.0.5
!
```

Проверим работу PBR(route-map R26)

Сделаем пинг и трассировку с VPC31 (VLAN2033) до удаленного роутера Лабытнанги и его ip 52.0.0.42

![тут скриншот 3](https://github.com/degreekeeper/otus-network/blob/master/less_09_PBR_IPSLA/screenshots/Screenshot_3.jpg)

Как видим R27(Лабытнанги пингуется), но трассировка идёт через R26(52.0.0.37)->R25(52.0.0.5)->R27(52.0.0.42). И это не смотря, на то, что в таблице маршрутизации маршрут по умолчанию через который доступна 52.0.0.42 идёт через R25(52.0.0.33)

##### 2.4 Проверим IP SLA  и PBR. 

Погасим на R28 линк eth0/0 в сторону R26.

```
R28(config)#interface ethernet 0/0
R28(config-if)#shutdown
R28(config-if)#end
```

Видим, что IP SLA 37 перешел в *state down*

```
*Sep 28 20:16:20.275: %TRACKING-5-STATE: 37 ip sla 37 reachability Up->Down
```

Так же видим увеличивающийся счетчик *failures*

![тут скриншот 4](https://github.com/degreekeeper/otus-network/blob/master/less_09_PBR_IPSLA/screenshots/Screenshot_4.jpg)

52.0.0.42 (R27 Лабытнанги) по прежнему доступен. Однако видим, что трассировка изменилась, идёт через оставшийся линк до R25.

![тут скриншот 5](https://github.com/degreekeeper/otus-network/blob/master/less_09_PBR_IPSLA/screenshots/Screenshot_5.jpg)



Восстанавливаем линк до R26 и видим как IP SLA снова переходит в *state UP*

```
R28#
*Sep 28 20:23:20.449: %TRACKING-5-STATE: 37 ip sla 37 reachability Down->Up
R28#
```

Удалим временные статические маршруты на R25  и R26, т.к. в скоре их заменит динамическая маршрутизация.



### 3. Настроим маршрут по умолчанию в офисе Лабытнанги (R27)



Тут будем использовать и адрес next-hop и выходной интерфейс:

```
!
ip route 0.0.0.0 0.0.0.0 Ethernet0/0 52.0.0.41
!
```



Пинг до loopback-интерфейса R25 есть. Маршрут по умолчанию работает:



![тут скриншот 6](https://github.com/degreekeeper/otus-network/blob/master/less_09_PBR_IPSLA/screenshots/Screenshot_6.jpg)



#### Конец.