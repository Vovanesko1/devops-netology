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

------------------------------------------------------------

# Добрый день!

Задание 1
Вы что-то ввели не так. Покажите скрин с командой и запросом.

По поводу сетей:

это первое занятие по данной теме;
сети будут на протяжении всего курса;
у нас есть курс сетевого инженера, можете записаться на него.
С уважением,
Алексей

Добрый день, Алексей. Да, в telnet запрос добавил лишнюю строку. Запрос по этому и выполнился с ошибкой. Ниже листинг и разбор успешного запроса.

```
~$ telnet stackoverflow.com 80
Trying 151.101.193.69...
Connected to stackoverflow.com.
Escape character is '^]'.
GET /questions HTTP/1.0
HOST: stackoverflow.com

HTTP/1.1 301 Moved Permanently
cache-control: no-cache, no-store, must-revalidate
location: https://stackoverflow.com/questions
x-request-guid: fe7ab9c1-669a-4e01-b7ab-dc6455813401
feature-policy: microphone 'none'; speaker 'none'
content-security-policy: upgrade-insecure-requests; frame-ancestors 'self' https://stackexchange.com
Accept-Ranges: bytes
Date: Sat, 12 Feb 2022 09:07:44 GMT
Via: 1.1 varnish
Connection: close
X-Served-By: cache-fra19134-FRA
X-Cache: MISS
X-Cache-Hits: 0
X-Timer: S1644656864.330457,VS0,VE85
Vary: Fastly-SSL
X-DNS-Prefetch-Control: off
Set-Cookie: prov=2fa658f9-e46a-067c-7057-f0a668198489; domain=.stackoverflow.com; expires=Fri, 01-Jan-2055 00:00:00 GMT; path=/; HttpOnly

Connection closed by foreign host.
```
## Разбор вывода
```
HTTP/1.1 301 Moved Permanently     
```
Код перенаправления
```
cache-control: no-cache, no-store, must-revalidate
```
Директивы для cache-control: 
**no-store**	запрещает кэшировать отклик при любых условиях.
**must-revalidate**	обязывает кэш жестко следовать предоставленной информации о свежести хранимого отклика. Обычно при определенных обстоятельствах (например, при отсутствии интернет-соединения) HTTP позволяет использовать просроченный кэш. Указывая этот заголовок, вы сообщаете о строгой необходимости следовать установленным правилам.
**no-cache**	обязывает кэш направлять все запросы на сервер-источник для подтверждения свежести хранимого в кэше отклика. Применяется, если необходимо сохранить семантическую прозрачность, не теряя преимуществ условного кэширования.
```
location: https://stackoverflow.com/questions
```
адрес страницы, куда перенаправили.
```
x-request-guid: fe7ab9c1-669a-4e01-b7ab-dc6455813401
```
Уникальный идентификатор запроса, сгенерированный для каждого запроса, поступающего на сервер
```
feature-policy: microphone 'none'; speaker 'none'
```
Заголовок, позволяющий владельцам сайтов включать или отключать специфические для определенной платформы функции. Используя этот заголовок, владелец сайта может ограничить некоторые функции браузера на своем ресурсе. В данном случае владелец отключил микрофон и аудио колонки на странице.
```
content-security-policy: upgrade-insecure-requests; frame-ancestors 'self' https://stackexchange.com
```
CSP - это механизм обеспечения безопасности, с помощью которого можно защищаться от атак с внедрением контента, например, межсайтового скриптинга (XSS, cross site scripting). CSP описывает безопасные источники загрузки ресурсов, устанавливает правила использования встроенных стилей, скриптов, а также динамической оценки JavaScript — например, с помощью eval. Загрузка с ресурсов, не входящих в «белый список», блокируется.
По нашим директивам:
**upgrade-insecure-requests** предписывает обрабатывать все небезопасные URL-адреса сайта (те, которые обслуживаются через HTTP), как если бы они были заменены безопасными URL-адресами (которые обслуживаются через HTTPS).
Директива **frame-ancestors** указывает допустимых родителей, которые могут встраивать страницу, используя <frame>, <iframe>, <object>, <embed> или <applet>. 'self' cсылается на источник, из которого обслуживается защищенный документ, включая ту же схему URL и номер порта.
```
Accept-Ranges: bytes
```
Это указание клиенту о поддержке "запросов по кускам". Его значение указывает единицу измерения, которая может быть использована для определения диапазона чтения: Биты.
```
Date: Sat, 12 Feb 2022 09:07:44 GMT
```
содержит дату и время создания сообщения.
```
Via: 1.1 varnish
```
Это производитель ПО добавил рекламы
```
Connection: close
```
Заголовок Connection определяет, остаётся ли сетевое соединение активным после завершения текущей транзакции (запроса). Если в запросе отправлено значение keep-alive, то соединение остаётся и не завершается, позволяя выполнять последующие запросы на тот же сервер. В нашем случае следующие запросы не пыполняются и соединение закрывается.
```
X-Served-By: cache-fra19134-FRA
```
Этот нестандартный заголовок содержит идентификатор кэш-сервера, выступающего в качестве узла доставки. Значение является точным на момент его создания, но не гарантируется в другой момент времени.
```
X-Cache: MISS
```
X-префикс указывает, что заголовок не является частью спецификации, поэтому его значение может различаться в разных реализациях прокси. Предположительно: X-Cache: Обрабатывал ли этот прокси результат из кеша (HIT для да, MISS для нет).
```
X-Cache-Hits: 0
```
Значение близко к значению «x-cache», только значение «0» эквивалентно «MISS», а любое положительное значение эквивалентно «HIT» (значение равно равно количеству попыток до попадания).
```
X-Timer: S1644656864.330457,VS0,VE85
```
Этот заголовок предоставляет информацию о времени пути запроса из конца в конец
```
Vary: Fastly-SSL
```
Указывает, использовало ли подключение на стороне клиента TLS для подключения к Fastly
```
X-DNS-Prefetch-Control: off
```
управляет предварительной выборкой DNS — функцией, с помощью которой браузеры активно выполняют разрешение доменных имен для ссылок, по которым пользователь может перейти, а также для URL-адресов элементов, на которые ссылается документ, включая изображения, CSS, JavaScript и т. д.
```
Set-Cookie: prov=2fa658f9-e46a-067c-7057-f0a668198489; domain=.stackoverflow.com; expires=Fri, 01-Jan-2055 00:00:00 GMT; path=/; HttpOnly
```
HTTP заголовок Set-Cookie используется для отправки cookies с сервера пользователю. Куки устанавливаются веб-сервером при помощи заголовка Set-Cookie. Затем браузер будет автоматически добавлять их в (почти) каждый запрос на тот же домен при помощи заголовка Cookie.
Директривы:
**prov=**    - не нашел описания
**domain=**  - По умолчанию куки доступно лишь тому домену, который его установил. Мы не сможем получить эти куки на поддомене
**expires=**  - По умолчанию, если куки не имеют ни одного из этих параметров (expires, max-age), то они удалятся при закрытии браузера. Такие куки называются сессионными («session cookies»). Чтобы помочь куки «пережить» закрытие браузера, надо установить значение опций expires или max-age.
**path=**    - URL-префикс пути, куки будут доступны для страниц под этим путём. Должен быть абсолютным. По умолчанию используется текущий путь. Как правило, указывают в качестве пути корень path=/, чтобы наше куки было доступно на всех страницах сайта. Если куки установлено с path=/admin, то оно будет доступно на страницах /admin и /admin/something, но не на страницах /home или /adminpage.
**HttpOnly**  - Эта настройка запрещает любой доступ к куки из JavaScript. Мы не можем видеть такое куки или манипулировать им с помощью document.cookie. Эта настройка используется в качестве меры предосторожности от определённых атак, когда хакер внедряет свой собственный JavaScript-код в страницу и ждёт, когда пользователь посетит её. Но если куки имеет настройку httpOnly, то document.cookie не видит его, поэтому такое куки защищено.
```
Connection closed by foreign host.
```
Соединение telnet закрыто сервером, в соответствии с заголовком **Connection: close**

