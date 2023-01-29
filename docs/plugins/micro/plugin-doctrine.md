---
layout: doc/document
title: Doctrine компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Doctrine

Компонент предоставляет интерфейс для работы с базами данных с помощью [ORM Doctrine](https://www.doctrine-project.org/).

## Установка

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-doctrine
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


## Концигурация

  * `ORM_CONNECTION_LIST` - список соединений с БД.
  * `ORM_{connection name}_DRIVER` - имя драйвера

Дополнительные конфигурации зависят от драйвера.

### Драйверы

##### pdo_mysql
  * `ORM_{connection name}_HOST` - по умолчанию `localhost`
  * `ORM_{connection name}_PORT` - по умолчанию `3306`
  * `ORM_{connection name}_DATABASE` - имя базы данных.
  * `ORM_{connection name}_USER` - имя пользователя
  * `ORM_{connection name}_PASSWORD` - пароль

##### pdo_sqlite
  * `ORM_{connection name}_PATH`
  * `ORM_{connection name}_IN_MEMORY`
  
##### pdo_pgsql
  * `ORM_{connection name}_HOST`
  * `ORM_{connection name}_PORT`
  * `ORM_{connection name}_DATABASE`
  * `ORM_{connection name}_USER` - имя пользователя
  * `ORM_{connection name}_PASSWORD` - пароль


## Дополнительная информация
  * Для описание свойств моделей необходимо [использовать атрибуты](https://www.doctrine-project.org/projects/doctrine-orm/en/2.14/reference/attributes-reference.html).
  * [Создание моделей и связей](https://www.doctrine-project.org/projects/doctrine-orm/en/2.14/reference/basic-mapping.html#basic-mapping)

## Утилиты для работы с cli
Подробное описание и набор команд можете посмотреть выполнив команду `php bin/console orm`

## Фасад
Фасад предоставляет доступ к EntityManager. 

##### Пример использования

```php
use Micro\Plugin\Doctrine\DoctrineFacadeInterface;

$facade = $container->get(DoctrineFacadeInterface::class);

$entityManager - $facade->getManager('example');
//Or
$defaultEntityManager = $facade->getDefaultManager();

$entity = new Entity();
$entityManager->persist($entity);
$entityManager->flush();
```
