# OSPF
## Введение
OSPF — протокол динамической маршрутизации, использующий концепцию разделения на области в целях масштабирования. Административная дистанция OSPF — 110

Основные свойства протокола OSPF:
- Быстрая сходимость
- Масштабируемость (подходит для маленьких и больших сетей)
- Безопасность (поддежка аутентиикации)
- Эффективность (испольование алгоритма поиска кратчайшего пути)
При настроенном OSPF маршрутизатор формирует таблицу топологии с использованием результатов вычислений, основанных на алгоритме кратчайшего
пути (SPF) Дейкстры. Алгоритм поиска кратчайшего пути основывается на данных о совокупной стоимости доступа к точке назначения. Стоимость доступа определятся на основе скорости интерфейса.

Чтобы повысить эффективность и масштабируемость OSPF, протокол поддерживает иерархическую маршрутизацию с помощью областей (area). Область OSPF (area) — Часть сети, которой ограничивается формирование базы данных о состоянии каналов. Маршрутизаторы, находящиеся в одной и той же области, имеют одну и ту же базу данных о топологии сети. Для определения областей применяются идентификаторы областей.

Протоколы OSPF бвывают 2-х версий:
- OSPFv2
- OSPFv3
Основным отличием протоколов является то, что OSPFv2 работает с IPv4, а OSPFv3 — c IPv6. Маршрутизаторы в OSPF классифицируются на основе выполняемой ими функции:
- Internal router (внутренний маршрутизатор) — маршрутизатор, все
интерфейсы которого находятся в одной и той же области.
- Backbone router (магистральный маршрутизатор) — это маршрутизатор, который находится в магистральной зоне (area 0).
- ABR (пограничный маргрутизатор области) — маршрутизатор, интерфейсы которого подключены к разным областям.
- ASBR (Граничный маршрутизатор автономной системы) — это маршрутизатор, у которого интерфейс подключен к внешней сети.
Также с помощью OSPF можно настроить ассиметричный роутинг. Ассиметричная маршрутизация — возможность пересекать сеть в одном направлении, используя один путь, и возвращаться через другой путь.
## Цели домашнего задания
Создать домашнюю сетевую лабораторию. Научится настраивать протокол OSPF в Linux-based системах.
## Описание домашнего задания
1. Развернуть 3 виртуальные машины
2. Объединить их разными vlan
- настроить OSPF между машинами на базе Quagga;
- изобразить ассиметричный роутинг;
- сделать один из линков "дорогим", но что бы при этом роутинг был
симметричным.
## Пошаговая инструкция выполнения домашнего задания
### Разворачиваем 3 виртуальные машины
Так как мы планируем настроить OSPF, все 3 виртуальные машины должны быть соединены между собой (разными VLAN), а также иметь одну (или несколько) доолнительных сетей, к которым, далее OSPF сформирует маршруты.
Создаём каталог, в котором будут храниться настройки виртуальной машины. В каталоге создаём Vagrantfile. Бокс подгружаем локально:
```
vagrant box add centos7 /home/sam/centos7.box
```
Результатом выполнения vagrant up будут 3 созданные виртуальные машины, которые соединены между собой сетями (10.0.10.0/30, 10.0.11.0/30 и 10.0.12.0/30). У каждого роутера есть дополнительная сеть:
- на router1 — 192.168.10.0/24
- на router2 — 192.168.20.0/24
- на router3 — 192.168.30.0/24
- 
На данном этапе ping до дополнительных сетей (192.168.10-30.0/24) с соседних роутеров будет недоступен.
### Установка пакетов для тестирования и настройки OSPF между машинами на базе Quagga
Для тестирования нам понадобятся пакеты vim, traceroute, tcpdump, net-tools, frr. Также на сервер будут скопированы конфиги frr для каждой машины. Пошагово это выглядит так:
- Отключаем файерволл ufw и удаляем его из автозагрузки
- Устанавливаем frr
- Разрешаем (включаем) маршрутизацию транзитных пакетов
- Включаем демон ospfd в FRR. Для этого в файле /etc/frr/daemons необходимо поменять параметры zebra и ospfd на yes
- На серверах создаем файл /etc/frr/frr.conf, который будет содержать в себе информацию о требуемых интерфейсах и OSPF.
Через ansible реализация конфига frr выглядит так (пример для router1):
```
!Указание версии FRR
frr version 8.1
frr defaults traditional
!Указываем имя машины
hostname {{ hostname1 }}
log syslog informational
no ipv6 forwarding
service integrated-vtysh-config
!
!Добавляем информацию об интерфейсе enp0s8
interface {{ name_int1_1 }}
!Указываем имя интерфейса
description {{ description1_1 }}
!Указываем ip-aдрес и маску (эту информацию мы получили в прошлом шаге)
ip address {{ ip_addr1_1 }}
!Указываем параметр игнорирования MTU
ip ospf mtu-ignore
!Если потребуется, можно указать «стоимость» интерфейса
!ip ospf cost 1000
!Указываем параметры hello-интервала для OSPF пакетов
ip ospf hello-interval 10
!Указываем параметры dead-интервала для OSPF пакетов
!Должно быть кратно предыдущему значению
ip ospf dead-interval 30
!
interface {{name_int1_2}}
description {{description1_2}}
ip address {{ip_addr1_2}}
ip ospf mtu-ignore
!ip ospf cost 45
ip ospf hello-interval 10
ip ospf dead-interval 30
interface {{name_int1_3}}
description {{description1_3}}
ip address {{ip_addr1_3}}
ip ospf mtu-ignore
!ip ospf cost 45
ip ospf hello-interval 10
ip ospf dead-interval 30
!
!Начало настройки OSPF
router ospf
!Указываем router-id
router-id {{id_router1}}
!Указываем сети, которые хотим анонсировать соседним роутерам
network {{network1_1}}
network {{network1_2}}
network {{network1_3}}
!Указываем адреса соседних роутеров
neighbor {{neighbor1_1}}
neighbor {{neighbor1_2}}
!Указываем адрес log-файла
log file /var/log/frr/frr.log
default-information originate always
```
И соответсвующие для конфига переменные:
```
---
# vars file for ospf

#Router1

hostname1: "router1"
name_int1_1: eth1
description1_1: "r1-r2"
ip_addr1_1: "10.0.10.1/30"
name_int1_2: "eth2"
description1_2: "r1-r3"
ip_addr1_2: "10.0.12.1/30"
name_int1_3: "eth3"
description1_3: "net_router1"
ip_addr1_3: "192.168.10.1/24"
id_router1: "1.1.1.1"
network1_1: "10.0.10.0/30 area 0"
network1_2: "10.0.12.0/30 area 0"
network1_3: "192.168.10.0/24 area 0"
neighbor1_1: "10.0.10.2"
neighbor1_2: "10.0.12.2"
#Router2
```
Для выполнения роли файлы и переменные лежат в соответсвущих каталогах main/files и main/vars

Тестирование:
В Vagrantfile указываем путь до плэйбука и выполняем команду vagrant provision. Подключаемся к машине router1. 
Проверям, что OSPF перезапустился без ошибок:
```
[root@router1 ~]# systemctl status frr
● frr.service - FRRouting (FRR)
   Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-08-10 23:10:37 UTC; 20min ago
  Process: 30832 ExecStop=/usr/lib/frr/frr stop (code=exited, status=0/SUCCESS)
  Process: 30915 ExecStart=/usr/lib/frr/frr start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/frr.service
           ├─30932 /usr/lib/frr/zebra -d -A 127.0.0.1
           ├─30941 /usr/lib/frr/ospfd -d -A 127.0.0.1
           └─30952 /usr/lib/frr/watchfrr -d -b_ -r/usr/lib/frr/frr_restart_%s -s/usr/lib/frr/frr_...

```
Пробуем сделать ping до ip-адреса 192.168.30.1:
```
[root@router1 ~]# ping 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=0.815 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=0.629 ms
^C
--- 192.168.30.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.629/0.722/0.815/0.093 ms
```
Запустим трассировку до адреса 192.168.30.1
```
[root@router1 ~]# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  1.363 ms  1.306 ms  1.266 ms
```
Проверим из интерфейса vtysh какие маршруты мы видим на данный момент:
```
[root@router1 ~]# vtysh

Hello, this is FRRouting (version 5.0.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router1# show ip route ospf
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR,
       > - selected route, * - FIB route

O   10.0.10.0/30 [110/100] is directly connected, eth1, 00:24:45
O>* 10.0.11.0/30 [110/200] via 10.0.10.2, eth1, 00:23:19
  *                        via 10.0.12.2, eth2, 00:23:19
O   10.0.12.0/30 [110/100] is directly connected, eth2, 00:24:45
O   192.168.10.0/24 [110/100] is directly connected, eth3, 00:24:45
O>* 192.168.20.0/24 [110/200] via 10.0.10.2, eth1, 00:23:55
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, eth2, 00:23:19
```

### Настройка ассиметричного роутинга
Настройки выполняем аналогично пункту выше ролью ansible. Во-первых для настройки ассиметричного роутинга нам необходимо выключить блокировку ассиметричной маршрутизации: sysctl net.ipv4.conf.all.rp_filter=0. Во-вторых на roter1 изменим «стоимость интерфейса» на 1000. 
Смотрим router1:
```
router1# show ip route ospf
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR,
       > - selected route, * - FIB route

O   10.0.10.0/30 [110/300] via 10.0.12.2, eth2, 00:05:35
O>* 10.0.11.0/30 [110/200] via 10.0.12.2, eth2, 00:05:35
O   10.0.12.0/30 [110/100] is directly connected, eth2, 00:05:46
O   192.168.10.0/24 [110/100] is directly connected, eth3, 00:06:35
O>* 192.168.20.0/24 [110/300] via 10.0.12.2, eth2, 00:05:35
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, eth2, 00:05:35
```
router2:
```
router2# show ip route ospf
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR,
       > - selected route, * - FIB route

O   10.0.10.0/30 [110/100] is directly connected, eth1, 00:07:17
O   10.0.11.0/30 [110/100] is directly connected, eth2, 00:07:09
O>* 10.0.12.0/30 [110/200] via 10.0.10.1, eth1, 00:06:58
  *                        via 10.0.11.1, eth2, 00:06:58
O>* 192.168.10.0/24 [110/200] via 10.0.10.1, eth1, 00:07:17
O   192.168.20.0/24 [110/100] is directly connected, eth3, 00:07:33
O>* 192.168.30.0/24 [110/200] via 10.0.11.1, eth2, 00:06:58
```
После внесения данных настроек, мы видим, что маршрут до сети 192.168.20.0/30 теперь пойдёт через router2, но обратный трафик от router2 пойдёт по другому пути.

Тестирование:

В Vagrantfile изменим параметр ansible.playbook на "ospf_asim_routering/provision.yml" и запровиженим машины/

На router1 запускаем пинг от 192.168.10.1 до 192.168.20.1: ping -I 192.168.10.1 192.168.20.1. На router2 запускаем tcpdump, который будет смотреть трафик только на порту eth2
```
[root@router2 ~]# tcpdump -i eth2
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), capture size 262144 bytes
20:23:47.316107 IP 192.168.10.1 > router2: ICMP echo request, id 7727, seq 73, length 64
20:23:48.316785 IP 192.168.10.1 > router2: ICMP echo request, id 7727, seq 74, length 64
20:23:49.317939 IP 192.168.10.1 > router2: ICMP echo request, id 7727, seq 75, length 64
20:23:50.319447 IP 192.168.10.1 > router2: ICMP echo request, id 7727, seq 76, length 64
```
Видим что данный порт только получает ICMP-трафик с адреса 192.168.10.1. На router2 запускаем tcpdump, который будет смотреть трафик только на порту eth1:
```
[root@router2 ~]# tcpdump -i eth2
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), capture size 262144 bytes
20:48:47.946409 IP 192.168.10.1 > router2: ICMP echo request, id 9581, seq 15, length 64
20:48:48.946922 IP 192.168.10.1 > router2: ICMP echo request, id 9581, seq 16, length 64
20:48:49.946408 IP 192.168.10.1 > router2: ICMP echo request, id 9581, seq 17, length 64
20:48:50.946945 IP 192.168.10.1 > router2: ICMP echo request, id 9581, seq 18, length 64
```
Видим что данный порт только отправляет ICMP-трафик на адрес 192.168.10.1, таким образом мы видим ассиметричный роутинг.


### Настройка симметричного роутинга
Так как у нас уже есть один «дорогой» интерфейс, нам потребуется добавить ещё один дорогой интерфейс, чтобы у нас перестала работать ассиметричная маршрутизация.Так как в прошлом задании мы заметили что router2 будет отправлять обратно трафик через порт eth1, мы также должны сделать его дорогим и далее проверить, что теперь используется симметричная маршрутизация, поменяв стоимость интерфейса eth1 на router2.

Тестирование:

В Vagrantfile изменим параметр ansible.playbook на "ospf_sim_routering/provision.yml" и запровиженим машины/

На router1 запускаем пинг от 192.168.10.1 до 192.168.20.1: ping -I 192.168.10.1 192.168.20.1, а на router2 запускаем tcpdump, который будет смотреть трафик только на порту eth2:
```
[root@router2 ~]# tcpdump -i eth2
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), capture size 262144 bytes
20:42:28.209874 IP 192.168.10.1 > router2: ICMP echo request, id 8656, seq 25, length 64
20:42:28.209908 IP router2 > 192.168.10.1: ICMP echo reply, id 8656, seq 25, length 64
20:42:29.212259 IP 192.168.10.1 > router2: ICMP echo request, id 8656, seq 26, length 64
20:42:29.212278 IP router2 > 192.168.10.1: ICMP echo reply, id 8656, seq 26, length 64
20:42:30.214593 IP 192.168.10.1 > router2: ICMP echo request, id 8656, seq 27, length 64
20:42:30.214623 IP router2 > 192.168.10.1: ICMP echo reply, id 8656, seq 27, length 64
20:42:31.215739 IP 192.168.10.1 > router2: ICMP echo request, id 8656, seq 28, length 64
20:42:31.215762 IP router2 > 192.168.10.1: ICMP echo reply, id 8656, seq 28, length 64
```
Теперь мы видим, что трафик между роутерами ходит симметрично.
