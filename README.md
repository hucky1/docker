## Требования

- Git.
- Docker engine 19.x и выше. 

## Возможности и особенности

- **PHP** — **8** с набором наиболее востребованных расширений. 
- Предварительно сконфигурированный веб-сервер **Nginx**.
- Базы данных:
  - **MySQL 8**.
- Настройка основных параметров окружения через файл **.env**.
- Возможность модификации сервисов через **docker-compose.yml**.
- Каталоги большинства docker-контейнеров, в которых хранятся пользовательские данные и параметры конфигурации смонтированы на локальную машину.

## Структура проекта

Рассмотрим структуру проекта.

```
├── .gitignore
├── README.md
├── docker-compose.yml
├── config
└── app
```

**Примечание:**

В некоторых каталогах можно встретить пустой файл **.gitkeep**. Он нужен лишь для того, чтобы была возможность добавить каталог под наблюдение **Git**. 

**.gitkeep** — является заполнением каталога, это фиктивный файл, на который не следует обращать внимание.

Пример файла с основными настройками среды разработки.

```dotenv
# Настройки Nginx
# Порт, который следует использовать
# для соединения с локального компьютера
NGINX_PORT=80

# Настройки общие для MySQL
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=test

# Настройки MySQL
# Порт, который следует использовать
# для соединения с локального компьютера
MYSQL_PORT=4308


# Настройки PHP 8.0
# Внешний порт, доступен с локального компьютера
PHP_PORT=9006

```

### .gitignore

Каталоги и файлы, в которых хранятся пользовательские данные, код ваших проектов и ssh-ключи внесены в .gitignore.

### README.md

Документация, которую вы сейчас читаете.

### docker-compose.yml

Документ в формате YML, в котором определены правила создания и запуска многоконтейнерных приложений Docker. 
В этом файле описана структура среды разработки и некоторые параметры необходимые для корректной работы web-приложений.


### mysql

Каталог базы данных MySQL.

```
├── conf.d
│   └── config-file.cnf
├── data
├── dump
└── logs
```

**config-file.cnf** — файл конфигурации. В этот файл можно добавлять параметры, которые при перезапуске MySQL будут применены.

**data** — эта папка предназначена для хранения пользовательских данных MySQL.

**dump** — каталог для хранения дампов.

**logs** — каталог для хранения логов.


### nginx

Эта папка предназначена для хранения файлов конфигурации Nginx и логов.

```
├── conf.d
│   ├── default.conf
│   └── vhost.conf
└── logs
```

**default.conf** — файл конфигурации, который будет применён ко всем виртуальным хостам.

**vhost.conf** — здесь хранятся настройки виртуального хоста web-проекта.

### php

Здесь хранится файл, в котором описаны действия, выполняемые при создании образов docker-контейнера PHP.

```
└── Dockerfile
```

**Dockerfile** — это текстовый документ, содержащий все команды, которые следует выполнить для сборки образов PHP.


Содержимое каталога **projects** доступно из контейнеров **php-7.1** и **php-7.3**. 

Если зайти в контейнер **php-7.1** или **php-7.3**, то в каталоге **/var/www** будут доступны проекты, которые расположены в **projects** на локальной машине.

## Программы в docker-контейнерах PHP

Полный перечень приложений, которые установлены в контейнерах **php-x.x** можно посмотреть в **php-n-workspace/Dockerfile**.

Здесь перечислим лишь некоторые, наиболее важные:

- bash
- htop
- curl
- Git
- Сomposer
- make
- wget
- NodeJS
- Supervisor
- npm


## Начало работы


**1**. Скопируйте файл **.env-example** в **.env**

**2**. Отредактируйте настройки виртуальных хостов **Nginx**.

Файл конфигурации виртуальных хостов находится в каталоге **./nginx/conf.d/**.

<br>

**3**. Настройте хосты (доменные имена) web-проектов на локальной машине. 

Необходимо добавить названия хостов web-проектов в файл **hosts** на вашем компьютере. 

В файле **hosts** следует описать связь доменных имён ваших web-проектов в среде разработки на локальном компьютере и IP docker-контейнера **Nginx**.

Строки, которые вы добавляете в этот файл, будут выглядеть примерно так:

```
127.0.0.1   app.localhost
```

В данном случае, мы исходим из того, что **Nginx**, запущенный в docker-контейнере, доступен по адресу **127.0.0.1** и web-сервер слушает порт **80**.

 
После того, как вам станет известен IP-адрес, укажите его в секции **extra_hosts** в описание сервисов **php-7.1** **php-7.3** в **docker-compose.yml**.


Именно эти параметры следует использовать для конфигурации web-проектов. 

Для соединения с базами данных с локальной машины:

- Хост для всех баз данных — **127.0.0.1**.
- Порты — значения указанные в **.env**.

<br>  
  
**4**. Создайте контейнеры и запустите их.

Выполните команду:

```shell script
docker-compose build && docker-compose up -d
```

## Вопросы и ответы

Несколько наиболее важных вопросов и ответов на них.

### Как останавливать и удалить контейнеры и другие ресурсы среды разработки, которые были созданы?

```shell script
docker-compose down
```

### Как удалить все контейнеры?

Удаление всех контейнеров:

```shell script
docker rm -v $(docker ps -aq)
```

Удаление всех активных контейнеров:

```shell script
docker rm -v $(docker ps -q) # Все активные
```

Удаление всех неактивных контейнеров:

```shell script
docker rm -v $(docker ps -aq -f status=exited) # Все неактивные
```


## Развёртывание дампов MySQL
   
Если для работы web-проектов требуются перенести данные в хранилища, то следуйте описанным ниже инструкциям.

### Как развернуть дамп MySQL?

Если требуется создать дополнительных пользователей, то следует это сделать перед началом процедуры загрузки дампа.  

В файле **mysql/conf.d/config-file.cnf** отключите лог медленных запросов **slow_query_log=0** или установите большое значение **long_query_time**, например 1000.

Если дамп сжат утилитой gzip, сначала следует распаковать архив:

```shell script
gunzip databases-dump.sql.gz
```

Затем можно развернуть дамп, выполнив на локальном компьютере команду:

```shell script
docker exec -i mysql mysql --user=root --password=secret --force < databases-dump.sql
````
Указывать пароль в командной строке — плохая практика, не делайте так в производственной среде. 

MySQL выдаст справедливое предупреждение:

>mysql: [Warning] Using a password on the command line interface can be insecure.

Ключ *--force* говорит MySQL, что ошибки следует проигнорировать и продолжить развёртывание дампа. Этот ключ иногда может пригодится, но лучше его без необходимости не применять. 
