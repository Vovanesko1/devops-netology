### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-02-py/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Никакого значения присвоено не будет, операция заканчивается ошибкой: "TypeError: unsupported operand type(s) for +: 'int' and 'str'. Вывод значения 'c': NameError: name 'c' is not defined"  |
| Как получить для переменной `c` значение 12?  | 	str(a); c=a+b; int( c ) |
| Как получить для переменной `c` значение 3?  | a=int(a) b=int(b) c=int( c ) c=a+b Можно было бы написать 'c=a+int(b)' сразу, но типы этих переменных были переопределны в примерах выше.  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/devops-netology", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ' , '~/netology/devops-netology/')
        print(prepare_result)
```

### Вывод скрипта при запуске при тестировании:
```
$ git status
On branch main
Your branch is ahead of 'origin/main' by 2 commits.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   2.md
        modified:   3.md
        modified:   444

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        myscrypt

no changes added to commit (use "git add" and/or "git commit -a")

$ python3 myscrypt
~/devops-netology/2.md
~/devops-netology/3.md
~/devops-netology/444
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import subprocess
from pathlib import Path

while (True):
	vvod = input('Введите полный путь к репозиторию: ')
	if vvod[0] != '/':
		pwd = os.popen('pwd').read()
		vvod = pwd.replace('\n' , '/') + vvod
	entries = Path(vvod)
		
	try:
			for entry in entries.iterdir(): 
					if entry.name == '.git':
							bash_command = ["cd " + vvod, "git status"]
							result_os = os.popen(' && '.join(bash_command)).read()
							for result in result_os.split('\n'): 
									if result.find('modified') != -1:
											print (result.replace('\tmodified:   ' , vvod + '/'))
			break
	except FileNotFoundError:
			print ('Нет такого каталога. Повторите ввод.')
```

### Вывод скрипта при запуске при тестировании:
```
$ python3 myscrypt2
Введите полный путь к репозиторию: /home/vagrant/devops-netology
/home/vagrant/devops-netology/2.md
/home/vagrant/devops-netology/3.md
/home/vagrant/devops-netology/444
```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3.9

import os
import time
import socket

arr_address = ["drive.google.com", "mail.google.com", "google.com"] 

old_ip_address = ["","",""]

for i in range(3):
    old_ip_address[i]=socket.gethostbyname(arr_address[i])
    print(arr_address[i], "-", old_ip_address[i])
    
while (True):
	time.sleep (6)
	flag = False
	for i in range(3):
		tmp = socket.gethostbyname(arr_address[i])
		if(tmp != old_ip_address[i]):
			flag = True
			print("[ERROR]",arr_address[i], "IP mismatch:", old_ip_address[i], tmp) 
			old_ip_address[i] = tmp
	if flag:
		print()
```

### Вывод скрипта при запуске при тестировании:
```
$ python3 myscrypt3
drive.google.com - 64.233.164.194
mail.google.com - 173.194.222.19
google.com - 173.194.222.100
[ERROR] mail.google.com IP mismatch: 173.194.222.19 173.194.222.18

[ERROR] google.com IP mismatch: 173.194.222.100 173.194.222.102

```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 

### Ваш скрипт:
```python
???
```

### Вывод скрипта при запуске при тестировании:
```
???
```