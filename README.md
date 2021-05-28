# Техническая документация v2

Содержание:

* [Методы получения уведомлений](#01)
* [Отладка получения уведомлений](#02)

## Методы получения уведомлений {#01}

### `receiveNotification`

Метод выполняет получение очередного уведомления из очереди.

Метод ожидает в течение 20 сек получения нового уведомления. Если в течение 20 сек новых уведомлений не поступает, то метод завершается с кодом 200 (ОК).
Если в течение 20 сек в пуле появляется уведомление, то вызов метода завершается сразу же, без ожидания.

После успешной обработки уведомления требуется вызывать метод `deleteNotification`, чтобы подтвердить обработку уведомления и перейти к следующему уведомлению.

```
GET {{host}}/waInstance{{idInstance}}/receiveNotification/{{apiTokenInstance}}
```

Параметры:

 - нет

Пример запроса:

```
GET https://api.green-api.com/waInstance33012345/receiveNotification/e8dc45b249b606615485432f5bb5e29166a7502e7e5ecfeb12
```

Пример ответа:

```json
{
    "receiptId": 488645,
    "body": {
        "typeWebhook": "outgoingMessageReceived",
        "instanceData": {
            "idInstance": 33012345,
            "wid": "79001234567@c.us",
            "typeInstance": "whatsapp"
        },
        "timestamp": 1588091580,
        "idMessage": "F7AEC1B7086ECDC7E6E45923F5EDB825",
        "senderData": {
            "chatId": "79001234568@c.us",
            "sender": "79001234568@c.us",
            "senderName": "Green API"
        },
        "messageData": {
            "typeMessage": "textMessage",
            "textMessageData": {
                "textMessage": "I use Green-API to send this message to you!"
            }
        }
    }
}
```

- receiptId - число - номер квитанции для последующего удаления уведомления методом deleteNotification
- typeWebhook - тип уведомления; для сообщений на отправку значение outgoingMessageReceived
- instanceData - данные об аккаунте; можно игнорировать
- idMessage - идентификатор сообщения; по данному идентификатору ожидаем получать статусы отправлено/доставлено/прочитано/нет воцап/ошибка
- senderData - данные об отправителе; можно игнорировать
- messageData - полезные данные для отправки сообщения
- messageData.typeMessage - тип сообщения (текст, видео, файл, картинка, звук, голос, контакт, геолокация и пр);
- messageData.textMessageData.textMessage - текст сообщения

### `deleteNotification`

Метод выполняет удаление уведомления из очереди. Вызов метода подтверждает успешную обработку уведомления.

```
DELETE {{host}}/waInstance{{idInstance}}/deleteNotification/{{apiTokenInstance}}/{{receiptId}}
```

Параметры:

- receiptId - число - номер квитанции, полученный методом `receiveNotification`

Пример запроса:

```
DELETE https://api.green-api.com/waInstance33012345/deleteNotification/e8dc45b249b606615485432f5bb5e29166a7502e7e5ecfeb12/488645
```

Пример ответа:

```json
{
    "result": true
}
```

- result - булево - удалось или нет удалить уведомление из очереди


## Отладка получения уведомлений {#02}

Для публикации уведомления в очередь требуется отправить POST запрос на адрес `https://webhook.green-api.com`. Далее это уведомление можно будет получить из очереди методом `receiveNotification` в порядке FIFO.

Тело запроса:

```json
{
    "typeWebhook": "outgoingMessageReceived",
    "instanceData": {
        "idInstance": 33012345,
        "wid": "79001234567@c.us",
        "typeInstance": "whatsapp"
    },
    "timestamp": 1588091580,
    "idMessage": "F7AEC1B7086ECDC7E6E45923F5EDB825",
    "senderData": {
        "chatId": "79001234568@c.us",
        "sender": "79001234568@c.us",
        "senderName": "Green API"
    },
    "messageData": {
        "typeMessage": "textMessage",
        "textMessageData": {
            "textMessage": "I use Green-API to send this message to you!"
        }
    }
}
```
