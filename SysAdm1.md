 sudo apt-get install dnsutils, mtr, traceroute

1. - Работа c HTTP через телнет.
Подключитесь утилитой телнет к сайту stackoverflow.com telnet stackoverflow.com 80
отправьте HTTP запрос
GET /questions HTTP/1.0
HOST: stackoverflow.com
[press enter]
[press enter]
В ответе укажите полученный HTTP код, что он означает?

```
HTTP/1.1 400 Bad Request
Connection: close
Content-Length: 0
Connection closed by foreign host.
```
400 Bad Request говорит о том, что возникла проблема на стороне пользователя. Зачастую сервер отправляет ее, когда появившаяся неисправность не подходит больше ни под одну категорию ошибок. 
Код 400 напрямую связан с клиентом и намекает на то, что отправленный запрос со стороны пользователя приводит к сбою еще до того, как его обработает сервер (вернее, так считает сам сервер).
В нашем случае ошибка из-за того, что по URL ждут https запрос.

2. - Повторите задание 1 в браузере, используя консоль разработчика F12.
- откройте вкладку Network
- отправьте запрос http://stackoverflow.com
- найдите первый ответ HTTP сервера, откройте вкладку Headers
- укажите в ответе полученный HTTP код.
```
Request URL: http://stackoverflow.com/
Request Method: GET
Status Code: 307 Internal Redirect
Referrer Policy: strict-origin-when-cross-origin
Location: https://stackoverflow.com/
Non-Authoritative-Reason: HSTS
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36
```
- проверьте время загрузки страницы, какой запрос обрабатывался дольше всего?
- приложите скриншот консоли браузера в ответ.
[скрин 1](https://drive.google.com/file/d/1Eh7gfP3CdtEqUmw2ZhXhPtVoXfvwUeIb/view?usp=sharing)
[скрин 2](https://drive.google.com/file/d/1I6MGtks98F__Nnm7R2ajNYCuJnmD0DYd/view?usp=sharing)

3. - Какой IP адрес у вас в интернете?
```
:~$ wget -qO- ident.me
94.242.44.182vagrant@vagrant:~$
```
4. - Какому провайдеру принадлежит ваш IP адрес? Какой автономной системе AS? Воспользуйтесь утилитой whois
```
 whois 94.242.44.182
% This is the RIPE Database query service.
% The objects are in RPSL format.
%
% The RIPE Database is subject to Terms and Conditions.
% See http://www.ripe.net/db/support/db-terms-conditions.pdf
% Note: this output has been filtered.
%       To receive output for a database update, use the "-B" flag.
% Information related to '94.242.32.0 - 94.242.47.254'
% Abuse contact for '94.242.32.0 - 94.242.47.254' is 'abuse@veesp.com'

inetnum:        94.242.32.0 - 94.242.47.254
netname:        RNet
descr:          RNet ISP network
country:        RU
admin-c:        FCOV1-RIPE
tech-c:         FCOV1-RIPE
status:         ASSIGNED PA
geoloc:         59.9100783 30.4076695
mnt-by:         FISHNET-MNT
created:        2019-07-05T08:03:25Z
last-modified:  2019-07-05T08:03:25Z
source:         RIPE

role:           Veesp / FNK - Contact Role
address:        Obukhovskoy oborony 45
address:        Saint Petersburg
address:        Russian Federation
phone:          +7 812 3136313
abuse-mailbox:  abuse@veesp.com
remarks:        *************************************************
remarks:        * For spam/abuse/security issues please contact *
remarks:        * abuse@veesp.com *
remarks:        * The contents of your abuse email will be *
remarks:        * forwarded directly on to our client for *
remarks:        * handling. *
remarks:        *************************************************
remarks:
remarks:        *************************************************
remarks:        * Any questions on Peering/Routing please send to *
remarks:        * noc@veesp.com *
remarks:        *************************************************
org:            ORG-OC18-RIPE
admin-c:        AI2525-RIPE
tech-c:         IR2525-RIPE
tech-c:         AI2525-RIPE
nic-hdl:        FCOV1-RIPE
mnt-by:         FISHNET-MNT
created:        2017-06-30T12:49:06Z
last-modified:  2022-02-07T08:55:22Z
source:         RIPE # Filtered

% Information related to '94.242.32.0/20AS43317'

route:          94.242.32.0/20
descr:          RNET ISP Network
origin:         AS43317
mnt-by:         FISHNET-MNT
created:        2008-12-16T11:04:46Z
last-modified:  2019-07-05T07:59:57Z
source:         RIPE
remarks:        region-id = Russia, Leningrad district, Saint-Petersburg
```
```
~$ whois -h whois.radb.net 94.242.44.182
route:          94.242.32.0/20
descr:          RNET ISP Network
origin:         AS43317
mnt-by:         FISHNET-MNT
created:        2008-12-16T11:04:46Z
last-modified:  2019-07-05T07:59:57Z
source:         RIPE
remarks:        region-id = Russia, Leningrad district, Saint-Petersburg
remarks:        ****************************
remarks:        * THIS OBJECT IS MODIFIED
remarks:        * Please note that all data that is generally regarded as personal
remarks:        * data has been removed from this object.
remarks:        * To view the original object, please query the RIPE Database at:
remarks:        * http://www.ripe.net/whois
remarks:        ****************************
```

Выделен провайдеру 'RNet'. Автономная система AS43317

5. - Через какие сети проходит пакет, отправленный с вашего компьютера на адрес 8.8.8.8? Через какие AS? Воспользуйтесь утилитой traceroute
```
traceroute -IAn 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  10.0.2.2 [*]  0.873 ms  0.714 ms  0.627 ms
 2  192.168.1.1 [*]  1.512 ms  1.404 ms  1.339 ms
 3  94.242.42.1 [AS43317]  1.683 ms  4.258 ms  4.204 ms
 4  94.242.0.28 [AS43317]  1.913 ms  4.101 ms  4.051 ms
 5  94.242.0.97 [AS43317]  3.999 ms  3.950 ms  3.900 ms
 6  94.242.0.50 [AS43317]  3.846 ms  3.018 ms  2.919 ms
 7  178.18.227.12 [AS50952]  1.803 ms  4.397 ms  4.272 ms
 8  74.125.244.180 [AS15169]  5.225 ms  8.160 ms  8.108 ms
 9  142.251.61.219 [AS15169]  8.054 ms  8.003 ms  7.953 ms
10  142.250.208.25 [AS15169]  7.899 ms  7.849 ms  7.799 ms
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  8.8.8.8 [AS15169]  5.178 ms *  4.893 ms
```

6. - Повторите задание 5 в утилите mtr. На каком участке наибольшая задержка - delay?

[mtr screen](https://drive.google.com/file/d/1jDy4i0XfGmfzaYRE2iPHSLoyI4Y4Knry/view?usp=sharing)
Наибольшая задержка на шаге 10.

7. - Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? воспользуйтесь утилитой dig
```
dig +trace dns.google

; <<>> DiG 9.16.1-Ubuntu <<>> +trace dns.google
;; global options: +cmd
.                       81794   IN      NS      l.root-servers.net.
.                       81794   IN      NS      g.root-servers.net.
.                       81794   IN      NS      h.root-servers.net.
.                       81794   IN      NS      k.root-servers.net.
.                       81794   IN      NS      c.root-servers.net.
.                       81794   IN      NS      a.root-servers.net.
.                       81794   IN      NS      m.root-servers.net.
.                       81794   IN      NS      j.root-servers.net.
.                       81794   IN      NS      f.root-servers.net.
.                       81794   IN      NS      b.root-servers.net.
.                       81794   IN      NS      e.root-servers.net.
.                       81794   IN      NS      d.root-servers.net.
.                       81794   IN      NS      i.root-servers.net.
;; Received 262 bytes from 127.0.0.53#53(127.0.0.53) in 132 ms

google.                 172800  IN      NS      ns-tld1.charlestonroadregistry.com.
google.                 172800  IN      NS      ns-tld2.charlestonroadregistry.com.
google.                 172800  IN      NS      ns-tld3.charlestonroadregistry.com.
google.                 172800  IN      NS      ns-tld4.charlestonroadregistry.com.
google.                 172800  IN      NS      ns-tld5.charlestonroadregistry.com.
google.                 86400   IN      DS      6125 8 2 80F8B78D23107153578BAD3800E9543500474E5C30C29698B40A3DB2 3ED9DA9F
google.                 86400   IN      RRSIG   DS 8 1 86400 20220223050000 20220210040000 9799 . Cd7UjNTHEFGM53D83oC4dfWJXip4+n09gCfoqBwVidA5T6NppG42yvb7 phktV7rhbxmKyub7ul2tKOtm/bSGJEuRxSQdT96cKpY1MleZn2CDihc8 0g0gLT+YWBAsuJik6NeswJwVKHawICZsLwJilmeKNC/QNZ860G5nVVdB PWw951ulLR5RymTJX0GTiD0SQoMImWBTxIp/0ynnV7V2cNRjg+mlwdr4 t8DakxiLoF0M6kZT6A1FnfrLcOhx/tgww/dxqK8XLzxIKRDWjFf0kUl8 mlndVX+tkcs35VF6mNX73dfU8Nd0XFIm/H8P2djPQ3MO5E3FHOqQ69xW o3SLCg==
;; Received 730 bytes from 199.7.91.13#53(d.root-servers.net) in 44 ms

dns.google.             10800   IN      NS      ns1.zdns.google.
dns.google.             10800   IN      NS      ns2.zdns.google.
dns.google.             10800   IN      NS      ns3.zdns.google.
dns.google.             10800   IN      NS      ns4.zdns.google.
dns.google.             3600    IN      DS      56044 8 2 1B0A7E90AA6B1AC65AA5B573EFC44ABF6CB2559444251B997103D2E4 0C351B08
dns.google.             3600    IN      RRSIG   DS 8 2 3600 20220302182859 20220208182859 39106 google. P4AxpPShH0+rAol37rX0myfKtKajydqbCbMFIg4L9SnboAx5R7ROnwrw 5USR7gFEHRc9aEm+O2/TmcpivMuGRsS6JvCMbqqlsAtrcMGmK8nabCXu yc82ipF67HTJwKmSO7t4YU/4ZBvdYDR9uolaN6UJJ2Gwtbfee2Qg4RoI 5tw=
;; Received 506 bytes from 216.239.36.105#53(ns-tld3.charlestonroadregistry.com) in 8 ms

dns.google.             900     IN      A       8.8.4.4
dns.google.             900     IN      A       8.8.8.8
dns.google.             900     IN      RRSIG   A 8 2 900 20220301194556 20220207194556 25800 dns.google. voKTmdCkO3DRupZHZ+LC6VTDV4UQkBdnidvQSJE/E5zZYWUI4YWTiOeD wuKgiHLdJTbGEZ9aF4s/JCzwKG7Vjy0aw4RM97XNNRNgy/nDMbVThIVx IK6L85Jfn7gtrqlVzOc8gXC1CwEt0ufsFU2c2MSjDcjSQCilICKCu09v vuw=
;; Received 241 bytes from 216.239.36.114#53(ns3.zdns.google) in 8 ms
```
Для работы с сервером dns.google по имени нужны минимум 3 сервера:
корневой d.root-servers.net
второго уровня ns-tld3.charlestonroadregistry.com
третьего уровня ns3.zdns.google
У этого имени сервера две записи типа А:
dns.google.             900     IN      A       8.8.4.4
dns.google.             900     IN      A       8.8.8.8

8. Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? воспользуйтесь утилитой dig
```
dig -x 8.8.8.8

; <<>> DiG 9.16.1-Ubuntu <<>> -x 8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32975
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;8.8.8.8.in-addr.arpa.          IN      PTR

;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.   3635    IN      PTR     dns.google.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Thu Feb 10 17:01:34 UTC 2022
;; MSG SIZE  rcvd: 73

 dig -x 8.8.4.4

; <<>> DiG 9.16.1-Ubuntu <<>> -x 8.8.4.4
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61492
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;4.4.8.8.in-addr.arpa.          IN      PTR

;; ANSWER SECTION:
4.4.8.8.in-addr.arpa.   68361   IN      PTR     dns.google.

;; Query time: 7 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Thu Feb 10 17:02:55 UTC 2022
;; MSG SIZE  rcvd: 73
``` 
8.8.8.8.in-addr.arpa.     dns.google.
4.4.8.8.in-addr.arpa.     dns.google.
