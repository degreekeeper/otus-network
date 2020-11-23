# IPSEC



### Топология



![Тут скрин 1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_1.jpg)



### Техническое задание



Цель: Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги

​                            

1. Настроите GRE поверх IPSec между офисами Москва и С.-Петербург
2. Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги
3. Все узлы в офисах в лабораторной работе должны иметь IP связность
4. План работы и изменения зафиксированы в документации 



#### Технические решения по задаче 1:



1.1 Шифроваться будут два GRE-туннеля между МСК и СПБ - это Tun1418 и Tun1518, соответственно между R14-R18 и R15-R18.

1.2 IPSEC авторизация будет происходить через пароли командой ***crypto isakmp key 'password' address y.y.y.y***

1.3 IPSEC будет работать в транспортном режиме ***mode transport***, ip-адреса шифроваться не будут. 

1.4 Применяться политика IPSEC будет с помощью ipsec-профайла командой ***crypto ipsec profile 'profile-name'***

1.5 Профайл будет применен к туннельным интерфейсам GRE с помощью команды ***tunnel protection ipsec profile 'profile-name'***

1.6 Стандартом шифрование первой фазы ISAKMP будут выбраны AES 256, SHA 256, DH group 2. Стандартом шифрования второй фазы IKE будет выбран  ESP AES 256



#### Настройки на R14



##### IPSEC



```
!
crypto isakmp policy 1418
 encr aes 256
 hash sha256
 authentication pre-share
 group 2
!
crypto isakmp key CISCO1418 address 52.0.0.26
!
!
crypto ipsec transform-set IPSEC-PRESHARE-R14R18 esp-aes 256
 mode transport
!
crypto ipsec profile IPSEC-PRESHARE-R14R18
 set transform-set IPSEC-PRESHARE-R14R18
!
```



##### Туннельный интерфейс



```
!
interface Tunnel1418
 description GRE R14-R18
 ip address 10.14.18.14 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 101.0.0.6
 tunnel destination 52.0.0.26
 tunnel protection ipsec profile IPSEC-PRESHARE-R14R18
!
```



#### Настройки на R15



##### IPSEC



```
!
crypto isakmp policy 1518
 encr aes 256
 hash sha256
 authentication pre-share
 group 2
!
crypto isakmp key CISCO1518 address 52.0.0.30
!
crypto ipsec transform-set IPSEC-PRESHARE-R15R18 esp-aes 256
 mode transport
!
crypto ipsec profile IPSEC-PRESHARE-R15R18
 set transform-set IPSEC-PRESHARE-R15R18
!
```



##### Туннельный интерфейс



```
!
interface Tunnel1518
 description GRE R15-R18
 ip address 10.15.18.15 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 30.1.0.2
 tunnel destination 52.0.0.30
 tunnel protection ipsec profile IPSEC-PRESHARE-R15R18
!
```



#### Настройки на R18



##### IPSEC



```
!
crypto isakmp policy 1418
 encr aes 256
 hash sha256
 authentication pre-share
 group 2
!
crypto isakmp policy 1518
 encr aes 256
 hash sha256
 authentication pre-share
 group 2
!
crypto isakmp key CISCO1418 address 101.0.0.6
crypto isakmp key CISCO1518 address 30.1.0.2
!
!
crypto ipsec transform-set IPSEC-PRESHARE-R15R18 esp-aes 256
 mode transport
crypto ipsec transform-set IPSEC-PRESHARE-R14R18 esp-aes 256
 mode transport
!
crypto ipsec profile IPSEC-PRESHARE-R14R18
 set transform-set IPSEC-PRESHARE-R14R18
!
crypto ipsec profile IPSEC-PRESHARE-R15R18
 set transform-set IPSEC-PRESHARE-R15R18
!
```



##### Туннельные интерфейсы



```
!
interface Tunnel1418
 description GRE R14-R18
 ip address 10.14.18.18 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 52.0.0.26
 tunnel destination 101.0.0.6
 tunnel protection ipsec profile IPSEC-PRESHARE-R14R18
!
interface Tunnel1518
 description GRE R15-R18
 ip address 10.15.18.18 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 52.0.0.30
 tunnel destination 30.1.0.2
 tunnel protection ipsec profile IPSEC-PRESHARE-R15R18
!
```



#### Проверка работы IPSEC



##### Проверяем состояние туннелей на R14



![тут скрин р14_1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R14_1.jpg)



##### Проверяем состояние туннелей на R15



![тут скрин р15_7](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_7.jpg)



##### Проверяем состояние туннелей на R18



![тут скрин р18_1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R18_1.jpg)



#### Проверка доступности туннельных адресов



##### Проверяем пинги на R14



![тут скрин р14_2](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R14_2.jpg)



##### Проверяем пинги на R15



![тут скрин р15_8](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_8.jpg)



##### Проверяем пинги на R18



![тут скрин р18-2](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R18_2.jpg)





#### 2.1 Технические решения по задаче 2:





2.1.1 В качестве CA сервера будет выбран R19. Т.к. на нем самом не будут поднят IPSEC. А так же потому что до него уже успешно протестирован проброс 23-его порта черезе NAT. Под http-сервер для CA будет проброшен 80-ый порт через оба бордер роутера R14 и R15. Это исключит потерю CA в случае аварии на одном из этих роутеров или на каналах до них. Для этого будет использована команда: 
***ip nat inside source static tcp 10.201.77.19 80 101.0.10.253 80 extendable***



2.1.2 На клиентах CA будет настроен IPSEC поверх уже имеющегося у нас DMVPN между Москвой, Лабытнанги, Чокурдах.

2.1.3 IPSEC авторизация будет происходить через сертификаты и сертификационный сервере CA на R19 с помощью команды ***authentication rsa-sig***

2.1.4 IPSEC будет настроен в туннельном режиме с помощью команд 

***crypto ipsec transform-set 'name'***
 ***mode tunnel***

2.1.5 Будет использован ipsec-профайл. Команду см. выше пункт 1.4

2.1.6 IPSEC-профайл будет применен на туннельных интерфейсах под DMVPN - на всех роутерах это ***Tunnel15***. Команду см. выше пункт 1.5



#### 2.2 Настройка проброса порта 80 под HTTP-сервер на R19 на R14 и R15



##### R14



```
!
ip nat inside source static tcp 10.201.77.19 80 101.0.10.253 80 extendable
!
```



##### R15



```
!
ip nat inside source static tcp 10.201.77.19 80 101.0.10.253 80 extendable
!
```



#### Проверка доступности http-сервера под CA-сервер по http через проброс 80-ого порта с R15, R27, R28



##### R15



![screen r19-6](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_6.jpg)



##### R27



![screen r19-4](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_4.jpg)



##### R28



![screen r19-5](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_5.jpg)





#### 2.3 Настройка CA (Certificate Authority) на R19



##### R19



```
!
ip domain name otus.lab
!
crypto pki server CA
 no database archive
!
!
ip http server
no ip http secure-server
!

```



##### теперь сгенерируем собственный сертификат CA и сделаем его экспортируемым



![тут скриншот р19_1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_1.jpg)



##### поднимаем сервер и выводим его из shutdown. Нам потребуется ввести 8-ми значный пароль. Видим сообщение, что сервер поднялся



![скрин р19-2](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_2.jpg)



##### Проверяем есть ли запросы на валидацию сертификатов от других роутеров. Видим что на данный момент запросов нет:



![тут скрин р19-11](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_11.jpg)



#### 2.4 Настройка клиентов обмена ключами CA и клиентов CA





##### Для начала посмотрим процесс обмена ключами и между CA и клиентами CA и их валидации друг у друга, это ещё не настройка самого IPSEC:



##### R27



Генерируем пару ключей сертификат на роутере-клиенте:



![скрин р27-0](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_0.jpg)



Настраиваем трастпоинт. Фактически это указание CA-сервера с которым работаем и которому доверяем и указание пары ключей сертификата, которую мы отправим на CA:



```
!
crypto pki trustpoint IPSEC-CA-R19
 enrollment url http://101.0.10.253:80
 subject-name CN=R27,OU=IPSEC-CA-R19,O=LABYTNANGI,C=RU
 revocation-check none
 rsakeypair IPSEC-CA-R19
!
```



Запрашиваем у CA его сертификат. Как получим нас спросят доверять ли ему, нажимаем yes. После чего сертификат CA скачается на роутер и будет принят:



![тут скрин р27-1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_1.jpg)



Дальше отправляем наш собственный сертификат, который мы сгенерировали до этого на CA для его валидации там. Нам потребуется ввести пароль 8 знаков. Так же уточнить добавить ли в сертификат информацию об ip-адресе роутера и серийном номере роутера. Можно ответить - нет, на случай если будет замена роутера или измениться ip-адрес. На вопрос "запросить сертификат у CA?" отвечаем - да.



![тут скрин р27-2](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_2.jpg)



Дальше смотрим список запросов на подпись сертификата на CA. Видим наш запрос и его стату pending(ожидание).



![тут скрин р27-3](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_3.jpg)



На нашем роутере сам сертификат так же в статусе ожидания




![тут скрин р27-4](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_4.jpg)



##### R28



Повторяем все пункты для R28



Генерация своего сертификата:



![тут r28-1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_1.jpg)



Настраиваем трастпоинт. Фактически это указание CA-сервера с которым работаем и которому доверяем и указание пары ключей сертификата, которую мы отправим на CA:



```
!
crypto pki trustpoint IPSEC-CA-R19
 enrollment url http://101.0.10.253:80
 subject-name CN=R28,OU=IPSEC-CA-R19,O=CHOCURAKH,C=RU
 revocation-check none
 rsakeypair IPSEC-CA-R19
!
```



запрос и скачивание сертификат у CA:



![тут  скрин р28-2](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_2.jpg)



отправка нашего собественного сертификат на CA для валидации:



![скрин р28-3](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_3.jpg)



Видим что сертификат в статусе ожидание



![скрин р28-4](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_4.jpg)



##### R15



Повторяем все пункты для R15



Генерация своего сертификата:



![тут r15-1](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_1.jpg)



Настраиваем трастпоинт. Фактически это указание CA-сервера с которым работаем и которому доверяем и указание пары ключей сертификата, которую мы отправим на CA. **Обращаем внимание, что R15 находится в сети офиса Москва и адрес CA нужно так же указаывать локальный 10.201.77.19**



```
!
crypto pki trustpoint IPSEC-CA-R19
 enrollment url http://10.201.77.19:80
 subject-name CN=R15,OU=IPSEC-CA-R19,O=SPB,C=RU
 revocation-check none
 rsakeypair IPSEC-CA-R19
!
```



![скрин р15-2](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_2.jpg)



запрос и скачивание сертификат у CA:



![тут  скрин р15-3](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_3.jpg)



отправка нашего собественного сертификат на CA для валидации:



![скрин р15-4](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_4.jpg)



Видим что сертификат в статусе ожидание:



![скрин р15-5](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_5.jpg)



##### R19



Проверяем ещё раз запросы на подпись сертификатов от других роутеров. Видим все три запроса от R15,R27,R28 в статусе ожидание:



![screen r19-7](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_7.jpg)



Подписываем все сертификаты



![тут скрин Р19-8](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_8.jpg)



Проверяем ещё раз статус запросов, видим что они перешли в статус ***granted(предоставлен)***



![тут скрин р19-9](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_9.jpg)



Ждём получения сертификатов на роутерах. ***Внимание: возможно из-за виртуализации получение каждого сертификата заняло по 10 минут. Итого 30 минут.*** Видим сообщение о получение на каждом роутере:



![тут р19-10](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_10.jpg)



Смотрим на R19 и не видим больше ни одного запроса. Т.к. они удаляются после того как CA получил подтверждение о получении сертификата от клиентов:



![тут скрин р19-11](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R19_11.jpg)



Проверяем статус сертификатов на R15, R27, R28 и видим что они перешли в статус "принят":



R15



![тут скрин р15-6](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_6.jpg)



R27



![тут скрин 27-5](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_5.jpg)



R28



![скрин р28-5](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_5.jpg)



#### 2.5 Настройка IPSEC на роутерах поверх DMVPN



##### R15



##### IPSEC



```
!
crypto isakmp policy 19
 authentication rsa-sig
!
!
crypto ipsec transform-set IPSEC-CA-R19 esp-aes
 mode tunnel
!
crypto ipsec profile IPSEC-CA-R19
 set transform-set IPSEC-CA-R19
!
```



##### Туннельный интерфейс



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
 tunnel protection ipsec profile IPSEC-CA-R19
!
```



##### R27



##### IPSEC



```
!
crypto isakmp policy 19
!
!
crypto ipsec transform-set IPSEC-CA-R19 esp-aes
 mode tunnel
!
crypto ipsec profile IPSEC-CA-R19
 set transform-set IPSEC-CA-R19
!
```



##### Туннельный интерфейс



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
 tunnel protection ipsec profile IPSEC-CA-R19
!
```



##### R28



##### IPSEC



```
!
crypto isakmp policy 19
!
!
crypto ipsec transform-set IPSEC-CA-R19 esp-aes
 mode tunnel
!
crypto ipsec profile IPSEC-CA-R19
 set transform-set IPSEC-CA-R19
!
```



##### Туннельный интерфейс



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
 tunnel protection ipsec profile IPSEC-CA-R19
!
```



#### 2.6 Проверим работу IPSEC поверх DMVPN



##### R15



Сессии ISAKMP и IKE



![тут скрин 15-11](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_11.jpg)



Соседства NHRP и DMVPN



![тут 15-9](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_9.jpg)



Проверям связность со SPOKE



![тут скрин 15-10](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R15_10.jpg)



##### R27



Сессии ISAKMP и IKE



![тут скрин 27-6](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_6.jpg)



Соседства NHRP и DMVPN



![тут 27-7](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_7.jpg)



Проверям связность с HUB и SPOKE



![тут скрин 27-8](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R27_8.jpg)



##### R28



Сессии ISAKMP и IKE



![тут скрин 28-6](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_6.jpg)



Соседства NHRP и DMVPN



![тут 28-7](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_7.jpg)



Проверям связность с HUB и SPOKE



![тут скрин 28-8](https://github.com/degreekeeper/otus-network/blob/master/less_16_ipsec/screenshots/Screenshot_R28_8.jpg)





### Конец.