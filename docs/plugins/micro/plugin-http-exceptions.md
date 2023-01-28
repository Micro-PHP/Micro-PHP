---
layout: doc/document
title: Http-Exceptions компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Http-Exceptions

Компонент приводит любые исключения к `Micro\Plugin\Http\Exception\HttpException` и генерирует ответ с необходимым статусом.
По умолчанию, если приложение выбросило любое исключение (кроме HttpException), то выбрасывается HttpException, где в роли `previous` служит оригинальный объект исключения.

## Установка

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-exceptions
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`
