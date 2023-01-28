---
layout: doc/document
title: Http-Exceptions-Dev компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Http-Exceptions-Dev

Компонент показывает страницу с трассировкий исключений.
Компонент активен только в случае если переменная конфигурации приложения `APP_ENV` начинается с `dev`.  


## Установка

###### Компонент включен в состав [micro/plugin-http-pack](/docs/plugins/micro/plugin-http-pack).

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-exceptions-dev
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`
