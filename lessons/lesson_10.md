# Аутентификация и авторизация

## Оглавление

1. Аутентификация </br>
1.1 Основные термины </br>
1.2 Модуль Crypto </br>
1.3 Аутентификация при помощи passport.js </br>
2. Авторизация </br>
2.1 Основные термины </br>
2.2 Oauth2 </br>
2.3 Авторизация при помощи passport.js
3. Безопасность </br>
3.1 Виды уязвимостей </br>
3.2 Helmet.js </br>
4. Домашнее задание </br>
4.1 Основные термины </br>
4.2 Практическое задание по работе с Json Web Token </br>
5. Источники </br>

### TODOS
- переделать раздел ниже на монгуз и добавить логику с crypto
- добавить еще одну стратегию
- в раздел oauth2 добавить пример постмана\curl

### 1. Аутентификация
### 1.1 Основные термины
<b>Аутентификация</b> - это проверка соответствия субъекта и того, за кого он пытается себя выдать, с помощью некой уникальной информации (отпечатки пальцев, цвет радужки, голос и тд.), в простейшем случае - с помощью имени входа и пароля.

### 1.2 Модуль Crypto

### 1.3 Аутентификация при помощи passport.js
<b>Passport</b> — это middleware для проверки подлинности, которую мы собираемся использовать для управления сессиями.

Наша цель — реализовать в нашем приложении следующий процесс аутентификации:

1. Пользователь вводит имя и пароль
2. Приложение проверяет, являются ли они корректными
3. Если имя и пароль корректны, приложение отправляет заголовок Set-Cookie, который будет использоваться для аутентификации дальнейших страниц
4. Когда пользователь посещает страницы в том же домене, ранее установленный cookie будет добавлен ко всем запросам
5. Аутентификация на закрытых страницах происходит с помощью этого файла cookie

Чтобы настроить такую стратегию аутентификации, выполните следующие два действия:

1. Настройте Express

```javascript
const express = require('express')
const passport = require('passport')
const session = require('express-session')
const RedisStore = require('connect-redis')(session)
const app = express()
app.use(session({
    store: new RedisStore({
        url: config.redisStore.url
    }),
    secret: config.redisStore.secret,
    resave: false,
    saveUninitialized: false
}))
app.use(passport.initialize())
app.use(passport.session())
```

2. Настройте Passport

<b>Passport</b> — отличный пример библиотеки, использующей плагины. В этом уроке мы добавляем модуль passport-local, который добавляет простую локальную стратегию аутентификации с использованием имён пользователей и паролей.

```javascript
const passport = require('passport')
const LocalStrategy = require('passport-local').Strategy
const user = {
    username: 'test-user',
    password: 'my-password',
    id: 1
}
passport.use(new LocalStrategy(
    function(username, password, done) {
        findUser(username, function (err, user) {
            if (err) {
                return done(err)
            }
            if (!user) {
                return done(null, false)
            }
            if (password !== user.password ) {
                return done(null, false)
            }
            return done(null, user)
        })
    }
))
```

### 2. Авторизация
### 2.1 Основные термины
<b>Авторизация</b> - это проверка и определение полномочий на выполнение некоторых действий в соответствии с ранее выполненной аутентификацией.

<b>OAuth 2.0</b> — протокол авторизации, позволяющий выдать одному сервису (приложению) права на доступ к ресурсам пользователя на другом сервисе. Протокол избавляет от необходимости доверять приложению логин и пароль, а также позволяет выдавать ограниченный набор прав, а не все сразу.

<b>Токены</b> предоставляют собой средство авторизации для каждого запроса от клиента к серверу. Токены(и соотвественно сигнатура токена) генерируются на сервере основываясь на секретном ключе(который хранится на сервере) и payload'e. Токен в итоге хранится на клиенте и используется при необходимости авторизации како-го либо запроса. Такое решение отлично подходит при разработке SPA.

<b>access token</b> — некий ключ (обычно просто набор символов), предъявление которого является пропуском к защищенным ресурсам.

<b>refresh token</b> - выдается сервером по результам успешной аутентификации и используется для получения нового access token'a и обновления refresh token'a


### 2.2 Oauth2
Общая схема работы приложения, использующего OAuth, такова:
1. получение авторизации
2. обращение к защищенным ресурсам

Результатом авторизации является <b>access token</b>

В протоколе описано несколько вариантов авторизации, подходящих для различных ситуаций:
1. авторизация для приложений, имеющих серверную часть (чаще всего, это сайты и веб-приложения)
2. авторизация для полностью клиентских приложений (мобильные и desktop-приложения)
3. авторизация по логину и паролю
4. восстановление предыдущей авторизации

Далее рассмотрим схему авторизации для приложений, имеющих серверную часть

![схема авторизации](https://habrastorage.org/storage/0309cc52/d3bd6b1e/b44ab1f6/64e01804.png)

1. Редирект на страницу авторизации
2. На странице авторизации у пользователя запрашивается подтверждение выдачи прав
3. В случае согласия пользователя, браузер редиректится на URL, указанный при открытии страницы авторизации, с добавлением в GET-параметры специального ключа — authorization code
4. Сервер приложения выполняет POST-запрос с полученным authorization code в качестве параметра. В результате этого запроса возвращается access token

### 2.3 Авторизация при помощи passport.js

Чтобы добавить защищенные разделы на сайт, мы используем шаблон middleware Express. Для этого сначала создадим middleware для аутентификации:

```javascript
const passport = require('passport')
app.get('/profile', passport.authenticationMiddleware(), renderProfile)
```

Она имеет только одну задачу — если пользователь аутентифицирован (имеет правильные cookie-файлы), она просто вызывает следующую middleware. В противном случае пользователь перенаправляется на страницу, где он может войти в систему.

Использование этой middleware также просто, как добавление любой другой middleware в определение роута.

### 3. Безопасность
### 3.1 Виды уязвимостей
- <b>contentSecurityPolicy</b> for setting Content Security Policy
- <b>crossdomain</b> for handling Adobe products’ crossdomain requests
- <b>dnsPrefetchControl</b> controls browser DNS prefetching
- <b>expectCt</b> for handling Certificate Transparency
- <b>frameguard</b> to prevent clickjacking
- <b>hidePoweredBy</b> to remove the X-Powered-By header
- <b>hpkp</b> for HTTP Public Key Pinning
- <b>hsts</b> for HTTP Strict Transport Security	
- <b>ieNoOpen</b> sets X-Download-Options for IE8+
- <b>noCache</b> to disable client-side caching
- <b>noSniff</b> to keep clients from sniffing the MIME type
- <b>referrerPolicy</b> to hide the Referer header
- <b>xssFilter</b> adds some small XSS protections
### 3.2 Helmet.js

Данная библиотека предоставляет защиту от перечисленных выше уязвимостей

Использовать ее просто:
```javascript
const express = require('express')
const helmet = require('helmet')

const app = express()

app.use(helmet())
```

Но по умолчанию включены не все, остальные можно включить таким образом:

```javascript
app.use(helmet.noCache())
app.use(helmet.frameguard())
```
https://github.com/helmetjs/helmet

### 4. Домашнее задание
### 4.1 Теория
<b> JSON Web Token (JWT)</b> — содержит три блока, разделенных точками: заголовок(header), набор полей (payload) и сигнатуру. Первые два блока представлены в JSON-формате и дополнительно закодированы в формат base64. Набор полей содержит произвольные пары имя/значения, притом стандарт JWT определяет несколько зарезервированных имен (iss, aud, exp и другие). Сигнатура может генерироваться при помощи и симметричных алгоритмов шифрования, и асимметричных. Кроме того, существует отдельный стандарт, отписывающий формат зашифрованного JWT-токена.

Пример подписанного JWT токена (после декодирования 1 и 2 блоков):

```
{ «alg»: «HS256», «typ»: «JWT» }.{ «iss»: «auth.myservice.com», «aud»: «myservice.com», «exp»: «1435937883», «userName»: «John Smith», «userRole»: «Admin» }.S9Zs/8/uEGGTVVtLggFTizCsMtwOJnRhjaQ2BMUQhcY
```
## Схема создания/использования токенов

1. Пользователь логинится в приложении, передавая логин/пароль на сервер

2. Сервер проверят подлинность логина/пароля, в случае удачи генерирует и отправляет клиенту два токена(access, refresh) и время смерти access token'а (expires_in поле, в unix timestamp). Также в payload refresh token'a добавляется user_id
    ```
    "accessToken": "...",
    "refreshToken": "...",
    "expires_in": 1502305985425
    ```

3. Клиент сохраняет токены и время смерти access token'а, используя access token для последующей авторизации запросов

4. Перед каждым запросом клиент предварительно проверяет время жизни access token'а (из expires_in)и если оно истекло использует refresh token чтобы обновить ОБА токена и продолжает использовать новый access token

## Схема рефреша токенов

1. Клиент проверяет перед запросом не истекло ли время жизни access token'на
2. Если истекло клиент отправляет на auth/refresh-token URL refresh token
3. Сервер берет user_id из payload'a refresh token'a по нему ищет в БД запись данного юзера и достает из него refresh token
4. Сравнивает refresh token клиента с refresh token'ом найденным в БД
5. Проверяет валидность и срок действия refresh token'а
6. В случае успеха сервер:</br>
6.1 Создает и перезаписывает refresh token в БД</br>
6.2 Создает новый access token</br>
6.3 Отправляет оба токена и новый expires_in access token'а клиенту</br>
7. Клиент повторяет запрос к API c новым access token'ом

Пример:
https://github.com/zmts/supra-api-nodejs/tree/master/actions/auth

### 4.2 Практическое задание по работе с Json Web Token
Добавить в приложение авторизацию по JWT

### Источники
- https://gist.github.com/zmts/802dc9c3510d79fd40f9dc38a12bccfc
- https://habr.com/post/145988/
- https://habr.com/company/mailru/blog/115163/
- https://habr.com/post/193458/#p5
- https://medium.com/devschacht/node-hero-chapter-8-27b74c33a5ce