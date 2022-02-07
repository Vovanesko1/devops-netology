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
Перешёл на другой терминал под пользователем 'root' и просмотрел вывод 'dmesg'. Запускал новый терминал и под тем же пользователем. Никаких записей связанных со сбоем не нашёл.

------------------------------------------------------------------------------------------------------

- ✨Александр Козлов отправил(a) решение на доработку 31 янв. в 14:02✨

Александр Козлов отправил(a) решение на доработку 31 янв. в 14:02

Александр Козлов
ПРЕПОДАВАТЕЛЬ
31 января 2022 14:02

Добрый день!

- По первому и второму заданию: небольшое замечание по поводу юнит файла — хорошо что вы поизучали механизм оверрайдов, но замечу, что на мой взляд, в этом примере его использовать это переусложнение. Он нужен для тех ситуаций, когда например юнит файл поставляется из пакета, а вам нужно изменить его поведение не меняя исходного юнит файла (т.к. он может быть изменен с обновлением пакета)

 Если без использования отдельного файла конфига для параметров, тогда правим UNIT файл (“/etc/systemd/system/node_exporter.service”), добавляя зарезервированную переменную $EXTRA_OPTS:
```
ExecStart=/usr/local/bin/node_exporter $EXTRA_OPTS
```
Теперь когда выполняется команда
“Systemctl start node_exporter --collector.disable-defaults --collector.cpu”
ищется файл внутри “/etc/systemd/” с указанным именем сервиса.
Читается найденный UNIT файл “/etc/systemd/system/node_exporter.service”.
Парамерты “--collector.disable-defaults --collector.cpu” передаются в процесс  “/usr/local/bin/node_exporter” **как опции**

Это позволит запускать процессы с нужными опциями, не правя конфиг перед запуском. Однако, как Вы и писали, мы вмешиваемся в UNIT файл. Если он меняется с обновлением, то наши измененийя могут быть утеряны.

- Как включать и отключать метрики — почитайте доку по node exporter на github, там это довольно ясно описано. Всё таки жду от вас список коллекторов, которые вы бы включили в экспортере по умолчанию, если возникнет задача замониторить инфраструктуру.

https://github.com/prometheus/node_exporter#collectors
Включаем и отключаем метрики по их **типам**:
--collector.<name>           # добавить тип метрики
--no-collector.<name>   # исключить тип метрики
--collector.disable-defaults  # отключить все метрики по умолчанию
--collector.perf --collector.perf.cpus=2-7:2  # предоставляет метрики на на ЦП 2,4,6 (шаг 2)

По умолчанию node_exporter предоставляет ВСЕ дефолтные метрики из включенных коллекторов.
Это рекомендуемый способ сбора метрик, чтобы избежать ошибок при сравнении метрик разных семейств. Так что отдельные метрики включать не стоит, если передавать собранные данные далее. Впрочем, как собрать отдельные метрики и не написано!

В качестве типов метрик для мониторинга инфраструктуры я бы использовал такие:
--collector.disable-defaults --collector.<name> ...

|Имя   |	Описание | Предпосылки 
| ------ | ------ | ---- |
| arp |	Предоставляет статистику ARP из /proc/net/arp | Если есть кольца в сети ловить arp шторм
| diskstats |  Exposes disk I/O statistic | Для понимания на какое по  скорости хранилище поместить и отлова аномального поведения софта
|filesystem |	Exposes filesystem statistics, such as disk space used | Контроль места на дисках
|loadavg |	Exposes load average | Востребованность сервисов и контроль ресурсов виртуалок
|netdev |	Exposes network interface statistics such as bytes transferred | Отслеживание высоких сетевых нагрузок
| ntp |	Exposes local NTP daemon health to check time | Ищем убежавшие часы и не настроенный NTP 
Утилизацию CPU и Memory даёт виртуализация, на хосты без виртуализации эти метрики можно поставить. 
     
**Где бы найти метрику, которая отслеживает дату устаревания сертификатов?!**

- Шестое задание: изучите ман повнимательней, там буквально в самом начале есть параметр, который вам нужен. В качестве ответа пришлите листинг, в котором вы запускаете интерпретатор в PID неймспейсе другого, уже запущенного процесса из задачи, и список процессов (чтобы вы и я могли убедиться что вы действительно находитесь в другом неймспейсе)

Использовал синтаксис из лекции.
```     
unshare  -f --pid  --mount-proc sleep 200 &
[1] 15847
ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root     15847  0.2  0.0  13952   788 tty1     SN   09:11   0:00 unshare -f --pid --mount-proc sleep 200
root     15848  0.2  0.0  13964   816 tty1     SN   09:11   0:00 sleep 200
root     15849  0.0  0.0  17656  2020 tty1     R    09:11   0:00 ps -aux
nsenter --target 15848 --pid --mount
ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  13964   816 tty1     SN+  09:11   0:00 sleep 200
root         2  0.3  0.0  18568  3572 tty1     S    09:11   0:00 -bash
root         6  0.0  0.0  17656  2020 tty1     R    09:12   0:00 ps -aux
```

Сложность была в том, что я пытался подключиться к процессу 'unshare', а не к 'sleep'
Как понять, что именно этот 'sleep' внутри именно этого пространства имён? Если выделенных пространств много и процессов много, то нужно понимание кто где крутится.

- Седьмое задание: хотелось бы увидеть более ясный ответ на вопросы: что именно делает «логическая бомба» как вы её назвали и за счёт чего происходит отказ? Вы написали в ответе про файловые дескрипторы, мне непонятно — как это связано? И объясните, пожалуйста, на что именно влияет установка ulimit как указано в вашем решении?
Остальные задания решены хорошо! Но всё же отправляю на доработку!

Я думал, что функция открывает файлы и исчерпывает дескрипторы. Файловые дескрипторы оказались не при чём. Функция порождает процесс (или 2, не понятно), который порождает новый процесс (или 2) и так бесконечно. Заканчивается выделенное пользователю число процессов или память, что быстрее. 
Утилиту 'ulimit' можно использовать для ограничения и файловых дескрипторов, если использовать ключ '-n', что и ввело меня в заблуждение.
В данном случае 'ulimit -u 100' используется для изменения максимального количества пользовательских процессов текущего пользователя до 100. 

 

 









