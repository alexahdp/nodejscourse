# Занятие 8


### ORM
Работать с данными как нативными примитивами js-а можно, но обычно не сильно удобно. Со временем появляется много кода, тесно связанного с экземплярами тех или иных объектов. Для удобства работы с данными обычно используют высокоуровневые обёртки, например ORM. Т.е. тип объекта описывается схемой и методами работы с ней (далее будем использовать промисы).


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
  username: 'Alex',
  email: 'alexcode61.ru',
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

Аналоги этих действий в виде чистого sql^
```sql
INSERT INTO users (
  username,
  email,
  password
) VALUES(
  'Alex',
  'alexcode61.ru',
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
      is: /^[a-z]+$/i,          // same as the previous example using real RegExp
      not: ["[a-z]",'i'],       // will not allow letters
      isEmail: true,            // checks for email format (foo@bar.com)
      isUrl: true,              // checks for url format (http://foo.com)
      isIP: true,               // checks for IPv4 (129.89.23.1) or IPv6 format
      isIPv4: true,             // checks for IPv4 (129.89.23.1)
      isIPv6: true,             // checks for IPv6 format
      isAlpha: true,            // will only allow letters
      isAlphanumeric: true,     // will only allow alphanumeric characters, so "_abc" will fail
      isNumeric: true,          // will only allow numbers
      isInt: true,              // checks for valid integers
      isFloat: true,            // checks for valid floating point numbers
      isDecimal: true,          // checks for any numbers
      isLowercase: true,        // checks for lowercase
      isUppercase: true,        // checks for uppercase
      notNull: true,            // won't allow null
      isNull: true,             // only allows null
      notEmpty: true,           // don't allow empty strings
      equals: 'specific value', // only allow a specific value
      contains: 'foo',          // force specific substrings
      notIn: [['foo', 'bar']],  // check the value is not one of these
      isIn: [['foo', 'bar']],   // check the value is one of these
      notContains: 'bar',       // don't allow specific substrings
      len: [2,10],              // only allow values with length between 2 and 10
      isUUID: 4,                // only allow uuids
      isDate: true,             // only allow date strings
      isAfter: "2011-11-05",    // only allow date strings after a specific date
      isBefore: "2011-11-05",   // only allow date strings before a specific date
      max: 23,                  // only allow values <= 23
      min: 23,                  // only allow values >= 23
      isCreditCard: true,       // check for valid credit card numbers

      // custom validations are also possible:
      isEven(value) {
        if (parseInt(value) % 2 != 0) {
          throw new Error('Only even values are allowed!')
          // we also are in the model's context here, so this.otherField
          // would get the value of otherField if it existed
        }
      }
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

### GUI
Работать с БД через консоль можно, но можно использовать графическое приложение [HeidiSQL ](https://www.heidisql.com/)


### Домашнее задание
1. описать все сущности, хранящиеся в БД, как схемы Sequelize и методы схем. Каждая сущность должна быть описана в отдельном файле. Например пользователь - `model/user.js`. Файл должен экспортировать уже готовую модель. Для удобства можно подключить все модели куда-нибудь в `global.model = { ... }`
2. переписать все запросы к БД на Sequelize
