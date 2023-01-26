---
layout: doc/document
title: Installation
show_sidebar: false
menubar: docs_menu
---

## Requirements
The minimum PHP version must be at least **8.2**

## Get started

It is recommended that you create your first application in a [docker environment](/docs/getting-started/docker).

Or you can create an application via [composer](https://getcomposer.org/).

```shell
$ composer create-project micro/micro {directory} --remove-vcs
$ cd {directory}
$ php -S localhost:10000 -t public/
```

Open `http://localhost:10000` and you will see HelloWorld page.

There is also support for console utilities.
```bash
php bin/console
``` 

## Running Tests

To run tests, run the following command. By default, phpstan, psalm, phpunit will be runned.

```bash
$ composer run-script test-all
``` 
