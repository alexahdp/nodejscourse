# Занятие 8

### Коннект
Будем использовать [sequelizejs](http://docs.sequelizejs.com/), поскольку позволяет работать с целым рядом популярных sql-баз данных:

1. Ставим модули
```bash
npm i --save sequelize
npm i --save pg pg-hstore
```

2. Создаем файл `sql.js` с содержимым
```js
sequelize.sync()
  .then(() => User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  }))
  .then(jane => {
    console.log(jane.toJSON());
  });
```

Думаем что не так.

3. Описание модели
```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres'
});

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

Что не так с кодом? Как поправить, чтобы работало? Что можно улучшить?

#### CRUD
http://docs.sequelizejs.com/manual/tutorial/models-usage.html

```sql
SELECT *
FROM users
WHERE (
  name = 'Alex'
)
LIMIT 1;
```

```js
User.findOne({
    where: {
        username: 'Alex'
    }
}).then(user => {
    user.password = '456';
    return user.save();
}).then(user => {
    return user.destroy();
});
```




 - Инструменты для работы с данными - PGAdmin 