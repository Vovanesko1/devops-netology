# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"
1. - Какой системный вызов делает команда cd? В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, поэтому запустить strace непосредственно на cd не получится. Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. Вам нужно найти тот единственный, который относится именно к cd. Обратите внимание, что strace выдаёт результат своей работы в поток stderr, а не в stdout.

Для работы 'grep' пришлось вывод ошибок вывести в стандартный вывод
```sh
~$ strace /bin/bash -c 'cd /tmp' 2>&1 | grep /tmp
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffdfcd4fbd0 /* 23 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")         
```
2. - Попробуйте использовать команду file на объекты разных типов на файловой системе. Например:
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64 
Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.

```sh
:~$ strace file /dev/sda 2>&1 | grep openat
(или strace -e openat file /dev/sda)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
```
Исключить библиотеки и папки. остаётся два расположения: 
"/usr/share/misc/magic.mgc"
"/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache"
Файл "gconv-modules.cache" содержит кэш программы преобразования кодировок.
Файл "magic.mgc" содержит ссылку в другое место.
```sh
$ ls -al
lrwxrwxrwx   1 root root      24 Jan 16  2020 magic.mgc -> ../../lib/file/magic.mgc
/lib/file$ cat magic.mgc
```
Содержимое файла 'magic.mgc' похоже на файл с шаблонами для определения типов файлов. Предположу, что /lib/file/magic.mgc и есть файл шабонов для работы команды 'file'.

3. - Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

Создаём
top > 1.log
ctrl+z
bg
rm 1.log

Смотрим какие открытые файлы имеют статус 'deleted', их файловый дескриптор, PID
```sh
 lsof | grep deleted
 top       2209                        vagrant    1w      REG              253,0        2    1059125 /home/vagrant/1.log (deleted)
```
Копируем текущее содержимое, если информация нужна, и обнуляем размер 
 ```sh
cp /proc/2209/fd/1 2209.log
cat 2209.log
> /proc/2209/fd/1
cp /proc/2209/fd/1 2209.log
cat 2209.log
 ```
В первом случае файл сорержит информацию, во втором уже нет.  'lsof | grep deleted' показывает фантомный размер.
 
4. - Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби-процесс занимает 32 КБ в таблице страниц памяти и еще 32 КБ использует для своей личной памяти.Process Private Memory. Page Table Memory.
Занимает PID. Большое количество зомби процессов может исчерпать все возможные PID, тогда рождение новых процессов станет невозможным. Говорят, Солярис научился убивать часть процессов при переполненни таблицы PID. Остальным только hardresset.
Решение - не дожидаясь переполнения таблицы завершить родительский процесс для зомби. Тогда он перейдёт в статус сироты. Сирот подберёт процесс с PID 1. И обработает, т.е. завершит.

5. - В iovisor BCC есть утилита opensnoop:
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом bpfcc-tools для Ubuntu 20.04. Дополнительные сведения по установке.

opensnoop отслеживает все файлы, которые были открыты в системе (системные вызовы трассировки open() )
```sh
sudo apt-get install bpfcc-tools
~sudo /usr/sbin/opensnoop-bpfcc
PID    COMM               FD ERR PATH
992    vminfo              5   0 /var/run/utmp
663    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
663    dbus-daemon        27   0 /usr/share/dbus-1/system-services
663    dbus-daemon        -1   2 /lib/dbus-1/system-services
663    dbus-daemon        27   0 /var/lib/snapd/dbus-1/system-services/
```

6. - Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в /proc, где можно узнать версию ядра и релиз ОС.

Запускаем утилиту execsnoop, она ожидает системные вызовы, на другом терминале выполняем 'uname -a', получаем результат на первом терминале:
```sh
~$ sudo /usr/sbin/execsnoop-bpfcc
PCOMM            PID    PPID   RET ARGS
uname            3722   2288     0 /usr/bin/uname -a
```

Можно через strace 
```sh
~$ strace -e uname uname -a
uname({sysname="Linux", nodename="vagrant", ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
Linux vagrant 5.4.0-94-generic #106-Ubuntu SMP Thu Jan 6 23:58:14 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
+++ exited with 0 +++
```
Ставим справку по системным вызовам
```sh
apt install manpages-dev
man 2 uname
       Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
cat /proc/sys/kernel/{ostype,hostname,osrelease,version,domainname}
Linux
uit-139
4.4.0-19041-Microsoft
cat: /proc/sys/kernel/version: No such file or directory
sz.ru
```

7. - Чем отличается последовательность команд через ; и через && в bash? Например:
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
Есть ли смысл использовать в bash &&, если применить set -e?

[;] Позволяет записывать две и более команд в одной строке.
echo hello; echo there
[&&] "И-список": благодаря этому все последующие команды будут выполнены только тогда, когда предыдущие завершатся успешно

'set' — это встроенная команда оболочки, которая позволяет отображать или устанавливать переменные оболочки и среды. 
параметр '-e' указывает оболочке выйти, если команда дает ненулевой статус выхода.
```sh
set -e echo 111 && echo 222
222
```
Получаем вывод только второй команды 'echo' на экран. Вывода первого 'echo' нет. Исчез, наверно посчитал первое 'echo' опцией команды 'set'.
Команду 'set –e' часто используют в скриптах.
Смысл использовать в bash &&, если применить 'set -e' - есть. В случае выполнения, например, такой команды
```sh
#!/bin/bash
set -e
echo 111 && echo3 222 && echo 333 && echo 7777
set -e
echo 444 && echo 555 && echo 666
Даёт вывод
111
./set.sh: line 3: echo3: command not found
444
555
666
```
Итого: можем делить скрипт на части. Ошибочное выполнение одной части не приведёт остановке выполнения всего скрипта.

8. - Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?

С этими ключами некоторые распространенные ошибки вызовут немедленный явный сбой скрипта. В противном случае можно получить скрытые баги, которые обнаруживаются только тогда, когда они проявляются в работе.

'-e'           предписывает bash немедленно завершить работу, если какая-либо команда имеет ненулевой статус выхода. 
'-x'           включает режим оболочки, в котором все выполненные команды выводятся в терминал.
'-u'           обрабатывает неустановленные или неопределенные переменные, за исключением специальных параметров, таких как                подстановочные знаки (*) или «@», как ошибки во время раскрытия параметра.
'-o pipefail'  Этот параметр предотвращает маскирование ошибок в конвейере. В случае сбоя какой-либо команды в конвейере                   этот код возврата будет использоваться как код возврата для всего конвейера. По умолчанию конвейер                          возвращает код последней команды, даже если она выполнена успешно.

9. - Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will
       display to describe the state of a process:

               D    uninterruptible sleep (usually IO)
               I    Idle kernel thread
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by its parent

       For BSD formats and when the stat keyword is used, additional characters may be displayed:

               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group

```sh
ps -e -o stat | grep -E  "Ssl|Ss|[S]" --count
138
```
137 процессов со статусом спит в ожидании пробуждения


