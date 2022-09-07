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


```bash
iva@c8:~/test $ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
iva@c8:~/test $ docker run -v /data:/data -dt --name centos centos
d901a793a565cb15f52d4f0d58d8b64706661e05a974d9367ca031479280971c
iva@c8:~/test $ docker run -v /data:/data -dt --name debian debian
2ebd47b23bd07a3ac8147ef0ccba7226ccd5b30242cdd31b741fdf089f702bec
iva@c8:~/test $ docker exec -it centos /bin/sh
sh-4.4# echo 'It from centos'>/data/from-centos
sh-4.4# exit
exit
iva@c8:~/test $ docker exec -it debian /bin/sh
# cat /data/from-centos
It from centos
# exit
iva@c8:~/test $ docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
2ebd47b23bd0   debian    "bash"        2 minutes ago   Up 2 minutes             debian
d901a793a565   centos    "/bin/bash"   2 minutes ago   Up 2 minutes             centos
iva@c8:~/test $ 
iva@c8:~ $ echo 'from localhost'>/data/localhost
iva@c8:~ $ docker exec -it debian /bin/sh
# ls -la /data
total 8
drwxrwxrwx. 2 root root 42 Jan 30 00:20 .
drwxr-xr-x. 1 root root 18 Jan 30 00:17 ..
-rw-rw-r--. 1 1000 1000 15 Jan 30 00:18 from-centos
-rw-rw-r--. 1 1000 1000 15 Jan 30 00:20 localhost
# exit
iva@c8:~ $ docker stop $(docker ps -a -q)
2ebd47b23bd0
d901a793a565
iva@c8:~ $ docker rm $(docker ps -a -q)
2ebd47b23bd0
d901a793a565
```



## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

процесс создания и публикации:

<details>
     <summary>Решение Задачи *</summary>
    <br>


```bash
iva@c8:~/Documents/docker/ansible $ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
iva@c8:~/Documents/docker/ansible $ docker build -t  egerpro/ansible:2.10.7 .
Sending build context to Docker daemon   2.56kB
Step 1/5 : FROM alpine:3.15
3.15: Pulling from library/alpine
59bf1c3509f3: Already exists 
Digest: sha256:21a3deaa0d32a8057914f36584b5288d2e5ecc984380bc0118285c70fa8c9300
Status: Downloaded newer image for alpine:3.15
 ---> c059bfaa849c
Step 2/5 : RUN CARGO_NET_GIT_FETCH_WITH_CLI=1 &&     apk --no-cache add         sudo         python3        py3-pip         openssl         ca-certificates         sshpass         openssh-client         rsync         git &&     apk --no-cache add --virtual build-dependencies         python3-dev         libffi-dev         musl-dev         gcc         cargo         openssl-dev         libressl-dev         build-base &&     pip install --upgrade pip wheel &&     pip install --upgrade cryptography cffi &&     pip install ansible==2.10.7 &&     pip install mitogen ansible-lint jmespath &&     pip install --upgrade pywinrm &&     apk del build-dependencies &&     rm -rf /var/cache/apk/* &&     rm -rf /root/.cache/pip &&     rm -rf /root/.cargo
 ---> Running in 9a6998f13796
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/57) Installing ca-certificates (20211220-r0)
(2/57) Installing brotli-libs (1.0.9-r5)
(3/57) Installing nghttp2-libs (1.46.0-r0)
(4/57) Installing libcurl (7.80.0-r0)
(5/57) Installing expat (2.4.3-r0)
(6/57) Installing pcre2 (10.39-r0)
(7/57) Installing git (2.34.1-r0)
(8/57) Installing openssh-keygen (8.8_p1-r1)
(9/57) Installing ncurses-terminfo-base (6.3_p20211120-r0)
(10/57) Installing ncurses-libs (6.3_p20211120-r0)
(11/57) Installing libedit (20210910.3.1-r0)
(12/57) Installing openssh-client-common (8.8_p1-r1)
(13/57) Installing openssh-client-default (8.8_p1-r1)
(14/57) Installing openssl (1.1.1l-r8)
(15/57) Installing libbz2 (1.0.8-r1)
(16/57) Installing libffi (3.4.2-r1)
(17/57) Installing gdbm (1.22-r0)
(18/57) Installing xz-libs (5.2.5-r0)
(19/57) Installing libgcc (10.3.1_git20211027-r0)
(20/57) Installing libstdc++ (10.3.1_git20211027-r0)
(21/57) Installing mpdecimal (2.5.1-r1)
(22/57) Installing readline (8.1.1-r0)
(23/57) Installing sqlite-libs (3.36.0-r0)
(24/57) Installing python3 (3.9.7-r4)
(25/57) Installing py3-appdirs (1.4.4-r2)
(26/57) Installing py3-certifi (2020.12.5-r1)
(27/57) Installing py3-charset-normalizer (2.0.7-r0)
(28/57) Installing py3-idna (3.3-r0)
(29/57) Installing py3-urllib3 (1.26.7-r0)
(30/57) Installing py3-requests (2.26.0-r1)
(31/57) Installing py3-msgpack (1.0.2-r1)
(32/57) Installing py3-lockfile (0.12.2-r4)
(33/57) Installing py3-cachecontrol (0.12.10-r0)
(34/57) Installing py3-colorama (0.4.4-r1)
(35/57) Installing py3-contextlib2 (21.6.0-r1)
(36/57) Installing py3-distlib (0.3.3-r0)
(37/57) Installing py3-distro (1.6.0-r0)
(38/57) Installing py3-six (1.16.0-r0)
(39/57) Installing py3-webencodings (0.5.1-r4)
(40/57) Installing py3-html5lib (1.1-r1)
(41/57) Installing py3-parsing (2.4.7-r2)
(42/57) Installing py3-packaging (20.9-r1)
(43/57) Installing py3-tomli (1.2.2-r0)
(44/57) Installing py3-pep517 (0.12.0-r0)
(45/57) Installing py3-progress (1.6-r0)
(46/57) Installing py3-retrying (1.3.3-r2)
(47/57) Installing py3-ordered-set (4.0.2-r2)
(48/57) Installing py3-setuptools (52.0.0-r4)
(49/57) Installing py3-toml (0.10.2-r2)
(50/57) Installing py3-pip (20.3.4-r1)
(51/57) Installing libacl (2.2.53-r0)
(52/57) Installing lz4-libs (1.9.3-r1)
(53/57) Installing popt (1.18-r0)
(54/57) Installing zstd-libs (1.5.0-r0)
(55/57) Installing rsync (3.2.3-r5)
(56/57) Installing sshpass (1.09-r0)
(57/57) Installing sudo (1.9.8_p2-r1)
Executing busybox-1.34.1-r3.trigger
Executing ca-certificates-20211220-r0.trigger
OK: 97 MiB in 71 packages
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/35) Upgrading libcrypto1.1 (1.1.1l-r7 -> 1.1.1l-r8)
(2/35) Upgrading libssl1.1 (1.1.1l-r7 -> 1.1.1l-r8)
(3/35) Installing pkgconf (1.8.0-r0)
(4/35) Installing python3-dev (3.9.7-r4)
(5/35) Installing linux-headers (5.10.41-r0)
(6/35) Installing libffi-dev (3.4.2-r1)
(7/35) Installing musl-dev (1.2.2-r7)
(8/35) Installing binutils (2.37-r3)
(9/35) Installing libgomp (10.3.1_git20211027-r0)
(10/35) Installing libatomic (10.3.1_git20211027-r0)
(11/35) Installing libgphobos (10.3.1_git20211027-r0)
(12/35) Installing gmp (6.2.1-r1)
(13/35) Installing isl22 (0.22-r0)
(14/35) Installing mpfr4 (4.1.0-r0)
(15/35) Installing mpc1 (1.2.1-r0)
(16/35) Installing gcc (10.3.1_git20211027-r0)
(17/35) Installing rust-stdlib (1.56.1-r0)
(18/35) Installing libxml2 (2.9.12-r2)
(19/35) Installing llvm12-libs (12.0.1-r0)
(20/35) Installing rust (1.56.1-r0)
(21/35) Installing cargo (1.56.1-r0)
(22/35) Installing openssl-dev (1.1.1l-r8)
(23/35) Installing libressl3.4-libcrypto (3.4.1-r0)
(24/35) Installing libressl3.4-libssl (3.4.1-r0)
(25/35) Installing libressl3.4-libtls (3.4.1-r0)
(26/35) Installing libressl-dev (3.4.1-r0)
(27/35) Installing libmagic (5.41-r0)
(28/35) Installing file (5.41-r0)
(29/35) Installing libc-dev (0.7.2-r3)
(30/35) Installing g++ (10.3.1_git20211027-r0)
(31/35) Installing make (4.3-r0)
(32/35) Installing fortify-headers (1.1-r1)
(33/35) Installing patch (2.7.6-r7)
(34/35) Installing build-base (0.5-r2)
(35/35) Installing build-dependencies (20220130.005824)
Executing busybox-1.34.1-r3.trigger
OK: 1079 MiB in 104 packages
Requirement already satisfied: pip in /usr/lib/python3.9/site-packages (20.3.4)
Collecting pip
  Downloading pip-21.3.1-py3-none-any.whl (1.7 MB)
Collecting wheel
  Downloading wheel-0.37.1-py2.py3-none-any.whl (35 kB)
Installing collected packages: wheel, pip
  Attempting uninstall: pip
    Found existing installation: pip 20.3.4
    Uninstalling pip-20.3.4:
      Successfully uninstalled pip-20.3.4
Successfully installed pip-21.3.1 wheel-0.37.1
Collecting cryptography
  Downloading cryptography-36.0.1-cp36-abi3-musllinux_1_1_x86_64.whl (3.8 MB)
Collecting cffi
  Downloading cffi-1.15.0.tar.gz (484 kB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting pycparser
  Downloading pycparser-2.21-py2.py3-none-any.whl (118 kB)
Building wheels for collected packages: cffi
  Building wheel for cffi (setup.py): started
  Building wheel for cffi (setup.py): finished with status 'done'
  Created wheel for cffi: filename=cffi-1.15.0-cp39-cp39-linux_x86_64.whl size=429250 sha256=4f73a0bbc791831c845e47e07645d9e58da220bb565101a31e7003a6ae56327d
  Stored in directory: /root/.cache/pip/wheels/8e/0d/16/77c97b85a9f559c5412c85c129a2bae07c771d31e1beb03c40
Successfully built cffi
Installing collected packages: pycparser, cffi, cryptography
Successfully installed cffi-1.15.0 cryptography-36.0.1 pycparser-2.21
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Collecting ansible==2.10.7
  Downloading ansible-2.10.7.tar.gz (29.9 MB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting ansible-base<2.11,>=2.10.5
  Downloading ansible-base-2.10.16.tar.gz (6.1 MB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting jinja2
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting PyYAML
  Downloading PyYAML-6.0.tar.gz (124 kB)
  Installing build dependencies: started
  Installing build dependencies: finished with status 'done'
  Getting requirements to build wheel: started
  Getting requirements to build wheel: finished with status 'done'
  Preparing metadata (pyproject.toml): started
  Preparing metadata (pyproject.toml): finished with status 'done'
Requirement already satisfied: cryptography in /usr/lib/python3.9/site-packages (from ansible-base<2.11,>=2.10.5->ansible==2.10.7) (36.0.1)
Requirement already satisfied: packaging in /usr/lib/python3.9/site-packages (from ansible-base<2.11,>=2.10.5->ansible==2.10.7) (20.9)
Requirement already satisfied: cffi>=1.12 in /usr/lib/python3.9/site-packages (from cryptography->ansible-base<2.11,>=2.10.5->ansible==2.10.7) (1.15.0)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp39-cp39-musllinux_1_1_x86_64.whl (30 kB)
Requirement already satisfied: pycparser in /usr/lib/python3.9/site-packages (from cffi>=1.12->cryptography->ansible-base<2.11,>=2.10.5->ansible==2.10.7) (2.21)
Building wheels for collected packages: ansible, ansible-base, PyYAML
  Building wheel for ansible (setup.py): started
  Building wheel for ansible (setup.py): finished with status 'done'
  Created wheel for ansible: filename=ansible-2.10.7-py3-none-any.whl size=48212991 sha256=09a4b441bf3ea524659e6be65f7119783ef5ad91062e4406d0c39b171cf1b0eb
  Stored in directory: /root/.cache/pip/wheels/d6/46/93/27a3ddde47e17cc3cb9d04ce9661b9cd39a186af92c7c48592
  Building wheel for ansible-base (setup.py): started
  Building wheel for ansible-base (setup.py): finished with status 'done'
  Created wheel for ansible-base: filename=ansible_base-2.10.16-py3-none-any.whl size=1880285 sha256=1bc82b9e5d00c01026d5e6dcb13b4d019e9319797115accecf4aac47360cf6a1
  Stored in directory: /root/.cache/pip/wheels/1f/e8/91/dab76006726209b0d1063b395df54b43675d95ee3441350bae
  Building wheel for PyYAML (pyproject.toml): started
  Building wheel for PyYAML (pyproject.toml): finished with status 'done'
  Created wheel for PyYAML: filename=PyYAML-6.0-cp39-cp39-linux_x86_64.whl size=45332 sha256=b01cdcefdea6120755182f4f3ec34b71244cda999f435f408c8c1dfd10995b51
  Stored in directory: /root/.cache/pip/wheels/b4/0f/9a/d6af48581dda678920fccfb734f5d9f827c6ed5b4074c7eda8
Successfully built ansible ansible-base PyYAML
Installing collected packages: MarkupSafe, PyYAML, jinja2, ansible-base, ansible
Successfully installed MarkupSafe-2.0.1 PyYAML-6.0 ansible-2.10.7 ansible-base-2.10.16 jinja2-3.0.3
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Collecting mitogen
  Downloading mitogen-0.3.2-py2.py3-none-any.whl (288 kB)
Collecting ansible-lint
  Downloading ansible_lint-5.3.2-py3-none-any.whl (115 kB)
Collecting jmespath
  Downloading jmespath-0.10.0-py2.py3-none-any.whl (24 kB)
Requirement already satisfied: packaging in /usr/lib/python3.9/site-packages (from ansible-lint) (20.9)
Collecting wcmatch>=7.0
  Downloading wcmatch-8.3-py3-none-any.whl (42 kB)
Requirement already satisfied: pyyaml in /usr/lib/python3.9/site-packages (from ansible-lint) (6.0)
Collecting tenacity
  Downloading tenacity-8.0.1-py3-none-any.whl (24 kB)
Collecting enrich>=1.2.6
  Downloading enrich-1.2.7-py3-none-any.whl (8.7 kB)
Collecting rich>=9.5.1
  Downloading rich-11.1.0-py3-none-any.whl (216 kB)
Collecting ruamel.yaml<1,>=0.15.37
  Downloading ruamel.yaml-0.17.20-py3-none-any.whl (109 kB)
Collecting commonmark<0.10.0,>=0.9.0
  Downloading commonmark-0.9.1-py2.py3-none-any.whl (51 kB)
Requirement already satisfied: colorama<0.5.0,>=0.4.0 in /usr/lib/python3.9/site-packages (from rich>=9.5.1->ansible-lint) (0.4.4)
Collecting pygments<3.0.0,>=2.6.0
  Downloading Pygments-2.11.2-py3-none-any.whl (1.1 MB)
Collecting ruamel.yaml.clib>=0.2.6
  Downloading ruamel.yaml.clib-0.2.6.tar.gz (180 kB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting bracex>=2.1.1
  Downloading bracex-2.2.1-py3-none-any.whl (12 kB)
Building wheels for collected packages: ruamel.yaml.clib
  Building wheel for ruamel.yaml.clib (setup.py): started
  Building wheel for ruamel.yaml.clib (setup.py): finished with status 'done'
  Created wheel for ruamel.yaml.clib: filename=ruamel.yaml.clib-0.2.6-cp39-cp39-linux_x86_64.whl size=745784 sha256=21f586cfb529b27b8c9b1a602737e8df2d0d1fb32cb2b72ab7490fb4a8b23c65
  Stored in directory: /root/.cache/pip/wheels/b1/c4/5d/d96e5c09189f4d6d2a9ffb0d7af04ee06d11a20f613f5f3496
Successfully built ruamel.yaml.clib
Installing collected packages: pygments, commonmark, ruamel.yaml.clib, rich, bracex, wcmatch, tenacity, ruamel.yaml, enrich, mitogen, jmespath, ansible-lint
Successfully installed ansible-lint-5.3.2 bracex-2.2.1 commonmark-0.9.1 enrich-1.2.7 jmespath-0.10.0 mitogen-0.3.2 pygments-2.11.2 rich-11.1.0 ruamel.yaml-0.17.20 ruamel.yaml.clib-0.2.6 tenacity-8.0.1 wcmatch-8.3
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Collecting pywinrm
  Downloading pywinrm-0.4.2-py2.py3-none-any.whl (44 kB)
Requirement already satisfied: requests>=2.9.1 in /usr/lib/python3.9/site-packages (from pywinrm) (2.26.0)
Collecting xmltodict
  Downloading xmltodict-0.12.0-py2.py3-none-any.whl (9.2 kB)
Collecting requests-ntlm>=0.3.0
  Downloading requests_ntlm-1.1.0-py2.py3-none-any.whl (5.7 kB)
Requirement already satisfied: six in /usr/lib/python3.9/site-packages (from pywinrm) (1.16.0)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (1.26.7)
Requirement already satisfied: certifi>=2017.4.17 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (2020.12.5)
Requirement already satisfied: charset_normalizer~=2.0.0 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (2.0.7)
Requirement already satisfied: idna<4,>=2.5 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (3.3)
Requirement already satisfied: cryptography>=1.3 in /usr/lib/python3.9/site-packages (from requests-ntlm>=0.3.0->pywinrm) (36.0.1)
Collecting ntlm-auth>=1.0.2
  Downloading ntlm_auth-1.5.0-py2.py3-none-any.whl (29 kB)
Requirement already satisfied: cffi>=1.12 in /usr/lib/python3.9/site-packages (from cryptography>=1.3->requests-ntlm>=0.3.0->pywinrm) (1.15.0)
Requirement already satisfied: pycparser in /usr/lib/python3.9/site-packages (from cffi>=1.12->cryptography>=1.3->requests-ntlm>=0.3.0->pywinrm) (2.21)
Installing collected packages: ntlm-auth, xmltodict, requests-ntlm, pywinrm
Successfully installed ntlm-auth-1.5.0 pywinrm-0.4.2 requests-ntlm-1.1.0 xmltodict-0.12.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: Ignoring https://dl-cdn.alpinelinux.org/alpine/v3.15/main: No such file or directory
WARNING: Ignoring https://dl-cdn.alpinelinux.org/alpine/v3.15/community: No such file or directory
(1/33) Purging build-dependencies (20220130.005824)
(2/33) Purging python3-dev (3.9.7-r4)
(3/33) Purging libffi-dev (3.4.2-r1)
(4/33) Purging linux-headers (5.10.41-r0)
(5/33) Purging cargo (1.56.1-r0)
(6/33) Purging rust (1.56.1-r0)
(7/33) Purging rust-stdlib (1.56.1-r0)
(8/33) Purging openssl-dev (1.1.1l-r8)
(9/33) Purging libressl-dev (3.4.1-r0)
(10/33) Purging libressl3.4-libssl (3.4.1-r0)
(11/33) Purging libressl3.4-libtls (3.4.1-r0)
(12/33) Purging build-base (0.5-r2)
(13/33) Purging file (5.41-r0)
(14/33) Purging g++ (10.3.1_git20211027-r0)
(15/33) Purging gcc (10.3.1_git20211027-r0)
(16/33) Purging binutils (2.37-r3)
(17/33) Purging libatomic (10.3.1_git20211027-r0)
(18/33) Purging libgomp (10.3.1_git20211027-r0)
(19/33) Purging libgphobos (10.3.1_git20211027-r0)
(20/33) Purging make (4.3-r0)
(21/33) Purging libc-dev (0.7.2-r3)
(22/33) Purging musl-dev (1.2.2-r7)
(23/33) Purging fortify-headers (1.1-r1)
(24/33) Purging patch (2.7.6-r7)
(25/33) Purging pkgconf (1.8.0-r0)
(26/33) Purging mpc1 (1.2.1-r0)
(27/33) Purging mpfr4 (4.1.0-r0)
(28/33) Purging isl22 (0.22-r0)
(29/33) Purging gmp (6.2.1-r1)
(30/33) Purging llvm12-libs (12.0.1-r0)
(31/33) Purging libxml2 (2.9.12-r2)
(32/33) Purging libressl3.4-libcrypto (3.4.1-r0)
(33/33) Purging libmagic (5.41-r0)
Executing busybox-1.34.1-r3.trigger
OK: 97 MiB in 71 packages
Removing intermediate container 9a6998f13796
 ---> 74b411380e5a
Step 3/5 : RUN mkdir /ansible &&     mkdir -p /etc/ansible &&     echo 'localhost' > /etc/ansible/hosts
 ---> Running in 0e86c4905174
Removing intermediate container 0e86c4905174
 ---> 41ac29e67793
Step 4/5 : WORKDIR /ansible
 ---> Running in 99ce71a3d5be
Removing intermediate container 99ce71a3d5be
 ---> f10052f8e553
Step 5/5 : CMD [ "ansible-playbook", "--version" ]
 ---> Running in 1b2a5b56d18f
Removing intermediate container 1b2a5b56d18f
 ---> b753e717a593
Successfully built b753e717a593
Successfully tagged egerpro/ansible:2.10.7
iva@c8:~/Documents/docker/ansible $ docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
egerpro/ansible   2.10.7    b753e717a593   7 seconds ago   390MB
alpine            3.15      c059bfaa849c   2 months ago    5.59MB
iva@c8:~/Documents/docker/ansible $ 
iva@c8:~/Documents/docker/ansible $ docker push egerpro/ansible:2.10.7 
The push refers to repository [docker.io/egerpro/ansible]
72a473e767ed: Pushed 
326a93401de2: Pushed 
8d3ac3489996: Mounted from library/alpine 
2.10.7: digest: sha256:5b5eec10471298e0d5437707ca30786e6c9a86580bb702d2a3f0cd4748a952c7 size: 947
iva@c8:~/Documents/docker/ansible $ 
```
</details>


https://hub.docker.com/repository/docker/egerpro/ansible

---
