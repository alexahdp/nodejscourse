# Занятие 7

## Реляционная БД

### Термины
- тип данных. Аналогия типа данных из ЯП
- ячейка. Минимальная единица данных. Аналог переменой из ЯП.
- колонка. Набор ячеек одного типа.
- запись. Набор ячеек экземпляра одной сущности.
- таблица. Набор записей.
- запрос. Текс определеной структуры (SQL), описывающий команду для обработки данных. Разные типы команд имеют разных синтаксис и могут как возвращать результат (таблицу), так и нет
- драйвер. Как правило, бд имеет бинарный протокол, используя который можно общаться с бд (отправлять запросы и получать в ответ данные). Аналог драйвера в ОС
- ORM. Объектно-ориентированная модель. Способ работы с данными релиационной бд как с экземплярами объектов/классов определённой схемы (стуктуры).

Таблица с данными пользователей

|  id | username |   email   |      dateCreate     |
|---|---|---|---|
| 345 | Паша     | p@mail.ru | 2006-01-11 07:56:20 |
| 346 | Саня     | s@mail.ru | 2018-08-13 19:17:04 |
| 347 | Димарик  | d@mail.ru | 2018-08-14 00:14:12 |


Эти же данные на стороне NodeJS:
```js
[
  {
    id: 345,
    username: 'Паша',
    email: 'p@mail.ru',
    dateCreate: '2006-01-11 07:56:20'
  },
  {
    id: 346,
    username: 'Саня',
    email: 's@mail.ru',
    dateCreate: '2018-08-13 19:17:04'
  },
  {
    id: 347,
    username: 'Димарик',
    email: 'd@mail.ru',
    dateCreate: '2018-08-14 00:14:12'
  }
]
```

### Установка
Будем использовать [MariaDB](https://mariadb.com/kb/en/library/installing-and-using-mariadb-via-docker/) - форк знаменийто MySQL.


Скачиваем образ последней стабильной версии
```sh
docker pull mariadb:latest
```


Запускаем контейнер командой
```sh
docker run --restart=always --name mariadb_test -v /mnt/mysql:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD='mypass' -p 3306:3306 -d mariadb -innodb_buffer_pool_size=1G
```
Что тут написано
* запустить контейнер c именем `mariadb_test`
* рестартовать контейнер при выключении системы `–restart=always`
* проброс директории в контейнер, чтобы при перезапуске контейнера не потерять данные в контейнере `-v /mnt/mysql:/var/lib/mysql`
* сразу задаем пароль `-e MYSQL_ROOT_PASSWORD='mypass'`
* указываем порт, по которому будет снаружи доступна база `-p 3306:3306`. Первая цифра 3306 - это мы используем для доступа снаружи контейнера, например указываем в драйвере в NodeJS. Второй порт 3306 - база должна слушать внутри контейнера. Т.е. мы можем запустить хоть 10 контейнеров с базой и они будут прекрасно работать.
* запусить в режиме демона `-d`
* имя образа `mariadb`. Можно указать конкретную версию базы через `:`
* опция уже самого процесса базы `-innodb_buffer_pool_size=1G`. Т.е. уже при запуске контейнера можно указывать настройки запуска и не нужно лезь внутрь контейнера


### Подключение через консоль
Выполнив команду
```bash
mysql -uroot --password -p 3306 --protocol=TCP
```
мы попадаем в режим командой строки уже самой базы. Как будто запустили интерпретатор NodeJS, только для базы данных


### Подключение через NodeJs
Будем использовать [mysql](https://github.com/mysqljs/mysql) - одноименный драйвер:

```bash
npm i --save mysql
```

Создадим файл `model/db.js` со следующим содержимым:
```js
const mysql      = require('mysql');

const sql = mysql.createConnection({
  host     : configl.db.host,
  user     : configl.db.user,
  password : configl.db.password,
  database : configl.db.database
});

sql.connect();


module.exports = { sql };
```

Теперь в любом месте проекта пишем
```javascript
const { sql } = require('model/db');
sql.query('SELECT 1 + 1 AS solution', function (error, results, fields) {
  if (error) throw error;
  console.log('The solution is: ', results[0].solution);
});
```


### Пулл коннектор
Код создания коннекта из предыдущего примера имеет существенные ограничения:
<details>
  <summary>Ответ</summary>
  параметры подключения захардкожены в файле, нет закрытия коннекта, нельзя делать несколько одновременных запроса
</details>

За нас уже подумали разработчики MariaDB и авторы драйвера, поэтому файл `model/db.js`
нужно поменять на:
```javascript
const mysql      = require('mysql');
var sql  = mysql.createPool({
  connectionLimit : 10,
  host     : configl.db.host,
  user     : configl.db.user,
  password : configl.db.password,
  database : configl.db.database
});

module.exports = { sql };
```

Теперь при вызове pool.query будут вызваны несколько действий:
* sql.getConnection(). Он получит объект connection из пулла коннектов. Если свободных коннектор нет, будет ждать пока появится.
* connection.query(). Собвственно выполнение запроса
* connection.release(). Освобождение запроса и пометка его в пулле коннектов как свободного.
Для пользователя это всё скрыто и можно использовать sql.query
Полный список настроек [тут](https://github.com/mysqljs/mysql#pool-options)


### Запросы
Все запросы к БД можно разделить на две вида:
1. выборка данных
```sql
SELECT email, username, dateCreate FROM users WHERE email = 'd@mail.ru'
SELECT SUBSTRING_INDEX(email, '@', -1) AS domain, COUNT(*) AS count FROM users GROUP BY domain ORDER BY count DESC LIMIT 5
```
2. выполнение команды
```sql
DELETE FROM users WHERE email LIKE '%@mail.ru%'
ALTER TABLE users ADD COLUMN phone VARCHAR(15) AFTER username;
```


### Построитель запросов
Как правило запрос должен содержать некоторые данные, полученные из вне

```javascript
// где-то в коде
const email = req.queru.email;

sql.query(`SELECT * FROM users WHERE email = ${email}`, ...;
```
Такой подход хорош все, кроме безопасности. Самым частым случаем взлома веб-приложений является передача от клиента параметров, изменяющих поведение sql-запрос на вредоностное. Чтобы "руками" каждый раз не проверять, что там передали от клиента, в драйвер встроенн механизм "очистки" данных от возможных проблем:
```javascript
sql.query(
  'SELECT * FROM users WHERE email = ? AND username = ?',
 email,
 username
  function (error, results) {
    
  }
);
```
При выполнении запроса драйвер вызовет метод `connection.escape()` и "очистит" значение от возможного вредоностного содержимого.

Для удобства составления запросов можно и нужно использовать постоитель запросов [squel](hiddentao.com/squel)
```javascript
let s = squel.select()
        .field("*")
        .from("users")
        .where("email = ?", email)
        .where("username = ?", username);
        
sql.query(s.toString(), ...)
```
Таким образом сущевственно упрощается написание запросов, поскольку часто повторяющиеся кусочки можно сохранить куда-то в точку входа приложение и делать что-то типа такого:
```javascript

// где-то в месте инициализации приложения
global.queryMap = {
 'getUser': squel.select()
        .field("*")
        .from("users")
};


// в нужном месте
let query = global.queryMap.clone()
 .where('username = ?', username)
 .toString();
        
```

### Домашнее задание
* добавить создание подключения в проект
* создать/изменить роут авторизации и регистрации, проверяющий данные пользователя в БД
* сделать все запросы через построитель запросов





