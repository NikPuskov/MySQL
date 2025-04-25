# MySQL

Цель домашнего задания:

Настроить репликацию MySQL

Описание домашнего задания:

Развернуть базу данных на мастере, и реплицировать на слейв таблицы bookmaker, competition, market, odds, outcome. Репликация должна выполняться с помощью GTID

Выполнение домашнего задания:

- Поднимаем две виртуальные машины - master и slave. Master - основная БД, slave - реплика с master'а:

`vagrant up`

- Устанавливаем на обе машины Percona Server for MySQL 5.7

`sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo`

`sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo`

`sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo`

`yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm`

`percona-release setup ps57`

`yum install Percona-Server-server-57`

- Те же действия повторяем на slave.

По умолчанию Percona хранит файлы в таком виде:

1 основной конфиг в /etc/my.cnf

2 так же инклюдится директория /etc/my.cnf.d/ - куда мы и будем складывать наши конфиги.

3 data файлы в /var/lib/mysql

- Копируем заранее подготовленные конфиги:

`cp /vagrant/conf.d/* /etc/my.cnf.d/`

- Запускаем службу MySQL:

![Image alt](https://github.com/NikPuskov/MySQL/blob/main/mysql1.jpg)

- Те же действия выполняем на slave.

- При установке Percona автоматически генерирует пароль для пользователя root и кладет его в файл /var/log/mysqld.log:

![Image alt](https://github.com/NikPuskov/MySQL/blob/main/mysql2.jpg)

- Меняем пароль:

[root@master ~]# `mysql -uroot -p'h&Hp,1oAav(7'`

mysql> `ALTER USER USER() IDENTIFIED BY 'AsdQaz123$';`

Query OK, 0 rows affected (0.00 sec)

- Репликацию будем настраивать с использованием GTID. GTID представляет собой уникальный 128-битный глобальный идентификационный номер (SERVER_UUID), который увеличивается с каждой новой транзакцией.

- Следует обратить внимание, что атрибут server-id на мастер-сервере должен обязательно отличаться от server-id слейв-сервера. Проверить какая переменная установлена на текущий момент можно следующим образом:

![Image alt](https://github.com/NikPuskov/MySQL/blob/main/mysql3.jpg)

- Параметр server-id у нас прописан в конфигурационном файле etc/my.cnf.d/01-base.cnf. Соответственно на slave его нужно изменить и перезагрузить mysql.

- Убеждаемся что GTID включён:

![Image alt](https://github.com/NikPuskov/MySQL/blob/main/mysql4.jpg)

- Создадим тестовую базу bet на master'е, загрузим в нее дамп и проверим:

mysql> `CREATE DATABASE bet;`

Query OK, 1 row affected (0.00 sec)

`mysql -uroot -p -D bet < /vagrant/bet.dmp`

![Image alt](https://github.com/NikPuskov/MySQL/blob/main/mysql5.jpg)

- Создадим пользователя для репликации и дадим ему права на эту самую репликацию:

![Image alt](https://github.com/NikPuskov/MySQL/blob/main/mysql6.jpg)

- Дампим базу для последующего залива на slave и игнорируем таблицы по заданию:

`mysqldump --all-databases --triggers --routines --master-data --ignore-table=bet.events_on_demand --ignore-table=bet.v_same_event -uroot -p > master.sql`

- На слейве раскомментируем в /etc/my.cnf.d/05-binlog.cnf строки:

#replicate-ignore-table=bet.events_on_demand

#replicate-ignore-table=bet.v_same_event

- Таким образом указываем таблицы которые будут игнорироваться при репликации.

- Заливаем на слейв дамп мастера и убеждаемся что база есть и она без лишних таблиц:

image10

- Видим что таблиц v_same_event и events_on_demand нет.

- Подключаем и запускаем slave:

image11

- Видно что репликация работает, gtid работает и игнорятся таблички по заданию.

- Проверим репликацию в действии. На мастере:

image12

- На слейве:

image13

- В binlog-ах на cлейве также видно последнее изменение, туда же он пишет информацию о GTID:

image14
