# Заметки по использованию PHPUnit

[PHPUnit](https://phpunit.de) -  система модульного тестирования для PHP.

<!--ts-->
<!--te-->

## Ссылки

1. [Документация на русском языке](https://phpunit.readthedocs.io/ru/latest).

## Установка на macOS

Сначала устанавливаем [HomeBrew](HomeBrew/readme.md), затем с его помощью устанавливаем PHPUnit, при условии что PHP уже установлен, потому что [версия PHPUnit устанавливается в зависимости от версии PHP](https://phpunit.de/getting-started/phpunit-7.html):

	$ brew install phpunit
	$ phpunit --version