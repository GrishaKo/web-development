# Заметки по использованию HTTP-сервера Apache

[Apache HTTP-сервер](https://ru.wikipedia.org/wiki/Apache_HTTP_Server) — свободный веб-сервер. 

<!--ts-->
<!--te-->

## Ссылки

1. [MacOS 10.14 Mojave Apache Setup: Multiple PHP Versions](https://getgrav.org/blog/macos-mojave-apache-multiple-php-versions).

## Установка на macOS

Сначала устанавливаем [HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем Apache.

Сразу следует установить пакеты [OpenLDAP](https://ru.wikipedia.org/wiki/OpenLDAP) и [libiconv](https://www.gnu.org/software/libiconv/):

	$ brew install openldap libiconv 

Затем, если запущен Apache 2.4 предустановленный в masOS Mojave, следует его остановить (в любом случае не помешает выполнить эти команды):

	$ sudo apachectl stop
	$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null

Теперь устанавливаем Apache:

	$ brew install httpd
	$ httpd -v

Запускаем Apache в режиме [демона](https://ru.wikipedia.org/wiki/Демон_(программа)), чтобы он работал в фоновом режиме и запускался автоматически после перезагрузки системы:

	$ sudo brew services start httpd

Теперь можно проверить работу Apache, открыв в браузере ссылку <http://localhost:8080> - должна отобразится страница с текстом "It works!".

В случае необходимости можно посмотреть журнал ошибок Apache:

	$ tail -f /usr/local/var/log/httpd/error_log

Команда apachectl управляет запуском, остановкой и перезапуском Apache:

	$ sudo apachectl start
	$ sudo apachectl stop
	$ sudo apachectl -k restart

>  Опция "-k" перезапускает Apache немедленно.

## Конфигурация

Основной файл конфигурации расположен в /usr/local/etc/httpd/httpd.conf.

Открываем httpd.conf из командной строки, например, в редакторе TextEdit:

	$ open -a /Applications/TextEdit.app /usr/local/etc/httpd/httpd.conf

> Можно настроить [алиас в терминале](Terminal), для сокращенного использования команды "open -a /Applications/TextEdit.app".


	