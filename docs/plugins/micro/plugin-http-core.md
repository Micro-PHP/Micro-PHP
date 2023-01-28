# Компонент HTTP-Core

Данный компонент отвечает за коммуникацию между сервером и клиентом по протоколу HTTP.

## Установка

###### Компонент включен в состав [micro/plugin-http-pack](/docs/plugins/micro/plugin-http-pack).

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-http-core
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


## Интерфейс

Плагин HTTP-Core предоставляет простой интерфейс фасада `Micro\Plugin\Http\Facade\HttpFacadeInterface` для маршрутизации и выполнения HTTP запросов.

#### Методы

| Метод  	                  |      Аргументы  	| Возвращаемый Тип | Описание |
|---	                      |---	              |---	             |---	      |
|  getDeclaredRoutesNames 	|  - 	              | string[]         | Список декларированных маршрутов   	      |
|  createRouteBuilder       |  -                | [Micro\Plugin\Http\Business\Route\RouteBuilderInterface](#RouteBuilderInterface) |  |
|  match                    | Symfony\Component\HttpFoundation\Request $request| RouteInterface | Возвращает маршрут, который подходит для конкретного запроса. |
|  execute                  | Symfony\Component\HttpFoundation\Request $request <br /> bool $flush = true | Symfony\Component\HttpFoundation\Response |Возвращает результат запроса. При flush=true, запрос сразу отправляется в output через Response::send()|
| generateUrlByRouteName    | string $routeName <br /> array\|null $parameters = []) | string | Генератор пути по имени маршрута. Параметры необходимы для определения переменных динамического маршрута. |

## Маршрутизация

  *  [micro/plugin-http-router-code](/docs/plugins/micro/plugin-http-router-code)  Регистрация маршрутов на основе интерфейса плагина.




