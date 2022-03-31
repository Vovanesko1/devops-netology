#Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"
 - Обязательная задача 1

Есть скрипт:
```
#!/usr/bin/env python3.9
a = 1
b = '2'
c = a + b
```
Вопросы:
| Вопрос	| Ответ |
|----|----|
|Какое значение будет присвоено переменной c? |	Никакого значения присвоено не будет, операция заканчивается ошибкой: "TypeError: unsupported operand type(s) for +: 'int' and 'str'. Вывод значения 'c': NameError: name 'c' is not defined" |
|Как получить для переменной c значение 12?|	str(a); c=a+b; int( c )|
|Как получить для переменной c значение 3?	| int(a) int(b) int( c ) c=a+b|
|Как получить для переменной c значение 3?	| a=int(a) b=int(b) c=int( c ) c=a+b|


2. - Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ' , '~/devops-netology/')
        print(prepare_result)
```

3. - Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.	
	
```
#!/usr/bin/env python3.9


import os

bash_command = [input('Введите полный путь к репозиторию: '), "git status"]
result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ' , '~/devops-netology/')
        print(prepare_result)
```
		
4. - Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: drive.google.com, mail.google.com, google.com.		
	
		
```
#!/usr/bin/env python3.9


import os	#подключаем библиотеку для работы с операционной системой
import time #подключаем библиотеку для работы со временем

arr_address = ["drive.google.com", "mail.google.com", "google.com"] 

old_ip_address = ["","",""]

for i in range(3):
	bash_command = ["ping -c 1 " + arr_address[i] + "|tr -s \'\\n\' \' \'|cut -f 3 -d \" \"|tr -d \'(\' |tr -d \')\'|tr -d \'\\n\'"] 
	#вывод утилиты ping (:-/, слава утилите пинг) обрезаем до адреса. "-с 1" - только один ping, "tr -s" заменяет символ, "tr -d" удаляет символ, "cut -f 3" - выбирает 3_е поле вывода с #разделителем "-d " "", и куча экранирований спец символов.
	old_ip_address[i] = os.popen(' && '.join(bash_command)).read()
	#функция от библиотеки "os" , которая выполняет составную команду в bash. ".read()" возвращает результат выполнения bash в переменную old_ip_address
	print(arr_address[i], "-", old_ip_address[i]) 
	#вывод адреса и ip адреса, если без перевода строки, то используем (end='')

while (True):
	time.sleep (6) #пауза 6 секунд
	flag = False #Флаг для перевода строки в нужный момент
	for i in range(3): # бегаем 3 раза, подставляя адреса
		bash_command = ["ping -c 1 " + arr_address[i] + "|tr -s \'\\n\' \' \'|cut -f 3 -d \" \"|tr -d \'(\' |tr -d \')\'|tr -d \'\\n\'"] 
		tmp = os.popen(' && '.join(bash_command)).read()
		if(tmp != old_ip_address[i]): #сравниваем новые адреса со старыми
			flag = True
			print("[ERROR]",arr_address[i], "IP mismatch:", old_ip_address[i], tmp) #если совпадения при сравнении нет - выводим сообщение
			old_ip_address[i] = tmp #в старый ip адрес записывается новое значение
	if flag: #вместо flag подставляется значение: True, False. Если True - выполняется действие
		print() # вывод превода строки
```

		
		
		
		
		
		