## Работа с нереляционными БД (на примере MongoDB)  
 
#### Mongo DB - самая популярная не реляционная база данных:

Основная особенность - не строгая схема данных (с версии 3.6 появилась `JSON Schema`)
[JSON Schema](https://docs.mongodb.com/manual/core/schema-validation/#json-schema)

Плюсы в сравнении с реляционной СУБД:
* Почти ни когда не нужно делать миграции
* Упрощение связей между сущностями 
Например:
```javascript
{
	Dialog: {
		_id     : ObjetId,
		users   : Array<ObjetId>,
		messages: Array<{ from: String, msg: String }>,
	},
}
```
При определенных требованиях, такой подход более удобный нежели создавать отдельную коллекцию для сообщений. Например, обычно сообщение, оторванное от контекста диалога, не имеет смысла.

Кстати в `mongoose` такая схема описана была бы так:

```javascript
const Schema = mongoose.Schema;
const messageSchema = new Schema({
	from: {
		type    : String,
		required: true,
	},
	
	msg: {
		type    : String,
		required: true,
	},
});

const dialogSchema = new Schema({
	users   : [ Schema.Types.ObjectId ],
	messages: [ messageSchema ],
});
```

А вот в мире `Sql` придется делать кучу разных `join`, для выборок.

* Так же исходя из предыдущего пункта, более легковесная и по этому может обрабатывать больше `rps` при прочих равных.


#### Mongo DB в связке с Mongoose - ООП в работе с базой данных  
* `ODM`;
* Mongoose - `ODM` для `mongodb` реализующий паттерн [active record](https://ru.wikipedia.org/wiki/ActiveRecord)
* Описание схемы монгуса
* [Грабли монгуса](https://habrahabr.ru/post/253395/)
* Дискрминаторы - [паттерн STI](http://design-pattern.ru/patterns/single-table-inheritance.html)
* Валидация (синхронная / ассинхронная)
* репликация
  * Replica Sets - Предпочтительный вариант
  * Master-Slave
* `.lean` - Помогает сэкономить CPU

### Настройка и запуск собственной базы данных Mongo DB на локальном сервере
 
 Устанавливается в докере https://hub.docker.com/_/mongo/
 Для хранения данных используется [named volume](https://docs.docker.com/storage/#good-use-cases-for-bind-mounts) 
 
 
 ### Инструменты для работы с данными
 * [Robo 3T](https://robomongo.org/)


### Задание

1) Написать json API с использование `mongodb` и `mongoose` как `ODM`

Не большое приложение по учету книг.
У книги есть: Название, Автор, Описание, Жанр.

Должно быть реализованно:
* Добавление
* Выборка (поиск по автору и названии книги)
* Редактирование существующей книги
* Можно сменить жанр книги
* Можно добавить, удалить автора
