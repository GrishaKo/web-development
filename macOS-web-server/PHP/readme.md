# Заметки по использованию PHP

[PHP](https://ru.wikipedia.org/wiki/PHP) - скриптовый язык общего назначения, интенсивно применяемый для разработки веб-приложений.

<!--ts-->
   * [Заметки по использованию PHP](#заметки-по-использованию-php)
      * [Ссылки](#ссылки)
      * [Установка на macOS](#установка-на-macos)
      * [Конфигурация Apache](#конфигурация-apache)
      * [Установка расширений для PHP на macOS](#установка-расширений-для-php-на-macos)

<!-- Added by: grisha_k, at:  -->

<!--te-->

## Ссылки

1. [MacOS 10.14 Mojave Apache Setup: Multiple PHP Versions](https://getgrav.org/blog/macos-mojave-apache-multiple-php-versions).
2. [MacOS 10.14 Mojave Apache Setup: MySQL, APC & More...](https://getgrav.org/blog/macos-mojave-apache-mysql-vhost-apc).

## Установка на macOS

Сначала устанавливаем [HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем [Apache](../Apache/readme.md) и после - PHP.

Если PHP был уже установлен в /usr/local/Homebrew/php, то нужно [удалить старые пакеты PHP и очистить настройки](../HomeBrew/readme.md#Удаление-пакетов).

Теперь можем установить несколько версий PHP в /usr/local/Homebrew/core: 

	$ brew install php@5.6
	$ brew install php@7.2
	
После установки мы должны создать необходимые символические ссылки на нужную версию PHP (например, 5.6), в том числе /usr/local/bin/php:

	$ brew link --force --overwrite php@5.6
	$ php -v
	PHP 5.6.39 (cli) 
	
Поменяем версию PHP c 5.6 на 7.2:	
	
	$ brew unlink php@5.6 && brew link --force --overwrite php@7.2
	$ php -v
	PHP 7.2.13 (cli)

Установим параметр [date.timezone](http://php.net/manual/ru/timezones.php) в конфигурационных файлах php.ini: 

		# сделаем копию конфигурационных файлов
	$ cp /usr/local/etc/php/5.6/php.ini /usr/local/etc/php/5.6/php.ini.bak
	$ cp /usr/local/etc/php/7.2/php.ini /usr/local/etc/php/7.2/php.ini.bak
		# установим date.timezone путем замены значения по умолчанию
	$ perl -i -pe 's/;date.timezone =/date.timezone = Europe\/Moscow/;' /usr/local/etc/php/5.6/php.ini
	$ perl -i -pe 's/;date.timezone =/date.timezone = Europe\/Moscow/;' /usr/local/etc/php/7.2/php.ini
		# либо откроем соответствующие файлы php.ini и изменим date.timezone вручную, например:
		# $ nano /usr/local/etc/php/5.6/php.ini
		# $ nano /usr/local/etc/php/7.2/php.ini

## Конфигурация Apache

[Откроем конфигурационный файл Apache](../Apache/readme.md#конфигурация-httpdconf), например, с помощью терминального редактора nano:

	$ nano /usr/local/etc/httpd/httpd.conf

Найдем последнюю строку с LoadModule:

	LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
	
И после нее с новой строки добавим строки подключения модулей libphp для каждой версии PHP, закомментировав все строки, кроме строки для текущей версии PHP, а также через пустую строку добавим указание на то, чтобы PHP обрабатывал файлы с расширением .php:

	#LoadModule php5_module /usr/local/opt/php@5.6/lib/httpd/modules/libphp5.so
	LoadModule php7_module /usr/local/opt/php@7.2/lib/httpd/modules/libphp7.so
	
	<FilesMatch \.php$>
	    SetHandler application/x-httpd-php
	</FilesMatch>

Теперь мы должны установить индексные файлы PHP для каталогов, для этого находим строки:

	<IfModule dir_module>
	   DirectoryIndex index.html
	</IfModule>

И заменяем на:

	<IfModule dir_module>
	    DirectoryIndex index.php index.html
	</IfModule>

Сохраняем изменения, перезапустив Apache:

	$ sudo apachectl -k restart
	$ sudo apachectl start
	
Теперь мы можем проверить, какую точно версию PHP использует Apache, для этого в одном из каталогов виртуальных хостов создадим следующий файл:

	$ echo "<?php phpinfo();" > /usr/local/var/www/info.php
	
> /usr/local/var/www - путь до корневой директории виртуального хоста localhost.

И можем открыть в браузере страницу виртуального хоста <https://localhost/info.php>, в результате увидим вывод команды phpinfo().

Для переключения версии PHP, используемой Apache, можно закомментировать одно подключение модуля "LoadModule php7_module ... libphp7.so" и раскомментировать другое подключение "LoadModule php5_module ... libphp5.so", либо установить [PHP Switcher Script](https://gist.github.com/rhukster/f4c04f1bf59e0b74e335ee5d186a98e2):

	$ curl -L https://gist.githubusercontent.com/rhukster/f4c04f1bf59e0b74e335ee5d186a98e2/raw > /usr/local/bin/sphp
	$ chmod +x /usr/local/bin/sphp

Затем необходимо добавить /usr/local/sbin в $PATH:
	
	$ sudo nano ~/.bash_profile
		# вставляем строку
	export PATH=/usr/local/bin:/usr/local/sbin:$PATH
		# сохраняем, выходим и выполняем	
	$ source ~/.bash_profile

Теперь мы можем переключаться между установленными версиями php:

	$ sphp 5.6
	$ php -v
	PHP 5.6.38 (cli)
	$ sphp 7.2
	$ php -v
	PHP 7.2.12 (cli)

Посмотреть более детальную информацию о конкретной версии PHP можно командой:

	$ brew info php@7.2
	
## Установка расширений для PHP на macOS

Установим расширение [Xdebug](https://ru.wikipedia.org/wiki/Xdebug), сначала для PHP 5.6:

	$ sphp 5.6
	$ pecl install xdebug-2.5.5
	
Теперь удалим или закомментируем строку (отключим расширение) zend_extension="xdebug.so" в /usr/local/etc/php/5.6/php.ini, добавленную PECL:

	$ perl -i -pe 's/zend_extension="xdebug.so"/;zend_extension="xdebug.so"/;' /usr/local/etc/php/5.6/php.ini

И создадим файл с подключением и конфигурацией расширения Xdebug:

	$ nano /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
		# добавим строки, сохраним изменения и выйдем
	[xdebug]
	zend_extension="xdebug.so"
	xdebug.remote_enable=1
	xdebug.remote_host=localhost
	xdebug.remote_handler=dbgp
	xdebug.remote_port=9000

Теперь перезагрузим Apache:

	$ sudo apachectl -k restart

Откроем в браузере страницу виртуального хоста <https://localhost/info.php> и посмотрим, что phpinfo() есть информация о подключенном Xdebug.

Можем установить переключатель "[Xdebug Toggler for OSX](https://github.com/w00fz/xdebug-osx)":

	$ curl -L https://gist.githubusercontent.com/rhukster/073a2c1270ccb2c6868e7aced92001cf/raw > /usr/local/bin/xdebug
	$ chmod +x /usr/local/bin/xdebug

Теперь мы можем быстро включать и выключать расширение Xdebug:
	
	$ xdebug on
	$ xdebug off

> Файл "/usr/local/etc/php/5.6/conf.d/ext-xdebug.ini" переименовывается в ".../ext-xdebug.ini.disabled".

Теперь мы можем переключиться на другую версию php и переустановить расширение Xdebug для другой версии php:

	$ sphp 7.2
	$ pecl uninstall -r xdebug
	$ pecl install xdebug
		# удалим или закомментируем подключение Xdebug в php.ini
	$ perl -i -pe 's/zend_extension="xdebug.so"/;zend_extension="xdebug.so"/;' /usr/local/etc/php/7.2/php.ini
		# скопируем файл с подключением и конфигурацией расширения Xdebug
	$ cp /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini /usr/local/etc/php/7.2/conf.d/ext-xdebug.ini
		# перезагрузим Apache
	$ sudo apachectl -k restart

Также мы можем установить расширение [Yaml через PECL] (https://pecl.php.net/package/yaml), предварительно установив библиотеку Yaml через HomeBrew:

	$ brew install libyaml
		# для php 5.6
	$ sphp 5.6
	$ pecl install yaml-1.3.1
	$ sudo apachectl -k restart
		# для php 7.2
	$ sphp 7.2
	$ pecl uninstall -r yaml && pecl install yaml
	$ sudo apachectl -k restart

Также как и для Xdebug, мы можем перенести подключение Yaml из php.ini в conf.d/ext-yaml.ini:

	[yaml]
	extension="yaml.so"