---
layout: doc/document
title: Logger Monolog компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Logger Monolog

Адаптер для [micro/plugin-logger-core], который предоставляет возможность использования [Monolog](https://seldaek.github.io/monolog/)

## Установка

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-logger-monolog
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`

## Принцип работы.

Логер - это объект, имплементирующий интерфейс логера формата [PSR-3](https://www.php-fig.org/psr/psr-3/).
Хендлер - это объект, который пишет сообщения логера в какой-либо источник.

Логер может иметь множество хендлеров.
Например, если логи нужно одновременно складировать в файловую систему и отправлять на сторонний ресурс, например, Graylog, Datadog, Amqp и т.д.

На сегодняшний день реализован только StreamHandler.

## Конфигурация

  * `LOGGER_LIST` - Список логеров через запятую. По умолчанию `default`
  * `LOGGER_HANDLER_LIST` - Список хендлеров.
  * `LOGGER_{loggern name}_HANDLERS`- список хендлеров для конкретного логера.
  * `LOGGER_HANDLER_{handler name}_TYPE` - тип хендлера. На текущий момент доступен только `stream`.
  * `LOGGER_{logger name}_LOG_LEVEL` - Уровень логов.
  * `LOGGER_{logger name}_PROVIDER_TYPE` - Провайдер логов. Данный плагин - это провайдер имеет названием `monolog`. 

#### Handler Stream
  * `LOGGER_HANDLER_{handler name}_FILE` - путь до файла с 
  * `LOGGER_HANDLER_{handler name}_USE_LOCKING`

## Пример использования.

Пример конфигурации
```dotenv

LOGGER_LIST=example
LOGGER_HANDLER_LIST=local

LOGGER_HANDLER_LOCAL_TYPE=stream
LOGGER_HANDLER_LOCAL_FILE=/var/log/example.log

LOGGER_EXAMPLE_HANDLERS=local
LOGGER_EXAMPLE_LOG_LEVEL=debug
LOGGER_EXAMPLE_PROVIDER_TYPE=monolog
```
Пример использования
```php

$loggerFacade = $container->get(Micro\Plugin\Logger\Facade\LoggerFacadeInterface::class);
$logger = $loggerFacade->getLogger('example');
$logger->info('Log message');

```


