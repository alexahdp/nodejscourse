# Занятие 4

## Теоретическая часть

### Последовательное выполнение асинхронного кода
 - сырым javascript  
 - с использованием async.waterfall, async.mapSeries  
 - обработка ошибок  

### Параллельное выполнение асинхронного кода
 - с использованием async.parallel  
 - обработка ошибок  

### Promise
 - создание и использование  
 - обработка ошибок  
 - Promise.all  
 - util.promisify  
 - bluebird  

### Async-await
 - отличие async-функций от обычных функций  
 - обработка ошибок  

### Queue
 - примеры использования  


## Практическая часть

Набор свободных api:
https://github.com/toddmotto/public-apis#books


### Задание 1
Есть открытый api: https://coinmarketcap.com/api/.
Нужно написать скрипт, который коневртирует BTC в LTC, LTC в PPC и PPC в USD.
Для этого нужно обращаться по урлу:  
https://api.coinmarketcap.com/v2/ticker/{CURRENCY_ID}/?convert={CURRENCY_NAME}  

Таблица соответствия CURRENCY_ID к CURRENCY_NAME ниже:  
BTC = 1
LTC = 2
PPC = 5

В качестве ответа от сервера приходит json вида:
```js
{
    "data": {
        "id": 6, 
        "name": "Novacoin", 
        "symbol": "NVC", 
        "website_slug": "novacoin", 
        "rank": 400, 
        "circulating_supply": 2146161.0, 
        "total_supply": 2146161.0, 
        "max_supply": null, 
        "quotes": {
            "LTC": {
                "price": 0.0612845261, 
                "volume_24h": 590.5546917273, 
                "market_cap": 131526.0, 
                "percent_change_1h": 0.12, 
                "percent_change_24h": -5.19, 
                "percent_change_7d": -15.33
            }, 
            "USD": {
                "price": 4.5577062854, 
                "volume_24h": 43919.3219191019, 
                "market_cap": 9781574.0, 
                "percent_change_1h": -0.25, 
                "percent_change_24h": -4.09, 
                "percent_change_7d": -19.86
            }
        }, 
        "last_updated": 1533666186
    }, 
    "metadata": {
        "timestamp": 1533665780, 
        "error": null
    }
}
```
При получении ответа от сервера необходимо обязательно проверять
наличие ошибки в metadata.error.  
Для обращения к серверу стоит использовать модуль request.  
Например, для конвертации BTC => LTC нужно обратиться по адресу:
https://api.coinmarketcap.com/v2/ticker/1/?convert=LTC
Тип запроса - get.  
Интересующее нас поле - price.  

### Задание 2
Нужно переписать логику работы задания №1 на использование async.waterfall.  

### Задание 3
Нужно переписать логику работы задания №2 на Promise-ы.  
(обратить внимание на util.promisify)  

### Задание 4
Нужно переписать логику работы задания №3 на async-await.  

### Задание 5
TODO проработать работу с очередями  

## Homework
Написать простой crawler сайта. Crawler - бот, который обходит страницы сайта
и извлекает из них информацию. Наш Crawler должен просто обходить первые 5 страниц сайта.
В качестве входных данных боту скармливается url главной страницы сайта.
Бот должен получить ее get-запросом, извлечь из нее ссылки, лежащие в <a href="/link">
и пройтись по первым 10 ссылкам. При получении страницы бот должен писать в консоль 
title страницы. Для извлечения ссылок и title-ов можно использовать модуль cheerio.