# Заметки по использованию Composer

[Composer](https://getcomposer.org/) - менеджер зависимостей для PHP.

<!--ts-->
  * [Установка Composer на macOS](#установка-composer-на-macos)
  * [Ссылки](#ссылки)
  * [Использование](#использование)
<!--te-->

<a id="links"></a>
## Ссылки

1. [Composer Cheat Sheet for developers](http://composer.json.jolicode.com) (ENG).
2. [Разберёмся с Composer](https://modzone.ru/blog/2016/12/02/understanding-composer/) (RUS).
3. [Composer: Всё о .lock файле](https://phpprofi.ru/blogs/post/15) (RUS).

<a id="installation"></a>
## Установка Composer на macOS

Сначала устанавливаем [../HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем Composer:

    $ brew install composer
    $ composer --version

Важно, что лучше чтобы сначала был установлен php через HomeBrew.

<a id="using"></a>
## Использование

[Packagist](https://packagist.org) - репозиторий пакетов PHP.

Добавить новый пакет в composer.json в текущем каталоге (в секцию "require") и установить:

    $ composer require doctrine/dbal

Или добавить новый пакет в секцию "require-dev"для использования только в среде разработки (на локальном компьютере):

    $ composer require doctrine/dbal —dev

Установка зависимостей из composer.json с учетом composer.lock:
    
    $ composer install

> Команда проверяет существует ли файл composer.lock если нет, проверяет зависимости и создает его, иначе устанавливает версии пакетов, указанные в нём - это нужно для того, чтобы у всех разработчиков были одинаковые версии пакетов.

На рабочем сервере, стоит установить зависимости исключив пакеты в секции "require-dev":

    $ composer install --no-dev
    
Обновление зависимостей до последних версий:

    $ composer update

> Команда обновит файл composer.lock.

Удаление пакета из composer.json и удаление ненужных зависимостей:

    $ composer remove doctrine/dbal
    $ composer update doctrine/dbal

Обновление автозагрузчика PHP классов:

    $ composer dump-autoload
    
На рабочем сервере стоит использовать оптимизацию автозагрузчика одной из команд:

    $ composer dump-autoload --optimize
    $ composer dump-autoload -o

> Оптимизация может ускорить работу автозагрузчика до 30%.