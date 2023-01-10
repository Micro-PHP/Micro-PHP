---
layout: doc/document
title: Plugins
show_sidebar: false
menubar: docs_menu
---

*Плагин* - это абсолютно любой класс, который может быть загружен и использован другими компонентами системы.
Поведение плагина описывается контакртами (интерфейсами).

***Важное замечание***: каждый плагин должен делать только свою конкретную задачу.

Основная цель данного подхода - оставить в сервисном контейнере только те сервисы, 
которые будут использоваться другими компонентами приложения, и избежать "сервис-хелла" в контейнере.

#### Пример написания и использовани плагинов

Для начала, давайте создадим контракты будущих плагинов:

```php

interface ExecutorPluginInterface
{
    public function execute(string $adapterName = null): void;
}

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

После чего добавим еще один плагин, который будет использовать вышеописанные контракты и зарегистрируем его в сервисном контейнере.

```php
use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Component\DependencyInjection\Container;

// Third plugin
class AdapterExecutor implements DependencyProviderInterface, ExecutorPluginInterface
{
    private KernelInterface $kernel;

    public function provideDependencies(Container $container): void
    {
        $container->register(ExecutorPluginInterface::class, function(
            KernelInterface $kernel
        ) {
            $this->kernel = $kernel;
            
            return $this;
        });
    }
    
    public function execute(string $adapterName = null): void
    {
        $executed = false;
        foreach ($this->kernel->plugins(AdapterPluginInterface::class) as $plugin) {
            if ($adapterName !== null && $adapterName !== $plugin->getName()) {
                continue;
            }
            
            $executed = true;
            $plugin->execute();
        }
        
        if(!$executed) {
            throw new \RuntimeException(sprintf('Failed to perform "%s" action.', $adapterName));
        }
    }
}
```

После чего нам необходимо будет зарегистрировать плагины в ядре и запустить приложение.
Если вы используете [micro/skeleton](/docs/getting-started/installation), то просто добавьте string-class в `etc/plugins.php`
  * `AdapterExecutor::class`
  * `AdvancedAdapterPlugin::class`
  * `DefaultAdapterPlugin::class`

После чего плагин `AdapterExecutor` можно использовать в других компонентах системы через [DI/LoC](/docs/architecture/di).
```php

$executor = $container->get(ExecutorPluginInterface::class)->execute();
// Hello, I'm default adapter
// Hello, I'm advanced adapter
```
