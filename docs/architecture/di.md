---
layout: doc/document
title: Dependency Injection
show_sidebar: false
menubar: docs_menu
---

## Register services

```php

namespace Vendor\Plugin;

use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Component\DependencyInjection\Container;

class ExamplePlugin implements DependencyProviderInterface
{
    public function provideDependencies(Container $container): void
    {
        $container->register('service-id', function() {
            return new ServiceExample();
        });
    }
}
```

## Locate services

```php
$service = $composer->get('service-id');
```

## Inject services

```php
namespace Vendor\Plugin;

use Micro\Framework\Kernel\Plugin\DependencyProviderInterface;
use Micro\Component\DependencyInjection\Container;

class ExamplePlugin implements DependencyProviderInterface
{
    public function provideDependencies(Container $container): void
    {
        $container->register('service-id', function(
            SomeServiceInterface $service
        ) {
            return new ServiceExample($service);
        });
    }
}
```

## Decorate services
```php

class ExamplePlugin implements DependencyProviderInterface
{
    public function provideDependencies(Container $container): void
    {
        $container->decorate(SomeServiceInterface::class, function(
            SomeServiceInterface $service,
            LoggerInterface $logger
        ) {
            return new ServiceDecoratorServiceExample($service, $logger);
        });
    }
}

```