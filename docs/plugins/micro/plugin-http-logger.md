---
layout: doc/document
title: Http-Logger компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Http-Logger

Компонент необходим для логирования запросов/ответов http.

## Установка

###### Компонент включен в состав [micro/plugin-http-pack](/docs/plugins/micro/plugin-http-pack).

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-exceptions
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`

По умолчанию устанавливает 

## Конфигурация

  * `MICRO_HTTP_LOGGER_ACCESS` - указатель на логер для журналирования запросов.
  * `MICRO_HTTP_LOGGER_ERROR` - указатель на логер, для журналирования исключений.
  * `MICRO_HTTP_LOGGER_HEADERS_SECURED` - заголовки, которые не должны отражаться в логах. По умолчанию: `authorization`.
  * `MICRO_HTTP_LOGGER_ACCESS_FORMAT` - формат сообщений для журналирования запросов. По умолчанию `{{remote_addr}} - {{remote_user}} [{{status}}] "{{request}}" {{request_header.http-referer}} {{request_header.user-agent}}'`
  * `MICRO_HTTP_LOGGER_ERROR_FORMAT` - формат сообщений для журналирования исключений. По умолчанию `{{remote_addr}} - {{remote_user}} [{{status}}] "{{request}}"`

[Полная конфигурация](https://github.com/Micro-PHP/http-logger/blob/master/src/HttpLoggerPluginConfiguration.php)
