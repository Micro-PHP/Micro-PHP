---
layout: doc/document
title: Console компонент
show_sidebar: false
menubar: docs_menu
---

## Компонент Console

Компонент реализован на базе [symfony/console](https://symfony.com/doc/current/components/console.html) и предоставляет удобные инструменты для работы с интерфейсом командной строки.

## Установка

Для установки библиотеки воспользуйтесь [composer](https://composer.org)

```shell
$ composer require micro/plugin-console
```

И добавьте в список плагинов. По умолчанию `{PROJECT ROOT DIR}/etc/plugins.php`


## Использование

Для того, чтобы зарегистрировать команду, просто создайте класс, наследуемый от `Symfony\Component\Console\Command\Command`.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

/**
 * @author Stanislau Komar <head.trackingsoft@gmail.com>
 */
class TestCounterCommand extends Command
{
    public function __construct(
        private readonly ExampleServiceFacadeInterface $exampleFacade
    )
    {
        parent::__construct('app:example');
    }

    public function execute(InputInterface $input, OutputInterface $output)
    {
        $result = $this->exampleFacade->execure();
        
        $output->writeln(sprintf('Result %s', $result));

        return self::SUCCESS;
    }
}
```

