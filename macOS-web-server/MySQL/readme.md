# Заметки по использованию MySQL

[MariaDB](https://ru.wikipedia.org/wiki/MariaDB) — ответвление от реляционной системы управления базами данных [MySQL](https://ru.wikipedia.org/wiki/MySQL).

<!--ts-->
  * [Ссылки](#ссылки)
  * [Установка на macOS](#установка-на-macos)
  * [Инструменты](#инструменты)
     * [Утилиты командной строки](#утилиты-командной-строки)
        * [mysql - Встроенный клиент командной строки MySQL](#mysql---встроенный-клиент-командной-строки-mysql)
        * [mysqladmin - Встроенный клиент для администрирования сервера MySQL](#mysqladmin---встроенный-клиент-для-администрирования-сервера-mysql)
        * [innotop - Утилита для мониторинга сервера MySQL в реальном времени](#innotop---утилита-для-мониторинга-сервера-mysql-в-реальном-времени)
        * [Percona Toolkit - Набор инструментов командной строки](#percona-toolkit---набор-инструментов-командной-строки)
     * [Интерфейсы](#интерфейсы)
        * [Sequel Pro - приложение на macOS для администрирования БД](#sequel-pro---приложение-на-macos-для-администрирования-бд)
        * [PhpMyAdmin - PHP-приложение для администрирования и мониторинга БД в браузере](#phpmyadmin---php-приложение-для-администрирования-и-мониторинга-бд-в-браузере)
        * [MySQL Workbench - приложение для визуального проектирования и администрирования БД](#mysql-workbench---приложение-для-визуального-проектирования-и-администрирования-бд)
  * [Конфигурация](#конфигурация)

<!-- Added by: grisha_k, at:  -->

<!--te-->

## Ссылки

1. [MacOS 10.14 Mojave Apache Setup: MySQL, APC & More...](https://getgrav.org/blog/macos-mojave-apache-mysql-vhost-apc).

## Установка на macOS

Сначала устанавливаем [HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем MariaDB/MySQL.

	$ brew update
	$ brew install mariadb
	$ mysql --version
	mysql  Ver 15.1 Distrib 10.3.10-MariaDB, for osx10.14 (x86_64) using readline 5.1
	
Запускаем MariaDB в режиме [демона](https://ru.wikipedia.org/wiki/Демон_(программа)), чтобы он работал в фоновом режиме и запускался автоматически после перезагрузки системы:
	
	$ brew services start mariadb
	
Для перезапуска или остановки используются следующие команды:

	$ brew services restart mariadb
	$ brew services stop mariadb

Необходимо выполнить базовые настройки безопасности ([MariaDB](https://mariadb.com/kb/en/library/mysql_secure_installation/)/[MySQL](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html)) на рабочем сервере и желательно на локальном, для этого необходимо запустить скрипт mysql\_secure\_installation в командной строке, установить пароль для пользователя root (по умолчанию пароль не установлен) и согласиться со всеми предложенными изменениями:

	$ mysql_secure_installation
		# $ which mysql_secure_installation
		# /usr/local/bin/mysql_secure_installation

> "Enter current password for root (enter for none): " - После установки MariaDB/MySQL не установлен пароль для пользователя root, поэтому если пароль не установлен, то оставляем поле пустым.
> 
> "Change the root password? [Y/n]: y" - Если пароль не установлен для пользователя root, то подтверждаем что будем его устанавливать.
> 
> "Remove anonymous users? [Y/n]: y" - Подтверждаем удаление анонимных пользователей.
> 
> "Disallow root login remotely? [Y/n]: y" - Подтверждаем, что пользователю root нельзя подключаться удаленно, только локально.
> 
> "Remove test database and access to it? [Y/n]: y" - Подтверждаем удаление тестовой базы данных и привилегий к ней.
> 
> "Reload privilege tables now? [Y/n]: y" - Подтверждаем перезагрузка таблиц привилегий, чтобы все изменения вступили в силу немедленно.

## Инструменты

### Утилиты командной строки

#### mysql - Встроенный клиент командной строки MySQL

Используется для входа в MySQL и выполнения SQL запросов:

	$ mysql -u root -p -h 127.0.0.1 -P 3306

> -u - имя пользователя.
> 
> -p - флаг для ввода пароля в интерактивном режиме (сразу вводить не нужно).
> 
> -h, -P - по желанию можно указать хост и номер порта.

Примеры SQL запросов для мониторинга сервера:

	> SHOW VARIABLES LIKE "%wait%";
	> SHOW GLOBAL STATUS;
	> SHOW FULL PROCESSLIST;
	> SHOW ENGINE INNODB STATUS\G
	> quit

#### mysqladmin - Встроенный клиент для администрирования сервера MySQL

Проверить работу сервера можно командой:

	$ mysqladmin -u root -p ping
		# $ which mysqladmin
		/usr/local/bin/mysqladmin
	Enter password: 
	mysqld is alive

Вывести текущий статус сервера:

	$ mysqladmin -u root -p status
	
Вывести все переменные состояния и их значения:

	$ mysqladmin -u root -p extended-status
	
Вывести все переменные и их значения:
	
	$ mysqladmin  -u root -p variables
	
Вывести все запущенные процессы сервера:

	$ mysqladmin -u root -p processlist
	
> Эквивалентно SHOW PROCESSLIST. С ключем "--verbose", эквивалентно SHOW FULL PROCESSLIST.

Убить процесс:
	
	$ mysqladmin -u root -p kill ID

> ID - ИД процесса.

Изменить пароль пользователя root:

	$ mysqladmin -u root -p password	

>  Далее необходимо будем ввести старый пароль, затем 2 раза новый.

Вывести текущую версию сервера:

	$ mysqladmin -u root -p version

Список все команд:

	$ mysqladmin --help

#### innotop - Утилита для мониторинга сервера MySQL в реальном времени

Установить [innotop](https://www.percona.com/blog/2013/10/14/innotop-real-time-advanced-investigation-tool-mysql/) можно командой:

	$ brew install innotop
	
Пример запуска утилиты:
	
	$ innotop -u root -p password -h 127.0.0.1 -P 3306

> Ключи такие же как и у встроенных утилит mysql, но отличие только в том, что после ключа "-p" нужно сразу вводить пароль в терминале (вместо password), поэтому после выхода лучше очистить историю терминала командой "history -c".

После вызова утилиты, можно вызвать справку с помощью ввода символа "?" [shift + ?], где будут отображены горячие клавиши и описания.

#### Percona Toolkit - Набор инструментов командной строки

Установить [Percona Toolkit](https://www.percona.com/software/database-tools/percona-toolkit) можно командой:

	$ brew install percona-toolkit

> После установки, в командной строке станет доступно [множество утилит с префиксом pt](https://www.percona.com/doc/percona-toolkit/LATEST/index.html).

### Интерфейсы
	
#### Sequel Pro - приложение на macOS для администрирования БД

Для macOS установим приложение [Sequel Pro](http://www.sequelpro.com):

	$ brew cask install sequel-pro

Откроем Sequel Pro:
 
 1. Выберем способ подключение.
 	* При стандартном подключении в качестве имени хоста (Host) укажем "127.0.0.1", номер порта (Port) оставляем по умолчанию 3306 (или 3307).
 	* При подключение через "Сокет" (Socket) по умолчанию используется сокет /tmp/mysql.sock, поэтому мы можем оставить поле сокет (Socket) пустым (пока не поменяем расположение сокета).
 1. В качестве имени (Name) укажем "localhost".
 2. Укажем имя пользователя (Username) root и пароль (Password).
 3. Сейчас у нас используется сокет (Socket) по умолчанию (/tmp/mysql.sock), поэтому можем оставить поле пустым (пока не поменяем расположение сокета).
 3. Добавим подключение в избранное (Add to Favorites) и нажмем "Соединить" (Connect).

#### PhpMyAdmin - PHP-приложение для администрирования и мониторинга БД в браузере 

Также можем установить PHP-приложение [PhpMyAdmin](https://ru.wikipedia.org/wiki/PhpMyAdmin) для локального использования в браузере:

	$ brew install phpmyadmin

> После установки будет выведена подсказка по настройке Apache и путь до конфигурационного файла /usr/local/etc/phpmyadmin.config.inc.php.

Добавим доступ к phpmyadmin из браузера, для этого либо добавим предложенные настройки конфигурации в /usr/local/etc/httpd/httpd.conf, либо настроем конфигурационный файла виртуальных хостов по умолчанию на 443 порту, для этого откроем конфигурационный файл:

	$ nano /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_80_.conf
	
И в конце блока \<VirtualHost\> перед строчкой \</VirtualHost\> добавим предложенный код:

	Alias /phpmyadmin /usr/local/share/phpmyadmin
    <Directory /usr/local/share/phpmyadmin/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        <IfModule mod_authz_core.c>
            Require all granted
        </IfModule>
        <IfModule !mod_authz_core.c>
            Order allow,deny
            Allow from all
        </IfModule>
    </Directory>

Перезапустим Apache:

	$ sudo apachectl -k restart

И откроем в браузере <https://localhost/phpmyadmin> (или по протоколу http, если настройки делали в httpd.conf).

Теперь откроем конфигурационный файл phpmyadmin:

	$ nano /usr/local/etc/phpmyadmin.config.inc.php

Найдем строку с параметров "blowfish_secret" и добавим для него значение из 32 любых символов:

	$cfg['blowfish_secret'] = '';
	
#### MySQL Workbench - приложение для визуального проектирования и администрирования БД

Установить приложение [MySQL Workbench](https://ru.wikipedia.org/wiki/MySQL_Workbench) можно командой:

	$ brew cask install mysqlworkbench

## Конфигурация

<https://tools.percona.com/wizard>

<http://qaru.site/questions/24764/how-to-discover-number-of-logical-cores-on-mac-os-x>

<https://mariadb.com/kb/en/library/mariadb-vs-mysql-compatibility/>