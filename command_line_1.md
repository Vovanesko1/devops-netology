# Домашнее задание к занятию "3.1. Работа в терминале, лекция 1"
## Ознакомиться с разделами man bash, почитать о настройках самого bash:
- какой переменной можно задать длину журнала history, и на какой строчке manual это описывается?

862 строка команда HISTSIZE

- что делает директива ignoreboth в bash?

Директива ignoreboth выполняет две директивы ignorespace и ignoredups.
ignoredups убирает одинаковые, введённые подряд, команды из истории
ignorespace убирает из истории команды, начинающиеся на пробел.
В man bash я не нашел синтаксиса как задать этот параметр. Пришлось воспользоваться посиковиком в Интернет: export HISTCONTROL=ignoreboth

- В каких сценариях использования применимы скобки {} и на какой строчке man bash это описано?

навигация по тексту, описанная в SUMMARY OF LESS COMMANDS:
```sh
25   ESC-}  ^RightArrow   Right to last column displayed.
26   ESC-{  ^LeftArrow    Left  to first column.
62   {  (  [           *  Find close bracket } ) ].
63   }  )  ]           *  Find open bracket { ( [.
```
В основном тексте описания bash:
```sh
 179 ! case  coproc  do done elif else esac fi for function if in select then until while { } time [[ ]]
 257 { list; }
list is simply executed in the current shell environment.  list must be terminated with  a  newline  or
semicolon.  This is known as a group command.  The return status is the exit status of list.  Note that
unlike the metacharacters ( and ), { and } are reserved words and must occur where a reserved  word  is
permitted  to be recognized.  Since they do not cause a word break, they must be separated from list by
whitespace or another shell metacharacter.
403 pound-command  (see Compound Commands above).  That command is usually a list of commands between { and }, but may be any command listed under Compound Commands above, with one exception.
639 BASH_SOURCE
An array variable whose members are the source filenames where the corresponding shell  function  names
in  the FUNCNAME array variable are defined.  The shell function ${FUNCNAME[$i]} is defined in the file
${BASH_SOURCE[$i]} and called from ${BASH_SOURCE[$i+1]}.
.........
 ```
И другие упоминания.
Вцелом, как я понял, фигурные скобки используются для перечисления не числовых параметров, команд, зарезервированных слов, создания массивов и навигаии по тексту.

- С учётом ответа на предыдущий вопрос, как создать однократным вызовом touch 100000 файлов? Получится ли аналогичным образом создать 300000? Если нет, то почему?

Создать 100 000 и 300 000 файлов командой touch не вышло. Изучив интернет я узнал про команду "ulimit" - команда оболочки Linux, которая может устанавливать или сообщать об ограничении ресурсов текущего пользователя. 
```sh
ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7597
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8000
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7597
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
Через неё нашел нужный параметр:
```sh
ulimit -s
stack size (kbytes, -s) 8000
```
Задаётся этот парамтр один раз за сессию. По крайней мере в моей ssh сесси он позволяет задавать себя только одн раз. Выйдя и зайдя по ssh задал его значение 20 000
```sh
ulimit -s 20000
```
После чего командой создал нужное число файлов. 
```sh
touch qq{1..300000}
```
Команда 'ls' для такого количества файлов отрабатывла больше 2_х минут :-).

- В man bash поищите по /\[\[. Что делает конструкция [[ -d /tmp ]]

```sh
When used with [[, the < and > operators sort lexicographically using the current locale.  The test command sorts using ASCII ordering.
       -d file              True if file exists and is a directory.
```
Конструкция [[ -d /tmp ]] проверяет существует директория 'tmp' в текущей директории.
Похоже, что это скриптовая команда. Т.к. стандартный вывод её пуст: [[ -d /tmp ]] > 1.txt

- Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке:
bash is /tmp/new_path_directory/bash
bash is /usr/local/bin/bash
bash is /bin/bash
```sh
type -a NAME
display all locations containing an executable named NAME; includes aliases, builtins, and functions
 ```
выводит все места, где существует имя NAME
Создал папку и файл new_path_directory/bash. Побпробовал создать алиас 
```sh
alias bash="/tmp/new_path_directory/bash"
```
Но вывод type -a bash оказался не очень похож:
```sh
bash is aliased to `/tmp/new_path_directory/bash'
bash is /usr/bin/bash
bash is /bin/bash
```sh
Тогда изменил пременную $PATH и скопировал файл в папку:
```sh
export PATH=$PATH:/tmp/new_path_directory/
cp /bin/bash /tmp/new_path_directory/
```sh
Вывод type -a bash оказался не в том порядке. Нашел команду:
```sh
 export PATH="/tmp/new_path_directory/:$PATH"
```
Это решило проблему добавления в начало нужной строки. Как удалить лишнюю нижнюю строчку я не разобрался. В материалах к лекции указывался "which – порядок в $PATH". Но его синтаксис, в данном случае, не понятен.
Прописывать в постоянные пользовательские (файл .bashrc) или системные переменные не стал. 
```sh
 type -a bash
bash is /tmp/new_path_directory/bash
bash is /usr/bin/bash
bash is /bin/bash
bash is /tmp/new_path_directory/bash
```

- Чем отличается планирование команд с помощью batch и at?

Batch подходит для планирования выполнения задач при загрузке сисемы ниже пороговой.
at подходит для выполнения задач в заданное время.
