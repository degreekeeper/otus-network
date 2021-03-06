

# Лабораторная работа. Базовая настройка протокола EIGRP для IPv4





![image-20200531171600562](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531171547474.png)



##### Таблица адресации и Задачи

![image-20200531171654199](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531171654199.png)





## Часть 1. Построение сети и проверка связи

Произведена базовая настройка маршрутизаторов и хостов. Все хосты успешно пингуют шлюзы. Все маршрутизаторы успешно пингуют друг друга:



![image-20200531180428289](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531180428289.png)



## Часть 2. Настройка маршрутизации EIGRP

###### Включим маршрутизацию EIGRP на маршрутизаторе R1. После чего объявим подключенные к нему сети, используя шаблонную маску.



```
R1(config)#router eigrp 10
R1(config-router)#network 10.1.1.0 0.0.0.3
R1(config-router)#network 192.168.1.0 0.0.0.255
R1(config-router)#network 10.3.3.0 0.0.0.3
R1(config-router)#end
```



> Почему рекомендуется использовать шаблонные маски при объявлении сетей? Можно ли исключить маску в какой-нибудь из вышеприведённых инструкций network? Если да, то в какой (в каких)?

Шаблонные маски рекомендуется использовать, т.к. без них автоматически в диапазон попадет вся классовая сеть и EIGRP может объявить сети которые не предполагалось объявлять. В данном случае можно убрать во всех сетях шаблонную маску т.к. сети классовые и все нужные подсети попадут в диапазон 10.0.0.0/8 и 192.168.0.0/16.

###### Включим маршрутизацию EIGRP и объявите напрямую подключенные сети на маршрутизаторах R2 и R3.

R2:

```
R2#conf t
R2(config)#router eigrp 10
R2(config-router)#network 192.168.2.0 0.0.0.255
R2(config-router)#network 10.1.1.0 0.0.0.3
*May 31 18:16:55.147: %DUAL-5-NBRCHANGE: EIGRP-IPv4 10: Neighbor 10.1.1.1 (Serial1/0) is up: new adjacency
R2(config-router)#network 10.2.2.0 0.0.0.3

*May 31 18:18:54.125: %DUAL-5-NBRCHANGE: EIGRP-IPv4 10: Neighbor 10.2.2.1 (Serial1/1) is up: new adjacency

R2(config-router)#end
```

R3:

```
R3#conf t
R3(config)#router eigrp 10
R3(config-router)#network 192.168.3.0 0.0.0.255
R3(config-router)#network 10.3.3.0 0.0.0.3
*May 31 18:18:27.778: %DUAL-5-NBRCHANGE: EIGRP-IPv4 10: Neighbor 10.3.3.1 (Serial1/0) is up: new adjacency
R3(config-router)#network 10.2.2.0 0.0.0.3
*May 31 18:18:54.136: %DUAL-5-NBRCHANGE: EIGRP-IPv4 10: Neighbor 10.2.2.2 (Serial1/1) is up: new adjacency
R3(config-router)#end
```

Как видим из логов сразу после включение объявления маршрутов на интерфейсах через них установилось соседство со смежным маршрутизатором.

Проверяем связность хостов между собой. Все хосты успешно пингуют друг друга:

![image-20200531202140376](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531202140376.png)



## Часть 3. Проверка маршрутизации EIGRP

###### Шаг 1:   Анализ таблицы соседних устройств EIGRP.

![image-20200531202434884](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531202434884.png)

###### Шаг 2:   Проанализируйте таблицу IP-маршрутизации EIGRP.

![image-20200531202539965](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531202539965.png)

> Почему у маршрутизатора R1 два пути к сети 10.2.2.0/30?

Потомучто до него существует два разноценных пути с одинаковой FD( Feasible Distance). Поэтому будет происходить балансировка трафика.



###### Шаг 3:  Проанализируйте таблицу соседних устройств EIGRP.

![image-20200531202850308](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531202850308.png)



> Почему в таблице топологии маршрутизатора R1 отсутствуют возможные преемники?

Потомучто данная команда не выводит возможных преемников. Вывести топологию со всеми направлениями, в том числе которые не являются ни преемниками, ни возможными преемниками можно командой 

```
show ip eigrp topology all-links
```

###### Шаг 4: Проверьте параметры маршрутизации EIGRP и объявленные сети.

![image-20200531203343337](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531202908994.png)



> Ответьте на следующие вопросы, используя результаты команды **show ip protocols**.

> Какой номер автономной системы используется?  - 10

> Какие сети объявляются?  -   10.1.1.0/30, 10.3.3.0/30,  192.168.1.0/24

> Каково значение административной дистанции для маршрутов EIGRP? - 90(для внутренних маршрутов), 170(для внешних маршрутов)

> Сколько маршрутов с равной стоимостью по умолчанию использует EIGRP? - 4



## Часть 4. Настройка пропускной способности и пассивных интерфейсов

В EIGRP используется пропускная способность по умолчанию, основанная на типе интерфейса маршрутизатора. В части 4 необходимо изменить эту пропускную способность, поскольку пропускная способность канала между маршрутизаторами R1 и R3 ниже, чем у каналов R1/R2 и R2/R3. Кроме того, необходимо настроить на каждом маршрутизаторе пассивные интерфейсы.

###### Шаг 1:   Изучим текущие настройки маршрутизации.

R1#show interfaces serial 1/0
Serial1/0 is up, line protocol is up
  Hardware is M4T
  Description: R2 to S1/0
  Internet address is 10.1.1.1/30
  MTU 1500 bytes, **BW 1544 Kbit/sec**, DLY 20000 usec,

> Какова пропускная способность по умолчанию для этого последовательного интерфейса? 

1544кбит/с

> Сколько маршрутов к сети 10.2.2.0/30 содержит таблица маршрутизации? 

2 маршрута

D        10.2.2.0/30 [90/2681856] via 10.3.3.2, 00:38:24, Serial1/1
                                [90/2681856] via 10.1.1.2, 00:38:24, Serial1/0

###### Шаг 2:   Изменим пропускную способность на маршрутизаторах.

```
R1#conf t
R1(config)#interface serial 1/0
R1(config-if)#bandwidth 2000
R1(config-if)#exit
R1(config)#interface serial 1/1
R1(config-if)#bandwidth 64
R1(config-if)#end
```

> Выполним на маршрутизаторе R1 команду **show ip route**. Появились ли изменения в таблице маршрутизации? Если да, в чем они заключаются?

Gateway of last resort is not set

  10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks

C        10.1.1.0/30 is directly connected, Serial1/0
L        10.1.1.1/32 is directly connected, Serial1/0
**D        10.2.2.0/30 [90/2681856] via 10.1.1.2, 00:01:13, Serial1/0**
C        10.3.3.0/30 is directly connected, Serial1/1
L        10.3.3.1/32 is directly connected, Serial1/1
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
D     192.168.2.0/24 [90/1817600] via 10.1.1.2, 00:01:13, Serial1/0
D     192.168.3.0/24 [90/2707456] via 10.1.1.2, 00:01:13, Serial1/0

Появились изменения. Теперь маршрут до сети 10.2.2.0/30 один через Serial1/0 т.к. из-за изменения bandwidth в большую сторону композитная метрика до него стала меньше и он стал выигрывать у маршрута через Serial1/1.

Изменим пропускную способность на остальных маршрутизаторах



```
R2#conf t
R2(config)#interface serial 1/0
R2(config-if)#bandwidth 2000
R2(config)#interface serial 1/1
R2(config-if)#bandwidth 2000
R2(config-if)#end
```



```
R3#conf t
R3(config)#interface serial 1/0
R3(config-if)#bandwidth 64
R3(config)#interface serial 1/1
R3(config-if)#bandwidth 2000
R3(config-if)#end
```



###### Шаг 3:   Проверим изменение пропускной способности.

R1#show interfaces serial 1/0
Serial1/0 is up, line protocol is up
  Hardware is M4T
  Description: R2 to S1/0
  Internet address is 10.1.1.1/30
  MTU 1500 bytes, **BW 2000 Kbit/sec**, DLY 20000 usec,
     reliability 255/255, txload 1/255, rxload 1/255

> Исходя из заданной пропускной способности, попробуйте определить, как будут выглядеть таблицы маршрутизации маршрутизаторов R2 и R3 до выполнения команды **show ip route**. Останутся ли их таблицы маршрутизации прежними или изменятся?

Т.к. с обоих сторон пропускная способность между R1-R3 сильно занижена. Он везде будет резервным. Соответственно на R2 ничего не изменится, на R3 сети с R1 будут видны с R2.



###### Шаг 4:   Настроим на маршрутизаторах R1, R2 и R3 интерфейс G0/0 как пассивный.

Пассивный интерфейс не позволяет передавать исходящие и входящие обновления маршрутизации через настроенный интерфейс. Команда **passive-interface** *интерфейс* заставляет маршрутизатор прекратить отправку и получение пакетов приветствия через интерфейс, но сеть, связанная с этим интерфейсом, по-прежнему будет объявляться для других маршрутизаторов через интерфейсы, не являющиеся пассивными. Интерфейсы маршрутизатора, подключенные к локальным сетям, обычно настраиваются как пассивные.



```
R1(config)#router eigrp 10
R1(config-router)#passive-interface ethernet 0/0
R1(config-router)#end
```

```
R2(config)#router eigrp 10
R2(config-router)#passive-interface ethernet 0/0
R2(config-router)#end
```

R2(config)#router eigrp 10
R2(config-router)#passive-interface ethernet 0/0
R2(config-router)#end

```
R3(config)#router eigrp 10
R3(config-router)#passive-interface ethernet 0/0
R3(config-router)#end
```



Вводим на маршрутизаторах команду show ip protocols и убеждаемся что интерфейс стал пассивным:

![image-20200531212027497](https://github.com/degreekeeper/otus-network/blob/master/less_08_EIGRP/EIGRP_Basic/screenshot/image-20200531212027497.png)

> При выполнении лабораторной работы можно было ограничиться только статической маршрутизацией. Каковы преимущества использования EIGRP?



Да можно было бы ограничиться статической маршрутизацией, но её конфигурация заняла бы больше времени. Так же не было автоматического переключения на резервный маршрут в случае аварии. Так же нельзя бы было сделать балансировку трафика при необходимости.



#### Конец.