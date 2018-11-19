# Установка

## Предпочтительный вариант через HomeBrew:

[Установить HomeBrew ](brew.md)

	$ brew install composer

Важно, что лучше чтобы сначала был установлен php через HomeBrew.

## Установка напрямую

<https://getcomposer.org/download/>

Скачиваем (по умолчанию в папку пользователя: ~/):

	$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

Проверяем:

	$ php -r "if (hash_file('SHA384', 'composer-setup.php') === '55d6ead61b29c7bdee5cccfb50076874187bd9f21f65d8991d46ec5cc90518f447387fb9f76ebae1fbbacf329e583e30') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

Устанавливаем «composer.phar»:

	$ php composer-setup.php

Удаляем installer:

	$ php -r "unlink('composer-setup.php');"

Используем, например:

	$ php composer-setup.php global require "laravel/installer"

Или ставим глобально (пример на Mac):

	$ mv composer.phar /usr/local/bin/composer

Тогда используем так, например:

	$ composer global require "laravel/installer"

# Использование

Пакеты для установки можно посмотреть на https://packagist.org/ там можно найти тот же Laravel.

Описание команд Composer https://modzone.ru/blog/2016/12/02/understanding-composer/

Команда для добавления нового пакета:

	$ composer require doctrine/dbal

Или для добавления нового пакета только для разработки (в секцию require-dev в composer.json):

	$ composer require phpdocumentor —dev

Удаление добавленного пакета либо через команду:

	$ composer remove doctrine/dbal

Либо удалить строчку из файла composer.json с названием пакета, и выполнить команду обновления пакетов:

	$ composer update

Список всех командам: 
http://composer.json.jolicode.com
