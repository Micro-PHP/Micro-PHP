---
layout: doc/document
title: Framework components status
show_sidebar: false
menubar: docs_menu
---

# Пример создания приложения на Micro Framework.

В данной статье я расскажу вам как создать простое приложение, которое с помощью [Packagist API](https://packagist.org/apidoc#search-packages) выполняет поиск библиотек и выводит результаты в браузер и консоль.

## Установка

###### Минимальная версия php - 8.2

Для начала, давайте установим MF.
Для этого воспользуемся [composer](https://composer.org).

```shell
$ composer require micro/skeleton Demo
```

У нас появился пустой проект и в нем мы увидим следующую структуру.

`bin/console` - утилита для работы с интерфейсом командной строки

`etc/plugins.php`  - *здесь можно найти список плагинов, которые включены в проект.

`public/index.php` - это точка входа интерфейса HTTP.

`.env` - *файл с конфигурацией проекта.

`src` - Исходный код нашего приложения. Давайте загянем внутрь.

## Предлагаемая, но необязательная структура каталогов.

Сейчас мы находимся внутри каталога `src` и видим схему дирректорий:

`Shared` - этот каталог содержит в себе набор публичных интерфейсов,
    которые будут реализованы в бизнес-слое нашего приложения.

`Business` Здесь расположен бизнес-слой приложения.
    Основная его задача - выполнение бизнес логики приложения.
    Внутри будут находится реализации публичных интерфейсов из каталога `Shared`

`View` - Здесь расположены плагины, которые обеспечивают доступ к шаблонам.
    По умолчанию мы будем использовать [micro/plugin-twig](/docs/plugins/micro/plugin-twig)

`Communication` - Данный каталог содержит в себе плагины, которые отвечают за I/O.
     - Для работы с HTTP мы будем использовать [micro/http-pack](/docs/plugins/micro/http-pack)
     - Для коммуникации через cli, мы используем [micro/plugin-console](/docs/plugins/micro/plugin-console)

`Plugin` - Здесь располагаются мета-плагины,
    которые обеспечивают подключение зависимых плагинов через интерфейс `Micro\Framework\Kernel\Plugin\PluginDependedInterface`

## Бизнес-слой.

Давайте определим то, что будет делать наше приложение. Как я писал выше, мы хотим выполнять простой поиск по репозиторию Packagist
и выводить результаты в интерфейс консоли и браузер.
Для этой задачи нам необходимо будет определить интерфейс. Чтож, давайте это сделаем.

В каталоге `src/Shared/Packagist/Search/` создадим необходимый нам интерфейс.

```php
<?php

declare(strict_types=1);

namespace App\Shared\Packagist\Search;

interface PackagistSearchInterface
{
    public function search(string $query): array;
}
```

А так же нам понадобится Facade для доступа к данному сервису.
Давайте создадим его интерфейс в `src/Shared/Packagist`

```php
<?php

declare(strict_types=1);

namespace App\Shared\Packagist;

use App\Shared\Packagist\Search\PackagistSearchInterface;

interface PackagistFacadeInterface extends PackagistSearchInterface
{
}
```

Инетрфейс фасада позволяет облегчить понимание того, как работает сервис.
Фасад может включать в себя множество интерфейсов из которых и будет состоять сервис, 
который в дальнейшем мы будем передавать в контейнер. Но об этом чуть позже.

### Имплементация бизнес-логики на основе абстракций.

Мы определили инетрфейсы наших сервисов, теперь неплохо было бы заставить их работать.
Давайте имплементируем наш интерфейс `App\Shared\Packagist\Search\PackagistSearchInterface`

Для этого создадим следующий класс в `App\Business\Packagist\Search\PackagistSearch.php`

```php
<?php

declare(strict_types=1);

namespace App\Business\Packagist\Search;

use App\Business\Packagist\PackagistBusinessPluginConfiguration;
use App\Shared\Packagist\Search\PackagistSearchInterface;

readonly class PackagistSearch implements PackagistSearchInterface
{
    public function __construct(
        private PackagistBusinessPluginConfiguration $pluginConfiguration
    ) {
    }

    public function search(string $query): array
    {
        $query = trim($query);
        if (!$query) {
            return [];
        }

        $url = sprintf(
            '%s/search.json?q=%s',
            $this->pluginConfiguration->getPackagistUrl(),
            $query
        );

        $results = file_get_contents($url);
        if (false === $results) {
            throw new \RuntimeException(sprintf('Fetch query can not be processed right now `%s`', $url));
        }

        return json_decode($results, true);
    }
}
```

Наш сервис поиска готов. Внутри он реализован достаточно просто и вполне подойдет для демонстрации работы нашего приложения.
Однако, этого мало.
Теперь сервис нужно передавать в фасад, чтобы в дальнейшем его можно было передать в DI.

Давайте же создадим фасад нашего сервиса Packagist.
Создадим новый класс в каталоге `src/Business/Packagist/Facade/PackagistFacade.php` и наполним его следующим содеримым.

```php
<?php

declare(strict_types=1);

namespace App\Business\Packagist\Facade;

use App\Shared\Packagist\PackagistFacadeInterface;
use App\Shared\Packagist\Search\PackagistSearchInterface;

readonly class PackagistFacade implements PackagistFacadeInterface
{
    public function __construct(
        private PackagistSearchInterface $packageReceiver
    ) {
    }
    
    public function search(string $query): array
    {
        return $this->packageReceiver->search($query);
    }
}
```

Обратите внимание, в качестве аргумента конструктора мы передали реализацию сервиса `PackagistSearchInterface`.
Наш бизнес-слой готов, но все еще недоступен из DI/IoC контейнера.

### Предоставление сервиса в контейнер.

Для этого мы напишем плагин, который будет предоставлять данный фасад в контейнер.
Создадим новый класс в `src/Business/Packagist/PackagistBusinessPlugin.php`

```php
<?php

declare(strict_types=1);

namespace App\Business\Packagist;

use App\Business\Packagist\Facade\PackagistFacade;
use App\Business\Packagist\Search\PackagistSearch;
use App\Shared\Packagist\PackagistFacadeInterface;
use App\Shared\Packagist\Search\PackagistSearchInterface;
use Micro\Component\DependencyInjection\Container;
use Micro\Framework\Kernel\Plugin\ConfigurableInterface;
use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Framework\Kernel\Plugin\PluginConfigurationTrait;

/**
 * @method PackagistBusinessPluginConfiguration configuration()
 */
class PackagistBusinessPlugin implements DependencyProviderInterface, ConfigurableInterface
{
    use PluginConfigurationTrait;

    public function provideDependencies(Container $container): void
    {
        $container->register(PackagistFacadeInterface::class, function () {
            return $this->createFacade();
        });
    }

    protected function createFacade(): PackagistFacadeInterface
    {
        return new PackagistFacade($this->createPackagistReceiver());
    }

    protected function createPackagistReceiver(): PackagistSearchInterface
    {
        return new PackagistSearch($this->configuration());
    }
}
```

Давайте подробнее рассмотрим этот класс, а особенно обратим внимание на его интерфейсы.
  * `Micro\Framework\Kernel\Plugin\DependencyProviderInterface` - интерфейс сообщает, что плагину нужен Di\IoC контейнер. 
  * `Micro\Framework\Kernel\Plugin\ConfigurableInterface` - интерфейс сообщает о том, что плагин имеет конфигурацию.
    Обратите внимание на аннотацию `@method`. Она здесь необходима для удобства и указывает какой объект конфигурации вернет метод `configuration()`.

Т.к. мы делем запрос на удаленный URL, его нам необходимо предоставить.
Давайте напишем класс, в котором мы определим все необходимые параметры для работы плагина.

```php
<?php

declare(strict_types=1);

namespace App\Business\Packagist;

use Micro\Framework\Kernel\Configuration\PluginConfiguration;

class PackagistBusinessPluginConfiguration extends PluginConfiguration
{
    public const CFG_PACKAGIST_URL = 'PACKAGIST_URL';

    public function getPackagistUrl(): string
    {
        return rtrim($this->configuration->get(self::CFG_PACKAGIST_URL, 'https://packagist.org/'), '/');
    }
}
```
Мы не будем останавливаться сейчас на том, что такое `PluginConfiguration`, какие у него есть свойства и как можно манипулировать конфигами.
Подробнее об этом можно узнать в [здесь](/docs/plugins/micro/kernel-boot-configuration).

## Коммуникационный слой - CLI

Для начала нам необходимо создать плагин, который бы выполнял роль подключения комманд к CLI интерфейсу.

```php
<?php

declare(strict_types=1);

namespace App\Communication\Packagist;


class PackagistCommunicationPlugin
{
}
```

В дальнейшем мы вернемся к нему, а пока проследуем дальше.

Сейчас давайте имплементируем нашу команду, который будет отвечать за коммуникацию бизнес-слоя и CLI интерфейса.
Для этого перейдем в каталог `src/Communication/Packagist/Command` и заполним класс `PackagistCommand` следующим содержимым.

```php
<?php

declare(strict_types=1);

namespace App\Communication\Packagist\Command;

use App\Shared\Packagist\PackagistFacadeInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Helper\Table;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class PackagistCommand extends Command
{
    public function __construct(private readonly PackagistFacadeInterface $packagistFacade)
    {
        parent::__construct('packagist:search');
    }

    public function configure(): void
    {
        $this->addArgument('q', InputArgument::REQUIRED, 'Search query string.', null);
    }

    public function execute(InputInterface $input, OutputInterface $output): int
    {
        $query = $input->getArgument('q');
        $queryResult = $this->packagistFacade->search($query);
        $tableHeaders = [
            'name',
            'description',
            'favers',
        ];
        $tableRows = [];
        foreach ($queryResult['results'] as $result) {
            $row = [];
            foreach ($tableHeaders as $key) {
                $row[] = $result[$key];
            }

            $tableRows[] = $row;
        }

        $table = new Table($output);
        $table
            ->setHeaders($tableHeaders)
            ->setRows($tableRows)
        ;
        $table->render();

        return self::SUCCESS;
    }
}
```
###### Мы используем для работы с CLI компонет.[symfony/console](https://symfony.com/doc/current/components/console.html), который предоставляется в MF через компонент [micro/plugin-console](/docs/plugins/micro/plugin-console).

## Почти готово.

Теперь нам осталось сообщить приложению о том, какие плагины необходимо загружать в приложение.
Для этого в `etc/plugins.php` у нас содержится список классов, которые должны быть подключены в приложении.
Однако, этот список может быть крайне большим и управлять им будет крайне сложно.
Для этого мы позаботились о [механизме подключения зависимых плагинов](/docs/plugins/micro/kernel-boot-plugin-depended) и теперь можем не паковать список всех плагинов в одном месте.

Давайте же займемся этим.
С точки зрения обывателя, у нас должен быть модуль, который реализует полный цикл работы поиска в Packagist.
Для этого у нас существует папка `Plugins`, где мы создадим мета-пакет, который будет подключать все необходимые нам плагины.
Определим мета-плагин для коммуникации всех слоев логики плагина Packagist.
Для этого создадим класс `Plugin/Packagist/PackagistPlugin.php`

```php
<?php

declare(strict_types=1);

namespace App\Plugin\Packagist;

use App\Business\Packagist\PackagistBusinessPlugin;
use App\Communication\Packagist\PackagistCommunicationPlugin;
use App\View\Packagist\PackagistViewPlugin;
use Micro\Framework\Kernel\Plugin\PluginDependedInterface;

class PackagistPlugin implements PluginDependedInterface
{
    public function getDependedPlugins(): iterable
    {
        return [
            PackagistBusinessPlugin::class,
            PackagistCommunicationPlugin::class,
        ];
    }
}
```
Таким образом мы описали зависимости нашего плагина и предоставили набор компонентов в ядро.
###### Чуть позже мы еще сюда вернемся.

И следующий плагин, который нам понадобится `AppPlugin` в `src/Plugin`.

```php
<?php

declare(strict_types=1);

namespace App\Plugin;

use App\Plugin\Packagist\PackagistPlugin;
use App\View\Base\BaseViewPlugin;
use Micro\Framework\Kernel\Plugin\PluginDependedInterface;

class AppPlugin implements PluginDependedInterface
{
    public function getDependedPlugins(): iterable
    {
        return [
            PackagistPlugin::class,
        ];
    }
}
```

И давайте подключим этот плагин в систему через `etc/plugins.php`
```php
return [
/// **** other plugins */
    App\Plugin\AppPlugin::class
];
```

## Проверим, что у нас получилось.
Теперь наш плагин будет доступен из интерфейса командной строки.
Для этого выполним следующую команду
```shell
$ bin/console packagist:search micro/kernel
```

Если мы все сделали правильно, то увидим примерно следующее содержимое:
```shell
+-----------------------------------+---------------------------------------------------------------------------------+--------+
| name                              | description                                                                     | favers |
+-----------------------------------+---------------------------------------------------------------------------------+--------+
| micro/kernel-boot-plugin-depended | Micro Framework: Kernel Boot loader - component to provide dependencies         | 1      |
| micro/kernel-boot-dependency      | Micro Framework: Kernel Boot loader - component to provide dependencies         | 0      |
| micro/kernel-boot-configuration   | Micro Framework: Kernel Boot loader - component to provide plugin configuration | 0      |
| micro/kernel-app                  | Micro Framework: App Kernel component                                           | 2      |
| micro/kernel                      |                                                                                 | 2      |
+-----------------------------------+---------------------------------------------------------------------------------+--------+
```

## HTTP Интерфейс.
Для начала, давайте создадим контроллер `App\Communication\Packagist\Controller\PackagistSearchController`

```php
<?php

declare(strict_types=1);

namespace App\Communication\Packagist\Controller;

use App\Shared\Packagist\PackagistFacadeInterface;
use Micro\Plugin\Twig\TwigFacadeInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

readonly class PackagistSearchController
{
    public function __construct(
        private PackagistFacadeInterface $packagistFacade,
        private TwigFacadeInterface $twigFacade
    ) {
    }

    public function index(Request $request): Response
    {
        $query = (string) $request->get('q');
        $results = $this->packagistFacade->search($query);

        $rendered = $this->twigFacade->render('@Packagist/home.html.twig', [
            'query' => $query,
            'results' => $results,
        ]);

        return new Response($rendered);
    }
}
```
###### Как в конструктор, так и в метод контроллера можно передавать необходимые зависимости из DI автоматически.

И теперь давайте вернемся в класс `App\Communication\Packagist\PackagistCommunicationPlugin`
И немного его модифицируем.
```php
<?php

declare(strict_types=1);

namespace App\Communication\Packagist;

use App\Communication\Packagist\Controller\PackagistSearchController;
use Micro\Plugin\Http\Facade\HttpFacadeInterface;
use Micro\Plugin\Http\Plugin\RouteProviderPluginInterface;

class PackagistCommunicationPlugin implements RouteProviderPluginInterface
{
    public function provideRoutes(HttpFacadeInterface $httpFacade): iterable
    {
        yield $httpFacade
            ->createRouteBuilder()
            ->setController([PackagistSearchController::class, 'index'])
            ->setUri('/')
            ->setName('home')
            ->build();
    }
}
```

Мы имплементировали в класс интерфейс `Micro\Plugin\Http\Plugin\RouteProviderPluginInterface` и теперь компонент [micro/plugin-http-core](/docs/plugins/micro/plugin-http-core) знает о том, что 6этот плагин является провайдером маршрутов нашего приложения.
###### Это один из доступных вариантов определения маршрутизации. Подробнее обо всех возможностях читайте в [документации](/docs/plugins/micro/plugin-http-core).

И давайте попробуем посмотреть что получилось.
Для этого перейдем в папку `public/` и выполним `$ php -s localhost:8000`, после чего перейдем в браузер и увидим красиво оформленную в Symfony-style страницу с информацией об ошибке.

```
    There are no registered paths for namespace "Packagist".
```

Она возникла из-за того, что мы не подключили слой *View* приложения.
Давайте исправим это.
Перейдем в папку `src/View/Packagist` и создадим там плагин `PackagistViewPlugin`.

```php
<?php

declare(strict_types=1);

namespace App\View\Packagist;

use Micro\Plugin\Twig\Plugin\TwigTemplatePluginInterface;

class PackagistViewPlugin implements TwigTemplatePluginInterface
{
    public function getTwigTemplatePaths(): array
    {
        return [
            __DIR__.'/templates',
        ];
    }

    public function getTwigNamespace(): ?string
    {
        return 'Packagist';
    }

    public function isTwigTemplatesPrepend(): bool
    {
        return true;
    }
}
```

Интерфейс `Micro\Plugin\Twig\Plugin\TwigTemplatePluginInterface` сооббщает компоненту [micro/plugin-twig] о том, что этот плагин предоставляет шаблоны в формате [Twig](https://twig.symfony.com/).
Далее, в соответствующей папке ` __DIR__.'/templates'` мы можем реализовать необходимые нам шаблоны.
Готовые шаблоны нашего приложения можно посмотреть [здесь](https://github.com/Asisyas/microframework-packagist-demo/tree/master/src/View).

Когда мы закончили с `View`, подключим его плагин в `src/Plugin/PackagistPlugin.php` по аналогии с предыдущими.

Если мы все сделали правильно, то в браузере увидим результат нашего труда :)

### Заключение.
Структура каталогов и взаимодействие плагинов в статье может не соответствовать в реальных проектах, адаптируйте ее под свои нужды.

Исходный код проекта доступен на [GitHub](https://github.com/Asisyas/microframework-packagist-demo).

Всем спасибо!


