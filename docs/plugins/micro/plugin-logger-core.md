---
layout: doc/document
title: Logger Core компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Logger Core

Компонент предоставляет абстракцию для логирования. Подключаемый логер должен соответствовать [PSR-3](https://www.php-fig.org/psr/psr-3/) формату.

## Установка

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-logger-core
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


## Концигурация

  * `LOGGER_{logger name}_LOG_LEVEL` - уровень логирования.
  * `LOGGER_{logger name}_PROVIDER_TYPE` - имя адаптера. 

## Адаптеры

  * [micro/plugin-logger-monolog](/docs/plugins/micro/plugin-logger-monolog)

## Имплементация адаптера

Для этого вам необходимо создать [плагин](/docs/architecture/plugins) и имплементировать в нем интерфейс [Micro\Plugin\Logger\PluginLoggerProviderPluginInterface](https://github.com/Micro-PHP/plugin-logger/blob/master/src/Plugin/LoggerProviderPluginInterface.php).

