## Домашнее задание к занятию «SQL. Часть 1»

# Задание 1
Получите уникальные названия районов из таблицы с адресами, которые начинаются на “K” и заканчиваются на “a” и не содержат пробелов.

select
distinct district
from
address
where
left (district, 1) = 'K' and right (district, 1) = 'a' and district not like '% %'
order by district;

# Задание 2
Получите из таблицы платежей за прокат фильмов информацию по платежам, которые выполнялись в промежуток с 15 июня 2005 года по 18 июня 2005 года включительно и стоимость которых превышает 10.00.

select
amount,
customer_id,
last_update,
payment_date,
payment_id,
rental_id,
staff_id
from
payment
where
date(payment_date) between 20050615 and 20050618;

либо

...
where
payment_date between 20050615000000 and 20050618235959;

# Задание 3
Получите последние пять аренд фильмов.

select
rental_date,
rental_id,
inventory_id
from
rental
order by
rental_date desc
limit
5;

# Задание 4
Одним запросом получите активных покупателей, имена которых Kelly или Willie.

Сформируйте вывод в результат таким образом:

все буквы в фамилии и имени из верхнего регистра переведите в нижний регистр, замените буквы 'll' в именах на 'pp'.

select
customer_id,
replace(lower(first_name), 'll', 'pp') as first_name,
replace(lower(last_name), 'll', 'pp') as last_name,
email,
active
from
customer
where active = 1 and first_name = 'kelly' or active = 1 and first_name = 'willie';
