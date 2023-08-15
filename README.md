# RabbitMQ для обработки платежей

## Общая информация

RabbitMQ - это брокер сообщений, который используется для передачи данных между различными компонентами системы. Основное преимущество RabbitMQ заключается в использовании "очередей" для хранения сообщений до их обработки. 

**Термины:**
- **Producer**: Это компонент (или приложение), который отправляет сообщения. В контексте системы обработки платежей, producer может отправлять детали транзакции.

- **Queue**: Очередь - это буфер, который хранит сообщения. RabbitMQ может хранить множество очередей, и каждая очередь идентифицируется своим уникальным именем.

- **Consumer**: Это компонент, который слушает очередь и обрабатывает поступающие сообщения. В контексте нашей системы, consumer будет обрабатывать детали транзакции, например, осуществляя платёж.

- **Теги и приоритеты**: RabbitMQ позволяет добавлять теги к сообщениям и устанавливать приоритеты, что делает систему очень гибкой. Например, вы можете пометить определенные платежи как "срочные", и обработать их в первую очередь.

RabbitMQ поддерживает множество протоколов обмена сообщениями и позволяет разрабатывать распределенные и отказоустойчивые системы. Он обеспечивает высокую пропускную способность и латентность, что делает его идеальным выбором для систем обработки платежей.

## Применение в процессинге платежей

RabbitMQ может быть использован для управления очередью транзакций, что делает обработку платежей асинхронной и масштабируемой. Это особенно полезно вслучаях, когда требуется обработка большого объема транзакций в короткие промежутки времени.

## Обоснование

Основные причины интеграции брокера сообщений в процессинг:
1. **Асинхронность**: RabbitMQ позволяет обрабатывать платежи асинхронно, увеличивая производительность и уменьшая время ожидания.
2. **Масштабируемость**: Систему можно легко масштабировать, добавляя новые узлы.
3. **Надежность**: В случае сбоя в одном из узлов, другие продолжат работать.
4. **Гибкость**: Можно легко интегрировать с различными платформами и языками программирования.


## Пример кода на JavaScript

### Отправка сообщения

```javascript
const amqp = require('amqplib/callback_api');

amqp.connect('amqp://localhost', function(error, connection) {
    if (error) {
        throw error;
    }
    connection.createChannel(function(error, channel) {
        if (error) {
            throw error;
        }
        const queue = 'transaction_queue';
        const msg = 'Transaction Details';

        channel.assertQueue(queue, {
            durable: false
        });
        channel.sendToQueue(queue, Buffer.from(msg));

        console.log("Sent 'Transaction Details'");
    });
    setTimeout(function() {
        connection.close();
    }, 500);
});
```

### Получение сообщения

```javascript
const amqp = require('amqplib/callback_api');

amqp.connect('amqp://localhost', function(error, connection) {
    if (error) {
        throw error;
    }
    connection.createChannel(function(error, channel) {
        if (error) {
            throw error;
        }
        const queue = 'transaction_queue';

        channel.assertQueue(queue, {
            durable: false
        });

        console.log('Waiting for messages. To exit press CTRL+C');
        
        channel.consume(queue, function(msg) {
            console.log(`Received ${msg.content.toString()}`);
        }, {
            noAck: true
        });
    });
});
```

## Пример кода на Python

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='transaction_queue')

channel.basic_publish(exchange='', routing_key='transaction_queue', body='Transaction Details')
print("Sent 'Transaction Details'")

connection.close()
```

## Полезные ресурсы

1. **[Официальная документация RabbitMQ](https://www.rabbitmq.com/)**: Лучший источник для глубокого понимания работы RabbitMQ.
2. **[Библиотека `amqplib` для Node.js](https://www.npmjs.com/package/amqplib)**: Основной инструмент для работы с RabbitMQ на JavaScript.
3. **[Туториалы RabbitMQ](https://www.rabbitmq.com/getstarted.html)**: Набор туториалов для разных языков программирования.
4. **[Форум сообщества RabbitMQ](https://groups.google.com/forum/#!forum/rabbitmq-users)**: Место, где можно задать вопросы и получить помощь от сообщества.
