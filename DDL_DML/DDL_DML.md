## Домашнее задание к занятию «Работа с данными (DDL/DML)»

# Задание 1
1.1. Поднимите чистый инстанс MySQL версии 8.0+. Можно использовать локальный сервер или контейнер Docker.
1.2. Создайте учётную запись sys_temp.

MySQL запускается в контейнере, образ mysql:8.0.34-debian

mysqlsh -h localhost -u root -P 3306

\option --persist defaultMode sql \sql CREATE USER sys_temp IDENTIFIED BY '123'; SELECT user,host FROM mysql.user; GRANT ALL PRIVILEGES ON . TO sys_temp; SHOW GRANTS FOR sys_temp; \quit

mysqlsh -h localhost -u sys_temp -P 3306

mysqlsh -h localhost -u sys_temp -P 3306 < ./sakila-db/sakila-schema.sql mysqlsh -h localhost -u sys_temp -P 3306 < ./sakila-db/sakila-data.sql

1.3. Выполните запрос на получение списка пользователей в базе данных. (скриншот)

![img](https://github.com/BelkaBro/Database/blob/main/DDL_DML/img/268452287-0cf25caf-101d-4acd-9e86-6e260d4ea6e0.png)


