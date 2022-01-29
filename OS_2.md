# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"
1. - На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:
поместите его в автозагрузку, предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron), удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

Загрузка, распаковка, копирование бинарника 
```sh
cd ~
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar zxvf node_exporter-1.3.1.linux-amd64.tar.gz
sudo cp ./node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
```
/usr/local/bin/ - исполняемые файлы для всех учетных записей
/usr/lib/systemd - тут хранятся юнит-файлы, которые установлены из пакетов
/etc/systemd/system/ - unit файлы автозапуска системы

Создаем пользователя node_exp, делаем его владельцем файла 'node_exporter', cоздаем unit файл 'node_exporter.service' в 'systemd':

```sh
sudo useradd --no-create-home --shell /bin/false node_exp
sudo chown -R node_exp:node_exp /usr/local/bin/node_exporter
```
vim /etc/systemd/system/node_exporter.service
```sh
[Unit]
Description=Node Exporter Service
After=network.target
[Service]
User=node_exp # - пользователь, от имени которого работат сервис
Group=node_exp
Type=simple
ExecStart=/usr/local/bin/node_exporter # — полный путь к исполняемому файлу программы с параметрами запуска
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
Перечитываем конфигурацию systemd:
```sh
systemctl daemon-reload
```
Разрешаем автозапуск:
```sh
systemctl enable node_exporter
```
Запускаем службу:
```sh
systemctl start node_exporter
```
В браузере 'http://10.0.2.15:9100/metric' видим много выводов по метрикам
Метрики можно посмотреть через  'node_exporter --help'

Для возможности ручного управления параметрами работы сервиса создам каталог '/etc/systemd/system/node_exporter.service.d' и файл 'override.conf'. В файл добавлю разделы и параметры, которые требуется преопределить для этого сервиса. Например такой:
```sh
[Service]
IgnoreSIGPIPE=false
```
В файл '/etc/systemd/system/node_exporter.service' добавлю путь к этому файлу:
```sh
[Service]
EnvironmentFile=-/etc/systemd/system/node_exporter.service.d/override.conf
```
```sh
systemctl stop node_exporter.service
systemctl daemon-reload
systemctl start node_exporter.service
systemctl status node_exporter.service
● node_exporter.service - Node_exporter_service
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/node_exporter.service.d
             └─override.conf
     Active: active (running) since Thu 2022-01-27 12:34:37 UTC; 2min 5s ago
   Main PID: 5594 (node_exporter)
      Tasks: 5 (limit: 2309)
     Memory: 7.4M
     CGroup: /system.slice/node_exporter.service
             └─5594 /usr/local/bin/node_exporter
```
Сервис работает
```sh
sudo reboot
systemctl status node_exporter.service
```
Сервис работает

2. - Ознакомьтесь с опциями node_exporter и выводом /metrics по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

По умолчанию собирается 495 метрик, взято со страницы браузера.
60 для CPU, 59 для memory, 67 для network, 34 для disk

Какие выбрать - не знаю! ИМХО никакие не нужны. Когда возникнет неоходимость решить проблему через мониоринг - включить все и искать аномалии.

Для включения и выключения отдельных опций отредактируем ранее созданный файл параметорв 
'/etc/systemd/system/node_exporter.service'
строку 'ExecStart=/usr/local/bin/node_exporter' заменим на строку 'ExecStart=/usr/local/bin/node_exporter $OPTIONS'
В файл ‘/etc/systemd/system/node_exporter.service.d/override.conf’ добавим значение переменной $OPTIONS, например
$OPTIONS='\fB--collector.cpu.info\fR'
(взято из node_exporter --help-man)

3. - Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (sudo apt install -y netdata). После успешной установки:
в конфигурационном файле /etc/netdata/netdata.conf в секции [web] замените значение с localhost на bind to = 0.0.0.0,
добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте vagrant reload:
config.vm.network "forwarded_port", guest: 19999, host: 19999
После успешной перезагрузки в браузере на своем ПК (не в виртуальной машине) вы должны суметь зайти на localhost:19999. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

```sh
sudo vim /etc/netdata/netdata.conf
                bind socket to IP = 0.0.0.0
```
В'G:\VirtualBox VMs\Vagrantfile' дабавил строку 'config.vm.network "forwarded_port", guest: 19999, host: 19999'
vagrant reload
И на основом компьютере на странице localhost:19999 видны красивые метрики.

4. - Можно ли по выводу dmesg понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

Ядро Linux, как и другие программы может и выводит различные информационные сообщения и сообщения об ошибках. Все они выводятся в буфер сообщения ядра, так называемый kernel ring buffer. Основная причина существования этого буфера - надо сохранить сообщения, которые возникают во время загрузки системы пока сервис Syslog ещё не запущен и не может их собирать.
Для получения сообщений из этого буфера можно просто прочитать файл /var/log/dmesg. Однако, более удобно это можно сделать с помощью команды 'dmesg'.

ОС Знает, что она виртуальная. Наличие таких строк в выводе сообщений ядра через команду 'dmesg' тому подтверждение:
```sh
BIOS-provided physical RAM map:
CPU MTRRs all blank - virtualized system.
Booting paravirtualized kernel on KVM
systemd[1]: Detected virtualization oracle.
Hypervisor detected: KVM
ata3.00: ATA-6: VBOX HARDDISK, 1.0, max UDMA/133
```

5. - Как настроен sysctl fs.nr_open на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (ulimit --help)?

sysctl - утилита для настройки параметров работы ядра ОС
```sh
sysctl fs.nr_open
fs.nr_open = 1048576
```
Это максимальное количество файлов (файловых дескрипторов), которые могут быть выделены одному процессу


Количество файловых дескрипторов, открытых всеми процессами, не может превышать /proc/sys/fs/file-max
```sh
 cat /proc/sys/fs/file-max
9223372036854775807
```
Количество файловых дескрипторов, открытых одним процессом, не может превышать мягкое ограничение Nofile в пользовательском ограничении.
```sh
 ulimit -n
1024
```
На просторах Интернет нашел такой запрос: "когда я попытался установить программное обеспечение на RedHat EL5, я получил ошибку, что ожидаемое значение soft/hard nofile составляет 4096, а по умолчанию 1024"
Для изменний числа одновременно открытых файлов используем команду
ulimit -n <количество файлов>: указать максимальное количество файлов, которые могут быть открыты одновременно

Мягкий предел Nofile не может превышать его жесткий предел (cat /etc/security/limits.conf)
жесткий предел nofile не может превышать системный /proc/sys/fs/nr_open (cat /proc/sys/fs/file-max)
```sh
 cat /proc/sys/fs/file-max
9223372036854775807
```
ulimit используется для ограничения ресурсов, занимаемых процессом запуска оболочки, и поддерживает следующие типы ограничений: размер создаваемого файла ядра, размер блока данных процесса, размер файла, создаваемого процессом оболочки, размер блокировки памяти, резидентная память Размер набора, количество дескрипторов открытых файлов, максимальный размер выделенного стека, время ЦП, максимальное количество потоков для одного пользователя и максимальная виртуальная память, которую может использовать процесс Shell. В то же время он поддерживает жесткие и мягкие ограничения ресурсов.

6. - Запустите любой долгоживущий процесс (не ls, который отработает мгновенно, а, например, sleep 1h) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через nsenter. Для простоты работайте в данном задании под root (sudo -i). Под обычным пользователем требуются дополнительные опции (--map-root-user) и т.д.

> Теория на память:
Пространство имён (англ. namespace) — это механизм ядра Linux, обеспечивающий изоляцию процессов друг от друга. Работа по его реализации была начата в версии ядра 2.4.19. На текущий момент в Linux поддерживается шесть типов пространств имён:

| Пространство имён |	Что изолирует 
| ------ | ------ |
|PID	|    PID процессов
|NETWORK|	Сетевые устройства, стеки, порты и т.п.
|USER|	ID пользователей и групп
|MOUNT|	Точки монтирования
|IPC|	SystemV IPC, очереди сообщений POSIX
|UTS|	Имя хоста и доменное имя NIS
> Для создания новых пространств имён PID используется системный вызов clone() c флагом CLONE_NEWPID
В отличие от fork() вызов clone() не просто создаёт копию, он позволяет процессу-потомку использовать некоторые части контекста выполнения совместно с вызывающим процессом, например: область памяти, таблица файловых дескрипторов и таблица обработчиков сигналов. Но позволяет и разделять элементы контекста выполнения между дочерним и родительским процессами (используя флаг CLONE_NEWPID напирмер).
В Linux они обычно представлены как файлы в директории /proc/<pid>/ns
Пространства имен также можно создавать с помощью системного вызова unshare. Разница между clone и unshare в том, что первый порождает новый процесс внутри нового набора пространств имен, а последний перемещает в новый набор пространств имен текущий процесс.

Использую **'unshare -Ufp --mount-proc'**

**-U, --user[=file]**
           Unshare the user namespace. If file is specified, then a persistent namespace is created by a bind mount.
**--mount-proc[=mountpoint]**
           Just before running the program, mount the proc filesystem at
           mountpoint (default is /proc). This is useful when creating a
           new PID namespace. It also implies creating a new mount
           namespace since the /proc mount would otherwise mess up
           existing programs on the system. The new proc filesystem is
           explicitly mounted as private (with MS_PRIVATE|MS_REC).
**-f, --fork**
           Fork the specified program as a child process of unshare
           rather than running it directly. This is useful when creating
           a new PID namespace. Note that when unshare is waiting for
           the child process, then it ignores SIGINT and SIGTERM and
           does not forward any signals to the child. It is necessary to
           send signals to the child process.
**-p, --pid[=file]**
           Unshare the PID namespace. If file is specified, then a
           persistent namespace is created by a bind mount. (Creation of
           a persistent PID namespace will fail if the --fork option is
           not also specified.)

Получим выделенное PID пространство пользователя nobody. Можно добавить ключ '-r' и будет под root.
В этом пространстве PID запустил sleep 600 и больше ничего кроме bash нет
```sh
nobody@vagrant:~$ ps -uax
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
nobody         1  0.0  0.2   7364  4316 pts/0    S    12:41   0:00 -bash
nobody        10  0.0  0.0   5476   588 pts/0    S    12:41   0:00 sleep 600
nobody        15  0.0  0.1   9084  3528 pts/0    R+   12:49   0:00 ps -uax
```
Как заглянуть в это пространство через утилиту 'nsenter' не понятно. Наверное никак.

7. - Найдите информацию о том, что такое :(){ :|:& };:. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (это важно, поведение в других ОС не проверялось). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов dmesg расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

Команда ':(){ :|:& };:'
Эта команда является логической бомбой. Она оперирует определением функции с именем ‘:‘, которая вызывает сама себя дважды: один раз на переднем плане и один раз в фоне. Функция пораждает двойников до бесконечности, пока не закончатся файловые дескрипторы.

ОС ububnu после запуска команды не стабилизировалась за 20 минут. Откатился на снапшот и поменял максимальное число файловых дескрипторов для текущего пользователя на 100 и запустил бомбу опять 
```sh
ulimit -u 100
:(){ :|:& };:
```
Командная строка с возможностью ввода теперь появилась довольно быстро, но команды не работали:
```sh
:~$ lsof
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
```
Так что система не восстановила свою работу под запустившим бомбу пользователем.
Перешёл на другой терминал под пользователем 'root' и просмотрел вывод 'dmesg'. Никаких записей связанных со сбоем не нашёл.





