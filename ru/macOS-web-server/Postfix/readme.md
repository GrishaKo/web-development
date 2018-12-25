# Заметки по использованию Postfix

[Postfix](https://en.wikipedia.org/wiki/Postfix) (ENG/[RUS](https://ru.wikipedia.org/wiki/Postfix)) - агент передачи почты.

<!--ts-->
<!--te-->

<a id="links"></a>
## Ссылки

1. [Postfix relay yandex](https://help.ubuntu.ru/wiki/postfix_relay_yandex) (RUS).

<a id="installation"></a>
## Установка на macOS

Postfix уже установлен на macOS:

	$ sudo postfix status
	postfix/postfix-script: the Postfix mail system is not running
	$ which postfix
	/usr/sbin/postfix
	$ postconf -d mail_version
	mail_version = 3.2.2
	
Документация:

	$ find /usr/share -name postfix
	/usr/share/doc/postfix

<a id="configuration"></a>	
## Конфигурация отправки почты через сервер почтового сервиса

### Использование почтового сервиса

Для организации электронной почты для домена, проще использовать почтовый сервис, например: [Yandex](https://yandex.ru/support/pdd/), [Google](https://support.google.com/a#topic=2426592) или [Zoho](https://www.zoho.com/mail/help/).

Для того, чтобы использовать почтовый сервис для управления аккаунтами электронной почты для домена, настроить почтовый клиент (macOS Mail и т.д.) или использовать веб-интерфейс почтового сервиса, достаточно настроить только MX-запись для домена, которая указывает на сервер, принимающий почту для домена, например:

	@ MX mx.yandex.net. 10
	
Дополнительно можно настроить CNAME-запись для домена, которая позволяет назначить поддомен, для перенаправления на страницу веб-интерфейса почтового сервиса, например:

	mail CNAME domain.mail.yandex.net.
	
> В таком случае, например, будет перенаправление с `mail.domain.tld` на `mail.yandex.ru/?pdd_domain=domain.tld`.

Но для того чтобы использовать все возможности почтового сервиса и правильно настроить пересылку почты со своего сервера через SMTP сервер почтового сервиса (например, при отправке почты с сайта средствами PHP), необходимо делегировать домен на серверы почтового сервиса - это означает, что для домена необходимо указать DNS-серверы почтового сервиса (в настройках домена на сайте регистратора), например:
	
	@ NS dns1.yandex.net.
	@ NS dns1.yandex.net.

После того как изменения в DNS вступят в силу (этот процесс может длиться до 72 часов), необходимо настроить необходимые DNS-записи для правильной работы почты: MX, CNAME, SPF, DKIM-подпись, DMARK.

[SPF-запись](https://en.wikipedia.org/wiki/Sender_Policy_Framework) (ENG/[RUS](https://ru.wikipedia.org/wiki/Sender_Policy_Framework)) помогает снизить риск того, что отправленное с домена письмо попадет в спам, например:

	@ TXT v=spf1 ip4:127.0.0.1 include:_spf.yandex.net ~all

> `127.0.0.1` - IP адреса сервера, который указывается в соответствующей DNS-записи для домена `@ A
127.0.0.1`.

[DKIM-подпись](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) (ENG/[RUS](https://ru.wikipedia.org/wiki/DomainKeys_Identified_Mail)) позволяет получателю письма удостовериться в том, что оно действительно пришло от предполагаемого отправителя, например:

	mail._domainkey TXT v=DKIM1; k=rsa; t=s; p=MIG...QAB
	_domainkey TXT o=-
	
> `MIG...QAB` - публичный ключ DKIM, который предоставляется почтовым сервисом.
> `_domainkey` - дополнительная запись с параметром `o=-`, указывает политику исходящей подписи, где `~` означает, что все письма подписываются (по умолчанию), или `~` - некоторые письма подписываются.  

[DMARK](https://en.wikipedia.org/wiki/DMARC) (ENG/[RUS](https://ru.wikipedia.org/wiki/DMARC)) определяет действия с подозрительными сообщениями, поступившими в домен. DMARK-запись настраивается после SPF и DKIM, например:

	_dmarc TXT v=DMARC1; p=none

Проверить, что изменения DNS-записей SPF, DKIM и DMARK вступили в силу (этот процесс может длиться до 72 часов), а также исключить ошибки, можно с помощью специальных сервисов:

- [Протестируйте Ваши письма на СПАМ](https://www.mail-tester.com/) (ENG/[RUS](https://www.mail-tester.com/?lang=ru)).
- [Authentication Checker](https://port25.com/authentication-checker/) (ENG).

### Конфигурация Postfix

Добавляем параметры конфигурации в конец конфигурационного файла Postfix, например, для отправки писем через SMTP сервер почтового сервиса [Yandex](https://yandex.ru/support/pdd/):

	$ sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.bak
	$ sudo nano /etc/postfix/main.cf
	relayhost =
	smtp_sasl_auth_enable = yes
	smtp_sasl_password_maps = hash:/private/etc/postfix/private/sasl_passwd
	smtp_sasl_security_options = noanonymous
	smtp_sasl_type = cyrus
	smtp_sasl_mechanism_filter = login
	smtp_sender_dependent_authentication = yes
	sender_dependent_relayhost_maps = hash:/etc/postfix/private/sender_relay
	sender_canonical_maps = hash:/etc/postfix/private/canonical
	smtp_use_tls = yes
	
Создаем каталог `/etc/postfix/private`:

	$ sudo mkdir /etc/postfix/private
	
Создаем файлы конфигурации для Postfix lookup tables:
	
	$ sudo nano /etc/postfix/private/sasl_passwd
	no-reply@domain.tld login:password
	
	$ sudo nano /etc/postfix/private/sender_relay
	no-reply@domain.tld [smtp.yandex.ru]
	
	$ sudo nano /etc/postfix/private/canonical
	@hostname no-reply@domain.tld
	
Создаем Postfix lookup tables:
	
	$ sudo postmap /etc/postfix/private/*

Посмотреть конфигурацию можно командой:

	$ postconf -n

Изменим права доступа к каталогу `/etc/postfix/private`, чтобы файлы в каталоге могли посмотреть только пользователи с правами суперпользователя:

	$ chmod 750 /etc/postfix/private


Проверяем:

echo "Message" | mail -s "Subject" ebanis@yandex.ru

PHP

	sendmail_path = /usr/sbin/sendmail -t -i -fno-	reply@pozvonochnik.info
	# или
	mail.force_extra_parameters = -fno-reply@pozvonochnik.info


Узнать и изменить имя хоста:

	$ hostname
	LocalHostName.local
	$ scutil --set HostName domain.tld

Логирование:

	log show --last 2h  --predicate  'process == "smtp"' --info --debug
	
	log show --start "2018-12-23"  --end "2018-12-25" --predicate  'process == "smtp"' --info --debug
	
	log stream --predicate  '(process == "smtpd") || (process == "smtp")' -info


	

