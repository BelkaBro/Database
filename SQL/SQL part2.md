## Домашнее задание к занятию «SQL.

# Задание 1
Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:

фамилия и имя сотрудника из этого магазина;
город нахождения магазина;
количество пользователей, закреплённых в этом магазине.

select concat(staff.last_name, ' ' ,staff.first_name) as staff_member, city.city as store_address, count(customer_id) as num_of_customers

from store

join customer on store.store_id = customer.store_id

join address on store.address_id = address.address_id

join city on address.city_id = city.city_id

join staff on store.store_id = staff.store_id

group by staff.staff_id, city.city_id

having num_of_customers > 300;

![img](
