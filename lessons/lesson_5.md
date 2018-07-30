# Занятие 5

## План занятия

### Теория

1) Http-протокол  
 - положение на модели OSI  
 - текстовый/бинарный (1.*, 2.*)  
 - структура  
 - статусы  
 - просмотр через devTools или postman

2) Что такое http-сервер  
 - модуль http  
 - открытие порта  
 - отдача ответа  

3) Зачем нужен express  
 - роутинг  
 - миддлвары  
 - шаблонизаторы  


### Практика

4) Установка express и создание проекта  
 - написание собственного middleware  
 - проверка прав доступа с помощью middleware  
 - готовые middleware-ы (static, cookieParser)  
 - создание сессий  

5) Авторизация
 - создание механизма авторизации/регистрации с сохранением пользовательсих данных только в сессии

5) Загрузка файлов на сервер  
 - что такое stream и когда их нужно использовать  
 - загрузка файла с использованием stream  


## Домашнее задание
 - Добавить в проект возможность авторизации/регистрации  
 - Должна быть проверка входящих данных (валидный email, пароль минимальной длины с наличием спец-символов)  
 - Загрузка файлов (аватар пользователя, обложка кинофильма, аудиозапись, ...)  