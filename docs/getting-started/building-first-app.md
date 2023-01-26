---
layout: doc/document
title: Framework components status
show_sidebar: false
menubar: docs_menu
---
# Пример создания приложения на Micro Framework.

В данной статье я расскажу вам как создать простое приложение, которое с помощью [Packagist API](https://packagist.org/apidoc#search-packages) выполняет поиск библиотек и выводит результаты в браузер и консоль.
###### Пример составлен на базе предустановленных компонентах из [micro/micro](https://micro-php/docs/plugins/micro/micro).

## Установка

Есть 2 варианта, которые 

Для начала, давайте установим MF.
Для этого воспользуемся [composer](https://composer.org)

###### Обратите внимание, для запуска MF требуется минмальная версия php 8.2

```shell
$ composer create-project micro/skeleton Demo --remove-vcs
```
Либо можем использовать готовое Docker окружение.
###### Убедитесь, что у вас установлена свежая версия [Docker Compose](https://docs.docker.com/compose/install/) (v2.10+)
```shell
$ git clone git@github.com:Micro-PHP/micro-docker.git App
$ cd App
$ rm -rf .git/
$ git init
$ make build && make up
```

##### После установки, нас особенно будут интересовать следующие каталоги и файлы

`src` - исходный код приложения

`public` - точка входа в приложение по протолоку HTTP

`assets` - исходный код front (css/js/statics)

`bin/console` - утилита для работы с командной строкой

`etc/` - место хранения служебных файлов. Например, `etc/plugins.php` содердит список подключенных плагинов по умолчанию.

`.env` (`env.{APP_ENV}`, `env.{APP_ENV}.php`) - - конфигурация нашего приложения.

После установки приложения по умолчанию, давайте убедимся, что мы все сделали верно.

#### Если мы устанавливали приложение с помощью docker окружения
Давайте просто запустил приложение

```shell
$ make up
```

после чего перейдем [https://localhost](https://localhost)

#### Если установка была произведена с помощью `composer`
```shell
$ cd public
$ php -S localhost:8000
```

И просто перейдите по ссылке [http://localhost:8000](http://localhost:8000)

Если в браузере мы увидели приветственное сообщение - подздравляю, вы все сделали правильно!

## Первый плагин

Для начала, давайте удалим App/Acme плагин т.к. он является демонстрационным и никакой ценности из себя не представляет.

```shell
$ rm -rf src/Acme
```

И создадим свой собственный `src/Demo/DemoAppPlugin.php`

```php
class DemoAppPlugin
{
}
```

После чего включим его в список плагинов для инициализации `etc/plugins.php`.

```php
<?php

return [
    // Separate system plugins from appliction plugins.
    Micro\Plugin\Configuration\Helper\ConfigurationHelperPlugin::class,
    Micro\Plugin\Console\ConsolePlugin::class,
    Micro\Plugin\Http\HttpPackPlugin::class,
    Micro\Plugin\Logger\Monolog\MonologPlugin::class,
    Micro\Plugin\Doctrine\DoctrinePlugin::class,
    Micro\Plugin\Twig\TwigPlugin::class,
    OleksiiBulba\WebpackEncorePlugin\WebpackEncorePlugin::class,
    
     //App plugin(s)
    App\Demo\DemoAppPlugin::class,
    // App\Acme\AcmePlugin::class <-- remove redundant plugin
];
```

## Бизнес-слой

Давайте наделим наше приожение логикой. Для начала, определим интерфейс взаимодейстаия с [API Packagist.org](https://packagist.org/apidoc#search-packages) для поиска.
Создадим мы его в `src/Demo/Business/Packagist/PackagistSearchInterface.php`

```php
<?php

declare(strict_types=1);

namespace App\Demo\Business\Packagist;

interface PackagistSearchInterface
{
    /**
     * @return array<string, mixed>
     */
    public function search(string $query): array;
}
```

После того как интерфейс готов, имплементируем его логику.

```php
<?php

declare(strict_types=1);

namespace App\Demo\Business\Packagist;

use App\Demo\DemoAppPluginConfiguration;

readonly class PackagistSearch implements PackagistSearchInterface
{
    public function __construct(
    ) {
    }

    public function search(string $query): array
    {
        $query = trim($query);
        if (!$query) {
            return [];
        }

        $url = sprintf(
            'https://packagist.org/search.json?q=%s',
            $this->pluginConfiguration->getPackagistUrl(),
            urlencode($query)
        );

        $results = file_get_contents($url);
        if (false === $results) {
            throw new \RuntimeException(sprintf('Fetching query from `%s` can not be processed right now.', $url));
        }

        return json_decode($results, true);
    }
}
```

Казалось бы, все хорошо, однако packagist может иметь зеркала и нам необходимо иметь возможность его конфигурировать.
Давайте для этого создадим конфигурационный класс клагина, который будет отвечать за общие его настройки.
Конфигурация приложения - это key/value хранилище, а значит мы можем определить ключ необходимого нам параметра для получения его значения.

Итак, создаем новый класс в `src/Demo/DemoAppPluginConfiguration.php`

```php
<?php

declare(strict_types=1);

namespace App\Demo;

use Micro\Framework\Kernel\Configuration\PluginConfiguration;

class DemoAppPluginConfiguration extends PluginConfiguration
{
    public const CFG_PACKAGIST_URL = 'PACKAGIST_URL';

    public function getPackagistUrl(): string
    {
        return rtrim($this->configuration->get(self::CFG_PACKAGIST_URL, 'https://packagist.org/'), '/');
    }
}
```
Сделующая задача - сделать доступным эту конфигурацию в нашем плагине.
Для этого мы имплементируем в плагин `Micro\Framework\Kernel\Plugin\ConfigurableInterface` и чтобы каждый раз не писать рутинные методы `setConfiguration`, `configuration()`, просто добавим трейт `Micro\Framework\Kernel\Plugin\PluginConfigurationTrait`.

```php
<?php

declare(strict_types=1);

namespace App\Demo;
use Micro\Framework\Kernel\Plugin\ConfigurableInterface;
use Micro\Framework\Kernel\Plugin\PluginConfigurationTrait;

/**
 * @method DemoAppPluginConfiguration configuration()
 */
class DemoAppPlugin implements ConfigurableInterface
{
    use PluginConfigurationTrait;
}
```

###### Обратите внимание, чтобы конфигурационный класс плагина нашелся корректно, он должен иметь имя `{PluginClassName}Configuration`
###### Аннотация `@method` имеет исключительно вспомогательную функцию для IDE. 

Когда конфигурация определена, мы можем предоставить ее в сервис поиска `PackagistSearch`
```php
<?php

declare(strict_types=1);

namespace App\Demo\Business\Packagist;

use App\Demo\DemoAppPluginConfiguration;

readonly class PackagistSearch implements PackagistSearchInterface
{
    public function __construct(
        private DemoAppPluginConfiguration $pluginConfiguration
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
            urlencode($query)
        );

        $results = file_get_contents($url);
        if (false === $results) {
            throw new \RuntimeException(sprintf('Fetching query from `%s` can not be processed right now.', $url));
        }

        return json_decode($results, true);
    }
}
```

## Фасады - Контракт работы с бизнес-слоем.
Мы не хотим предоставлять все классы в сервис-контейнер. Важно иметь контроль над тем, что мы регистрируем в качестве сервиса, однако важно, чтобы сторонние сервисы, которые используют наш фасад могли работать с заранее определенным контрактом. 
Для этого к нам приходит на помощь такой паттерн проектирования как Фасад. 
Фасады нужны для того, чтобы агрегировать и определить контракт работы с бизнес-логикой плагина и последующей регистрации его в DI.

Давайте создадим наш первый фасад - `src/Demo/Facade/DemoAppFacadeInterface.php`

```php
<?php

declare(strict_types=1);

namespace App\Demo\Facade;

use App\Demo\Business\Packagist\PackagistSearchInterface;

interface DemoAppFacadeInterface extends PackagistSearchInterface
{
}
```

Обратите внимание, данный фасад наследуется от `App\Demo\Business\Packagist\PackagistSearchInterface`, однако он может наследовать множесто внутренних интерфейсов. Это позволяет предоставить единую точку входа к функционалу плагина, при этом иметь четкое понимание того, какие задачи он сопосбен решать.
И теперь давайте сделаем реализацию данного контракта `src/Demo/Facade/DemoAppFacade.php`

```php
<?php

declare(strict_types=1);

namespace App\Demo\Facade;

use App\Demo\Business\Packagist\PackagistSearchInterface;

readonly class DemoAppFacade implements DemoAppFacadeInterface
{
    public function __construct(private PackagistSearchInterface $packagistSearch)
    {
    }

    public function search(string $query): array
    {
        return $this->packagistSearch->search($query);
    }
}
```

## Регистрация сервиса в DI
Разумеется, мало написать сервис. Нужно предоставить его в сервисный контейнер.
Давайте немного расширим наш плагин.
```php
<?php

declare(strict_types=1);

namespace App\Demo;

use App\Demo\Business\Packagist\PackagistSearch;
use App\Demo\Business\Packagist\PackagistSearchInterface;
use App\Demo\Facade\DemoAppFacade;
use App\Demo\Facade\DemoAppFacadeInterface;
use Micro\Component\DependencyInjection\Container;
use Micro\Framework\Kernel\Plugin\ConfigurableInterface;
use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Framework\Kernel\Plugin\PluginConfigurationTrait;

/**
 * @method DemoAppPluginConfiguration configuration()
 */
class DemoAppPlugin implements DependencyProviderInterface, ConfigurableInterface
{
    use PluginConfigurationTrait;

    public function provideDependencies(Container $container): void
    {
        $container->register(DemoAppFacadeInterface::class, fn () => $this->createFacade());
    }
    
    protected function createFacade(): DemoAppFacadeInterface
    {
        return new DemoAppFacade($this->createPackagistSearchService());
    }

    protected function createPackagistSearchService(): PackagistSearchInterface
    {
        return new PackagistSearch($this->configuration());
    }
}
```

Что у нас получилось - мы создали и зарегистрировали в контейнере сервис с контрактом `App\Demo\Facade\DemoAppFacadeInterface`.
Давайте попробуем пообщаться с нашим сервисом.

## Cli интерфейс.

Здесь все будет предельно просто. Мы просто возьмем и создадим класс CLI I/O (Input/Output) и в зависимости от переданных параметров через консольную строку будем выдавать ответ в нужном формате.
Для этого создадим новый класс `src/Demo/Communication/Command/PackagistSearchCommand.php`
```php
<?php

declare(strict_types=1);

/**
 * This file is part of the Micro framework package.
 *
 * (c) Stanislau Komar <head.trackingsoft@gmail.com>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace App\Demo\Communication\Command;

use App\Demo\Facade\DemoAppFacadeInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Helper\Table;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

/**
 * @author Stanislau Komar <head.trackingsoft@gmail.com>
 */
class PackagistSearchCommand extends Command
{
    public function __construct(private readonly DemoAppFacadeInterface $facade)
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
        $queryResult = $this->facade->search($query);
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
В качестве рендера ответов в CLI мы решаем, что хватит такой информации как:
 - имя пакета
 - его описание
 - количество "звезд" на гитхабе.

Сделано это для того, чтобы выводимая информация без труда поместилась на экран.

Давайте попробуем что у нас получислось. В корне проекта выполним команду:
```shell
$ bin/console packagist:search "micro kernel app"
```

Если мы используем docker окружение, то
```shell
$ make micro c="packagist:search 'micro kernel app'"
```

И получим примено похожие результаты
```
+-----------------------------+---------------------------------------------+--------+
| name                        | description                                 | favers |
+-----------------------------+---------------------------------------------+--------+
| micro/kernel-app            | Micro Framework: App Kernel component       | 2      |
| symlex/di-microkernel       | Micro-Kernel for PHP Applications           | 6      |
| phonetworks/pho-microkernel | Social-Enabled App Infrastructure           | 23     |
| lastzero/di-microkernel     | Micro-Kernel for PHP Applications           | 6      |
| jmleroux/symfony-micro-rest | Symfony micro kernel application for rest   | 1      |
| lou117/wake                 | Web-Application KErnel, PHP micro-framework | 0      |
+-----------------------------+---------------------------------------------+--------+

```

Это может свидетельствует о том, что мы все делали правильно:
  - В `PackagistSearchCommand` была внедрена зависимость нашего бизнес-слоя
  - Сервис `DemoAppFacadeInterface` независимо от I/O способен выдавать необходимый результат для дальнейшего его представления во View.

## HTTP
Давайте отобразим наши данные в браузере.
Чтобы сделать это в красивом виде, мы будем использовать
 - В качестве шаблонизатора [Twig](https://twig.symfony.com/).
 - Для стилизации мы будем использовать [Bootstrap](https://getbootstrap.com/)
 - Чтобы связать стили и JS мы будем использовать уже предустановленный плагин [micro/plugin-twig-webpack-encore](https://micro-php.net/docs/plugins/micro/plugin-twig-webpack-encore).
 - Чтобы выполнять сборку frontend - нам понадобится nodejs и менеджер зависимостей (yarn/npm) последних версий.** Но об этом мы расскажем немного позже.

Давайте приступим к созданию контроллера.
Расположим его в `src/Demo/Communication/Controller/PackagistSearchController.php`

```php
<?php

declare(strict_types=1);

namespace App\Demo\Communication\Controller;

use App\Demo\Facade\DemoAppFacadeInterface;
use Micro\Plugin\Twig\TwigFacadeInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

readonly class PackagistSearchController
{
    public function __construct(
        private DemoAppFacadeInterface $packagistFacade,
        private TwigFacadeInterface $twigFacade
    ) {
    }

    public function search(Request $request): Response
    {
        $query = (string) $request->get('q');
        $exception = null;
        $results = null;
        try {
            $results = $this->packagistFacade->search($query);
        } catch (\RuntimeException $exception) {
        }

        $rendered = $this->twigFacade->render('@DemoAppPlugin/Packagist/search.html.twig', [
            'query' => $query,
            'results' => $results,
            'exception' => $exception,
        ]);

        return new Response($rendered);
    }
}
```

Обратите внимание, в контроллере нам требуются сервисы `App\Demo\Facade\DemoAppFacadeInterface` у которого есть одна задача - генерировать из Twig шаблона контент, в нашем случае, HTML для жальнейшей передачи его в `Response`, а так же, как вы уже узнали, сервис нашего бизнес-слоя `DemoAppFacadeInterface`.
Осталось сообщить ядру то, что наш плагин является еще и провайдером маршрутов и шаблонов Twig.
Для этого вернемся в код нашего плагина `src/Demo/DemoAppPlugin.php` и имплементируем в него интерфейсы
  - `Micro\Plugin\Twig\Plugin\TwigTemplatePluginInterface` - контракт, сообщающий о том, откуда и с каким неймспейсом забирать шаблоны.
  - `Micro\Plugin\Http\Plugin\RouteProviderPluginInterface` - контракт предоставления маршрутов плагина.

Давайте добавим в наш плагин эту функциональность.

```php
<?php

declare(strict_types=1);

namespace App\Demo;

use App\Demo\Business\Packagist\PackagistSearch;
use App\Demo\Business\Packagist\PackagistSearchInterface;
use App\Demo\Communication\Controller\PackagistSearchController;
use App\Demo\Facade\DemoAppFacade;
use App\Demo\Facade\DemoAppFacadeInterface;
use Micro\Component\DependencyInjection\Container;
use Micro\Framework\Kernel\Plugin\ConfigurableInterface;
use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Framework\Kernel\Plugin\PluginConfigurationTrait;
use Micro\Plugin\Http\Facade\HttpFacadeInterface;
use Micro\Plugin\Http\Plugin\RouteProviderPluginInterface;
use Micro\Plugin\Twig\Plugin\TwigTemplatePluginInterface;
use Micro\Plugin\Twig\Plugin\TwigTemplatePluginTrait;

/**
 * @method DemoAppPluginConfiguration configuration()
 */
class DemoAppPlugin implements DependencyProviderInterface, TwigTemplatePluginInterface, ConfigurableInterface, RouteProviderPluginInterface
{
    use PluginConfigurationTrait;
    use TwigTemplatePluginTrait;

    public function provideDependencies(Container $container): void
    {
        $container->register(DemoAppFacadeInterface::class, fn () => $this->createFacade());
    }

    protected function createFacade(): DemoAppFacadeInterface
    {
        return new DemoAppFacade($this->createPackagistSearchService());
    }

    protected function createPackagistSearchService(): PackagistSearchInterface
    {
        return new PackagistSearch($this->configuration());
    }

    public function provideRoutes(HttpFacadeInterface $httpFacade): iterable
    {
        yield $httpFacade->createRouteBuilder()
            ->setUri('/')
            ->setController(PackagistSearchController::class)
            ->setName('search')
            ->build()
        ;
    }
}
```

Здесь стоит обратить внимание на метод `provideRoutes`. ОДнако, думаю, в излишних комментариях он не нуждается.
А вот трейт `Micro\Plugin\Twig\Plugin\TwigTemplatePluginTrait` реализует сразу 2 метода:
  - `TwigTemplatePluginTrait::getTwigTemplatePaths()` - указывает массив с адресами каталогов где нужно искать шаблоны. По умолчанию это `<PLUGIN ROOT DIR>/templates`
  - `TwigTemplatePluginTrait::getTwigNamespace` - Указывает на неймспейс. По умолчанию это shortName класса плагина.

### Шаблоны
Мы не будем останавливаться в данной статье на работе шаблонизатора, лучше всего об этом расскажет [официальная документация](https://twig.symfony.com/doc/3.x/).
Но мы [подгтовили для вас готоые шаблоны](https://github.com/Asisyas/microframework-packagist-demo/tree/master/src/Demo/templates), чтобы вы смогли использовать их для демонстрационных целей.
Просто разместите их в каталог `src/Demo/templates/`
А [frontend часть]((https://github.com/Asisyas/microframework-packagist-demo/tree/master/assets)) разместите в `assets/`

### Давайте соберем наш фронт
Для того, чтобы html контент выглядел презентаельно, вышеупомянутые шаблоны подготовлены для работы с CSS фреймворком [Bootstrap](https://getbootstrap.com/)
И мы пришли к необходимости установки NodeJS.

###### Если вы используете docker, вы можете добавить в `docker-compose.override.yml` подходящий для этого контейнер, либо установить nodejs локально.
Oдин из вариантов добавления nodejs в `docker-compose.yml`. После чего, самый простой вариант установки нужного контейрера `make down && make up`
```yaml
services:
# other services
    node:
        image: node:lts
        working_dir: /app
        volumes:
          - - ./:/app
```
###### Если вы не используете Docker, то можете просто использовать интерфейс командной строки для работы.
```shell
$ yarn install
```
Давайте установим Bootstrap и требуемый для него jQuery.

```shell
$ yarn add bootstrap jquery
# or
$ npm i bootstrap jqeury --save
```
После чего давайте запустим сборку

```shell
yarn build
```
## Завершение
Давайте проверим то, как работает наше приложение

###### Если мы используем Docker
Если наше приложение не запущено, то давайте стартуем его

```shell
$ make up
```

Откроем в браузере [https://localhost](https://localhost)

###### Если мы используем локальный php-cli
```shell
$ cd public
$ php -S localhost:8000 
```
Откроем в браузере [http://localhost:8000]


### Поздравляю, мы собрали первое приложение на MicroPHP.
Надеемся, данный пример раскрыл часть возмодностей MicroPHP и вам понравилось на нем работать.

## Послесловие
  * Исходный код готового приложения можно найти на [GitHub](https://github.com/Asisyas/microframework-packagist-demo)
  * В демонстрационных целях мы показали как можно работать с интерфейсами компонентов MF, однако, мы придерживаемся правила, что каждый плагин должен выполнять только свою функцию: I/O, Business-Layer, View и т.д., однако, вам решать как обустраивать экосистему своего приложения.
  * Спасибо, что используете нас. Это очень мотивирует делать продукт более удобным, качественным и придерживаться высоких стандартов в его разработке.
