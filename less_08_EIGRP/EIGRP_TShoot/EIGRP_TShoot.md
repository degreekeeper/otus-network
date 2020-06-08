# Лабораторная работа. Настройка расширенных функций EIGRP для IPv4



**![image-20200604225548982](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200604225548982.png)**

![image-20200604225645757](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200604225645757.png)

### Часть 1:   Создание сети и настройка основных параметров устройства





### Часть 2:   Настройка EIGRP и проверка подключения

Настроим EIGRP для общей связности сети и выставим значение bandwidth на интерфейсах:

R1

```
R1(config)#router eigrp 1
R1(config-router)#network 192.168.1.0 0.0.0.255
R1(config-router)#network 192.168.12.0 0.0.0.3
R1(config-router)#network 192.168.13.0 0.0.0.3
R1(config-router)#passive-interface  ethernet 0/0
R1(config-router)#ex
R1(config)#interface serial 1/0
R1(config-if)#bandwidth 1024
R1(config-if)#exit
R1(config)#interface serial 1/1
R1(config-if)#bandwidth 64
R1(config-if)#end
```

R2

```
R2#conf t
R2(config)#router eigrp 1
R2(config-router)#network 192.168.2.0 0.0.0.255
R2(config-router)#network 192.168.12.0 0.0.0.3
R2(config-router)#network 192.168.23.0 0.0.0.3
R2(config-router)#passive-interface ethernet 0/0
R2(config-router)#end
R2#conf t
R2(config)#interface serial 1/0
R2(config-if)#bandwidth 1024
R2(config-if)#end
```

R3

```
R3#conf t
R3(config)#router eigrp 1
R3(config-router)#network 192.168.3.0 0.0.0.255
R3(config-router)#network 192.168.13.0 0.0.0.3
R3(config-router)#network 192.168.23.0 0.0.0.3
R3(config-router)#passive-interface ethernet 0/0
R3(config-router)#end
R3#conf t
R3(config)#interface serial 1/0
R3(config-if)#bandwidth 64
R3(config-if)#end
```



Проверьте связь всех ПК между собой:



![image-20200608224042238](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200608224042238.png)



### Часть 3:   Настройка EIGRP для автоматического объединения

Настроим EIGRP для автоматического обновления. Для начал посмотрим настройки на R1:

![image-20200608224401455](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200608224401455.png)



Как видимо авто-суммаризация по умолчанию выключена

Настроим loopback-интерфейсы на R1 согласно таблице и добавим их в EIGRP анонсы:

```
interface Loopback1
 ip address 192.168.11.1 255.255.255.252
!
interface Loopback5
 ip address 192.168.11.5 255.255.255.252
!
interface Loopback9
 ip address 192.168.11.9 255.255.255.252
!
interface Loopback13
 ip address 192.168.11.13 255.255.255.252
!
R1#conf t
R1(config)#router eigrp 1
R1(config-router)#network 192.168.11.0 0.0.0.3
R1(config-router)#network 192.168.11.4 0.0.0.3
R1(config-router)#network 192.168.11.8 0.0.0.3
R1(config-router)#network 192.168.11.12 0.0.0.3
R1(config-router)#end
```

R1(config)#router eigrp 1
R1(config-router)#network 192.168.11.0 0.0.0.3
R1(config-router)#network 192.168.11.4 0.0.0.3
R1(config-router)#network 192.168.11.8 0.0.0.3
R1(config-router)#network 192.168.11.12 0.0.0.3
R1(config-router)#end



На R2 путь до loopback-интерфейсов R1 видны как отдельные маршруты:

![image-20200608231322637](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200608231322637.png)



Включим автоматическое объединение маршрутов на R1:

```
R1#conf t
R1(config)#router eigrp 1
R1(config-router)#auto-summary
```

Теперь таблица R2 выглядит следующим образом:

![image-20200608232151153](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200608232151153.png)



Видим что остался один маршрут до объединенной сети 192.168.11.0/24

Настроим loopback-интейрфейсы и авто-объединение на R3:

```
interface Loopback1
 ip address 192.168.33.1 255.255.255.252
interface Loopback5
 ip address 192.168.33.5 255.255.255.252
interface Loopback9
 ip address 192.168.33.9 255.255.255.252
interface Loopback13
 ip address 192.168.33.13 255.255.255.252
R3#conf t
R3(config)#router eigrp 1
R3(config-router)#network 192.168.33.0 0.0.0.3
R3(config-router)#network 192.168.33.4 0.0.0.3
R3(config-router)#network 192.168.33.8 0.0.0.3
R3(config-router)#network 192.168.33.12 0.0.0.3
R3(config-router)#auto-summary
R3(config-router)#end
```



### Часть 4:   Настройка и распространение статического маршрута по умолчанию

Настроим loopback-интерфейс и маршурт по умолчанию на R2, а так же добавим его в анонсы EIGRP: 

```
interface Loopback1
description Loop1
ip address 192.168.22.1 255.255.255.252
end
R2(config)#ip route 0.0.0.0 0.0.0.0 Loopback 1
R2(config)#router eigrp 1
R2(config-router)#redistribute static
R2(config-router)#end
```

по выводу команды видим что статические маршруты распространяются:

![image-20200608233707833](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200608233707833.png)



Посмотрим команду на R1 по маршрутам EIGRP:



```
R1#show ip route eigrp | include 0.0.0.0
Gateway of last resort is 192.168.12.2 to network 0.0.0.0
D*EX  0.0.0.0/0 [170/3139840] via 192.168.12.2, 00:06:40, Serial1/0
```



### Часть 5:   Подгонка EIGRP

Настроим параметры использования пропускной способности для EIGRP на R1 и R2 чтобы разрешить трафику EIGRP использовать только 75% пропускной способности:



```
R1#conf t
R1(config)#interface serial 1/0
R1(config-if)#ip bandwidth-percent eigrp 1 75



R2#conf t
R2(config)#interface serial 1/0
R2(config-if)#ip bandwidth-percent eigrp 1 75
R2(config-if)#end
```



Настроим линк между R1 на R3 на использование под EIGRP только 40% канала



```
R1#conf t
R1(config)#interface serial 1/1
R1(config-if)#ip bandwidth-percent eigrp 1 40
R1(config-if)#end



R3#conf t
R3(config)#interface serial 1/0
R3(config-if)#ip bandwidth-percent eigrp 1 40
R3(config-if)#end
```



### Настроим интервал отправки пакетов приветствия (hello) и таймер удержания для EIGRP.

Смотрим стандартные настройки таймеров на R2

![image-20200608234804085](C:\Users\va\AppData\Roaming\Typora\typora-user-images\image-20200608234804085.png)



Hello - 5 секунд, Hold - 15 секунд

престароим интервалы таймеров на R1:

```
R1#conf t
R1(config)#interface serial 1/0
R1(config-if)#ip hello-interval eigrp 1 60
R1(config-if)#ip hold-time eigrp 1 180
R1(config-if)#exit
R1(config)#interface serial 1/1
R1(config-if)#ip hello-interval eigrp 1 60
R1(config-if)#ip hold-time eigrp 1 180
R1(config-if)#end
```

Перенастроим аналогично R2 и R3:

```
R2#conf t
R2(config)#interface serial 1/0
R2(config-if)#ip hello-interval eigrp 1 60
R2(config-if)#ip hold-time eigrp 1 180
R2(config)#interface serial 1/1
R2(config-if)#ip hello-interval eigrp 1 60
R2(config-if)#ip hold-time eigrp 1 180
R2(config-if)#end



R3#conf t
R3(config)#interface serial 1/0
R3(config-if)#ip hello-interval eigrp 1 60
R3(config-if)#ip hold-time eigrp 1 180
R3(config-if)#exit
R3(config)#interface serial 1/1
R3(config-if)#ip hello-interval eigrp 1 60
R3(config-if)#ip hold-time eigrp 1 180
R3(config-if)#end
```





### Конец.





















