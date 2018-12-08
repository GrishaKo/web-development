# Заметки по использованию HTTP-сервера Apache

[Apache HTTP-сервер](https://ru.wikipedia.org/wiki/Apache_HTTP_Server) — свободный веб-сервер. 

<!--ts-->
   * [Заметки по использованию HTTP-сервера Apache](#заметки-по-использованию-http-сервера-apache)
      * [Ссылки](#ссылки)
      * [Установка на macOS](#установка-на-macos)
      * [Конфигурация](#конфигурация)
         * [Ссылки](#ссылки-1)
         * [Конфигурация httpd.conf](#конфигурация-httpdconf)
         * [Логирование](#логирование)
         * [Конфигурация виртуальных хостов](#конфигурация-виртуальных-хостов)
         * [Локальное использование домена test](#локальное-использование-домена-test)
         * [Конфигурация SSL/TLS для работы виртуальных хостов по протоколу HTTPS](#конфигурация-ssltls-для-работы-виртуальных-хостов-по-протоколу-https)

<!-- Added by: grisha_k, at:  -->

<!--te-->

## Ссылки

1. [MacOS 10.14 Mojave Apache Setup: Multiple PHP Versions](https://getgrav.org/blog/macos-mojave-apache-multiple-php-versions).
2. [Apache HTTP Server Version 2.4 Documentation](https://httpd.apache.org/docs/2.4/)

## Установка на macOS

Сначала устанавливаем [HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем Apache.

Сразу следует установить библиотеки [OpenLDAP](https://ru.wikipedia.org/wiki/OpenLDAP) и [libiconv](https://www.gnu.org/software/libiconv/):

	$ brew install openldap libiconv 

Затем, если запущен Apache 2.4 предустановленный в masOS Mojave, следует его остановить (в любом случае не помешает выполнить эти команды):

	$ sudo apachectl stop
	$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null

Теперь устанавливаем Apache:

	$ brew install httpd
	$ httpd -v

Запускаем Apache в режиме [демона](https://ru.wikipedia.org/wiki/Демон_(программа)), чтобы он работал в фоновом режиме и запускался автоматически после перезагрузки системы:

	$ sudo brew services start httpd

> Теперь можно проверить работу Apache, открыв в браузере ссылку <http://localhost:8080> - должна отобразится страница с текстом "It works!".

В случае необходимости можно посмотреть журнал ошибок Apache:

	$ tail -f /usr/local/var/log/httpd/error_log

Команда apachectl управляет запуском, остановкой и перезапуском Apache:

	$ sudo apachectl start
	$ sudo apachectl stop
	$ sudo apachectl -k restart

>  Опция "-k" перезапускает Apache немедленно.

## Конфигурация

Глобальный файл конфигурации расположен в /usr/local/etc/httpd/httpd.conf, дополнительные в /usr/local/etc/httpd/extra.

Оригиналы всех конфигурационных файлов расположены в /usr/local/etc/httpd/original.

### Ссылки

1. [MacOS 10.14 Mojave Apache Setup: SSL](https://getgrav.org/blog/macos-mojave-apache-ssl).
2. [Генерация CSR-запроса на Linux/MacOS](https://1cloud.ru/help/ssl/orderssllinux).
3. [Mozilla's Server Side TLS Guidelines](https://wiki.mozilla.org/Security/Server_Side_TLS).

### Конфигурация httpd.conf

Открыть httpd.conf можно из командной строки, например, в редакторе TextEdit:

	$ open -a /Applications/TextEdit.app /usr/local/etc/httpd/httpd.conf
		# или в терминальном редакторе nano
		# $ nano /usr/local/etc/httpd/httpd.conf

> Можно настроить [алиас в терминале](../Terminal/readme.md), для сокращенного использования команды "open -a /Applications/TextEdit.app".
>
> Следует учесть, что по умолчанию TextEdit заменяет программистские кавычки на "Smart кавычки", что можно изменить в настройках редактора.

**Меняем прослушиваемый порт с 8080 на 80**, для этого находим строку:

	Listen 8080

И заменяем на:

	Listen 80
	
> Теперь можно проверить работу Apache, открыв в браузере ссылку <http://localhost> (без добавления порта) - должна отобразится страница с текстом "It works!".

**Включим mod_rewrite**, для этого раскомментируем строку, удалив в начале символ комментария "#":

	LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so

**Укажем текущее имя пользователя**, вместо your_user, в строке:

	User your_user
	Group staff

> По умолчанию администратор macOS Mojave состоит в группе staff.
> 
> Для примера в приложении macOS Yosemite Server использовалось одинаковое имя пользователя и группы "_www", но из-за этого возникали сложности с правами доступа к файлам и каталогам, так как пользователь не был добавлен в системе.

**Укажем глобальное имя хоста**, для этого находим строку:

	ServerName www.example.com:8080
	
И заменяем на:

	ServerName localhost

> Если не указать имя хоста, будет использовать имя компьютера и об этом будет выводиться сообщение при перезагрузке Apache.

Для применения всех изменений в конфигурационных файлах, всегда следует перезапускать Apache:

	$ sudo apachectl -k restart

### Логирование

**Уровень логов записываемых в [Errorlog](https://ru.wikipedia.org/wiki/Error.log) оставим по умолчанию**:

	LogLevel warn

> Уровень "предупреждения".

**Добавим имя хоста в формат записи обращений к сайту в лог файл [CustomLog (access_log)](https://ru.wikipedia.org/wiki/Access.log),** для этого во-первых найдем строку:

	LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
	
И добавим сразу после нее новую строку:

	LogFormat "\"%v\" %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combinedvhost

> "%v" - название хоста (в двойных кавычках), которое мы добавили в начале формата записи в лог файл, только этим значением данный формат отличается от "combined".

Во-вторых найдем строку:

	LogFormat "%h %l %u %t \"%r\" %>s %b" common

И добавим сразу после нее новую строку:

	LogFormat "\"%v\" %h %l %u %t \"%r\" %>s %b" commonvhost

В-третьих найдем строку:

	LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio

И добавим сразу после нее новую строку:

	LogFormat "\"%v\" %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinediovhost

> Теперь мы сможем указывать свой формат записи в лог, например: 
> 
> CustomLog "/usr/local/var/log/httpd/access_log" combinedvhost
> 
> Таким образом мы можем не создавать отдельный файл логов, для каждого виртуального хоста, а записывать все в один лог.

В приложении macOS Console в списке логов нам доступен каталог в домашней директории пользователя ~/Library/Logs/Homebrew/httpd, поэтому создадим символическую ссылку на этот каталог в каталоге по умолчанию:

	$ mv /usr/local/var/log/httpd /usr/local/var/log/httpd.bak
	$ ln -s ~/Library/Logs/Homebrew/httpd /usr/local/var/log/httpd 

### Конфигурация виртуальных хостов

Далее создадим каталог для конфигурационных файлов виртуальных хостов:

	mkdir /usr/local/etc/httpd/extra/httpd-vhosts

Теперь подключим этот каталог в конфигурационный файл, для этого находим строку:

	#Include /usr/local/etc/httpd/extra/httpd-vhosts.conf

И заменяем на:

	Include /usr/local/etc/httpd/extra/httpd-vhosts/*.conf

> Таким образом мы будем добавлять все виртуальные хосты не в один файл httpd-vhosts.conf, а каждый виртуальный хост в свой файл с расширением ".conf".

**Создадим конфигурационный файл виртуальных хостов по умолчанию** (если для виртуального хоста не настроена отдельная конфигурация, он будет обрабатываться в конфигурации этого файла):

	$ nano /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_80_.conf

> Имена файлов могут быть любые, но для удобства использования каждый виртуальный хост для каждого порта записывается в отдельный файл. Сделано по примеру программы macOS Yosemite Server.
>
> 0000 - порядок сортировки добавляется для каждого файла одинаковый (на всякий случай), но в случае необходимости можно будет поменять на "0001" и т.д.
> 
> any - означает, что обрабатываются все запросы по 80 порту, что обозначено звездочкой в секции конфигурации виртуального хоста \<VirtualHost \*:80\>.
> 
> 80 - номер порта (или 443), который также указывается в секции виртуального хоста \<VirtualHost \*:80\>. 
> 
> Знак подчеркивание "_" - после которого, также будет указываться имя хоста (ServerName), для именованных хостов.
			
Добавим в конфигурационный файл необходимые записи, сохраним и выйдем:

	<VirtualHost *:80>
		ErrorLog "/usr/local/var/log/httpd/error_log"
		CustomLog "/usr/local/var/log/httpd/access_log" combinedvhost
	
		<Directory "/usr/local/var/www">
			Require local
		</Directory>
	</VirtualHost>

> \<VirtualHost *:80\>...\</VirtualHost\> - секция конфигурации виртуального хоста для всех запросов по 80 порту.
>  
> ErrorLog - лог ошибок. Если путь совпадает с глобальной конфигурацией в httpd.conf, то его можно не указывать.
> 
> CustomLog - лог записи обращений к сайту. Если путь совпадает с глобальной конфигурацией в httpd.conf, то его можно не указывать. В данном случае мы поменяли формат записи в лога файл на "combinedvhost".
> 
> \<Directory "/usr/local/var/www"\>...\</Directory\> - в общем файле конфигурации мы оставляем директорию по умолчанию, где будут размещаться публичные файлы сайта.
> 
> Require local - в данном случае мы указываем, что обращение к хостам по умолчанию, может быть только с локального компьютера (localhost, 127.0.0.1).

**Теперь создадим конфигурационный файл для виртуального хоста, например, site.localhost:**

	$ nano /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_80_site.localhost.conf

> В отличие от конфигурационного файла по умолчанию, после нижнего подчеркивания мы добавили имя хоста (домена). Таким образом файл конфигурации по умолчанию для 80 порта будет всегда первым, при условии использования в начале имени файла всех хостов записи "0000_any_80_" (для в случае необходимости, будем менять порядок сортировки на 0001 и т.д.).
	
Добавим в конфигурационный файл необходимые записи, сохраним и выйдем:

	<VirtualHost *:80>
		DocumentRoot "/Users/$USER/Sites/site.localhost"
    
		ServerName site.localhost
		ServerAlias site.localhost *.site.localhost
		
		ErrorLog "/usr/local/var/log/httpd/error_log"
		CustomLog "/usr/local/var/log/httpd/access_log" combinedvhost
	
		<Directory "/Users/$USER/Sites/site.localhost">
			Options -Indexes +FollowSymLinks
			AllowOverride all
			Require all granted
		</Directory>
	</VirtualHost>
	
> DocumentRoot - сначала мы добавим новый путь до корневого каталога c файлами сайта, где $USER - имя текущего пользователя, а site.localhost - это каталог сайта. Вообще правильно в каталоге сайта создавать отдельный публичный каталог и использовать его в качестве корневого, например, /Users/USER/Sites/site/public, где будет размещаться только статика (изображения, компилированные css и js файлы, публичные файлы) и один файл index.php, на который будут перенаправляться все запросы с помощью директив mode_rewrite настроенных в файле /Users/USER/Sites/site/public/.htaccess, а уже /Users/USER/Sites/site/public/index.php будет перенаправлять все запросы к не публичному каталогу сайта /Users/USER/Sites/site.
>  
> ErrorLog - лог ошибок можно также создавать отдельно для каждого виртуального хоста, например: "/usr/local/var/log/httpd/site.localhost.error_log".
> 
> CustomLog - лог записи обращений к сайту можно также создавать отдельно для каждого виртуального хоста, например: "/usr/local/var/log/httpd/site.localhost.access_log".
> 
> ServerName - укажем имя хоста, теперь все запросы к указанному домену (site.localhost) будут обрабатываться в этой конфигурации хоста.
> 
> ServerAlias - укажем алиасы, т.е. поддомены для текущего домена (www.site.localhost и все остальные).
> 
> \<Directory "/Users/USER/Sites/site"\>...\</Directory\> - в этой секции укажем настройки для корневого каталога:
> 
> - "Options -Indexes +FollowSymLinks" - запрещаем отображать список файлов в каталоге, если не найден индексный файл, и разрешаем использовать символические ссылки. Мы использовали знаки мину и плюс, таким образом директива Options будет также наследовать все глобальные опции указанные в httpd.conf (т.е. зная, что уже FollowSymLinks у нас разрешена, мы можем указать просто "Options -Indexes"), иначе мы бы указали "Options FollowSymLinks".
> - AllowOverride all - разрешаем использовать все директивы в файле .htaccess в каталогах сайта.
> - Require all granted - разрешаем доступ всем.

Теперь создадим каталог для хостов (сайтов) в домашнем каталоге текущего пользователя и затем создадим каталог для нового хоста (сайта/домена):

	$ mkdir ~/Sites
	$ mkdir ~/Sites/site.localhost

Перезапустим Apache:

	$ sudo apachectl -k restart

Теперь добавим тестовый индексный файл для каталога сайта site.localhost:

	$ echo '<h1>Home page of the site</h1>' > ~/Sites/site.localhost/index.html

И откроем в браузере <http://site.localhost>, мы должны увидеть надпись "Home page of the site".

Домен первого уровня localhost и его поддомены \*.localhost зарезервированы в [IETF](https://en.wikipedia.org/wiki/.test) и не требуют дополнительной настройки в /ets/hosts в macOS Mojave. Но поддомены \*.localhost при этом работают в браузере Safari только при условии подключения к интернету (точно при подключении по Wi-Fi), без подключения к интернету поддомены не работают в Safari, но работают в браузере [Google Сhrome](https://www.google.com/chrome/).

> Не стоит использовать домены *.dev, потому что корневой домен dev куплен Google и не работает по https в Chrome для локальных доменов. 

В случае необходимости можно прописать нужные домены в /ets/hosts, например, добавив перенаправление site.localhost на локальный IP 127.0.0.1:

	$ sudo nano /etc/hosts
		# добавляем перенаправление для каждого поддомена с новой строки
	127.0.0.1       site.localhost
	127.0.0.1       www.site.localhost

Теперь все хосты *.localhost для которых не добавлены конфигурационные файлы в /usr/local/etc/httpd/extra/httpd-vhosts, будут обрабатываться конфигурацией в файле 0000_any_80_.conf, т.е. будут доступны только локально и будут ссылаться на корневой каталог /usr/local/var/www. Таким образом можно будет открывать, например, <http://localhost> для проверки работы Apache, в случае если настроенный хост не открывается.

### Локальное использование домена test

Также для локальной разработки зарезервирован домен test, и чтобы не прописывать каждый поддомен в /ets/hosts, можно использовать [Dnsmasq](https://ru.wikipedia.org/wiki/Dnsmasq), использовав для установки HomeBrew:

	$ brew install dnsmasq
	
Теперь необходимо настроить хосты *.test, для этого добавим запись в конфигурацию dnsmasq:

	$ echo 'address=/.test/127.0.0.1' > /usr/local/etc/dnsmasq.conf
	
Теперь запустим демон dnsmasq:

	$ sudo brew services start dnsmasq
		# перезагрузка
		# $ sudo brew services restart dnsmasq
		# остановка
		# $ sudo brew services stop dnsmasq

И выполним последнюю настройку dnsmasq:

	$ sudo mkdir -v /etc/resolver
	$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/test'

Но также как и с доменами *.localhost, в Safari не будет работать без подключения к интернету (решения пока не нашел).

### Конфигурация SSL/TLS для работы виртуальных хостов по протоколу HTTPS

[HTTPS](https://ru.wikipedia.org/wiki/HTTPS) (HyperText Transfer Protocol Secure) — расширение протокола HTTP для поддержки шифрования в целях повышения безопасности.

**Включим SSL/TLS**, для этого откроем и настроим httpd.conf:

	$ code /usr/local/etc/httpd/httpd.conf
		# или
		# nano /usr/local/etc/httpd/httpd.conf

> code - если команда добавлена [в алиас для терминала](#Конфигурация).

Затем раскомментируем 3 строки, удалив в начале символ комментария "#":

	LoadModule socache_shmcb_module lib/httpd/modules/mod_socache_shmcb.so
	
	LoadModule ssl_module lib/httpd/modules/mod_ssl.so
	
	Include /usr/local/etc/httpd/extra/httpd-ssl.conf
	
Теперь открываем и настраиваем конфигурационный файл httpd-ssl.conf:

	$ code /usr/local/etc/httpd/extra/httpd-ssl.conf
	
**Меняем прослушиваемый порт с 8443 на 443**, для этого находим строку:

	Listen 8443

И заменяем на:

	Listen 443
	
Затем поменяем порт в секции VirtualHost, для этого находим строку:

	<VirtualHost _default_:8443>

И заменяем на:

	<VirtualHost _default_:443>
	
И также **закомментируем ненужные нам сейчас директивы в файле httpd-ssl.conf**, добавив в начале символ комментария "#":

	#DocumentRoot "/usr/local/var/www"
	#ServerName www.example.com:8443

**Теперь сохраним изменения и создадим конфигурационный файл виртуальных хостов по умолчанию на 443 порту** (если для виртуального хоста не настроена отдельная конфигурация, он будет обрабатываться в конфигурации этого файла):

	$ nano /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_443_.conf
	
Добавим в конфигурационный файл необходимые записи, сохраним и выйдем:

	<VirtualHost *:443>
		ErrorLog "/usr/local/var/log/httpd/error_log"
		CustomLog "/usr/local/var/log/httpd/access_log" combinedvhost
	
		SSLEngine on
		SSLCertificateFile "/usr/local/etc/httpd/certificates/*.localhost.crt"
		SSLCertificateKeyFile "/usr/local/etc/httpd/certificates/private/*.localhost.key"
	
		<Directory "/usr/local/var/www">
			Require local
		</Directory>
	</VirtualHost>

> SSLEngine on - включаем SSL/TLS для виртуального хоста.
> 
> SSLCertificateFile - директива для указания файла с данными сертификата в формате PEM.
> 
> SSLCertificateKeyFile - директива для указания файла закрытого (приватного) ключа сертификата, закодированного с помощью PEM.
	
**Теперь создадим конфигурационный файл для виртуального хоста site.localhost на 443 порту**:

	$ nano /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_443_site.localhost.conf
	
Добавим в конфигурационный файл необходимые записи, сохраним и выйдем:

	<VirtualHost *:443>
		DocumentRoot "/Users/$USER/Sites/site.localhost"
    
		ServerName site.localhost
		ServerAlias site.localhost *.site.localhost
    
		ErrorLog "/usr/local/var/log/httpd/error_log"
		CustomLog "/usr/local/var/log/httpd/access_log" combinedvhost
    
		SSLEngine on
		SSLCertificateFile "/usr/local/etc/httpd/certificates/*.localhost.crt"
		SSLCertificateKeyFile "/usr/local/etc/httpd/certificates/private/*.localhost.key"
		
		<Directory "/Users/$USER/Sites/site.localhost">
			Options -Indexes +FollowSymLinks
			AllowOverride all
			Require all granted
		</Directory>	
	</VirtualHost>
	
При создании еще одного виртуального хоста, мы можем копировать конфигурацию для уже созданного хоста и заменить в ней название хоста, например:

	$ cp /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_443_site.localhost.conf /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_443_site2.localhost.conf
	$ perl -i -pe 's/site.localhost/site2.localhost/;' /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_443_site2.localhost.conf
		# И например, можем сразу поменять домен localhost на test, тем самым изменив и путь до сертификата
	$ perl -i -pe 's/localhost/test/;' /usr/local/etc/httpd/extra/httpd-vhosts/0000_any_443_site2.localhost.conf

**Создадим каталог для данных сертификатов и закрытых ключей:**

	$ mkdir /usr/local/etc/httpd/certificates
	$ mkdir /usr/local/etc/httpd/certificates/private
	
**Создадим каталог для сертификатов, запросов сертификатов и приватных ключей**:

	$ mkdir /usr/local/etc/httpd/certificates
	$ mkdir /usr/local/etc/httpd/certificates/csr
	$ mkdir /usr/local/etc/httpd/certificates/private

Изменим права доступа каталога .../certificates/private, чтобы доступ к нему имел только текущий пользователь компьютера:

	$ chmod 700 /usr/local/etc/httpd/certificates/private
	
**Теперь необходимо [создать самоподписанный сертификат для локального использования](../SSL/readme.md#генерация-самоподписанного-ssltls-сертификата-для-локального-использования).**

Затем создадим символические ссылки для файлов сертификата и закрытого ключа, указанных в соответствующих директивах SSLCertificateFile и SSLCertificateKeyFile в /usr/local/etc/httpd/extra/httpd-ssl.conf:

	$ ln -s /usr/local/etc/httpd/certificates/*.localhost.crt /usr/local/etc/httpd/server.crt
	$ ln -s /usr/local/etc/httpd/certificates/private/*.localhost.key /usr/local/etc/httpd/server.key
		
Запустим тест:

	$ sudo apachectl configtest
	Syntax OK

Перезапустим Apache:

	$ sudo apachectl -k restart

И откроем в браузере <https://site.localhost>, мы должны увидеть надпись "Home page of the site".