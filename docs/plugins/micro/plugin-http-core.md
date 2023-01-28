---
layout: doc/document
title: Http-Core компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент HTTP-Core

Данный компонент отвечает за коммуникацию между сервером и клиентом по протоколу HTTP.

## Установка

###### Компонент включен в состав [micro/plugin-http-pack](/docs/plugins/micro/plugin-http-pack).

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-core
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


### Интерфейс

Плагин HTTP-Core предоставляет простой интерфейс фасада [Micro\Plugin\Http\Facade\HttpFacadeInterface](https://github.com/Micro-PHP/plugin-http-core/blob/master/src/Facade/HttpFacadeInterface.php) для маршрутизации и выполнения HTTP запросов.
