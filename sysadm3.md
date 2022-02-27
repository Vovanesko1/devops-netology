Домашнее задание к занятию "3.8. Компьютерные сети, лекция 3"
1. - Подключитесь к публичному маршрутизатору в интернет. Найдите маршрут к вашему публичному IP
```
telnet route-views.routeviews.org
Username: rviews
show ip route x.x.x.x/32
show bgp x.x.x.x/32
```
```
>show ip route 94.242.44.182
Routing entry for 94.242.32.0/20
  Known via "bgp 6447", distance 20, metric 0
  Tag 6939, type external
  Last update from 64.71.137.241 2d09h ago
  Routing Descriptor Blocks:
  * 64.71.137.241, from 64.71.137.241, 2d09h ago
      Route metric is 0, traffic share count is 1
      AS Hops 2
      Route tag 6939
      MPLS label: none
```
Таблица маршрутов тут очень большая. Мой адрес живёт в сети 94.242.32.0/20 и первым адресом для связи между маршрутизатором route-views.routeviews.org и моим стоит адрес шлюза 64.71.137.241
```
>show bgp summary
BGP router identifier 128.223.51.103, local AS number 6447
BGP table version is 314953311, main routing table version 314953302
Path RPKI states: 6763733 valid, 12811026 not found, 13875 invalid
927488 network entries using 230017024 bytes of memory
19588634 path entries using 2350636080 bytes of memory
3234835/154349 BGP path/bestpath attribute entries using 802239080 bytes of memory
2975124 BGP AS-PATH entries using 152152948 bytes of memory
189182 BGP community entries using 26074592 bytes of memory
2575 BGP extended community entries using 205416 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3561325140 total bytes of memory
BGP activity 33747322/32673727 prefixes, 1201460359/1177831724 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
4.68.4.46       4         3356 1852460   14760 314953079    0    0 4d16h      867481
5.101.110.2     4        14061       0       0        1    0    0 never    Idle
12.0.1.63       4         7018 11375942   37276 314953079    0    0 3w2d       868499
37.139.139.17   4        57866 1118423    5373 314953079    0    0 1d17h      869929
64.71.137.241   4         6939 136842726  579150 314953079    0    0 1y0w       898299
66.59.190.221   4         6539       0       0        1    0    0 never    Idle
66.110.0.86     4         6453       0       0        1    0    0 never    Idle
66.185.128.48   4         1668       0       0        1    0    0 never    Active
69.31.111.244   4         4436       0       0        1    0    0 never    Idle
80.241.176.31   4        20771       0       0        1    0    0 never    Active
89.149.178.10   4         3257 32462998   39075 314953079    0    0 17w3d      868724
91.218.184.60   4        49788 51144191  126573 314953079    0    0 15w1d      872806
94.142.247.3    4         8283 1358464    2043 314953079    0    0 1d17h      873388
95.85.0.2       4        14061       0       0        1    0    0 never    Idle
103.197.104.1   4       134708       0       0        1    0    0 never    Idle
103.247.3.45    4        58511       0       0        1    0    0 never    Idle
103.255.249.22  4        58443       0       0        1    0    0 never    Active
114.31.199.1    4         4826       0       0        1    0    0 never    Active
123.108.254.218 4         9902       0       0        1    0    0 never    Active
129.250.0.11    4         2914       0       0        1    0    0 never    Active
129.250.1.66    4         2914       0       0        1    0    0 never    Idle
132.198.255.253 4         1351  384270    2710 314953079    0    0 1d17h      909730
134.222.87.1    4          286       0       0        1    0    0 31w2d    Idle
137.39.3.55     4          701  804239    7441 314953079    0    0 4d16h      868955
140.192.8.16    4        20130  675326    5378 314953079    0    0 1d17h      901375
144.228.241.130 4         1239   63928    8772 314953079    0    0 1w2d        60839
154.11.12.212   4          852  590598    5379 314953079    0    0 1d17h      870073
158.106.197.135 4        46450       0       0        1    0    0 never    Active
162.243.188.2   4        14061       0       0        1    0    0 never    Active
162.250.137.254 4         4901 46877362  382076 314953079    0    0 1y5w       872945
162.251.163.2   4        53767  995884   14746 314953079    0    0 4d16h      866720
173.205.57.234  4        53364       0       0        1    0    0 never    Idle
178.238.225.14  4        58901       0       0        1    0    0 never    Idle
192.203.116.253 4        22388       0       0        1    0    0 never    Idle
192.241.164.4   4        14061       0       0        1    0    0 never    Active
193.0.0.56      4         3333 1184834    5375 314953079    0    0 1d17h      884523
193.251.245.74  4         5511       0       0        1    0    0 never    Active
194.85.40.15    4         3267       0       1        1    0    0 00:09:53 OpenSent
195.66.232.239  4         5459       0       0        1    0    0 never    Active
195.208.112.161 4         3277       0       0        1    0    0 36w4d    Active
198.32.252.33   4        20080   19446    5377 314953079    0    0 1d17h       19440
198.58.198.254  4         1403       0       0        1    0    0 48w3d    Idle
198.58.198.255  4         1403       0       0        1    0    0 48w3d    Active
202.93.8.242    4        24441    3608    2718 314953079    0    0 1d17h        4577
202.232.0.2     4         2497 13434139   69673 314953079    0    0 10w3d      882798
203.62.252.83   4         1221 13388311  107959 314953079    0    0 16w1d      878230
203.181.248.168 4         7660 111620462  127836 314953079    0    0 1y5w       715626
206.24.210.80   4         3561 4397092   23068 314953079    0    0 3w3d       867631
207.46.32.34    4         8075       0       0        1    0    0 never    Active
207.172.6.1     4         6079       0       0        1    0    0 never    Idle
207.172.6.20    4         6079       0       0        1    0    0 never    Idle
208.51.134.254  4         3549 1196791   14730 314953079    0    0 4d16h      868914
208.74.64.40    4        19214 26262747  593978 314953079    0    0 1y1w       356993
209.124.176.223 4          101 25495835  225452 314953079    0    0 10w1d      872409
212.66.96.126   4        20912 2145403   22642 314953079    0    0 2w0d       872621
217.192.89.50   4         3303 1311572   19496 314953079    0    0 2w6d       895158
```
```
>show bgp 94.242.44.182
BGP routing table entry for 94.242.32.0/20, version 311647358
Paths: (23 available, best #7, table default)
  Not advertised to any peer
  Refresh Epoch 1
  8283 20764 43317
    94.142.247.3 from 94.142.247.3 (94.142.247.3)
      Origin IGP, metric 0, localpref 100, valid, external
      Community: 8283:1 8283:101 20764:1410 20764:3002 20764:3010 20764:3021
      unknown transitive attribute: flag 0xE0 type 0x20 length 0x18
        value 0000 205B 0000 0000 0000 0001 0000 205B
              0000 0005 0000 0001
      path 7FE1598A6F50 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  1351 6939 43317
    132.198.255.253 from 132.198.255.253 (132.198.255.253)
      Origin IGP, localpref 100, valid, external
      path 7FE0452BAE40 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  852 3356 20764 43317
    154.11.12.212 from 154.11.12.212 (96.1.209.43)
      Origin IGP, metric 0, localpref 100, valid, external
      path 7FE184ABB368 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  57866 9002 43317
    37.139.139.17 from 37.139.139.17 (37.139.139.17)
      Origin IGP, metric 0, localpref 100, valid, external
      Community: 9002:0 9002:64667
      path 7FE0F0744AE8 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  20130 6939 43317
    140.192.8.16 from 140.192.8.16 (140.192.8.16)
      Origin IGP, localpref 100, valid, external
      path 7FE0C8F8B810 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3333 20764 43317
    193.0.0.56 from 193.0.0.56 (193.0.0.56)
      Origin IGP, localpref 100, valid, external
      Community: 20764:1410 20764:3002 20764:3010 20764:3021
      path 7FE11AA727F8 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  6939 43317
    64.71.137.241 from 64.71.137.241 (216.218.252.164)
      Origin IGP, localpref 100, valid, external, best
      path 7FE0D110CD98 RPKI State not found
      rx pathid: 0, tx pathid: 0x0
  Refresh Epoch 1
  701 1273 31500 43317
    137.39.3.55 from 137.39.3.55 (137.39.3.55)
      Origin IGP, localpref 100, valid, external
      path 7FE165306AE8 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3549 3356 20764 43317
    208.51.134.254 from 208.51.134.254 (67.16.168.191)
      Origin IGP, metric 0, localpref 100, valid, external
      Community: 3356:2 3356:22 3356:100 3356:123 3356:507 3356:903 3356:2111 3549:2581 3549:30840 20764:1410 20764:3002 20764:3010 20764:3021
      path 7FE0B72A3200 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  53767 14315 6453 6453 3356 20764 43317
    162.251.163.2 from 162.251.163.2 (162.251.162.3)
      Origin IGP, localpref 100, valid, external
      Community: 14315:5000 53767:5000
      path 7FE0CD5DD940 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3356 20764 43317
    4.68.4.46 from 4.68.4.46 (4.69.184.201)
      Origin IGP, metric 0, localpref 100, valid, external
      Community: 3356:2 3356:22 3356:100 3356:123 3356:507 3356:903 3356:2111 20764:1410 20764:3002 20764:3010 20764:3021 65000:714 65000:6185 65000:16509 65000:16625 65000:20940
      path 7FE0EB0B87D0 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  4901 6079 3356 20764 43317
    162.250.137.254 from 162.250.137.254 (162.250.137.254)
      Origin IGP, localpref 100, valid, external
      Community: 65000:10100 65000:10300 65000:10400
      path 7FE1479D0480 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  20912 3257 3356 20764 43317
    212.66.96.126 from 212.66.96.126 (212.66.96.126)
      Origin IGP, localpref 100, valid, external
      Community: 3257:8070 3257:30515 3257:50001 3257:53900 3257:53902 20912:65004
      path 7FE17DF87608 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3303 31500 43317
    217.192.89.50 from 217.192.89.50 (138.187.128.158)
      Origin IGP, localpref 100, valid, external
      Community: 3303:1004 3303:1006 3303:1030 3303:1031 3303:3081 31210:31500 65005:10643
      path 7FE10E049AD0 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  7018 3356 20764 43317
    12.0.1.63 from 12.0.1.63 (12.0.1.63)
      Origin IGP, localpref 100, valid, external
      Community: 7018:5000 7018:37232
      path 7FE15416A0C8 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3561 3910 3356 20764 43317
    206.24.210.80 from 206.24.210.80 (206.24.210.80)
      Origin IGP, localpref 100, valid, external
      path 7FE144DB4380 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  101 174 20764 43317
    209.124.176.223 from 209.124.176.223 (209.124.176.223)
      Origin IGP, localpref 100, valid, external
      Community: 101:20100 101:20110 101:22100 174:21101 174:22014
      Extended Community: RT:101:22100
      path 7FE15C1F5148 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 2
  2497 1273 31500 43317
    202.232.0.2 from 202.232.0.2 (58.138.96.254)
      Origin IGP, localpref 100, valid, external
      path 7FE146CA6130 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  7660 2516 1273 31500 43317
    203.181.248.168 from 203.181.248.168 (203.181.248.168)
      Origin IGP, localpref 100, valid, external
      Community: 2516:1030 7660:9003
      path 7FE12E970408 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  49788 12552 20764 43317
    91.218.184.60 from 91.218.184.60 (91.218.184.60)
      Origin IGP, localpref 100, valid, external
      Community: 12552:12000 12552:12100 12552:12101 12552:22000
      Extended Community: 0x43:100:1
      path 7FE0D2044A58 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  1221 4637 31500 43317
    203.62.252.83 from 203.62.252.83 (203.62.252.83)
      Origin IGP, localpref 100, valid, external
      path 7FE10F8A7FC0 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  3257 3356 20764 43317
    89.149.178.10 from 89.149.178.10 (213.200.83.26)
      Origin IGP, metric 10, localpref 100, valid, external
      Community: 3257:8794 3257:30043 3257:50001 3257:54900 3257:54901
      path 7FE111353610 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  19214 174 20764 43317
    208.74.64.40 from 208.74.64.40 (208.74.64.40)
      Origin IGP, localpref 100, valid, external
      Community: 174:21101 174:22014
      path 7FE053BBEE30 RPKI State not found
      rx pathid: 0, tx pathid: 0
```
Paths: (23 available, best #7, table default) - 23 возможных маршрута до моего адреса. Выбран маршрут №7. 64.71.137.241 from 64.71.137.241 (216.218.252.164). Что сходится с выводом команды 'show ip route 94.242.44.182'

2. - Создайте dummy0 интерфейс в Ubuntu. Добавьте несколько статических маршрутов. Проверьте таблицу маршрутизации.

Любой интерфейс маршрутизатора потенциально может уйти в down, но сессии BGP и SSH должны работать. Поскольку loopback или dummy самопроизвольно не уйдет в down никогда, можно присвоить ему отдельный адрес и анонсировать его остальной сети через протокол OSPF (Open Shortest Path First), который использует multicast и не зависит от конкретных адресов интерфейсов. В этом случае адрес на loopback и dummy останется доступным, если у маршрутизатора есть хотя бы один живой канал в остальную сеть.
Добавление интерфейса до перезагрузки
```
sudo ip link add dummy0 type dummy
```
Добавляем постоянный интерфейс
```
sudo vi /etc/netplan/02-dummy.yaml

network:
  version: 2
  renderer: networkd
  bridges:
    dummy0:
      dhcp4: no
      dhcp6: no
      accept-ra: no
      interfaces: [ ]
      addresses:
              -  169.254.1.200/32

sudo netplan apply

ip  address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b1:28:5d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 85978sec preferred_lft 85978sec
    inet6 fe80::a00:27ff:feb1:285d/64 scope link
       valid_lft forever preferred_lft forever
3: dummy0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether f2:72:ef:a2:b1:c6 brd ff:ff:ff:ff:ff:ff
    inet 169.254.1.200/32 scope global dummy0
       valid_lft forever preferred_lft forever
    inet6 fe80::f072:efff:fea2:b1c6/64 scope link
       valid_lft forever preferred_lft forever
```
Добавим статический маршрут.
Статический маршрут задается для конкретного интерфейса, также в конфигурационном файле netplan:
```
sudo vi /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      nameservers:
        addresses: [8.8.8.8, 77.88.8.8]
      routes:
        - to: 192.168.0.0/24
          via: 10.0.2.1
          on-link: true
          metric: 90
```
Формат файла необычайно чувствителен к проблелам, уровням и минусам. При том, что сайт 'http://www.yamllint.com/' проверку синтаксиса проходит успешно, конфиг всё равно не работал. Кто придумал использовать текстовый редактор для его редактирования? Наверно только я.
Просмотр маршрутов:
```
 ip route
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15
10.0.2.2 dev eth0 proto dhcp scope link src 10.0.2.15 metric 100
192.168.0.0/24 via 10.0.2.1 dev eth0 proto static metric 90 onlink
```

3. - Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.

```
sudo ss -t
```
|State|Recv-Q|Send-Q|Local Address:Port|Peer Address:Port|Process|
| ------ | ------ | ------ | ------ | ------ | ------ |
|ESTAB|0|0|10.0.2.15:ssh|10.0.2.2:56501
```
 sudo ss -tlpn
```
|State|Recv-Q|Send-Q|Local Address:Port|Peer Address:Port|Process|
| ------ | ------ | ------ | ------ | ------ | ------ |
|LISTEN|0|4096|127.0.0.53%lo:53|0.0.0.0:*|users:(("systemd-resolve",pid=12966,fd=13))
|LISTEN |                 0             |          128          |                                 0.0.0.0:22          |                                0.0.0.0:*     |                users:(("sshd",pid=12619,fd=3))
|LISTEN              |    0           |            128           |                                    [::]:22  |                                           [::]:*             |        users:(("sshd",pid=12619,fd=4))
Установленное соединение одно - по ssh протоколу. На соединение открыты 2 порта: 53, 22.
Ubuntu по умолчанию прослушивает порт 53 процессом systemd. Это помешает поднять собственный DNS-сервер. Так же прослущивается 22 порт для подключения к sshd.
Ubuntu 12.04 теперь запускает dnsmasq - компактный DNS-сервер. Причина этого, по словам разработчиков , заключается в том, чтобы справиться с ситуациями, когда ваша машина имеет несколько сетевых интерфейсов с разными DNS-серверами. Это может создавать проблемы для людей, использующих виртуальные частные сети (VPN) и в других так называемых «многосетевых» контекстах. Таким образом, начиная с 12.04, на каждой машине с Ubuntu по умолчанию будет работать локальный DNS-сервер. Однако он должен прослушивать только порт 53 на интерфейсе localhost (127.0.0.1). И 53 порт не виден при сканировании снаружи.
```
sudo lsof -i | grep TCP
```
|COMMAND  |   PID       |     USER  | FD |  TYPE|DEVICE | SIZE/OFF | NODE NAME
| ------ | ------ | ------ | ------ | ------ | ------ |------ |------ |
|sshd |      1155       |     root  |  4u | IPv4 | 27562  |    0t0 | TCP vagrant:ssh->_gateway:56501 (ESTABLISHED)
|sshd|       1204      |   vagrant   | 4u | IPv4 | 27562  |    0t0 | TCP vagrant:ssh->_gateway:56501 (ESTABLISHED)
|sshd  |    12619    |        root   | 3u | IPv4  |43271   |  0t0 | TCP *:ssh (LISTEN)
|sshd  |    12619     |       root  |  4u|  IPv6 | 43273  |   0t0 | TCP *:ssh (LISTEN)
|systemd-r| 12966 |systemd-resolve  | 13u | IPv4|  43998    |  0t0 | TCP localhost:domain (LISTEN)


4. - Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?

```
 sudo ss -ulpn
 ```
 |State|Recv-Q|Send-Q|Local Address:Port|Peer Address:Port|Process|
 | ------ | ------ | ------ | ------ | ------ | ------ |
|UNCONN         |        0              |        0                |                         127.0.0.53%lo:53                             |             0.0.0.0:*       |              users:(("systemd-resolve",pid=12966,fd=12))
|UNCONN     |            0           |            0         |                               10.0.2.15%eth0:68                                   |       0.0.0.0:*          |           users:(("systemd-network",pid=23908,fd=20))
UNCONN - Unconnected connection
```
sudo lsof -i | grep UDP
```
|COMMAND  |   PID       |     USER  | FD |  TYPE|DEVICE | SIZE/OFF | NODE NAME
| ------ | ------ | ------ | ------ | ------ | ------ |------ |------ |
|systemd-r | 12966| systemd-resolve  | 12u| IPv4|  43997  |    0t0 | UDP localhost:domain
|systemd-n| 23908 |systemd-network |  20u | IPv4|  53438  |    0t0 |UDP vagrant:bootpc

5. - Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали.

в формате drawio
https://drive.google.com/file/d/1-URISx-M-EuGkAUnNZXt3MCoLdZzRumL/view?usp=sharing
в формате jpg
https://drive.google.com/file/d/1NPqKb0neFz_xLta_dd31l2uzxJAq0APN/view?usp=sharing
