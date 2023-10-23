# Руководство подключению к API 
> ОБРАТИТЕ ВНИМАНИЕ!
> 
> Ключи показаные в руководстве **НЕ ПРЕДОСТАВЛЯЮТ ДОСТУПА К ФУНКЦИОНАЛУ**, но позволят Вам проверить правильность работы подписи в вашем приложении.
>
> Для работы с API необходимо быть зарегистрированым на https://beribit.com/

## Навигация
[Получение курса из стаканов](#`получение-курса-из-стаканов`)

## Общая информация
- Базовый адрес: https://api.beribit.com
- Все поля, связанные со временем, указываются с нулевым смещением (UTC+0) в формате _YYYY-MM-DD'T'hh:mm:ss_ <br>**Пример:** 2023-09-14T12:44:31 (Московское время: 2023-09-14T15:44:31)
- API-ключи **чувствительны к регистру**.

## HTTP-коды возврата
|Код возврата| Описание |
|--|--|
| 200 | Операция выполнена успешно, возвращается объект JSON |
| 4XX | Используются для некорректно сформированных запросов; проблема на стороне отправителя |
| 401 | Используется при ошибке авторизации |
| 408 | Используется если превышен интервал запроса |
| 5XX | Используются для внутренних ошибок, проблема на стороне BERIBIT. **Важно НЕ рассматривать это как неудачную операцию; состояние выполнения НЕИЗВЕСТНО и могло быть выполнено успешно** |

## Авторизация

* **Обязательно:** Для каждого запроса ВНЕ ЗАВИСИМОСТИ ОТ ТИПА добавляется штамп времени (текущее время в формате UTC), передается в параметре запроса timestamp.
> GET https://api.beribit.com/accounts?timestamp=2023-08-20T13:51:00<br>
> POST https://api.beribit.com/deposit/generate_address?timestamp=2023-08-20T13:51:00

Для авторизации необходимо поместить следущие параметры в заголовок запроса:
|Заголовок| Содержание |
|--|--|
| UID | **Персональный ключ** |
| SIGNATURE | **HMACSHA256-хеш** |

### GET-запрос
|Параметр| Описание | Пример|
|--|--|--|
|PRIVATE_KEY|Приватный ключ HMACSHA256|<code>ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==</code>|
|PARAMS|Все параметры из HTTP запроса |<code>?timestamp=2023-08-20T13:51:00&page=2</code>|

Подписью является HMACSHA256("{PARAMS}"), где SecretKey HMACSHA256 является PRIVATE_KEY

Пример тела подписи для GET-запроса:

    ?timestamp=2023-08-20T13:51:00&page=2

Полученая подпись HMACSHA256

    e4704526be7d4be0663cba9e410560c1801fcf9653cf299041098e1e4ffa5296

### POST-запрос
|Параметр| Описание | Пример|
|--|--|--|
|UID|Персональный ключ|<code>e7742caf-5e74-498c-8f4f-d4ae0a6f2bf3</code>|
|PRIVATE_KEY|Приватный ключ|<code>ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==</code>|
|PARAMS|Все параметры из HTTP запроса |<code>?timestamp=2023-08-20T13:51:00&page=2</code>|
|PAYLOAD|Тело запроса|<code>{"AddressTo" : "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666", "Blockchain": "TRC20", "Amount": "10945.00", "Token": "USDT"}</code>|


Подписью является HMACSHA256("{PARAMS}:{PAYLOAD}"), где SecretKey HMACSHA256 является PRIVATE_KEY

Пример тела подписи для POST-запроса:

    ?timestamp=2023-08-20T13:51:00&page=2:{"AddressTo" : "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666", "Blockchain": "TRC20", "Amount": "10945.00", "Token": "USDT"}

Полученая подпись HMACSHA256

    39d69127df0fde4ec42374e7b673b1261851129a187710eaad0ead04028f8c27
    
## Общие функции

### `Получение баланса по всем валютам`

Тип запроса: **GET**

Путь: **/accounts**

Описание: **данная функция отвечает за возвращение списка криптовалютных балансов.**

Результат успешного выполнения: **возвращается название криптовалюты, сумма на открытом счету, сумма на закрытом счету и таймштамп.**

Пример ответа:
```<json>
{
    "Success": true,
    "Result": [
        {
            "Currency": "RUB",
            "Balance": 10000.00,
            "Locked": 2000.00,
            "Time": "2023-09-15T09:48:40.8485648Z"
        },
        {
            "Currency": "ETH",
            "Balance": 300.053021,
            "Locked": 50.00,
            "Time": "2023-09-15T09:48:40.848655Z"
        },
        {
            "Currency": "USDT",
            "Balance": 300.04,
            "Locked": 2560.73,
            "Time": "2023-09-15T09:48:40.8486553Z"
        }
    ]
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```

### `Функция получения баланса по конкретной валюте`

Тип запроса: **GET**

Путь: **/account/{currency}**

Описание: **данная функция отвечает за возвращение баланса заданной валюты.**

Результат успешного выполнения: **возвращается сумма на открытом счету, сумма на закрытом счету и таймштамп.**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Currency | string | обязательный | валюта, баланс которой требуется получить |

Пример запроса:
```<json>
{
  "Currency": "USDT",
}
```

Пример ответа:
```<json>
{
    "Success": true,
    "Result": {
        "Balance": 10000.00,
        "Locked": 3500.05,
        "Time": "2023-09-15T09:47:29.2933083Z"
    }
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```

## Пополнение криптовалюты

### `Сгенерировать адрес с внутренним id`

! ВАЖНО: НА ДАННЫЙ МОМЕНТ ЗАПРОС РАБОТАЕТ ТОЛЬКО ДЛЯ СЕТИ **TRC20** ! 

Тип запроса: **POST**

Путь: **deposit/generate_address**

Описание: **генерация адреса для депозита.**

Результат успешного выполнения: **возвращается json с идентификатором адреса, криптовалютным адресом и датой выполнения данной функции.**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Blockchain | string | обязательный | название сети |

Пример запроса:
```<json>
{
  "Blockchain": "TRC20"
}
```

Пример ответа:
```<json>
{
    "Success": true,
    "Result": {
        "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
        "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
        "Time": "2023-09-15T09:49:11.8341287Z"
    }
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Code": 401,
    "DescriptionCode": 401,
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```

### `Получить историю пополнений`

! ВАЖНО: НА ДАННЫЙ МОМЕНТ ЗАПРОС РАБОТАЕТ ТОЛЬКО ДЛЯ СЕТИ **TRC20** ! 

Тип запроса: **GET**

Путь: **deposit/history**

Описание: **данная функция отвечает за получение истории пополнения, также можно задать опциональные параметры для получения информации за конкретные периоды**

Результат успешного выполнения: **возвращается список данных, каждая запись содержит в себе информацию: об адресе, об идентификаторе адреса, о хеше транзакции, о сети, о названии валюты, о сумме, о статусе и о времени**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| AddressId | string | обязательный | - | идентификатор адреса |
| Blockchain | string | обязательный | - | название сети |
| Limit | int? | опциональный | 100 | количество выгружаемых операций |
| Offset | int? | опциональный | 0 | параметр, отвечающий за пропуск некоторого числа первых сделок |
| FromDate | DateTime? | опциональный | DateTime.MinValue | нижний лимит выборки по времени создания операции |
| ToDate | DateTime? | опциональный | DateTime.MaxValue | верхний лимит выборки по времени создания операции |

Статусы заявки:

- Pending (ожидание исполнения операции);
- Executed (операция исполнена);
- Cancelled (операция отменена).

Пример запроса:
```<json>
{
  "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
  "Blockchain": "TRC20",
  "Limit": "250",
  "Offset": "50",
  "FromDate": "2023-03-05T10:25:06.6590684Z",
  "ToDate": "2023-09-16T10:25:06.6590684Z",
}
```

Пример ответа:
```<json>
{
    "Success": true,
    "Result": [
        {
            "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
            "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
            "Txid": "6d58e075ff11c423a533a0b986238a36e60a41ef7716ef8393384f3b955e5a04",
            "Blockchain": "TRC20",
            "Currency": "USDT",
            "Amount": 12950.59,
            "Status": "Executed",
            "Time": "2023-09-15T09:49:40.3404432Z"
        },
        {
            "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
            "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
            "Txid": "431498142770f180c5f1b808c9c070656461ea766496da8321507100b35a5776",
            "Blockchain": "TRC20",
            "Currency": "USDT",
            "Amount": 501.42,
            "Status": "Pending",
            "Time": "2023-09-15T09:49:40.3404895Z"
        },
        {
            "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
            "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
            "Txid": null,
            "Blockchain": "TRC20",
            "Currency": "USDT",
            "Amount": 1792.64,
            "Status": "Cancelled",
            "Time": "2023-09-15T09:49:40.3404899Z"
        }
    ]
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```

## Вывод криптовалюты

### `Вывод криптовалюты`
> ОБРАТИТЕ ВНИМАНИЕ! ВРЕМЕННО НЕДОСТУПНО!

Тип запроса: **POST**

Путь **withdraw/send**

Описание: **данная функция отвечает за вывод криптовалюты, для ее работы нужно передать в качестве параметров следующие значения: адрес кошелька, на который переводятся средства, название сети, сумма и токен.**

Результат успешного выполнения: **возвращается идентификатор операции, таймштамп и статус.**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно;**
- **вы указали id несуществующего кошелька;**
- **вы ввели название несуществующей или неподдерживающейся платформой криптовалюты;**
- **вы ввели название несуществующей или неподдерживающейся платформой сети.**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| AddressTo | string | обязательный | адрес кошелька на который переводится криптовалюта |
| Blockchain | string | обязательный | название сети |
| Amount | decimal | обязательный | сумма операций |
| Token | string | обязательный | значение токена |

Статусы заявки:

- Pending (ожидание исполнения операции);
- Executed (операция исполнена);
- Cancelled (операция отменена).

Пример запроса:
```<json>
{
    "AddressTo" : "TYb3dNMA6v75B7Fi3d1ckjXrHEBxEBYj42",
    "Blockchain": "TRC20",
    "Amount": "10945.00",
    "Token": "USDT"
}
```

Пример ответа:
```<json>
{
    "Success": true,
    "Result": {
        "OperationId": "bb91595b-e19d-4b72-aa30-7c12a681abd0",
        "Status": "Pending",
        "Time": "2023-09-15T09:50:13.0719673Z"
    }
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```

### `Получить историю выводов`
> ОБРАТИТЕ ВНИМАНИЕ! ВРЕМЕННО НЕДОСТУПНО!

! ВАЖНО: НА ДАННЫЙ МОМЕНТ ЗАПРОС РАБОТАЕТ ТОЛЬКО ДЛЯ СЕТИ **TRC20** ! 

Тип запроса: **GET**

Путь: **withdraw/history**

Описание: **данная функция отвечает за получение истории вывода, также можно задать опциональные параметры для получения информации за конкретные периоды**

Результат успешного выполнения: **возвращается список данных, каждая запись содержит в себе информацию: об адресе, о хеше транзакции, о названии валюты, о сумме, о комиссии, о статусе и о времени**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| Address | string | опциональный | - | адрес |
| Blockchain | string | опциональный | - | название сети |
| Limit | int? | опциональный | 100 | количество выгружаемых операций |
| Offset | int? | опциональный | 0 | параметр, отвечающий за пропуск некоторого числа первых сделок |
| FromDate | DateTime? | опциональный | DateTime.MinValue | нижний лимит выборки по времени создания операции |
| ToDate | DateTime? | опциональный | DateTime.MaxValue | верхний лимит выборки по времени создания операции |

Статусы заявки:

- Pending (ожидание исполнения операции);
- Executed (операция исполнена);
- Cancelled (операция отменена).

Пример запроса:
```<json>
{
    "Address": "TYb3dNMA6v75B7Fi3d1ckjXrHEBxEBYj42",
    "Blockchain": "TRC20",
    "Limit": "250",
    "Offset": "50",
    "FromDate": "2023-03-05T10:25:06.6590684Z",
    "ToDate": "2023-09-16T10:25:06.6590684Z"
}
```
Пример ответа:
```<json>
{
    "Success": true,
    "Result": [
        {
            "Address": "TYb3dNMA6v75B7Fi3d1ckjXrHEBxEBYj42",
            "Txid": "6d58e075ff11c423a533a0b986238a36e60a41ef7716ef8393384f3b955e5a04",
            "Currency": "USDT",
            "Amount": 12950.59,
            "Blockchain": "TRC20",
            "Fee": 2.0,
            "Status": "Executed",
            "Time": "2023-09-15T10:24:16.3197628Z"
        },
        {
            "Address": "TYb3dNMA6v75B7Fi3d1ckjXrHEBxEBYj42",
            "Txid": "431498142770f180c5f1b808c9c070656461ea766496da8321507100b35a5776",
            "Currency": "USDT",
            "Blockchain": "TRC20",
            "Amount": 501.42,
            "Fee": 2.0,
            "Status": "Pending",
            "Time": "2023-09-15T10:24:16.3198069Z"
        },
        {
            "Address": "TYb3dNMA6v75B7Fi3d1ckjXrHEBxEBYj42",
            "Txid": null,
            "Currency": "USDT",
            "Blockchain": "TRC20",
            "Amount": 1792.64,
            "Fee": 2.0,
            "Status": "Cancelled",
            "Time": "2023-09-15T10:24:16.3198072Z"
        }
    ]
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```
## Callbacks:
> ОБРАТИТЕ ВНИМАНИЕ! ВРЕМЕННО НЕДОСТУПНО!
> Адрес callback настраивается в личном кабинете beribit.com

### `Callback после факта пополнения на указанный адрес с id`
```<json>
{
    "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
    "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
    "Txid": "6d58e075ff11c423a533a0b986238a36e60a41ef7716ef8393384f3b955e5a04",
    "Blockchain": "TRC20",
    "Currency": "USDT",
    "Amount": 12950.59,
    "Status": "Executed",
    "Time": "2023-09-15T09:49:40.3404432Z"
}
```
### `Callback после факта изменения статуса выплаты`
```<json>
{
    "Address": "TYb3dNMA6v75B7Fi3d1ckjXrHEBxEBYj42",
    "Txid": null,
    "Currency": "USDT",
    "Blockchain": "TRC20",
    "Amount": 12950.59,
    "Fee": 2.0,
    "Status": "Cancelled",
    "Time": "2023-09-15T10:24:16.3197628Z" 
}
```
### `Перевод валюты внутри платформы`

Тип запроса: **POST**

Путь: **withdraw/internal**

Описание: **данная функция отвечает за перевод внутри платформы между клиентами**

Результат успешного выполнения: **возвращается код перевода (уникальный номер)**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| UserToId | string | обязательный | - | Beribit ID клиента (Доступен в разделе 'внутренние переводы' (https://beribit.com/wallet/codes)|
| Amount | string | обязательный | - | Количество валюты |
| Token | string | обязательный | - | Токен |

Список токенов:

- RUB;
- USDT;
- BTC;
- ETH;
- BNB
- TRX.

Пример запроса:
```<json>
{
    "Token": "USDT",
    "Amount": 100.00,
    "userToId": "PDBW8MWCFMB"
}
```
Пример ответа:
```<json>
{
    "Success": true,
    "Result": "32e2e57c-1e02-4b7c-77ff-bfb680d9374c",
    "Time": "2023-10-20T12:54:31.9614616Z"
}
```

Пример ошибки:

```<json>
{
    "Success": false,
    "Error": {
        "Message": "Insufficient funds"
    },
    "Time": "2023-10-20T12:53:32.316102Z"
}
```
### `Получение курса из стаканов`

Тип запроса: **GET**

Путь: **depth/get-all**

Описание: **данная функция отвечает за получение актуальных курсов прямо из стаканов**

Результат успешного выполнения: **возвращается список данных, каждая запись содержит в себе название стакана и курс обмена**

Пример ответа:
```<json>
[
    {
        "symbol": "USDT_RUB",
        "price": 96.9
    },
    {
        "symbol": "ETH_USDT",
        "price": 1674.34
    },
    {
        "symbol": "BTC_USDT",
        "price": 30609.01
    },
    {
        "symbol": "BNB_USDT",
        "price": 219.48
    },
    {
        "symbol": "TRX_USDT",
        "price": 0.0902
    }
]
```
