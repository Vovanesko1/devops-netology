# Домашнее задание к занятию "3.9. Элементы безопасности информационных систем"
1. - Установите Bitwarden плагин для браузера. Зарегестрируйтесь и сохраните несколько паролей.

[plugin](https://github.com/Vovanesko1/devops-netology/blob/66c64cdc667b602f20b5bee9c48449bee8868050/1.jpg)

2. - Установите Google authenticator на мобильный телефон. Настройте вход в Bitwarden акаунт через Google authenticator OTP.

[2FA](https://github.com/Vovanesko1/devops-netology/blob/66c64cdc667b602f20b5bee9c48449bee8868050/2fa%20enable.jpg)

3. - Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.
```
sudo apt-get install apache2
sudo ufw app list
sudo ufw allow 'Apache'
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
sudo systemctl status apache2
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
> note: 
openssl: базовый инструмент командной строки для создания и управления сертификатами, ключами и другими файлами OpenSSL.
req: эта подкоманда указывает, что на данном этапе нужно использовать запрос на подпись сертификата X.509 (CSR). X.509 – это стандарт инфраструктуры открытого ключа, которого придерживаются SSL и TLS при управлении ключами и сертификатами. То есть, данная команда позволяет создать новый сертификат X.509.
-x509: эта опция вносит поправку в предыдущую субкоманду, сообщая утилите о том, что вместо запроса на подписание сертификата необходимо создать самоподписанный сертификат.
-nodes: пропускает опцию защиты сертификата парольной фразой. Нужно, чтобы при запуске сервер Apache имел возможность читать файл без вмешательства пользователя. Установив пароль, придется вводить его после каждой перезагрузки.
-days 365: эта опция устанавливает срок действия сертификата (как видите, в данном случае сертификат действителен в течение года).
-newkey rsa:2048: эта опция позволяет одновременно создать новый сертификат и новый ключ. Поскольку ключ, необходимый для подписания сертификата, не был создан ранее, нужно создать его вместе с сертификатом. Данная опция создаст ключ RSA на 2048 бит.
-keyout: эта опция сообщает OpenSSL, куда поместить сгенерированный файл ключа.
-out: сообщает OpenSSL, куда поместить созданный сертификат.

При использовании OpenSSL нужно также создать ключи Диффи-Хеллмана, которые нужны для поддержки PFS
```
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
Создать сниппет конфигураций, указывающий место хранения файлов SSL-сертификата и ключа.
```
sudo nano /etc/apache2/conf-available/ssl-params.conf
```
```
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3
SSLHonorCipherOrder On
Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
SSLCompression off
SSLSessionTickets Off
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"
```
Настроить виртуальный хост Apache для поддержки сертификата SSL.

Отредактировать директивы ServerAdmin и ServerName, изменить параметры SSL, указав файлы ключа и сертификата, и раскомментировать раздел конфигураций, отвечающих за совместимость с устаревшими версиями браузеров.
```
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf.bak
sudo nano /etc/apache2/sites-available/default-ssl.conf
```
Можно использовать Mozilla SSL configuration generator
```
<IfModule mod_ssl.c>
<VirtualHost _default_:443>
ServerAdmin your_email@example.com
ServerName server_domain_or_IP
DocumentRoot /var/www/html
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
SSLEngine on
SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
<FilesMatch "\.(cgi|shtml|phtml|php)$">
SSLOptions +StdEnvVars
</FilesMatch>
<Directory /usr/lib/cgi-bin>
SSLOptions +StdEnvVars
</Directory>
BrowserMatch "MSIE [2-6]" \
nokeepalive ssl-unclean-shutdown \
downgrade-1.0 force-response-1.0
</VirtualHost>
</IfModule>
```
Настроить незашифрованные виртуальные хосты для автоматической переадресации запросов на зашифрованный хост.
Отключить незашифрованный трафик HTTP.
```
sudo nano /etc/apache2/sites-available/000-default.conf
```
```
<VirtualHost *:80>
Redirect "/" "https://127.0.0.1"
</VirtualHost>
```
Чтобы добавить поддержку трафика HTTPS, нужно включить профиль Apache Full и отключить профиль Apache.
```
sudo ufw allow 'Apache Full'
sudo ufw delete allow 'Apache'
```
Включите модуль Apache для SSL, mod_ssl, и модуль mod_headers, который необходим для работы сниппета SSL:
```
sudo a2enmod ssl
sudo a2enmod headers
```
Включите подготовленный виртуальный хост:
```
sudo a2ensite default-ssl
```
Итак, теперь сайт и все необходимые модули включены. Проверьте синтаксис на наличие ошибок:
```
sudo apache2ctl configtest
```
Если ошибок нет, команда вернёт: AH00558
```
sudo systemctl restart apache2
```
[https web](https://github.com/Vovanesko1/devops-netology/blob/e9a3baa06dd6777059f7ce64b84f7997df07477b/https.jpg)

4. - Проверьте на TLS уязвимости произвольный сайт в интернете (кроме сайтов МВД, ФСБ, МинОбр, НацБанк, РосКосмос, РосАтом, РосНАНО и любых госкомпаний, объектов КИИ, ВПК ... и тому подобное).
```
git clone --depth 1 https://github.com/drwetter/testssl.sh.git
cd testssl.sh
./testssl.sh -U  iz.ru
```
```
 Testing vulnerabilities

 Heartbleed (CVE-2014-0160)                not vulnerable (OK), no heartbeat extension
 CCS (CVE-2014-0224)                       not vulnerable (OK)
 Ticketbleed (CVE-2016-9244), experiment.  not vulnerable (OK)
 ROBOT                                     not vulnerable (OK)
 Secure Renegotiation (RFC 5746)           supported (OK)
 Secure Client-Initiated Renegotiation     VULNERABLE (NOT ok), DoS threat (6 attempts)
 CRIME, TLS (CVE-2012-4929)                not vulnerable (OK)
 BREACH (CVE-2013-3587)                    At least 1/4 checks failed (HTTP header request stalled and was terminated, debug: warn_killed:yes POODLE, SSL (CVE-2014-3566)               not vulnerable (OK)
 TLS_FALLBACK_SCSV (RFC 7507)              Downgrade attack prevention supported (OK)
 SWEET32 (CVE-2016-2183, CVE-2016-6329)    not vulnerable (OK)
 FREAK (CVE-2015-0204)                     not vulnerable (OK)
 DROWN (CVE-2016-0800, CVE-2016-0703)      not vulnerable on this host and port (OK)
                                           make sure you don't use this certificate elsewhere with SSLv2 enabled services
                                           https://censys.io/ipv4?q=5EDFE4A2A560024F2AFBF5DD9568D1A9490A66F7587BA966166CFB9B4A75D68B could help you to find out
 LOGJAM (CVE-2015-4000), experimental      not vulnerable (OK): no DH EXPORT ciphers, no DH key detected with <= TLS 1.2
 BEAST (CVE-2011-3389)                     not vulnerable (OK), no SSL3 or TLS1
 LUCKY13 (CVE-2013-0169), experimental     potentially VULNERABLE, uses cipher block chaining (CBC) ciphers with TLS. Check patches
 Winshock (CVE-2014-6321), experimental    not vulnerable (OK)
 RC4 (CVE-2013-2566, CVE-2015-2808)        no RC4 ciphers detected (OK)
```
Из серьёзных выявлена одна ошибка:
Secure Client-Initiated Renegotiation     VULNERABLE (NOT ok), DoS threat (6 attempts)
[Об ошибке:](https://blog.qualys.com/product-tech/2011/10/31/tls-renegotiation-and-denial-of-service-attacks)
Apache, IIS и Lighttpd исправили эту проблему в своих последних выпусках.
Надо отключить или ограничьте возможность клиента инициировать повторное согласование. Например использовать счётчик для подсчета количества раз, когда происходит SSL_CB_HANDSHAKE_START.

Есть еще одна потенциальная проблема:
LUCKY13 (CVE-2013-0169), experimental     potentially VULNERABLE, uses cipher block chaining (CBC) ciphers with TLS. Check patches
Советуют использовать более новую версию OpenSSL. Хотя уязвимость может быть и ложной.
[Обсуждение](https://github.com/drwetter/testssl.sh/issues/1011)

5. - Установите на Ubuntu ssh сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.

```
apt-get install openssh-server
systemctl start sshd.service
ssh-keygen
ssh-copy-id vagrant@192.168.1.129
```
```
ssh vagrant@192.168.1.130

Enter passphrase for key '/home/vagrant/.ssh/id_rsa':
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 22 Mar 2022 04:30:29 PM UTC

  System load:  0.0                Processes:             116
  Usage of /:   11.2% of 30.88GB   Users logged in:       1
  Memory usage: 21%                IPv4 address for eth0: 192.168.1.130
  Swap usage:   0%


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Tue Mar 22 15:53:58 2022 from 192.168.1.150
```

6. - Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH клиента, так чтобы вход на удаленный сервер осуществлялся по имени сервера.

```
cd /home/vagrant/.ssh
ls
authorized_keys  id_rsa  id_rsa.pub  known_hosts
```
id_rsa - приватный ключ
id_rsa.pub - публичный ключ
```
 mv id_rsa.pub new_key.pub
 mv id_rsa new_key
 ssh -i /home/vagrant/.ssh/new_key vagrant@192.168.1.130
Enter passphrase for key '/home/vagrant/.ssh/new_key':
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 22 Mar 2022 04:52:58 PM UTC

  System load:  0.0                Processes:             114
  Usage of /:   11.2% of 30.88GB   Users logged in:       1
  Memory usage: 21%                IPv4 address for eth0: 192.168.1.130
  Swap usage:   0%
```

7. - Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.

```
sudo apt-get install tcpdump
tcpdump -D
1.eth0 [Up, Running]
2.lo [Up, Running, Loopback]
3.any (Pseudo-device that captures on all interfaces) [Up, Running]
4.bluetooth-monitor (Bluetooth Linux Monitor) [none]
5.nflog (Linux netfilter log (NFLOG) interface) [none]
6.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]

sudo tcpdump -c 100 -i eth0 -w dmp100.pcap
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
100 packets captured
100 packets received by filter
0 packets dropped by kernel

sudo apt-get install tftp
tftp
tftp> put 192.168.1.150:dmp100.pcap
tftp> quit
```
[скан из виндового wireshark ](https://github.com/Vovanesko1/devops-netology/blob/1bb66852448850fc12a72f5643bacb720a8295ac/wireshark.jpg)

Дважды создавал файл и дважды он содержал только первый пакет из 100.

8. - Просканируйте хост scanme.nmap.org. Какие сервисы запущены?

```
sudo apt install nmap
sudo nmap -A scanme.nmap.org

Starting Nmap 7.80 ( https://nmap.org ) at 2022-03-22 18:18 UTC
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.18s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 993 closed ports
PORT      STATE    SERVICE    VERSION
19/tcp    filtered chargen
22/tcp    open     ssh        OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 ac:00:a0:1a:82:ff:cc:55:99:dc:67:2b:34:97:6b:75 (DSA)
|   2048 20:3d:2d:44:62:2a:b0:5a:9d:b5:b3:05:14:c2:a6:b2 (RSA)
|   256 96:02:bb:5e:57:54:1c:4e:45:2f:56:4c:4a:24:b2:57 (ECDSA)
|_  256 33:fa:91:0f:e0:e1:7b:1f:6d:05:a2:b0:f1:54:41:56 (ED25519)
80/tcp    open     http       Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Go ahead and ScanMe!
111/tcp   filtered rpcbind
1900/tcp  filtered upnp
9929/tcp  open     nping-echo Nping echo
31337/tcp open     tcpwrapped
Aggressive OS guesses: Linux 2.6.32 - 3.13 (93%), Linux 2.6.22 - 2.6.36 (91%), Linux 3.10 - 4.11 (91%), Linux 3.10 (91%), Linux 2.6.32 (90%), Linux 3.2 - 4.9 (90%), Linux 2.6.32 - 3.10 (90%), Linux 2.6.18 (90%), Linux 3.16 - 4.6 (90%), HP P2000 G3 NAS device (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 25 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   0.69 ms   point (192.168.1.1)
2   1.22 ms   94.242.42.1
3   1.08 ms   94.242.0.28
4   3.95 ms   94.242.0.97
5   2.14 ms   spb-bm18-cr1.ae61-1253.rascom.as20764.net (80.64.98.226)
6   ... 7
8   12.49 ms  be3376.ccr21.sto03.atlas.cogentco.com (130.117.50.225)
9   23.38 ms  be2555.rcr21.cph01.atlas.cogentco.com (154.54.61.237)
10  27.58 ms  be2504.ccr42.ham01.atlas.cogentco.com (154.54.61.229)
11  ...
12  130.59 ms be12488.ccr42.lon13.atlas.cogentco.com (130.117.51.41)
13  132.41 ms be2317.ccr41.jfk02.atlas.cogentco.com (154.54.30.185)
14  141.11 ms be2889.ccr21.cle04.atlas.cogentco.com (154.54.47.49)
15  131.33 ms be2717.ccr41.ord01.atlas.cogentco.com (154.54.6.221)
16  141.48 ms be2831.ccr21.mci01.atlas.cogentco.com (154.54.42.165)
17  156.00 ms be3035.ccr21.den01.atlas.cogentco.com (154.54.5.89)
18  163.40 ms be3037.ccr21.slc01.atlas.cogentco.com (154.54.41.145)
19  239.99 ms be3109.ccr21.sfo01.atlas.cogentco.com (154.54.44.137)
20  179.47 ms be3178.ccr21.sjc01.atlas.cogentco.com (154.54.43.70)
21  181.24 ms be2095.rcr21.b001848-1.sjc01.atlas.cogentco.com (154.54.3.138)
22  181.93 ms 38.142.11.154
23  176.17 ms if-1-6.csw6-fnc1.linode.com (173.230.159.65)
24  175.08 ms if-1-6.csw6-fnc1.linode.com (173.230.159.65)
25  176.58 ms scanme.nmap.org (45.33.32.156)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.43 seconds
```
Открытые порты:
19 TCP - Узел сети может установить соединение с сервером, поддерживающим Character Generator Protocol с использованием TCP или UDP через порт 19. После открытия TCP-соединения, сервер начинает отправлять клиенту случайные символы и делает это непрерывно до закрытия соединения.
22 - SSH
80 - WEB server
111 - rpcbind
Когда на сервере запускается демон nfsd он может начать слушать порт 2049, а получить какой-нибудь другой порт, если 2049 занят. Он сообщает об этом запущенному на том же сервере rpcbind'у, который всегда слушает порт 111. Клиент сначала обращается к rpcbind'у на сервере, узнаёт, запущен ли nfsd и на каком порту, и после этого уже обращается к nfsd.
1900 - upnp
Simple Service Discovery Protocol. Каждое устройство, поддерживающее UPnP и предоставляющее какой-то сервис, систематически отправляет SSDP Notify пакет на 1900 UDP на multicast-адрес. Формат данных пакетов — HTTP.
9929 - nping-echo Nping echo
Существует утилита под именем nping. Nping является частью пакета nmap. Nping имеет несколько режимов работы: ICMP mode,TCP connect mode,TCP mode,UDP mode,Echo mode.
31337 - tcpwrapped
TCPWrapper - это программное решение на стороне клиента для машин Linux / BSD, которое предоставляет функции брандмауэра. Он отслеживает все входящие пакеты на машину, и, если внешний узел пытается подключиться, программа проверяет, авторизован ли узел на основе различных критериев, которые вы можете указать.

9. - Установите и настройте фаервол ufw на web-сервер из задания 3. Откройте доступ снаружи только к портам 22,80,443

Ранее уже настраивался файервол.
```
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```
Вместо sudo ufw allow 80, sudo ufw allow 443 можно использовать:
```
sudo ufw allow 'Apache Full'
```
Вместо sudo ufw allow ssh можно использовать:
```
sudo ufw allow 'OpenSSH'
```

