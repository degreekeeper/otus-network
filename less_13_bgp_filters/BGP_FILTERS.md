# BGP. Управление анонсами

### Топология



![тут скриншот 0](https://github.com/degreekeeper/otus-network/blob/master/less_13_bgp_filters/screenshots/Screenshot_0.jpg)



### Техническое задание



Цель: Настроить фильтрацию для офисе Москва
Настроить фильтрацию для офисе С.-Петербург

​                                   

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург
5. Все сети в лабораторной работе должны иметь IP связность
6. План работы и изменения зафиксированы в документации 



### 1. Описание технических решений проекта:



1.1 IPV6 настраивается так же как и IPV4 где это возможно.

1.2 R14 и R15 для исключения транзитного трафика будут анонсировать EBGP соседям только сети из своей AS 1001. Для этого на них на всех EBGP-сессиях будет настроен исходящий фильтр as-path, использующий регулярное выражение ***^$*** - ***ip as-path access-list 1 permit ^$***.  Данный фильтр пропускает только анонсы с пустым значением в атрибуте as-path, что как раз соответствует сетям зарожденным внутри AS.

1.3 R18 для исключения транзитного трафика будет анонсировать EBGP соседям только сети из своей AS 2042. Для этого на всех EBGP-сессиях будет настроен исходящий фильтр с prefix-list пропускающий только агрегированную сеть самого офиса - 10.202.0.0/16, с помощью команды - ***ip prefix-list XXXXX seq 10 permit YYYYYY ge ZZ***.

1.4 Провайдер Киторн(AS 101) будет анонсировать своему клиенту офису Москва(AS1001) только маршрут по умолчания через EBGP. Для этого на BGP-сессии будет настроен исходящий фильтр с prefix-list - с помощью аналогичной команды которая приведена в пункте 1.3. 

1.5 Провайдер Ламас(AS 301) будет анонсировать своему клиенту офису Москва(AS 1001) маршрут по умолчанию и все сети офиса СПБ(AS 2042). 

1.5.1 Для этого на IPV4-сесии будет настроен ***route-map OUT_AS1001***, который будет включать в себя две части(два clause). Первый clause будет пропускать маршрут по умолчанию с помощью prefix-list, команда префикс-листа будет аналогина пункту 1.3. Условие в route-map будет выглядеть как - ***match ip address prefix-list XXXX***. Второй будет пропускать только сети зарожденные в AS2042 с помощью as-path фильтра с регулярным выражением ***_2042$***, которое соответствует атрибуту as-path в анонсах начинающемуся с 2042. Делаться это будет аналогичной командой из пункта 1.2. Условия в route-map будет выглядеть так -  ***match as-path 1***.

1.5.2 Для IPV6 будут настроены на исходящем направлении будет настроены два отдельных фильтра. Первый prefix-list - имеющий два записи:

***ipv6 prefix-list IPV6_ONLY_DEFAULT seq 10 permit ::/0***
***ipv6 prefix-list IPV6_ONLY_DEFAULT seq 20 permit ::/0 ge 1***

Вторым будет тот же as-path фильтр, что и для IPV4:

***ip as-path access-list 1 permit _2042$***



Настроены они буду по отдельности на нейборе-IPV6:

  ***neighbor 2001:A:C:1::2 prefix-list IPV6_ONLY_DEFAULT out***
  ***neighbor 2001:A:C:1::2 filter-list 1 out***





### 2. Настройки фильтрации в офисе Москва



#### Настройки R14



```
!
router bgp 1001
 !
 address-family ipv4
  neighbor 101.0.0.5 filter-list 1 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:A:A:1::1 filter-list 1 out
 exit-address-family
!
ip as-path access-list 1 permit ^$
!
```



##### Таблица BGP до применения фильтрации

тут скриншот

##### Таблцица BGP после применения фильтрации

тут скриншот


#### Настройки R15



```
!
router bgp 1001
 !
 address-family ipv4
  neighbor 30.1.0.1 filter-list 1 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:A:C:1::1 filter-list 1 out
 exit-address-family
!
ip as-path access-list 1 permit ^$
!
```



##### Таблица BGP до применения фильтрации

тут скриншот

##### Таблцица BGP после применения фильтрации

тут скриншот



### 3. Настройки фильтрации в офисе СПБ



#### Настройки R18



```
!
router bgp 2042
 !
 address-family ipv4
  neighbor 52.0.0.25 prefix-list DENY-TRANSIT out
  neighbor 52.0.0.29 prefix-list DENY-TRANSIT out
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:A:B:7::1 prefix-list IPV6-DENY-TRANSIT out
  neighbor 2001:A:B:8::1 prefix-list IPV6-DENY-TRANSIT out
 exit-address-family
!
ip prefix-list DENY-TRANSIT seq 10 permit 10.202.0.0/15 ge 16
!
ipv6 prefix-list IPV6-DENY-TRANSIT seq 5 permit 2001:F:F2::/47 ge 48
!
```



##### Таблица BGP до применения фильтрации

тут скриншот

##### Таблцица BGP после применения фильтрации

тут скриншот



#### 4. Настройки фильтрации в провайдере Киторн

 

#### Настройки на R22



```
!
router bgp 101
 !
 address-family ipv4
  neighbor 101.0.0.6 prefix-list ONLY_DEFAULT out
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:A:A:1::2 prefix-list IPV6_ONLY_DEFAULT out
 exit-address-family
!
ip prefix-list ONLY_DEFAULT seq 10 permit 0.0.0.0/0
!
ipv6 prefix-list IPV6_ONLY_DEFAULT seq 10 permit ::/0
!
```



##### Таблица BGP до применения фильтрации

тут скриншот

##### Таблцица BGP после применения фильтрации

тут скриншот





### 5. Настройки фильтрации в провайдере Ламас



#### Настройки на R21



```
!
router bgp 301
 !
 address-family ipv4
  neighbor 30.1.0.2 route-map OUT_AS1001 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2001:A:C:1::2 prefix-list IPV6_ONLY_DEFAULT out
  neighbor 2001:A:C:1::2 filter-list 1 out
 exit-address-family
!
ip as-path access-list 1 permit _2042$
!
ip prefix-list ONLY_DEFAULT seq 10 permit 0.0.0.0/0
!
ipv6 prefix-list IPV6_ONLY_DEFAULT seq 10 permit ::/0
ipv6 prefix-list IPV6_ONLY_DEFAULT seq 20 permit ::/0 ge 1

!
route-map OUT_AS1001 permit 10
 match ip address prefix-list ONLY_DEFAULT
!
route-map OUT_AS1001 permit 20
 match as-path 1
!
```



##### Таблица BGP до применения фильтрации

тут скриншот

##### Таблцица BGP после применения фильтрации



![](https://github.com/degreekeeper/otus-network/blob/master/less_13_bgp_filters/screenshots/Screenshot_20.jpg)

--
![](https://github.com/degreekeeper/otus-network/blob/master/less_13_bgp_filters/screenshots/Screenshot_21.jpg)



#### Конец.