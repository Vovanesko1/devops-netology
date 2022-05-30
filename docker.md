# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"

- Задача 1
Сценарий выполения задачи:
    создайте свой репозиторий на https://hub.docker.com;
    выберете любой образ, который содержит веб-сервер Nginx;
    создайте свой fork образа;
    реализуйте функциональность: запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.

```
apt-get install docker.io
docker login
cat /root/.docker/config.json
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "dm92....."
		}
	}
docker search nginx
docker pull nginx
docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    0e901e68141f   28 hours ago   142MB
```

настрйка образа без докер манифеста:
```
docker run -it --name="nginx_clear" nginx:latest /bin/bash
root@3a482e9e11f2:cat > /usr/share/nginx/html/index.html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>

service nginx start

docker container inspect nginx_clear
 "IPAddress": "172.17.0.2",
 
http://172.17.0.2
exit

docker commit -m="This image with new start page" nginx_clear nginx_new_indexhtml

docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
nginx_new_indexhtml   latest    78a703916ee5   50 seconds ago   142MB
nginx                 latest    0e901e68141f   30 hours ago     142MB

docker tag 78a703916ee5 vovanesko/netology:1.0

docker login

docker push vovanesko/netology:1.0
```
https://hub.docker.com/r/vovanesko/netology/tags


Настройка докера через манифест файл:
```
mkdir mydockers
cd mydockers
touch Dockerfile
nano Dockerfile
FROM nginx
RUN apt-get update
COPY ./index.html /usr/share/nginx/html/index.html
EXPOSE 80

touch index.html
nano index.html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>

docker build -t nginx_file:v1 .
docker run -d nginx_file:v1
docker ps
docker container inspect 064767131db7
 "IPAddress": "172.17.0.3",
```

В браузере:
http://172.17.0.3
Hey, Netology
Iâ€™m DevOps Engineer!
```
docker system prune
docker rmi ....
```

- Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос: "Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"
Детально опишите и обоснуйте свой выбор.
Сценарий:

    Высоконагруженное монолитное java веб-приложение;
```    
    Основное ограничение монолита - это то, что он не может быть отмасштабирован частично. Наращивать мощности проще не при использовании docker контейнеров или физической мащины, а при виртуальной - по необходимости. Выбор - виртуализация.
``` 
    Nodejs веб-приложение;
```    
    Чаще всего программную платформу Node.js применяют для создания:
    мессенджеров, автоматизированных систем управления, стриминговых сервисов, фреймворков веб-сервисов, браузерных игр, одностраничных приложений.
    Лучше использвать docker контейнеры для быстрого разворачивания и лёгкого масштабирования.
```    
    Мобильное приложение c версиями для Android и iOS;
```    
    Docker не очень подходит для кроссплатформенной разработки. 
    Главным достоинством сборки Android-приложений в Docker контейнере является то, что при необходимости сборки приложения на другом устройстве нет необходимости в установке всего ПО (кроме Android SDK, необходимо чтобы была последняя версия Kotlin, Gradle, platform-tools, build-tools)
    Разработка мобильных приложений android возможна и на виртуалках и через docker
    Разработкой приложений под IOS лучше заниматься на физическом компьютере. Лучше использовать отдельный компьютер, чем разбираться с нюансами работы в виртуальной среде.
```
    Шина данных на базе Apache Kafka;
```    
    Это распределенный горизонтально масштабируемый отказоустойчивый журнал коммитов. Акцент на потоковом контенте, работа с большими потоками данных.
    Это может быть группа отдельных серверов, виртуальная машина или Docker-контейнеры.
```        
    Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
```    
    Хранилище для логов на базе Elasticsearch, Logstash и Kibana называют ELK Stack. В это хранилище можно настроить отправку практически любых логов в разных форматах и большого объема.
    Elasticsearch используется для хранения, анализа, поиска по логам.
	Kibana представляет удобную и красивую web панель для работы с логами.
	Logstash сервис для сбора логов и отправки их в Elasticsearch. 
    Используем виртуализацию для контроля ресурсов в этой связке. 
```     
    Мониторинг-стек на базе Prometheus и Grafana;
```    
    Prometheus отвечает за сбор данных, а Grafana отвечает за отображение данных. 
    Проще настроить через Docker с внешним хранилищем.
```    
    MongoDB, как основное хранилище данных для java-приложения;
```
    СУБД, предлагает документо-ориентированную модель данных. Используем физический сервер. Если нужен кластер, то потребуется уже 3 сервера и для экономии ресурсов можно воспользоваться виртуализацией.
    
    Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
	Преднастроенный Gitlab и Docker Registry образы есть в Docker. Я бы разворачивали их там, подключив внешний volume
```

- Задача 3

    Запустите первый контейнер из образа centos c любым тэгом в фоновом режиме, подключив папку /data из текущей рабочей директории на хостовой машине в /data контейнера;
    Запустите второй контейнер из образа debian в фоновом режиме, подключив папку /data из текущей рабочей директории на хостовой машине в /data контейнера;
    Подключитесь к первому контейнеру с помощью docker exec и создайте текстовый файл любого содержания в /data;
    Добавьте еще один файл в папку /data на хостовой машине;
    Подключитесь во второй контейнер и отобразите листинг и содержание файлов в /data контейнера.
```    
    docker pull debian
    docker pull centos
    mkdir data
    docker run -it -v /home/user/mydockers/data:/mnt/data --name=cent1 centos
    ctrl+p ctrl+q
    docker run -it -v /home/user/mydockers/data:/mnt/data --name=deb1 debian
	ctrl+p ctrl+q
	docker  exec -it cent1 touch  /mnt/data/1.txt
	touch /home/user/mydockers/data/2.txt
	docker  exec -it deb1 ls  /mnt/data/
	1.txt  2.txt
```
Запуск пришлось делать в интерактивном режиме. С ключом -d контейнеры сразу после создания закрывались. Вероятно из-за отсутствия работающих полезных процессов и сервисов.
	
- Задача 4 (*)
Воспроизвести практическую часть лекции самостоятельно.
Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

