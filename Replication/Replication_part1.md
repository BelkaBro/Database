# Домашнее задание к занятию «Репликация и масштабирование. Часть 1»

## Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

*Master-Slave*. В данном случае может существовать только один Master, а вот Slave-машин может быть несколько. Вся основная работа происходит на машине Master, с машины Slave информацию можно только считывать. Но тут, на мой взгляд, есть недостаток: если Master по какой-то причине выйдет из строя, то встанут все процессы и придется срочно переделывать какой-нибудь Slave на Master, если тот не удастся восстановить. Также из-за того, что всю информацию машины Slave берут из Master, на последний идет большая нагрузка.

*Master-Master*. Здесь же в плане отказоустойчивости дела получше. Каждый Master одновременно является Slave и наоборот - на каждую машину можно и заносить информацию, и читать. А это - уменьшение нагрузки на каждую машину. Если один Master выйдет из строя, его сразу заменит другой. Но в данной схеме могут возникнуть конфликты, если базы на нескольких Master обновляется одновременно.


## Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

*MySQL был установлен на две виртуальные машины, созданные в Яндекс-облаке. Master - 172.16.0.11, Slave - 172.16.0.16*

Файл конфигурации `/etc/mysql/mysql.conf.d/mysqld.cnf` на машине Master:

```
bind-address   = 172.16.0.11
log_error      = /var/log/mysql/error.log
server-id      = 1
log_bin        = /var/log/mysql/mysql-bin.log
```
Файл конфигурации `/etc/mysql/mysql.conf.d/mysqld.cnf` на машине Slave:

```
server-id      = 2
log_error      = /var/log/mysql/error.log
log_bin        = /var/log/mysql/mysql-bin.log
relay-log      = /var/log/mysql/mysql-relay-bin.log
```

### *Действия на Master*

`CREATE USER 'replica_user'@'172.16.0.16' IDENTIFIED WITH mysql_native_password BY '12345';`

`GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'172.16.0.16';`

`FLUSH PRIVILEGES;`

`SHOW MASTER STATUS;`

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270578679-c71d614f-7efc-47f1-ab5c-7bc40be68a91.png)


### *Действия на Slave*

`CHANGE REPLICATION SOURCE TO SOURCE_HOST='172.16.0.11', SOURCE_USER='replica_user', SOURCE_PASSWORD='12345', SOURCE_LOG_FILE='mysql-bin.000001', SOURCE_LOG_POS=2807;`

`START REPLICA;`

`SHOW REPLICA STATUS\G;`

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270579958-32793fe0-b6fa-47d4-b822-5cd64401b8ea.png)

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270580090-b9e80de5-bc91-482c-a1dd-be63952a9b19.png)

### *Действия на Master*

`SHOW DATABASES;`

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270580623-e42a25c3-a852-4f71-937b-763d4c1d2f51.png)

### *Действия на Slave*

`SHOW DATABASES;`

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270580771-563bd37c-0935-4969-9d9c-5b83f94686c3.png)

### *На Master создаю БД и проверяю*

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270581337-eca81a36-24f2-4dde-818b-1a0f5171d22d.png)

### *Проверяю репликацию на Slave, база появилась*

![img](https://github.com/BelkaBro/Database/blob/main/Replication/img/270581404-0d76cf7b-dcff-4f9f-bddb-c43bb7016638.png)
