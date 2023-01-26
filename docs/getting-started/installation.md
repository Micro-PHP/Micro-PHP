---
layout: doc/document
title: Установка
show_sidebar: false
menubar: docs_menu
---

## Минимальные требования
Минимальная версия PHP не ниже **8.2**

## Установка

Рекомендовано использовать [docker environment](/docs/getting-started/docker). для установки, но это не обязательно.

Либо вы можете использовать [composer](https://getcomposer.org/).

```shell
$ composer create-project micro/micro {directory} --remove-vcs
$ cd {directory}
$ php -S localhost:10000 -t public/
```

Откройте `http://localhost:10000` и вы увидите простой `hello world` ответ.

А так же, у вас есть возможность использовать инструменты командной строки
```bash
php bin/console
``` 

## Запуск тестов

Тесты - неотъемлемая часть любого профессионального ПО.
По умолчанию уже интегрированы `phpstan`, `psalm`, `phpunit`.

Для запуска тестов выполните команду 
```bash
$ composer run-script test-all
``` 

## Что дальше ?
Давайте попробуем [создать наше первое приложение](/docs/getting-started/building-first-app).

