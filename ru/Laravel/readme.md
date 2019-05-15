# Заметки по Laravel

[Laravel](https://en.wikipedia.org/wiki/Laravel) (ENG/[RUS](https://ru.wikipedia.org/wiki/Laravel)) - php-фреймворк.

<!--ts-->
<!--te-->

<a id="links"></a>
## Ссылки

1. [Laravel.com/docs](https://laravel.com/docs) (ENG).
2. [Laracasts.com](https://laracasts.com/series/laravel-from-scratch-2017/) (ENG).
3. [Laravel.ru](https://laravel.ru) (RUS).
4. [Laravel.su](http://laravel.su/docs/5.4/lifecycle) (RUS).
5. [Layered Structure](https://www.toptal.com/php/maintain-slim-php-mvc-frameworks-with-a-layered-structure) (ENG/[RUS](http://developer.uz/blog/layered-structure-for-yii-app/)).
6. [Laravel best practices](https://github.com/alexeymezenin/laravel-best-practices) (ENG/[RUS](https://github.com/alexeymezenin/laravel-best-practices/blob/master/russian.md)).

<a id="packages"></a>
## Пакеты

1. [barryvdh/laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper) + [PHPStorm's Laravel Facades Issue](https://laracasts.com/series/how-to-be-awesome-in-phpstorm/episodes/15) (ENG).
2. [barryvdh/laravel-debugbar](https://github.com/barryvdh/laravel-debugbar).

<a id="packages"></a>
## Перенос на рабочий сервер с помощью Git

Сначала на рабочем сервере устанавливаем [../Git](../macOS-web-server/Git/readme.md), после чего клонируем удаленный Git-репозиторий:

	$ cd ~/Sites
	$ git clone GIT@SERVER:REPOSTIROTY.git SITE
		
> `GIT@SERVER` - удаленный Git-сервер;
> 
> `REPOSITORY.git` - удаленный репозиторий (например: site.git).
> 
> `SITE` - при необходимости указываем имя каталога проекта отличное от `REPOSTIROTY` (например: site.tld, вместо, site).

Переходим в папку проекта:

	$ cd ~/Sites/SITE

Затем устанавливаем [../Composer](../macOS-web-server/Composer/readme.md) и при необходимости обновляем  [../PHP](../macOS-web-server/PHP/readme.md), после чего устанавливаем зависимости из composer.json:
    
    $ composer install
    	# или
    	# $ composer install --no-dev

> `--no-dev` - исключение пакетов в секции "require-dev".

Далее устанавливаем [Node.js](https://nodejs.org/en/download/), после чего устанавливаем зависимости из package.json:

	$ npm install

Настраиваем файл `.env`:

	$ cp .env.example .env
		# устанавливаем значение параметра APP_KEY
	$ php artisan key:generate
	$ nano .env

Теперь выполняем миграцию базы данных:

	$ php artisan migrate
	
Устанавливаем права для записи в следующие каталоги:

	$ chmod -R 777 storage && chmod -R 777 bootstrap/cache