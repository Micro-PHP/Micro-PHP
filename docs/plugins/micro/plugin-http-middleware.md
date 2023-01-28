---
layout: doc/document
title: HTTP Middleware компонент
show_sidebar: false
menubar: docs_menu
---
## Описание

Компонент предоставляет возможность вмешиваться в логику выполнения маршрута вне зависимости от того был ли он найден.

## Установка

###### Компонент включен в состав [micro/plugin-http-pack](/docs/plugins/micro/plugin-http-pack).

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-middleware
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


### Применение.

Плагин дает возможность внедрения какого-либо вне зависимости от того будет ли найден маршрут, либо нет. Внедренный код будет выполнен до того как будет выполнен коллбэк контроллера. 

Метод `HttpMiddlewarePluginInterface::getRequestMatchPath` возвращает match паттерн.
Метод `HttpMiddlewareOrderedPluginInterface::getMiddlewarePriority` возвращает приоритет выполнения конкретного middleware.


Чтобы внедрить middleware, вам необходимо создать (либо использовать существующие плагины) и имплементировать один из интерфейсов

  * [HttpMiddlewareOrderedPluginInterface](https://github.com/Micro-PHP/http-middleware/blob/master/src/Plugin/HttpMiddlewareOrderedPluginInterface.php)
  * [HttpMiddlewarePluginInterface](https://github.com/Micro-PHP/http-middleware/blob/master/src/Plugin/HttpMiddlewarePluginInterface.php)

Для удобства, советуем использовать специально подготовленный трейт [MiddlewareAllowAllTrait](https://github.com/Micro-PHP/http-middleware/blob/master/src/Plugin/MiddlewareAllowAllTrait.php)

#### Пример использования

```php
<?php

use Micro\Plugin\Http\Plugin\HttpMiddlewarePluginInterface;
use Micro\Plugin\Http\Exception\HttpUnauthorizedException

class HttpMiddlewareAccessPlugin implements HttpMiddlewarePluginInterface
{
  public function getRequestMatchMethods(): array
  {
        return ['post', 'patch', 'put', 'delete'];
  }
  
  public function getRequestMatchPath(): string
  {
    return '^/user'; // match pattern
  }

  public function processMiddleware(Request $request): void
  {
    throw new HttpUnauthorizedException();
  }
}

/**
 * Request example
 */

use Micro\Plugin\Http\Facade\HttpFacadeInterface;

$request = Request::create('/user/123', 'POST', [ 'username' => 'changed', ]);
$container->get(HttpFacadeInterface::class)->execute($request); // Throws HttpUnauthorizedException

```


