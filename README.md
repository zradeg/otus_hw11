# Docker
 
## Задачи 
1. Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен
отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)
2. Определите разницу между контейнером и образом. Вывод опишите в домашнем задании.
3. Ответьте на вопрос: Можно ли в контейнере собрать ядро?
4. Собранный образ необходимо запушить в docker hub и дать ссылку на ваш
репозиторий.
5. Задание со * (звездочкой)
Создайте кастомные образы nginx и php, объедините их в docker-compose.
После запуска nginx должен показывать php info.
Все собранные образы должны быть в docker hub


## Выполнение

Создаю структуру директорий
```
.
├── docker #здесь докерфалй и конфиги сервисов
│   ├── nginx
│   │   ├── Dockerfile
│   │   └── etc-nginx
│   │       ├── conf.d
│   │       │   └── default.conf
│   │       └── nginx.conf
│   └── php-fpm
│       ├── Dockerfile
│       └── php-fpm.conf
├── docker-compose.yml	#собственно, docker-compose.yml
├── nginx-single	#здесь Dockerfile и конфиги для nginx без php
│   ├── Dockerfile
│   ├── default.conf
│   └── nginx.conf
└── src	#здесь скрипт, вызывающий php-info
    └── index.php
```

* Для проверки первого задания, необходимо запустить докер контейнер, находясь в директории nginx-single
* Для проверки второго задания, необходимо запустить docker-compose up, находясь в корне этой структуры.


1. Создание своего кастомного образа

Формирую докерфайл:
```
#указываю базовый образ
FROM alpine:3.11
#обновляю БД пакетного менеджера, устанавливаю nginx и очищаю кэш
RUN apk update && apk add nginx && rm -rf /var/cache/apk/*
#копирую конфиг nginx внутрь контейнера
COPY ./nginx.conf /etc/nginx/
#копирую конфиг сайта внутрь контейнера
COPY ./default.conf /etc/nginx/conf.d/
#создаю домашнюю директорию для сайта и создаю index.html
RUN mkdir -p /usr/share/nginx/html  && echo "Hello Docker" > /usr/share/nginx/html/index.html
#транслирую 80 порт из контейнера в хостовую систему
EXPOSE 80
#указываю, что будет являться процессом внутри контейнера
CMD [ "nginx" ]
```

Запускаю построение образа на основе Dockerfile, находящегося в текущей директории
```
[root@hw11-otus vagrant]# docker build -t zradeg/nginx4php .
Sending build context to Docker daemon  12.8 kB
Step 1/7 : FROM alpine:3.11
 ---> f70734b6a266
Step 2/7 : RUN apk update && apk add nginx && rm -rf /var/cache/apk/*
 ---> Running in 655ad6ac59ee

fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
v3.11.6-6-g95aea5ceff [http://dl-cdn.alpinelinux.org/alpine/v3.11/main]
v3.11.6-5-g28d0524933 [http://dl-cdn.alpinelinux.org/alpine/v3.11/community]
OK: 11270 distinct packages available
(1/2) Installing pcre (8.43-r0)
(2/2) Installing nginx (1.16.1-r6)
Executing nginx-1.16.1-r6.pre-install
Executing busybox-1.31.1-r9.trigger
OK: 7 MiB in 16 packages
 ---> 1a3101c7ccbc
Removing intermediate container 655ad6ac59ee
Step 3/7 : COPY ./nginx.conf /etc/nginx/
 ---> eaf38f8f151b
Removing intermediate container cb97180f333d
Step 4/7 : COPY ./default.conf /etc/nginx/conf.d/
 ---> 427f25193558
Removing intermediate container a68f564faad8
Step 5/7 : RUN mkdir -p /usr/share/nginx/html  && echo "Hello Docker" > /usr/share/nginx/html/index.html
 ---> Running in d0b765d59700

 ---> f23c2e14f7c8
Removing intermediate container d0b765d59700
Step 6/7 : EXPOSE 80
 ---> Running in 323087ee6a4a
 ---> 3953af442f4a
Removing intermediate container 323087ee6a4a
Step 7/7 : CMD nginx
 ---> Running in ca4a22122bdf
 ---> d16b5c85feb0
Removing intermediate container ca4a22122bdf
Successfully built d16b5c85feb0
```

Логинюсь в doker hub и отправляю туда полученный образ
```
[root@hw11-otus vagrant]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: zradeg
Password:
Login Succeeded

[root@hw11-otus vagrant]# docker push zradeg/nginx
The push refers to a repository [docker.io/zradeg/nginx]
7411241d58d1: Pushed
230c22b2515f: Pushed
e39ba9e229b6: Pushed
a9a0e5d40dea: Pushed
2de72f81bccc: Pushed
beee9f30bc1f: Mounted from library/alpine
latest: digest: sha256:2adbe14aa9d984c7c2ff290b2d6a2666316bb5b9d28c7c7ba88d03dc166f813b size: 1567
```

Запускаю контейнер
```
[root@hw11-otus vagrant]# docker run -d -p 80:80 zradeg/nginx
f550b1866bc4a72e5d42723ee4906307296e9a554a140b76bf91c6b1fa4a4f23
```

Проверяю результат
```
[root@hw11-otus vagrant]# curl http://localhost
Hello Docker
```



2. Определите разницу между контейнером и образом
Docker-образ - это шаблон, на основе которого запускается контейнер. Шаблон по своей сути неизменяем, на его основе может быть запущен один или несколько одинаковых инстансов, в процессе работы которых в них могут произойти изменения, но существовать эти изменения будут только до рестарта контейнера. В свою очередь, произведя изменения в запущенном контейнере, можно сохранить его в новый образ.


3. Можно ли в контейнере собрать ядро?
Собрать ядро можно, но использовать его для загрузки в контейнере не получится - контейнер буде запущен на основе ядра хостовой системы.


4. [https://hub.docker.com/repository/docker/zradeg/nginx](https://hub.docker.com/repository/docker/zradeg/nginx)


5. Задание со звездочкой

Аналогично заданию №1 собираю образы и отправляю их в docker hub:
```
[root@hw11-otus vagrant]# docker build -t zradeg/nginx4php ./docker/nginx/
 Sending build context to Docker daemon 6.144 kB
Step 1/6 : FROM alpine:3.11
 ---> f70734b6a266
Step 2/6 : RUN apk update && apk add nginx
 ---> Using cache
 ---> b07b99f52c6e
Step 3/6 : RUN mkdir -p /usr/share/nginx/html
 ---> Using cache
 ---> 87738e750b35
Step 4/6 : RUN ln -s /dev/stdout /var/log/nginx/error.log && ln -s /dev/stdout /var/log/nginx/access.log
 ---> Using cache
 ---> 9cbd66fb3519
Step 5/6 : EXPOSE 80
 ---> Using cache
 ---> 8550286a8186
Step 6/6 : CMD nginx -g daemon off;
 ---> Using cache
 ---> d7eb702a950a
Successfully built d7eb702a950a


[root@hw11-otus vagrant]# docker push zradeg/nginx4php
The push refers to a repository [docker.io/zradeg/nginx4php]
3a6959a5a0c6: Pushed
bb335aceb216: Pushed
b3bab3cf421c: Pushed
3e207b409db3: Pushed
latest: digest: sha256:9a11622cb9b85ae4f63b5c3c57bdc5c75d9994e67699415b604ca3b9e8a34600 size: 1153
 
 
[root@hw11-otus vagrant]# docker build -t zradeg/phpfpm5 ./docker/php-fpm/
Sending build context to Docker daemon  25.6 kB
Step 1/5 : FROM alpine:3.11
 ---> f70734b6a266
Step 2/5 : RUN apk add --update --repository https://sjc.edge.kernel.org/alpine/v3.8/main --repository https://sjc.edge.kernel.org/alpine/v3.8/community php5-fpm && rm -rf /var/cache/apk/*
 ---> Using cache
 ---> 9441b8a0e1f2
Step 3/5 : RUN ln -s /dev/stdout /var/log/php-fpm.log
 ---> Using cache
 ---> e19e1b1af52a
Step 4/5 : EXPOSE 9000
 ---> Using cache
 ---> d7f3b74984dd
Step 5/5 : CMD /usr/bin/php-fpm5 --nodaemonize
 ---> Using cache
 ---> 0864684928e7
Successfully built 0864684928e7

[root@hw11-otus vagrant]# docker push zradeg/phpfpm5
The push refers to a repository [docker.io/zradeg/phpfpm5]
4ef190f8698b: Pushed
dbec69612f4d: Pushed
3e207b409db3: Mounted from zradeg/nginx4php
latest: digest: sha256:bcd676e48d1fdcd08f393a922de93a3e17c1ab9532effef4bacf349533dcdf7c size: 946
```

Формирую docker-compose.yml
```
version: '3.7'

services:
  php-fpm:
    build:
      context: ./docker/php-fpm
    networks:
      - nginx_phpfpm
    ports:
      - "9000:9000"
    volumes:
      - ./docker/php-fpm/php-fpm.conf:/etc/php5/php-fpm.conf
      - ./src:/var/www

  nginx:
    build:
      context: ./docker/nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/etc-nginx/conf.d/:/etc/nginx/conf.d/
      - ./docker/nginx/etc-nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./src:/var/www
    depends_on:
      - php-fpm
    networks:
      - nginx_phpfpm

networks:
  nginx_phpfpm:
```

Существенные моменты, на которые может уйти неоправданно много времени:
* для php-fpm также необходимо пробросить объем www, иначе не будет работать
* в конфиге php-fpm по-умолчанию настроено прослушивание локалхоста - это необходимо исправить
* в секции описания сетей, название сети должно обязательно оканчиваться двоеточием