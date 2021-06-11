Добавление кнопки
```sh
$ export GITHUB_USERNAME=Lam0Rka
```
Телепортируемся туда, где будем работать
```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
Это делать, уже традиция какая-то
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab777 lab888
$ cd lab888
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab888
```
Создаём докер и указываем базовый образ
```sh
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```
Говорим докеру, что нужно будет установить
```sh
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```
Копируем файлы и нашего каталога и задаём рабочую директорию для докера
```sh
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```
В докеры делаем cmake
```sh
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
Устанавливаем значение LOG_PATH
```sh
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
```
Говорим докеру где будут храниться файлы, который останутся после работы с контейнером
```sh
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
```
переходим в директорию
```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```
Создаём точку входа в приложение
```sh
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```
А вот тут у меня начались проблемы, у меня докер отказывался работать и к чему-то подключаться.
После бездумных копипаст в терминал комманд, которые должны исправить ситуацию в голову пришла идея написать sudo после чего всё удачно запустилось.
Может помогли комманды из инетрнета, может проблема изначально была в sudo, но всё заработало
Тут происходит сборка образа с тегом  logger
```sh
$ docker build -t logger .
```
Просим докер показать нам, какие существующие образы у нас есть
```sh
$ docker images
```
создаём директорию logs, создаём интеррактивный процесс logger и записываем туда текст
```sh
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3
<C-D>
```
Просим докер вывести подробную информацию о контейнере
```sh
$ docker inspect logger
```
Проверяем сохранены ли файлы
```sh
$ cat logs/log.txt
```
используем gsed
```sh
$ gsed -i 's/lab07/lab08/g' README.md
```
редактируем трэвис ямл
```sh
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```
Добавляем всё это чудо на гитхаб
```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```
Логинимся в трэвис и запускаем его
```sh
$ travis login --token
$ travis enable
```
