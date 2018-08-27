# Занятие 7

## Теоретическая часть
 - что такое базы данных и для чего их стоит использовать  
 - какие бывают базы данных (key-value, реляционные, sql/nosql, графовые, колоночные)  
 - SQL-базы данных: mysql, mariadb, postrgres (основные отличия)  
 - синтаксис SQL (очень коротко)  
 - ORM  
 - разработка схемы данных  
 - валидация данных  


Пример таблицы с данными пользователей:  
|  id | username |   email   |      dateCreate     |
|---|---|---|---|
| 345 | Bob     | p@mail.ru | 2006-01-11 07:56:20 |
| 346 | Alice     | s@mail.ru | 2018-08-13 19:17:04 |
| 347 | Mark  | d@mail.ru | 2018-08-14 00:14:12 |


Эти же данные на стороне NodeJS в виде объектов:
```js
[
  {
    id: 345,
    username: 'Bob',
    email: 'p@mail.ru',
    dateCreate: '2006-01-11 07:56:20'
  },
  {
    id: 346,
    username: 'Alice',
    email: 's@mail.ru',
    dateCreate: '2018-08-13 19:17:04'
  },
  {
    id: 347,
    username: 'Mark',
    email: 'd@mail.ru',
    dateCreate: '2018-08-14 00:14:12'
  }
]
```

## Практическая часть

### Установка
В качестве sql-базы данных будем использовать [MariaDB](https://mariadb.com/kb/en/library/installing-and-using-mariadb-via-docker/)  

Скачиваем образ последней стабильной версии через docker
```sh
docker pull mariadb:latest
```

Запускаем контейнер командой
```sh
docker run --restart=always --name mariadb_test -v /mnt/mysql:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD='mypass' -p 3306:3306 -d mariadb -innodb_buffer_pool_size=1G
```
Что тут написано:
* запустить контейнер c именем `mariadb_test`
* рестартовать контейнер при выключении системы `–restart=always`
* проброс директории в контейнер, чтобы при перезапуске контейнера не потерять данные в контейнере `-v /mnt/mysql:/var/lib/mysql`
* сразу задаем пароль `-e MYSQL_ROOT_PASSWORD='mypass'`
* указываем порт, по которому будет снаружи доступна база `-p 3306:3306`. Первая цифра 3306 - это мы используем для доступа снаружи контейнера, например указываем в драйвере в NodeJS. Второй порт 3306 - база должна слушать внутри контейнера. Т.е. мы можем запустить хоть 10 контейнеров с базой и они будут прекрасно работать.
* запусить в режиме демона `-d`
* имя образа `mariadb`. Можно указать конкретную версию базы через `:`
* опция уже самого процесса базы `-innodb_buffer_pool_size=1G`. Т.е. уже при запуске контейнера можно указывать настройки запуска и не нужно лезь внутрь контейнера

### GUI
Работать с БД через консоль можно, но можно использовать графическое приложение [HeidiSQL ](https://www.heidisql.com/)

### Подключение через консоль
Внутри docker-контейнера с mysql нужно выполнить команду:
```bash
mysql -uroot --password -p 3306 --protocol=TCP
```
после этого мы попадаем в режим командой строки mysql. Как будто запустили интерпретатор NodeJS, только для базы данных. 


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

Теперь в любом месте проекта пишем:
```javascript
const { sql } = require('model/db');
sql.query('SELECT 1 + 1 AS solution', function (error, results, fields) {
  if (error) throw error;
  console.log('The solution is: ', results[0].solution);
});
```

### Пулл коннектор
Код создания коннекта из предыдущего примера имеет существенные ограничения:
  параметры подключения захардкожены в файле, нет закрытия коннекта, нельзя делать несколько одновременных запросов. Это легко исправляется путем изменений конфига подключения к БД:
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
(поле connectionLimit)

Теперь при вызове pool.query будут вызваны несколько действий:
* sql.getConnection(). Он получит объект connection из пулла коннектов. Если свободных коннектор нет, будет ждать пока появится.
* connection.query(). Собвственно выполнение запроса
* connection.release(). Освобождение запроса и пометка его в пулле коннектов как свободного.
Для пользователя это всё скрыто и можно использовать sql.query
Полный список настроек [тут](https://github.com/mysqljs/mysql#pool-options)


### Примеры sql-запросов

1. получение данных
```sql
SELECT email, username, dateCreate FROM users WHERE email = 'd@mail.ru'
SELECT SUBSTRING_INDEX(email, '@', -1) AS domain, COUNT(*) AS count FROM users GROUP BY domain ORDER BY count DESC LIMIT 5
```
2. изменение данных
```sql
DELETE FROM users WHERE email LIKE '%@mail.ru%'
ALTER TABLE users ADD COLUMN phone VARCHAR(15) AFTER username;
```

### Построитель запросов
Как правило, запрос должен содержать некоторые данные полученные извне

```javascript
// где-то в коде
const email = req.query.email;

sql.query(`SELECT * FROM users WHERE email = ${email}`, ...;
```
Такой подход хорош всем, кроме безопасности. Самым частым случаем взлома веб-приложений является передача от клиента параметров, изменяющих поведение sql-запроса на вредоносное. Чтобы "руками" каждый раз не проверять, что же пришло от клиента, в драйвер встроен механизм "очистки" данных от возможных проблем:
```javascript
sql.query(
  'SELECT * FROM users WHERE email = ? AND username = ?',
 email,
 username
  function (error, results) {
    
  }
);
```
При выполнении запроса драйвер вызовет метод `connection.escape()` и очистит значение от возможного вредоносного содержимого.

Для удобства составления запросов можно и нужно использовать постоитель запросов [squel](hiddentao.com/squel)
```javascript
let s = squel.select()
        .field("*")
        .from("users")
        .where("email = ?", email)
        .where("username = ?", username);
        
sql.query(s.toString(), ...)
```

### ORM
Работать с данными как нативными примитивами js-а можно, но обычно не очень удобно. Со временем появляется много кода тесно связанного с экземплярами тех или иных объектов. Для удобства работы с данными обычно используют высокоуровневые обёртки, например ORM. Т.е. тип объекта описывается схемой и методами работы с ней (далее будем использовать промисы).

### Коннект
Будем использовать [sequelizejs](http://docs.sequelizejs.com/), поскольку позволяет работать с целым рядом популярных sql-баз данных:

1. Ставим модули
```bash
npm i --save sequelize
npm i --save mysql2
```

2. Создаем файл `model/orm.js` с содержимым
```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'
});


module.exports = { sequelize, Sequelize };
```

Создадим схему пользователя `model/user.js`:
```javascript
const { Sequelize, sequelize } = require('model/sequelize');

const User = sequelize.define('user', {
  username: Sequelize.STRING,
  email: {
      type: Sequelize.STRING(255),
      validate: {
        isEmail: true
      }
  },
  dateCreate: {
      type: Sequelize.DATE,
      defaultValue: Sequelize.NOW
  },
  password: {
      type: Sequelize.STRING(255)
  }
});

module.exports = User;
```

3. Создаем схему объекта с БД (если такая таблица уже есть, но другой структуры, ничего не произойдёт)
```javascript

const User = require('model/user');

sequelize.sync()
  .then(() => User.create({
    username: 'Alex',
    email: 'alexcode61.ru',
    password: 123
  }))
  .then(user => {
      console.log(user.toJSON());
  });
```

#### CRUD
Обычный цикл работы с объектами БД выглядит так
1. создать документ
```javascript
let user = await User.create({
  username: 'Bob',
  email: 'test@gmail.com',
  password: 123
});
```

2. Найти
```javascript
const email = ...
let user = await User.findOne({
  where: {
    email: email
  }
});
```

3. Обновить
```javascript
let user = ...
user.email = newEmail;
await user.save();
```

4. Удалить
```javascript
let user = ...;
await user.destroy();
```

http://docs.sequelizejs.com/manual/tutorial/models-usage.html

Аналоги этих действий в виде чистого sql:
```sql
INSERT INTO users (
  username,
  email,
  password
) VALUES(
  'Bob',
  'test@gmail.com',
  123
);

SELECT *
FROM users
WHERE (
  email = '...'
)
LIMIT 1;

UPDATE TABLE users SET email = '...' WHERE id = ...;

DELETE FROM  users WHERE id = ...;
```

### Методы
Здесь и далее примеры из [документации](http://docs.sequelizejs.com/manual/tutorial/models-definition.html)
При описании схемы объекта, помимо описания полей, можно указать методы:

1. Статические методы. Методы, общие для схемы
```javascript
// объявление
User.version = () => 'v4';


// использование
console.log(global.model.User.version());
```

2. Методы экземпляра объекта
```javascript

// объявление
User.prototype.getDislpayName = function() {
  return `${this.username} (${this.email})`;
};

// использование
const user = ...;

console.log(user.getDislpayName());
```

3. Валидатор. Далее идёт копипаста с документации, просто чтобы оценить встроенные возможности
```javascript

// объявление
const User = sequelize.define('user', {
  foo: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // will only allow letters
    }
  }
});


// использование
let user = ...;
user.email = ...
let errors = user.validate();
if (errors) {
  // если мы зашли сюда, значит схеме что-то не понравилось в email
}

4. Getter/Setter

// объявление
const User = sequelize.define('user', {
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  getterMethods: {
    fullName() {
      return this.firstname + ' ' + this.lastname
    }
  },

  setterMethods: {
    fullName(value) {
      const names = value.split(' ');

      this.setDataValue('firstname', names[0]);
      this.setDataValue('lastname', name[1]);
    },
  }
});


// использование
let user = ...;
console.log(user.fullName);

user.fullName = 'Alex Pez';

console.log(user.fullName);

```

### Задание 1
Добавить создание подключения в проект. Код подключения должен располагаться в отдельном файле. Перенести запуск mysql в docker-контейнере в docker-compose файл.

### Задание 2
Изменить код авторизации и регистрации так, чтобы данные о пользователе хранились в БД.
Должна быть таблица users. В этой таблице должны быть следующие поля:
 - id  
 - email  
 - password  
 - createdAt  
 - updatedAt  
 - emailVerified  
 - firstName  
 - lastName  
(должен быть специальный файл с командой для создания таблицы)

При разработке нужно учитывать следующие факты:
 - при регистрации пользователь с указанным email может уже существовать в базе  
 - пользователь может попытаться ввести некорректный email  
 - на клиент нужно отдавать ошибки валидации  
 - на клиент не нужно отдавать системные ошибки  
 - firstName и lastName не должны быть пустыми  
 - при регистрации нового пользователя нужно отправлять на его email письмо о подтверждении  
 - до тех пор, пока пользователь не подтвердит свой email, его нельзя авторизовывать  
 - должна быть отдельная страница с формой "Забыли пароль?"  
 - количество пользователей будет большим и поиск пользователей будет выполняться по email  
 - должна быть bin-команда для верификации пользователей, пример использования:
```js
bin/user.js verify --email bob@gmail.com
```


## Homework

### Задание 1

Реализовать защиту от перебора паролей брутфорсом. Нужно завести отдельную таблицу и в этой таблице считать количество попыток авторизации. Поскольку с одного ip-адреса могут сидеть множество пользователей скрытых за NAT-ом, то просто блокировать всех подряд по ip-адресу не очень хорошая идея. Вместо этого блокировать будем возможность входа в определенный аккаунт по некоторому ip-адресу. Структура таблицы:
 - ip-адрес  
 - email авторизации  
 - время попытки авторизации  

В конфиге должен быть параметр "Максимальное количество попыток авторизации". Если количество попыток авторизации за последний час превысило значение максимального количества попыток - не пытаться авторизовать пользователя, вернуть 403 статус и текстовое сообщение о временной блокировке пользователя. Таким образом, через час пользователь снова сможет авторизовываться.

Также стоит написать bin-скрипт, который будет запускаться по крону раз в сутки ночью и удалять старые записи, чтобы они не скапливались.
