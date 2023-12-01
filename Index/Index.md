# Домашнее задание к занятию «Индексы»

# Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```
select round (sum(index_length) / sum(data_length) * 100) as '% index'
from INFORMATION_SCHEMA.TABLES;
```

![img](https://github.com/BelkaBro/Database/blob/main/Index/img/268876762-1e04c294-f9cb-499b-9a6b-4b6fff8ee6e6.png)

# Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

```

-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=16615..16616 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=16615..16615 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=16615..16615 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=13569..16046 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=13569..14031 rows=642000 loops=1)
                    -> Stream results  (cost=22.1e+6 rows=16.3e+6) (actual time=2.2..12572 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=22.1e+6 rows=16.3e+6) (actual time=2.18..11494 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=20.5e+6 rows=16.3e+6) (actual time=2.17..8173 rows=642000 loops=1)
                                -> Nested loop inner join  (cost=18.8e+6 rows=16.3e+6) (actual time=2.15..4853 rows=642000 loops=1)
                                    -> Inner hash join (no condition)  (cost=1.61e+6 rows=16.1e+6) (actual time=2.1..476 rows=634000 loops=1)
                                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.68 rows=16086) (actual time=0.12..29.2 rows=634 loops=1)
                                            -> Table scan on p  (cost=1.68 rows=16086) (actual time=0.0485..15.9 rows=16044 loops=1)
                                        -> Hash
                                            -> Covering index scan on f using idx_title  (cost=103 rows=1000) (actual time=0.0865..1.07 rows=1000 loops=1)
                                    -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1.01) (actual time=0.00244..0.00369 rows=1.01 loops=634000)
                                -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=0.00137..0.002 rows=1 loops=642000)
                            -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=0.00135..0.00199 rows=1 loops=642000)
```

# Старый вариант

Как понимаю данный запрос выдает платежи людей, взявших в аренду фильмы за определенную дату. И в запросе, на мой взгляд, много лишней информации: например, инвентаризация (inventory_id), дата аренды (rental_date), фильмы (film). Из-за чего исходный запрос у меня получился аж на 16615 милисекунд (16 сек) и прочитанных строк вышло 642000.

Я решила пойти путем не добавления индексов (на что нужно потратить лишнее время), а путем удаления ненужной информации, что было подсказано на вебинаре. 

Запрос у меня получился следующий:

```
select distinct concat(c.last_name, ' ', c.first_name) as Клиент, sum(p.amount) over (partition by c.customer_id) as 'Общий платеж'
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id;
```

![img](https://github.com/BelkaBro/Database/blob/main/Index/img/268892838-b5dfb42c-ac7a-4dba-bd63-20426147bd14.png)

В следствие чего анализ запроса получился уже другим: время 40 милисекунд и строк прочитано 634.
```
-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=39.8..40.5 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=39.8..40.2 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=39.8..39.8 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=37.2..39.3 rows=634 loops=1)
                -> Sort: c.customer_id  (actual time=37.1..37.6 rows=634 loops=1)
                    -> Stream results  (cost=7263 rows=16086) (actual time=0.176..36.5 rows=634 loops=1)
                        -> Nested loop inner join  (cost=7263 rows=16086) (actual time=0.166..35.3 rows=634 loops=1)
                            -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1633 rows=16086) (actual time=0.139..30.9 rows=634 loops=1)
                                -> Table scan on p  (cost=1633 rows=16086) (actual time=0.0675..17.1 rows=16044 loops=1)
                            -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=0.00259..0.00336 rows=1 loops=634)
```

# Новый вариант

Создан индекс для даты платежа

```
create index day_of_payment on payment(payment_date);
```

Переделан запрос с объединением таблиц. Таблица film не используется.

```
SELECT concat(c.last_name, ' ', c.first_name) AS Клиент, SUM(p.amount) as Платеж
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id 
JOIN payment p ON r.rental_date = p.payment_date 
join inventory i on i.inventory_id = r.inventory_id 
where date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY c.customer_id;
````

Вывод анализа. Скорость обработки с индексом стала еще быстрее по сравнению с самым первым моим варинатом.

```
-> Limit: 200 row(s)  (cost=12152 rows=190) (actual time=1.62..121 rows=200 loops=1)
    -> Group aggregate: sum(p.amount)  (cost=12152 rows=190) (actual time=1.61..120 rows=200 loops=1)
        -> Nested loop inner join  (cost=12133 rows=190) (actual time=0.987..120 rows=317 loops=1)
            -> Nested loop inner join  (cost=8042 rows=187) (actual time=0.174..76.1 rows=7694 loops=1)
                -> Nested loop inner join  (cost=4021 rows=187) (actual time=0.162..28.1 rows=7694 loops=1)
                    -> Index scan on c using PRIMARY  (cost=0.0228 rows=7) (actual time=0.0526..0.45 rows=284 loops=1)
                    -> Index lookup on r using idx_fk_customer_id (customer_id=c.customer_id)  (cost=6.69 rows=26.7) (actual time=0.0356..0.0598 rows=27.1 loops=284)
                -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.25 rows=1) (actual time=0.00226..0.00292 rows=1 loops=7694)
            -> Index lookup on p using day_of_payment (payment_date=r.rental_date), with index condition: ((cast(p.payment_date as date) >= '2005-07-30') and (cast(p.payment_date as date) < <cache>(('2005-07-30' + interval 1 day))))  (cost=0.254 rows=1.02) (actual time=0.00352..0.00357 rows=0.0412 loops=7694)
```

![img](https://github.com/BelkaBro/Database/blob/main/Index/img/269170668-898e8418-59cd-49cc-a3c7-907566eb6bb8.png)

Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

# В PostgreSQL:
— B-дерево
— Хеш-индекс
— GiST (Generalized Inverted Storage Tree)
— GIN (Generalized Inverted Index)
— BRIN (Block Range INdex)
— Bloom
# В MySQL:
— B-дерево
— Хеш-индекс
— GiST
— GIN
— BRIN
Однако, MySQL поддерживает более широкий набор типов данных, чем PostgreSQL, включая DECIMAL, FLOAT и DOUBLE, а также поддерживает пользовательские типы данных. Кроме того, MySQL поддерживает индексы выражений и индексы по столбцам представлений.
