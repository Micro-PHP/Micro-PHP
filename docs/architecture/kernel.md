---
layout: doc/document
title: Kernel usages
show_sidebar: false
menubar: docs_menu
---

[Kernel](https://github.com/Micro-PHP/micro-kernel) - это главный компонент системы фреймворка. Он отвечает за загрузку плагинов и корректную их работу.

## Plugins
Плагин - это абсолютно любой класс, который может быть загружен и использован другими компонентами системы.
Поведение плагина описывается контакртами (интерфейсами). Подробнее про систему плагинов читайте [здесь](/docs/architecture/plugins).

## Bootloaders

Bootloader - это класс, который позволяет вмешаться в процесс инициализации плагина.
Фактически, Bootloader является одним из способов внедрения любых побочных служебных компонентов в систему.

Вот несколько примеров его применения:
  * [Создание конфигурации плагина](https://github.com/Micro-PHP/kernel-bootloader-configuration)
  * [Подгрузка зависимых плагинов](https://github.com/Micro-PHP/kernel-bootloader-plugin-depends)
  * [Предоставление сервисного контейнера в плагин](https://github.com/Micro-PHP/kernel-bootloader-dependency)

## Использование плагинов в приложении.

Т.к. плагин - любой объект, функциональность которого определяется его контрактом (интерфейсом), существует возможность поиска всех плагинов, которые удовлетворяют тому или иному контракту.

#### Пример использования

Для начала, давайте создадим контракт плагина:

```php

interface AdapterPluginInterface
{
    public function getName(): string;
    public function execute(): void;
}
```

И имплементируем вышеописанный контракт.

```php
// First plugin
class DefaultAdapterPlugin implements AdapterPluginInterface
{
    public function getName(): string
    {
        return 'default';
    }
    
    public function execute(): void
    {
        print_r(sprintf('Hello, I\'m %s adapter', $this->getName()));
    }
}

// Second plugin
class AdvancedAdapterPlugin implements AdapterPluginInterface
{
    public function getName(): string
    {
        return 'advanced';
    }
    
    public function execute(): void
    {
        print_r(sprintf('Hello, I\'m %s adapter', $this->getName()));
    }
}
```

После чего добавим еще один плагин, который будет использовать вышеописанные контракты.

```php
use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Component\DependencyInjection\Container;

// Third plugin
class AdapterExecutor implements DependencyProviderInterface
{
    private KernelInterface $kernel;

    public function provideDependencies(Container $container): void
    {
        $container->register(AdapterExecutor::class, function(
            KernelInterface $kernel
        ) {
            $this->kernel = $kernel;
            
            return $this;
        });
    }
    
    public function execute()
    {
        /** @var AdapterPluginInterface $plugin */
        foreach ($this->kernel->plugins(AdapterPluginInterface::class) as $plugin) {
            $plugin->execute();
        }
    }
}
```

После чего нам необходимо будет зарегистрировать плагины в ядре и запустить приложение.
**Если вы используете [micro/skeleton](/docs/getting-started/installation), то просто добавьте string-class в `etc/plugins.php` **


После чего плагин `AdapterExecutor` можно использовать в других компонентах системы через DI/LoC.
```php

$executor = $container->get(AdapterExecutor::class)->execute();
// Hello, I'm default adapter
// Hello, I'm advanced adapter
```

## Подробный пример  инициализации ядра

Наглядный пример подготовки простейшего приложения. Эта информация дана исключительно в обучающих целях.
Советуем использовать уже готовые инструменты:
  * [micro/skeleton](/docs/getting-started/installation)
  * [micro/docker](/docs/getting-started/docker)



```php
use Micro\Framework\Kernel\Configuration\DefaultApplicationConfiguration;
use Micro\Framework\Kernel\Plugin\PluginDependedInterface;
use Micro\Framework\Kernel\Plugin\ApplicationPluginInterface;
use Micro\Component\DependencyInjection\Container;
use Micro\Framework\Kernel\Plugin\PluginBootLoaderInterface;
use Micro\Framework\Kernel\KernelBuilder;
use Micro\Framework\Kernel\Configuration\ApplicationConfigurationInterface;

interface DependencyProviderInterface
{
    public function provideDependencies(Container $container): void;
}

interface ConfigurableInterface
{
    public function provideApplicationConfiguration(ApplicationConfigurationInterface $configuration): void;
}

// Create simple plugin
readonly class TestPlugin implements DependencyProviderInterface, ConfigurableInterface
{
    public function provideDependencies(Container $container): void
    {
        print_r('Provided DI container');
    }
    
    public function provideApplicationConfiguration(ApplicationConfigurationInterface $configuration): void
    {
        print_r($configuration->get('APP_ENV')); //dev
    }
}

readonly class DependencyProviderBootLoader implements PluginBootLoaderInterface
{

    public function __construct(private Container $container)
    {
    }
    
    public function boot(object $applicationPlugin): void
    {
        if (!($applicationPlugin instanceof DependencyProviderInterface)) {
            return;
        }
        
        $applicationPlugin->provideDependencies($this->container);
    }
}

readonly class ConfigurationProviderBootLoader implements PluginBootLoaderInterface
{

    public function __construct(private ApplicationConfigurationInterface $configuration)
    {
    }
    
    public function boot(object $applicationPlugin): void
    {
        if (!($applicationPlugin instanceof ConfigurableInterface)) {
            return;
        }
        
        $applicationPlugin->provideApplicationConfiguration($this->configuration);
    }
}


$kernelBuilder  = new KernelBuilder();
$container      = new Container();
$configuration  = new DefaultApplicationConfiguration(['APP_ENV' => 'dev']);
$kernel         = $kernelBuilder
    ->setApplicationConfiguration($configuration)
    ->setContainer($container)
    ->setApplicationPlugins([
        TestPlugin::class
    ])
    ->addBootLoaders([
        new DependencyProviderLoader($container),
        new ConfigurationProviderBootLoader($configuration),
    ])
    ->build();
;

$kernel->run();
```