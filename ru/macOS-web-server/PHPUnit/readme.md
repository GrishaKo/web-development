# Заметки по использованию PHPUnit

[PHPUnit](https://phpunit.de) -  система модульного тестирования для PHP.

<!--ts-->
  * [Ссылки](#ссылки)
  * [Установка на macOS](#установка-на-macos)
<!--te-->

<a id="links"></a>
## Ссылки

1. [Документация](https://phpunit.readthedocs.io/en/latest/) (ENG/[RUS](https://phpunit.readthedocs.io/ru/latest)).

<a id="installation"></a>
## Установка на macOS

Сначала устанавливаем [../HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем PHPUnit, при условии что PHP уже установлен, потому что [версия PHPUnit устанавливается в зависимости от версии PHP](https://phpunit.de/getting-started/phpunit-7.html) (ENG):

	$ brew install phpunit
	$ phpunit --version
