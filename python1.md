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



#!/usr/bin/env python3.9

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ' , '~/devops-netology/')
        print(prepare_result)

		
		



#!/usr/bin/env python3.9


import os

bash_command = [input('Введите полный путь к удалённому рипозиторию: '), "git status"]
result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ' , '~/devops-netology/')
        print(prepare_result)

		
		

#bash_command = ["cd ~/devops-netology", "git status"]


		
		
#!/usr/bin/env python3.9


import os	#подключаем библиотеку для работы с файловой системой
import time #подключаем библиотеку для работы со временем

arr_address = ["drive.google.com", "mail.google.com", "google.com"] 

old_ip_address = ["","",""]

for i in range(3):
	bash_command = ["ping -c 1 " + arr_address[i] + "|tr -s \'\\n\' \' \'|cut -f 3 -d \" \"|tr -d \'(\' |tr -d \')\'|tr -d \'\\n\'"] 
	#вывод утилиты ping (:-/, слава утилите пинг) обрезаем до адреса. "-с 1" - только один ping, "tr -s" заменяет символ, "tr -d" удаляет символ, "cut -f 3" - выбирает 3_е поле вывода с #разделителем "-d " "", и куча экранирований спец символов.
	old_ip_address[i] = os.popen(' && '.join(bash_command)).read()
	#функция от библиотеки "os" , которая выполняет составную команду в bash. ".read()" возвращает результат выполнения bash в переменную old_ip_address
	print(arr_address[i], "-", old_ip_address[i]) 
	#вывод адреса и ip адреса, ели без перевода строки, то используем (end='')
	

while (True):
	time.sleep (6) #пауза 6 секунд
	flag = False #Флаг для перевода строки в нужный момент
	for i in range(3): # бегаем 3 раза подставляя адреса
		bash_command = ["ping -c 1 " + arr_address[i] + "|tr -s \'\\n\' \' \'|cut -f 3 -d \" \"|tr -d \'(\' |tr -d \')\'|tr -d \'\\n\'"] 
		tmp = os.popen(' && '.join(bash_command)).read()
		if(tmp != old_ip_address[i]): #сравниваем новые адреса со старыми
			flag = True
			print("[ERROR]",arr_address[i], "IP mismatch:", old_ip_address[i], tmp) #если совпадения нет - выводим сообщение об этом
			old_ip_address[i] = tmp #в старый ip адрес записывается новое значение
	if flag: #вместа flag подставляется значение: true false. Если True - выполняется действие
		print()

		
		
		
		
		
		