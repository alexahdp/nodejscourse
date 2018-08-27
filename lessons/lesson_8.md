# Занятие 8

## Теоретическая часть
 - миграции - http://docs.sequelizejs.com/manual/tutorial/migrations.html  
 - транзакции - http://docs.sequelizejs.com/manual/tutorial/transactions.html  
 - индексы - какие бывают, как строить - http://docs.sequelizejs.com/manual/tutorial/models-definition.html#indexes  
 - профилирование запросов (EXPLAIN)  


## Практическая часть

### Задание 1


### Задание 1
Создать страницу по адресу /profile, на которой будет выводиться информация о пользователе.
На странице должна быть возможность указать город проживания. В базе у пользователя должно появиться поле city. На странице же /proile должны выводиться страна, регион и город проживания.
Для этого задания нужно использовать дамп db.sql в каталоге data. Вот структура таблиц в дампе:

**city:**  
 - id_city
 - id_region
 - id
 - id_country
 - name

**region:**
 - id_region
 - id_country
 - name

**country:**
 - id_country
 - name

# TODO
в дампе кодировка cp1251, надо заменить на utf8

### Задание 2


## Homework

### Задание 1
