# Заметки по использованию MySQL

[MySQL](https://ru.wikipedia.org/wiki/MySQL) ([EN](https://ru.wikipedia.org/wiki/MySQL)) - реляционной системы управления базами данных.

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
2. [MySQL по максимуму. 3-е издание. Шварц Бэрон, Зайцев Пётр, Ткаченко Вадим](https://books.google.ru/books?id=A7daDwAAQBAJ&lpg=PA1&hl=ru&pg=PA1#v=onepage&q&f=false) ([EN](https://books.google.ru/books?id=D0b_Xg3UeXEC&lpg=PP1&hl=ru&pg=PP1#v=onepage&q&f=false)).

## Установка на macOS

Сначала устанавливаем [HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем [MariaDB/MySQL](https://ru.wikipedia.org/wiki/MariaDB) ([EN](https://en.wikipedia.org/wiki/MariaDB)).

	$ brew update
	$ brew install mariadb
	$ mysql --version
	mysql  Ver 15.1 Distrib 10.3.10-MariaDB, for osx10.14 (x86_64) using readline 5.1
	
Запускаем MariaDB в режиме [демона](https://ru.wikipedia.org/wiki/Демон_(программа)) ([EN](https://en.wikipedia.org/wiki/Daemon_(computing))), чтобы он работал в фоновом режиме и запускался автоматически после перезагрузки системы:
	
	$ brew services start mariadb
	
Для перезапуска или остановки используются следующие команды:

	$ brew services restart mariadb
	$ brew services stop mariadb

Необходимо выполнить базовые настройки безопасности ([MariaDB](https://mariadb.com/kb/en/library/mysql_secure_installation/)/[MySQL](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html)) на рабочем сервере и желательно на локальном, для этого необходимо запустить скрипт `mysql_secure_installation` в командной строке, установить пароль для пользователя root (по умолчанию пароль не установлен) и согласиться со всеми предложенными изменениями:

	$ mysql_secure_installation
		# $ which mysql_secure_installation
		# /usr/local/bin/mysql_secure_installation

> `Enter current password for root (enter for none):` - После установки MySQL не установлен пароль для пользователя root, поэтому если пароль не установлен, то оставляем поле пустым.
> 
> `Change the root password? [Y/n]:y` - Если пароль не установлен для пользователя root, то подтверждаем что будем его устанавливать.
> 
> `Remove anonymous users? [Y/n]:y` - Подтверждаем удаление анонимных пользователей.
> 
> `Disallow root login remotely? [Y/n]:y` - Подтверждаем, что пользователю root нельзя подключаться удаленно, только локально.
> 
> `Remove test database and access to it? [Y/n]:y` - Подтверждаем удаление тестовой базы данных и привилегий к ней.
> 
> `Reload privilege tables now? [Y/n]:y` - Подтверждаем перезагрузка таблиц привилегий, чтобы все изменения вступили в силу немедленно.

## Инструменты

### Утилиты командной строки

#### mysql - Встроенный клиент командной строки MySQL

1. <https://dev.mysql.com/doc/refman/5.7/en/mysql.html>
2. <https://mariadb.com/kb/en/library/mysql-command-line-client/>

Используется для входа в MySQL и выполнения SQL запросов:

	$ mysql -u root -p -h 127.0.0.1 -P 3306

> `-u` - имя пользователя.
> 
> `-p` - флаг для ввода пароля в интерактивном режиме (сразу вводить не нужно).
> 
> `-h, -P` - по желанию можно указать хост и номер порта.

Примеры SQL запросов для мониторинга сервера:

	mysql> SHOW VARIABLES LIKE "%wait%";
	
Вывести все переменные состояния и их глобальные значения:

	mysql> SHOW GLOBAL STATUS;

Вывести значения переменных состояния, начинающихся с фразы "Key_read", для текущего подключения:

	mysql> SHOW STATUS like 'Key_read%';

Вывести все переменные и их значения:

	mysql> SHOW VARIABLES;

Вывести значение переменной datadir:

	$ SHOW VARIABLES like 'datadir';

Вывести все запущенные процессы сервера:

	mysql> SHOW FULL PROCESSLIST;
	mysql> SHOW FULL PROCESSLIST\G

> `\G` - выводит табличную информацию построчно.
	
> Без ключевого слова `FULL` в поле "Информация" (Info) отображаются только первые 100 символов каждого оператора.

Вывести информацию о состоянии механизма хранения InnoDB:

	mysql> SHOW ENGINE INNODB STATUS\G

#### mysqladmin - Встроенный клиент для администрирования сервера MySQL

1. <https://dev.mysql.com/doc/refman/5.7/en/mysqladmin.html>
2. <https://mariadb.com/kb/en/library/mysqladmin/>

Проверить работу сервера можно командой:

	$ mysqladmin -u root -p ping
		# найти расположение утилиты можно командой
		# $ which mysqladmin
		# или
		# $ sudo find /usr/local -name mysqladmin
		# /usr/local/bin/mysqladmin
	Enter password: 
	mysqld is alive

Вывести текущий статус сервера:

	$ mysqladmin -u root -p status
	
Вывести все переменные состояния и их значения:

	$ mysqladmin -u root -p extended-status

Вывести значение переменной состояния `Key_reads` и обновлять данные каждые 10 секунд:

	$ mysqladmin extended-status -r -i 10 | grep Key_reads
	
Вывести все переменные и их значения:
	
	$ mysqladmin  -u root -p variables

Вывести значение переменной `datadir`:

	$ mysqladmin  -u root -p variables | grep datadir
	
Вывести все запущенные процессы сервера:

	$ mysqladmin -u root -p processlist
	
> Эквивалентно `SHOW PROCESSLIST`. С ключем `--verbose`, эквивалентно `SHOW FULL PROCESSLIST`.

Убить процесс:
	
	$ mysqladmin -u root -p kill ID

> `ID` - ИД процесса.

Изменить пароль пользователя root:

	$ mysqladmin -u root -p password	

>  Далее необходимо будем ввести старый пароль, затем 2 раза новый.

Вывести текущую версию сервера:

	$ mysqladmin -u root -p version

Список все команд:

	$ mysqladmin --help
	
#### mysqldumpslow - Встроенный клиент для анализа журнала медленных запросов MySQL	

	$ mysqldumpslow -s at -t 1 /usr/local/mysql/data/server-slow.log

#### innotop - Утилита для мониторинга сервера MySQL в реальном времени

Установить [innotop](https://www.percona.com/blog/2013/10/14/innotop-real-time-advanced-investigation-tool-mysql/) можно командой:

	$ brew install innotop
	
Пример запуска утилиты:
	
	$ innotop -u root -p password -h 127.0.0.1 -P 3306

> Ключи такие же как и у встроенных утилит mysql, но отличие только в том, что после ключа `-p` нужно сразу вводить пароль в терминале (вместо password), поэтому после выхода лучше очистить историю терминала командой `history -c`.

После вызова утилиты, можно вызвать справку с помощью ввода символа `?`, где будут отображены горячие клавиши и описания.

#### Percona Toolkit - Набор дополнительных инструментов командной строки для MySQL

Установить [Percona Toolkit](https://www.percona.com/software/database-tools/percona-toolkit) можно командой:

	$ brew install percona-toolkit

После установки, в командной строке станет доступно [множество утилит с префиксом pt](https://www.percona.com/doc/percona-toolkit/LATEST/index.html), например, утилита для улучшенного анализа лога медленных запросов:

	$ pt-query-digest /usr/local/var/mysql/mysql-slow.log > ~/mysql-slow.log

> `> ~/mysql-slow.log` - вывод результата лога в файл в домашнем каталоге.

### Интерфейсы
	
#### Sequel Pro - приложение на macOS для администрирования БД

Для macOS установим приложение [Sequel Pro](http://www.sequelpro.com):

	$ brew cask install sequel-pro

Откроем Sequel Pro:
 
 1. Выберем способ подключение.
 	* При стандартном подключении в качестве имени хоста (Host) укажем `127.0.0.1`, номер порта (Port) оставляем по умолчанию `3306` (или 3307).
 	* При подключение через "Сокет" (Socket) по умолчанию используется сокет `/tmp/mysql.sock`, поэтому мы можем оставить поле сокет (Socket) пустым (пока не поменяем расположение сокета).
 1. В качестве имени (Name) укажем `localhost`.
 2. Укажем имя пользователя (Username) `root` и пароль (Password).
 3. Сейчас у нас используется сокет (Socket) по умолчанию, поэтому можем оставить поле пустым (пока не поменяем расположение сокета).
 3. Добавим подключение в избранное (Add to Favorites) и нажмем "Соединить" (Connect).

#### PhpMyAdmin - PHP-приложение для администрирования и мониторинга БД в браузере 

Также можем установить PHP-приложение [PhpMyAdmin](https://ru.wikipedia.org/wiki/PhpMyAdmin) ([EN](https://ru.wikipedia.org/wiki/PhpMyAdmin)) для локального использования в браузере:

	$ brew install phpmyadmin

> После установки будет выведена подсказка по настройке Apache и путь до конфигурационного файла `/usr/local/etc/phpmyadmin.config.inc.php`.

Добавим доступ к phpmyadmin из браузера, для этого либо добавим предложенные настройки конфигурации в `/usr/local/etc/httpd/httpd.conf`, либо настроем конфигурационный файла виртуальных хостов по умолчанию на 443 порту, для этого откроем конфигурационный файл:

	$ nano /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_80_.conf
	
И в конце блока `<VirtualHost>` перед строчкой `</VirtualHost>` добавим предложенный код:

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

И откроем в браузере <https://localhost/phpmyadmin> (или по протоколу http, если настройки делали в `httpd.conf`).

Теперь откроем конфигурационный файл phpmyadmin:

	$ nano /usr/local/etc/phpmyadmin.config.inc.php

Найдем строку с параметров `blowfish_secret` и добавим для него значение из 32 любых символов:

	$cfg['blowfish_secret'] = '';
	
#### MySQL Workbench - приложение для визуального проектирования и администрирования БД

Установить приложение [MySQL Workbench](https://ru.wikipedia.org/wiki/MySQL_Workbench) ([EN](https://en.wikipedia.org/wiki/MySQL_Workbench)) можно командой:

	$ brew cask install mysqlworkbench

## Конфигурация

Конфигурация MySQL осуществляется путем настройки файла `my.cnf`, возможные пути его расположения можно посмотреть командой:

	$ mysqld --help --verbose | grep my.cnf 
	/usr/local/etc/my.cnf ~/.my.cnf 

В файле `/usr/local/etc/my.cnf` есть указание на то, что конфигурация также читается из каталога `my.cnf.d`:

	!includedir /usr/local/etc/my.cnf.d
	
Откроем для редактирования наш новый конфигурационный файл MySQL:

	$ nano /usr/local/etc/my.cnf.d/server.cnf
		# или создаем файл и открываем в другом текстовом редакторе
		# $ > /usr/local/etc/my.cnf.d/server.cnf
		# $ code /usr/local/etc/my.cnf.d/server.cnf

> В данном примере команда code - это [алиас для команды терминала](../Terminal/readme.md#Добавление-алиасов-для-команд-терминала), которая открывает файл в указанном текстовом редакторе.
> 
> Имя `server.cnf` выбрано исходя из книги "[Red Hat Enterprise Linux Troubleshooting Guide (Chapter 3: Troubleshooting a Web Application / Finding the MariaDB Folder)](https://books.google.ru/books?id=kQCACwAAQBAJ&lpg=PA83&dq=mariadb%20catalog%20my.cnf.d&hl=ru&pg=PA82#v=onepage&q=mariadb%20catalog%20my.cnf.d&f=false)".

Для стартовой конфигурации MySQL воспользуемся онлайн-инструментом "[Percona Tools for MySQL](https://tools.percona.com/wizard)" и далее будем настраивать под себя, используя также другие профессиональные рекомендации:

1. [MySQL по максимуму. 3-е издание. Шварц Бэрон, Зайцев Пётр, Ткаченко Вадим. (Глава 8: Оптимизация параметров сервера / Создание конфигурационного файла MySQL)](https://books.google.ru/books?id=A7daDwAAQBAJ&lpg=PA1&hl=ru&pg=PA397#v=onepage&q&f=false) ([EN](https://books.google.ru/books?id=D0b_Xg3UeXEC&lpg=PP1&hl=ru&pg=PA342#v=onepage&q&f=false)).
2. [Настройка MySQL для веб-приложений от Николая Лавлинского](https://youtu.be/NTfjB-TUock).
3. [Анализ производительности MySQL для ускорения веб-приложений от Николая Лавлинского](https://youtu.be/UMXbfRHoCMk).

Значения по умолчанию в конфигурации [MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) и, например, [MariaDB/MySQL](https://mariadb.com/kb/en/library/server-system-variables/) отличаются, следует ознакомиться со значениями по умолчанию для изменяемых параметров и рекомендациями по их изменению.

**Базовая настройка для секции [client]:**

	[client]

	socket                         = /usr/local/var/	mysql/mysql.sock
	port                           = 3306

> `[client]` - настройки для [утилит командной строки](#утилиты-командной-строки).

**Базовые настройки:**

	[mysqld]

	# GENERAL

	datadir                        = /usr/local/var/mysql/
	socket                         = /usr/local/var/mysql/mysql.sock
	pid_file                       = /usr/local/var/mysql/mysql.pid
	user                           = mysql
	port                           = 3306
	default_storage_engine         = InnoDB

> `[mysqld]` - настройки для демона mysqld, все остальные настройки далее будут относится к этой секции.
>
> `datadir` - путь до каталог с данными указываем как по умолчанию.
> 
> `socket` - меняем расположение сокета (по умолчанию обычно `/tmp/mysql.sock`).
> 
> `pid_file` - меняем имя pid файла (путь не меняем).
> 
> `user` - указываем имя пользователя.
> 
> `port` - имя порта указываем как по умолчанию.
> 
> `default_storage_engine` - механизм хранения данных, который будет использоваться по умолчанию.

**Настройка логирования:**

	# LOGGING
	
	log_error                      = /usr/local/var/mysql/mysql-error.log
	
	slow_query_log_file            = /usr/local/var/mysql/mysql-slow.log
	slow_query_log                 = 0
	long_query_time                = 0
	log_queries_not_using_indexes  = 1
	
	#log_bin                        = /usr/local/var/mysql/mysql-bin
	expire_logs_days               = 14
	sync_binlog                    = 1

>  `log_error` - указываем имя лога ошибок (путь не меняем).
> 
> `slow_query_log_file` - имя лога медленных запросов (путь не меняем).
> 
> `slow_query_log` - лог медленных запросов указываем выключенным, чтобы его можно было включать динамические на определенный срок для сборка статистики и последующего анализа.
> 
> `long_query_time` - убираем ограничение на время выполнения запроса (по умолчанию 10 секунд), если лог будет включен не всегда и для его анализа будет использоваться утилита [pt-query-digest](#Percona-Toolkit---Набор-дополнительных-инструментов-командной-строки-для-MySQL), то будем собирать статистику всех запросов и на ее основе оценивать процентное соотношение медленных запросов.
> 
> `log_queries_not_using_indexes` - запросы, не использующие индексы, будут записываться в лог медленных запросов.
> 
> `#log_bin` - укажем имя бинарного лога, но включать не будем, закомментируем это параметр, потому что его включении MySQL сервер буде работать медленнее, а необходим лог впервую очередь при репликации и полезен при восстановлении после сбоев (подробнее в документации [MySQL](https://dev.mysql.com/doc/refman/5.5/en/binary-log.html) и [MariaDB](https://mariadb.com/kb/en/library/overview-of-the-binary-log/)).
> 
> `expire_logs_days` - время жизни бинарного лога.
> 
> `sync_binlog` - указываем, как часто сервер MySQL должен синхронизировать двоичный журнал на диск.

Для анализа медленных запросов нужно включать лог медленных запросов на время (например, на 24 часа), без перезагрузки сервера MySQL, командой:

	$ mysql -u root -p
	mysql> set global slow_query_log = 1;

> После сбора лога медленных запросов отключаем его такой же командой, указав 0 вместо 1.

В приложении macOS Console в списке логов нам доступен каталог в домашней директории пользователя `~/Library/Logs/Homebrew/mariadb` (или mysql), поэтому создадим в этом каталоге жесткие ссылки на лог-файлы ошибок и медлeнных запросов в каталоге по умолчанию: 

	$ ln /usr/local/var/mysql/mysql-error.log ~/Library/Logs/Homebrew/mariadb/mysql-error.log
	$ ln /usr/local/var/mysql/mysql-slow.log ~/Library/Logs/Homebrew/mariadb/mysql-slow.log
	
**Настройка для MyISAM:**

	# MyISAM
	
	key_buffer_size                = 32M
	
	myisam_recover_options         = BACKUP,FORCE
	
> `key_buffer_size` - размер буфера ключей наиболее важный параметр для использования MyISAM таблиц. Даже если MyISAM таблицы не используются, стоит оставлять минимальное значение для внутренних целей MySQL.
> 
> `myisam_recover_options` - параметр автоматического восстановление таблиц после сбоя. `BACKUP` - означает делать резервную копию файла .MYD перед восстановлением. `FORCE` - означает запуск восстановление, даже если мы потеряем более одной строки из файла .MYD (подробнее в документации [MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_myisam-recover-options) и [MariaDB](https://mariadb.com/kb/en/library/myisam-system-variables/#myisam_recover_options)).

Размер `key_buffer_size` не стоит выставлять больше чем размер всех MYI-файлов, что можно подсчитать командой:

	$ mysql -u root -p
	mysql> SELECT SUM(INDEX_LENGTH)/1024/1024 FROM INFORMATION_SCHEMA.TABLES WHERE ENGINE='MyISAM';
		# или посчитать размер файлов на диске
		# $ du -sch `find /usr/local/mysql/ -name "*.MYI"`

После изменения значения переменной `key_buffer_size` нужно оценить эффективность изменений спустя 24 часа. Для оценки производительности чтения буфера ключей можно вычислить соотношение переменных состояния "`Key_reads` / `Key_read_requests`", которое обычно должно быть меньше 0,01 (в идеале не больше 0,001). Для более полной оценки эффективности значения переменной `key_buffer_size` можно воспользоваться следующими рекомендациями:

1. [MySQL - Check the performance of the key buffer](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_key_buffer_size).
2. [MariaDB - Optimizing key-buffer-size](https://mariadb.com/kb/en/library/optimizing-key_buffer_size/).
3. [MySQL по максимуму. 3-е издание. Шварц Бэрон, Зайцев Пётр, Ткаченко Вадим. (Глава 8: Оптимизация параметров сервера / Кэш ключей MyISAM)](https://books.google.ru/books?id=A7daDwAAQBAJ&lpg=PA1&hl=ru&pg=PA384#v=onepage&q&f=false) ([EN](https://books.google.ru/books?id=D0b_Xg3UeXEC&lpg=PP1&hl=ru&pg=PA351#v=onepage&q&f=false)).

**Кеш и лимиты:**

	# CACHE AND LIMITS
	
	tmp-table-size                 = 32M
	max-heap-table-size            = 32M
	
	query_cache_type               = 0
	query_cache_size               = 0
	
	max_allowed_packet             = 16M
	max_connect_errors             = 1000000
	max_connections                = 500
	
	thread_cache_size              = 50
	
	open_files_limit               = 65535
	
	table_open_cache               = 2048
	table_definition_cache         = 1024

> `tmp-table-size`, `max-heap-table-size` - Максимальный размер временных таблиц в памяти, в зависимости от того, какой из этих параметров меньше. Для таблиц MEMORY ограничением является только значение `max-heap-table-size`. Чем меньше временных таблиц создается на диске, тем лучше, что можно проанализировать, вычислив какой процент составляют временные таблицы на диске от всех временных таблиц, используя переменные состояния: `Created_tmp_disk_tables` / (`Created_tmp_disk_tables` + `Created_tmp_tables`) * 100 (подробнее в документации [MySQL](https://dev.mysql.com/doc/refman/5.7/en/internal-temporary-tables.html) и [MariaDB](https://mariadb.com/kb/en/library/server-system-variables/#tmp_table_size)). 
> 
> `query_cache_type`, `query_cache_size` - отключаем кеширование MySQL, так как на нашем сервере будет очень часто происходить добавление/обновление записей в БД, что приведет к частому сбрасыванию кэша и в результате высокой нагрузки на сервер. При этом если для `query_cache_type` указать значение 2, то в таком случае кэш будет применяться только к тем запросам, для которых принудительно указано в самом запросе о необходимости кэширования "[SELECT SQL_CACHE](https://dev.mysql.com/doc/refman/5.7/en/query-cache-in-select.html)".
>  
> `max_allowed_packet` - максимальный размера пакета, устанавливаем минимально рекомендуемое значение и в случае возникновения ошибки, указывающей на недостаточный размер этого параметра, увеличиваем его значение (подробнее в документации [MySQL](https://dev.mysql.com/doc/refman/8.0/en/packet-too-large.html) и [MariaDB](https://mariadb.com/kb/en/library/server-system-variables/#max_allowed_packet)).
> 
> `max_connect_errors` - установлено рекомендуемое ограничение на количество последовательных неудачных соединений с хостом, прежде чем хост будет заблокирован от дальнейших соединений (подробнее в документации [MySQL - Host 'host_name' is blocked](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_max_connect_errors)).
> 
> `max_connections` - увеличиваем максимальное количество одновременных клиентских подключений для исключения ошибки [Too many connections](https://www.percona.com/blog/2013/11/28/mysql-error-too-many-connections/).
> 
> `thread_cache_size` - максимальное количество потоков сохраняемых в кэше потоков, после закрытия соединения. При новом открытии соединения поток используется из кэша, что позволяет открывать соединения очень быстро и экономить ресурсы при большом количестве одновременных соединений. По умолчанию - это динамическая величина и автоматически определяется в зависимости от значения переменной `max_connections` (подробнее в документации [MySQL](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_thread_cache_size) и [MariaDB](https://mariadb.com/kb/en/library/server-system-variables/#thread_cache_size)).
> 
> `open_files_limit` - максимальное количество открытых файлов рекомендуется ставить достаточно большим чтобы исключить ошибку "Слишком много открытых файлов".
> 
> `table_open_cache`, `table_definition_cache` - переменные служат для конфигурации кэша открытых таблиц и кэша определений таблиц из которых состоит кэш таблиц (подробнее в документации [MySQL](https://dev.mysql.com/doc/refman/5.7/en/table-cache.html) и [MariaDB](https://mariadb.com/kb/en/library/optimizing-table_open_cache/)). 

Для определения в необходимости установки `thread_cache_size` следует посмотреть значения переменной статуса `Threads_created`, которое должно быть равно или чуть больше `Threads_cached`, если разница большая, то следует стремится к уменьшению разницы, для это необходимо увеличить значение `thread_cache_size`, установив его равным значению `Threads_connected` или чуть больше чем максимальное количество одновременных соединений с момента старта сервера, отображаемое в переменной статуса `Max_used_connections`:

	SHOW GLOBAL STATUS WHERE variable_name LIKE 'Threads%' OR variable_name LIKE 'Max_used_connections';
	+----------------------+-------+
	| Variable_name        | Value |
	+----------------------+-------+
	| Max_used_connections | 20    |
	| Threads_cached       | 19    |
	| Threads_connected    | 1     |
	| Threads_created      | 20    |
	| Threads_running      | 1     |
	+----------------------+-------+

При этом значение `thread_cache_size` следует ограничить, если оно значительно больше, чем `Max_used_connections`. Поэтому если количество одновременных соединений не более 50, то можно ограничить `thread_cache_size` значением 50. Дополнительные рекомендации:

1. [MySQL/MariaDB: тюнинг производительности #1: thread-cache-size](https://rtfm.co.ua/mysqlmariadb-tyuning-proizvoditelnosti-1-thread_cache_size/).
2. [MySQL Optimization Tip - thread_cache_size](http://anothermysqldba.blogspot.com/2013/09/mysql-optimization-tip-threadcachesize.html).

**Настройка для InnoDB:**

	# INNODB
	
	innodb_file_per_table          = 1
	innodb_flush_method            = O_DIRECT
	innodb_buffer_pool_size        = 8G
	
	innodb_log_file_size           = 256M
	innodb_log_files_in_group      = 2
	innodb_flush_log_at_trx_commit = 2
	
	
--	
	
https://www.percona.com/blog/2017/10/18/chose-mysql-innodb_log_file_size/
https://blog.programs74.ru/how-to-change-innodb_log_file_size-safely/#
	
	SHOW STATUS LIKE 'Threads_connected';
	SHOW STATUS LIKE 'Threads_created';
	SHOW STATUS LIKE 'Threads_cached';
	SHOW STATUS LIKE 'Slow_launch_threads';

<http://qaru.site/questions/24764/how-to-discover-number-of-logical-cores-on-mac-os-x>

<https://mariadb.com/kb/en/library/mariadb-vs-mysql-compatibility/>

https://dev.mysql.com/doc/refman/8.0/en/problems-with-mysql-sock.html