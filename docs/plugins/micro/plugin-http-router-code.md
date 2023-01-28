---
layout: doc/document
title: HTTP Router Code компонент
show_sidebar: false
menubar: docs_menu
---

## Установка

###### Компонент включен в состав [micro/plugin-http-pack](/docs/plugins/micro/plugin-http-pack).

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-router-code
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


### Создание маршрутов.

Для этого необходимо создать плагин (либо использовать существующие) и имплементировать в нем интерфейс [Micro\Plugin\Http\Plugin\RouteProviderPluginInterface](https://github.com/Micro-PHP/http-router-code/blob/master/src/Plugin/RouteProviderPluginInterface.php).
Обратите внимание, маршрутизатор не ограничен лишь одним классом. Их может быть неограниченное количество.
В случае попытки зарегистрировать 2 маршрута с одинаковыми именами, будет вызвано исключение `Micro\Plugin\Http\Exception\RouteAlreadyDeclaredException`

#### Пример использования

```php
<?php

use Micro\Plugin\Http\Plugin\RouteProviderPluginInterface;
use use Micro\Plugin\Http\Facade\HttpFacadeInterface;

class ApplicationRouterPlugin implements RouteProviderPluginInterface
{
  public function provideRoutes(HttpFacadeInterface $http): iterable
  {
    $builder = $http->createRouteBuilder();
    
    // Alowed by default: GET, POST, PUT, PATCH, DELETE
    yield $builder
      ->setName('home')
      ->setController([HomeController::class, 'index']) // Set controller class and method
      ->setUri('/')
      ->build();
      
    yield $builder
      ->setName('post')
      ->setMethods(['GET'])
      ->setController(PostController::class) // If the method is not set, the route name is used by default
      ->setUri('/post/{post_name}')
      ->build();

    yield $builder
      ->setName('hello')
      ->setController(fn(Request $request) => new Response('Hello, ' . $request->get('recipient_name'))) // If the method is not set, the route name is used by default
      ->setUri('/hello/{recipient_name}')
      ->build();
  }
}

```


