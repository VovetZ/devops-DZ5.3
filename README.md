# devops-DZ5.3

# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"

## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.

## Решение

- Регистрируемся [hub.docker.com](https://hub.docker.com/)
- Скачиваем образ nginx:stable  (1.22.0) на момент выполнения.

```bash
vk@vk-desktop:~/Docker/nginx$ sudo docker pull nginx:stable
stable: Pulling from library/nginx
7a6db449b51b: Pull complete 
139f0492fc3f: Pull complete 
54186ccb0ecc: Pull complete 
397e52387904: Pull complete 
3346bded5d4c: Pull complete 
90a40225b3c0: Pull complete 
Digest: sha256:f2dfca5620b64b8e5986c1f3e145735ce6e291a7dc3cf133e0a460dca31aaf1f
Status: Downloaded newer image for nginx:stable
docker.io/library/nginx:stable
vk@vk-desktop:~/Docker/nginx$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        stable    1b84ed9be2d4   2 weeks ago   142MB
```
- Готовим Dockerfile вида

```
FROM nginx:stable
COPY index.html /usr/share/nginx/html/
```

- Готовим index.html вида

```html
<html>
    <head>Hey, Netology</head>
    <body>
        <h1>I’m DevOps Engineer!</h1>
    </body>
</html>
```

- Собираем образ
```bash
vk@vk-desktop:~/Docker/nginx$ sudo docker build -t vkuzevanov/nginx-netology:1.22.0 .
Sending build context to Docker daemon  15.87kB
Step 1/2 : FROM nginx:stable
 ---> 1b84ed9be2d4
Step 2/2 : COPY index.html /usr/share/nginx/html/
 ---> 2d96954b90cf
Successfully built 2d96954b90cf
Successfully tagged vkuzevanov/nginx-netology:1.22.0
vk@vk-desktop:~/Docker/nginx$ sudo docker images
REPOSITORY                  TAG       IMAGE ID       CREATED          SIZE
vkuzevanov/nginx-netology   1.22.0    2d96954b90cf   40 seconds ago   142MB
nginx                       stable    1b84ed9be2d4   2 weeks ago      142MB
```

- Запускаем контейнер, тестируем
```bash
vk@vk-desktop:~/Docker/nginx$ sudo docker run -d --name test-netology-page -p 8080:80 vkuzevanov/nginx-netology:1.22.0
ab2d72d2106df945a317e4cf45e69ccefe52c1e070b8031455e61fd8052230ca
vk@vk-desktop:~/Docker/nginx$ curl localhost:8080
<html>
    <head>Hey, Netology</head>
    <body>
	<h1>I’m DevOps Engineer!</h1>
    </body>
</html>
vk@vk-desktop:~/Docker/nginx$ sudo docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS         PORTS                                   NAMES
ab2d72d2106d   vkuzevanov/nginx-netology:1.22.0   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   test-netology-page
```

- Push-им созданный образ
```bash
vk@vk-desktop:~/Docker/nginx$ sudo docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: vkuzevanov
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
vk@vk-desktop:~/Docker/nginx$ sudo docker push vkuzevanov/nginx-netology:1.22.0
The push refers to repository [docker.io/vkuzevanov/nginx-netology]
d1c19334752a: Pushed 
9760b00a1245: Mounted from library/nginx 
053c9e98185f: Mounted from library/nginx 
fdf60923d025: Mounted from library/nginx 
59dc71c10295: Mounted from library/nginx 
e08753aa4850: Mounted from library/nginx 
6485bed63627: Mounted from library/nginx 
1.22.0: digest: sha256:46252f177d48200f47dbcb1408fa0adf045cfdc4a293056ccaeb9122115250d0 size: 1777
```

ссылка на docker-репозиторий: (https://hub.docker.com/repository/docker/vkuzevanov/nginx-netology)

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

## Решение

- Высоконагруженное монолитное java веб-приложение;
Лучше использовать виртуальные/физические машины. Docker для этого меньше подходит, т.к. монолитное приложение тяжеловесно, имеет длительное время запуска, как правило, выполняется несколько процессов. Кроме этого, имеются зависимости, которые следует учитывать при проектировании других контейнеров. 

- Nodejs веб-приложение;
Однозначно, Docker. Простота развертывания, лёгковесность, масштабирование, простое воспроизведения зависимостей в рабочих средах

- Мобильное приложение c версиями для Android и iOS;
Виртуальные машины, проще для тестирования, размещения на одной хостовой машине

- Шина данных на базе Apache Kafka;
Docker. Есть готовые образы для Apache Kafka, + изолированность приложений, легкий возврат на стабильные версии в случае обнаружения багов

- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
При организации логирования с использованием elk-стека необходимо определить: 1. Объём логов 2. Период хранения 3. Скорость поиска.
В случае высоконагруженных систем с большими объёмами и сроками хранения логов лучше использовать физические/виртуальные сервера

- Мониторинг-стек на базе Prometheus и Grafana;
Docker. Есть готовые образы, приложения не хранят данные, что удобно при контейнеризации, удобно масштабировать, быстро разворачивать.

- MongoDB, как основное хранилище данных для java-приложения;
Лучше физические или виртуальные серверы. Сложно администрировать MongoDB внутри контейнера, есть вероятность потери данных при потере контейнера.

- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
Здесь больше подходят физические или виртуальные сервера. Docker не подходит, т.к. при потере контейнера будет сложно восстановить частоизменяемые данные. 

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.

## Решение

```bash
vk@vk-desktop:~/Docker$ sudo docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
vk@vk-desktop:~/Docker$ mkdir data

vk@vk-desktop:~/Docker$ sudo docker run -v /data:/data --name centos -dt centos
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
acf6cbda6ec97979ba981977eee128047a46c850438f048b766bee11ac5537dc

vk@vk-desktop:~/Docker$ sudo docker run -v /data:/data --name debian -dt debian
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
1671565cc8df: Pull complete 
Digest: sha256:d52921d97310d0bd48dab928548ef539d5c88c743165754c57cfad003031386c
Status: Downloaded newer image for debian:latest
65361a0d9c713adddd376daf2e8319076d51ed6eb048daaa1987dc6479f68fd0

vk@vk-desktop:~/Docker$ sudo docker ps 
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
65361a0d9c71   debian    "bash"        16 seconds ago   Up 14 seconds             debian
acf6cbda6ec9   centos    "/bin/bash"   2 minutes ago    Up 2 minutes              centos

vk@vk-desktop:~/Docker$ sudo docker exec -it centos /bin/sh 
sh-4.4# echo 'CentOS forever!' > /data/from-centos
sh-4.4# exit
exit
vk@vk-desktop:~/Docker$ touch data/from-host
vk@vk-desktop:~/Docker$ sudo docker exec -it debian /bin/sh 
# ls -l /data
total 8
-rw-r--r-- 1 root root 16 Sep  7 17:16 from-centos
-rw-r--r-- 1 root root 14 Sep  7 17:17 from-host
```




