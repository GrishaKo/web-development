# Заметки по использованию SSL/TLS-сертификатов

[TLS](https://ru.wikipedia.org/wiki/TLS) (Transport Layer Security — протокол защиты транспортного уровня) - протокол обеспечивающий защищённый обмен данных, как и его предшественник [SSL](https://ru.wikipedia.org/wiki/SSL)(Secure Sockets Layer - уровень защищённых cокетов).

<!--ts-->
  * [Ссылки](#ссылки)
  * [Подготовка веб-сервера для обработки HTTPS-соединений](#подготовка-веб-сервера-для-обработки-https-соединений)
  * [Генерация запроса на получение SSL/TLS-сертификата (CSR)](#генерация-запроса-на-получение-ssltls-сертификата-csr)
  * [Генерация самоподписанного SSL/TLS-сертификата для локального использования](#генерация-самоподписанного-ssltls-сертификата-для-локального-использования)
  * [Установка SSL-сертификата](#установка-ssl-сертификата)

<!-- Added by: grisha_k, at:  -->

<!--te-->

## Ссылки

1. [CSR Generation: Using OpenSSL (Apache w/mod_ssl, NGINX, OS X)](https://support.comodo.com/index.php?/comodo/Knowledgebase/Article/View/1/66/).
2. [HowTo: Create CSR using OpenSSL Without Prompt (Non-Interactive)](https://www.shellhacks.com/create-csr-openssl-without-prompt-non-interactive/) / [Как: Создать CSR в OpenSSL без Вопросов (Неинтерактивно)](https://www.shellhacks.com/ru/create-csr-openssl-without-prompt-non-interactive/).
3. [Certificate Installation: Apache & mod_ssl](https://support.comodo.com/index.php?/comodo/Knowledgebase/Article/View/637/66/).
4. [Mozilla's Server Side TLS Guidelines](https://wiki.mozilla.org/Security/Server_Side_TLS).

## Подготовка веб-сервера для обработки HTTPS-соединений

Сначала устанавливаем и настраиваем [Apache](../Apache/readme.md), после чего создадим каталог для сертификатов, запросов сертификатов и приватных ключей:

	$ mkdir /usr/local/etc/httpd/certificates
	$ mkdir /usr/local/etc/httpd/certificates/csr
	$ mkdir /usr/local/etc/httpd/certificates/private

Изменим права доступа для каталога приватных ключей, чтобы доступ к нему имел только текущий пользователь компьютера:

	$ chmod 700 /usr/local/etc/httpd/certificates/private
	
## Генерация запроса на получение SSL/TLS-сертификата (CSR)

Создадим каталог для сертификатов, запросов сертификатов и приватных ключей:

	$ mkdir /usr/local/etc/httpd/certificates
	$ mkdir /usr/local/etc/httpd/certificates/csr
	$ mkdir /usr/local/etc/httpd/certificates/private

Изменим права доступа для каталога приватных ключей, чтобы доступ к нему имел только текущий пользователь компьютера:

	$ chmod 700 /usr/local/etc/httpd/certificates/private

Создадим приватный ключ и [CSR](https://en.wikipedia.org/wiki/Certificate_signing_request), с помощью [OpenSSL](https://ru.wikipedia.org/wiki/OpenSSL) и заполним в интерактивном режиме требуемые поля:
	
	$ openssl req -nodes -newkey rsa:2048 -keyout /usr/local/etc/httpd/certificates/private/example.com.key -out usr/local/etc/httpd/certificates/csr/example.com.csr
	Country Name (2 letter code) []: RU
	State or Province Name (full name) []:Moscow
	Locality Name (eg, city) []:Moscow
	Organization Name (eg, company) []:example.com
	Organizational Unit Name (eg, section) []:IT
	Common Name (eg, fully qualified host name) []:example.com
	Email Address []:admin@example.com
	A challenge password []:

> "Organizational Unit Name" - всегда можно указывать IT или IT department, т.е. для ИТ отдела организации.
> 
> "Common Name" - для [Wildcard-сертификата](https://ru.wikipedia.org/wiki/Wildcard-сертификат) необходимо указывать "*.example.com".
> 
> "Email Address" - необязательное поле, но рекомендуется указать тот же e-mail, что будет использован для подтверждения владения доменом (часто это admin@).
> 
> "A challenge password" - оставим пустым, нажав Enter.

Или без использования интерактивного режима можно сразу указать все необходимые данные:
	
	$ openssl req -nodes -newkey rsa:2048 -keyout /usr/local/etc/httpd/certificates/private/example.com.key -out /usr/local/etc/httpd/certificates/csr/example.com.csr -subj "/C=RU/ST=Moscow/L=Moscow/O=example.com/OU=IT/CN=example.com/emailAddress=admin@example.com" 
	
> [openssl req](https://www.openssl.org/docs/man1.0.2/apps/openssl-req.html)	- утилита запроса сертификата и генерации сертификата.
> 
> -nodes - секретный ключ не шифруется.

> -newkey	- создание нового запроса сертификата и нового приватного ключа.
> 
> rsa:2048 - генерация 2048-битного RSA ключа.
> 
> -keyout	- имя файл для нового приватного ключа.
> 
> -out	- имя файл для нового запроса сертификата.
> 
> -subj - данные запроса сертификата, в формате "/ключ=значение/ключ2=...", символы могут экранироваться \ (обратным слэшем), пробелы не пропускаются.
	
Или можно использовать уже существующий приватный ключ:

	$ openssl req -new -key /usr/local/etc/httpd/certificates/private/example.com.key -out /usr/local/etc/httpd/certificates/csr/example.com.csr
	
> -new	- создание нового запроса сертификата.
> 
> -key	- имя файл существующего приватного ключа.
	
Проверка CSR файла и просмотр декодированных данных:
	
	$ openssl req -noout -verify -text -in /usr/local/etc/httpd/certificates/csr/example.com.csr
		# в начале должно быть отображено
	verify OK
		# затем - данные запроса

> -noout - не выводить содержимое файла CSR.
> 
> -verify - проверка CSR.
> 
> -text - вывести декодированные данные CSR.

## Генерация самоподписанного SSL/TLS-сертификата для локального использования

Создадим приватный ключ и самоподписанный сертификат:

	$ openssl req -x509 -days 365 -nodes -newkey rsa:2048 -keyout /usr/local/etc/httpd/certificates/private/*.localhost2.key -out /usr/local/etc/httpd/certificates/*.localhost2.crt -subj "/C=RU/ST=Moscow/L=Moscow/O=localhost2/OU=IT/CN=*.localhost2"

> CN (Common Name) - укажем имя хоста "\*.localhost" для использования со всеми поддоменами localhost.

Чтобы не подтверждать сертификат в браузере Safari при каждом входе на новый поддомен localhost:

1. Откроем сертификат /usr/local/etc/httpd/certificates/*.localhost.crt в приложение "Связка ключей" (Keychain Access).
2. Затем перейдем в секцию "Доверять" (Trust).
3. И выберем "Всегда доверять" (Always Trust) в поле "Параметры использования сертификата" (When using this certificate).

## Установка SSL-сертификата

Добавляем внутрь секции \<VirtualHost\> необходимые записи в конфигурации виртуального хоста (или в глобальном конфигурационном файле /usr/local/etc/httpd/extra/httpd-ssl.conf), например:

	<VirtualHost *:443>
		...
	    
	    SSLEngine on    
	    SSLCertificateKeyFile /usr/local/etc/httpd/certificates/private/example.com.key
	    SSLCertificateFile /usr/local/etc/httpd/certificates/example.com.crt
	    SSLCertificateChainFile /usr/local/etc/httpd/certificates/example.com.ca-bundle
	    
	    ...
	</VirtualHost>
	
> SSLEngine on - включаем SSL/TLS для виртуального хоста.
> 
> SSLCertificateKeyFile - директива для указания файла закрытого (приватного) ключа сертификата, закодированного с помощью PEM.
> 
> SSLCertificateFile - директива для указания файла с данными сертификата в формате PEM.
> 
> SSLCertificateChainFile - директива для указания файла содержащего цепочку сертификатов (корневой + промежуточный сертификаты), не нужно указывать для самоподписанных сертификатов.