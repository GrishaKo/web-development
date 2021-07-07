# Заметки по использованию SSH

[SSH](https://en.wikipedia.org/wiki/SSH) (ENG/[RUS](https://ru.wikipedia.org/wiki/SSH)) - сетевой протокол для удаленного управления компьютером (сервером) с помощью командной строки (macOS/Linux/Windows).

- [Ссылки](#links)
- [Подключение к удаленному компьютеру по SSH на macOS и IOS](#terminal)
- [Удаленный вход на macOS по SSH](#ssh)
- [Настройка SSH аутентификации по ключам для подключения к удаленному компьютеру](#ssh-keys-authentication)
 - [Ссылки](#ssh-keys-authentication-links)
 - [Создание SSH-ключей](#ssh-keygen)
 - [Передача публичного SSH-ключа на удаленный компьютер](#ssh-key-transfer)
 - [Конфигурация SSH-сервера на удаленном комьюетере](#sshd-config)
 - [Настройка алиаса для команды подключения к удаленному компьютеру по SSH](#ssh-alias)
- [Подключение к MySQL на удаленном компьютере через SSH туннель](#ssh-mysql)
- [Обмен файлами с удаленным компьютером Mac через Finder с помощью SFTP](#sftp)

<a name="links"></a>
## Ссылки

1. [SSH KEY](https://www.ssh.com/ssh/key/) (ENG).
2. [Настройка доступа по ssh (обновленная версия)](https://www.youtube.com/watch?v=5bF-DTxvvmQ) (RUS).
3. [SSH ключи для работы с удаленными Git-репозиториями](https://www.youtube.com/watch?v=KqzVaUTCPbQ) (RUS).
4. [How to Access Your Mac over SSH with Remote Login](https://www.booleanworld.com/access-mac-ssh-remote-login/) (ENG).

<a name="terminal"></a>
## Подключение к удаленному компьютеру по SSH на macOS и IOS

[../MacOS Terminal](../Terminal/readme.md) - встроенное приложение в macOS для работы с командной строкой.

[Termius - SSH client](https://www.termius.com) - кроссплатформенное приложение для мобильных, планшетов и компьютеров.

<a name="ssh"></a>
## Удаленный вход на macOS по SSH 

1. Открываем "Системные настройки" (System Preferences).
2. Переходим в "Общий доступ" (Sharing).
3. В списке служб слева выбираем "Удаленный вход" (Remote Login).
4. Справа под индикатором "Удаленный вход: вкл" (Remote Login: on) копируем ссылку для удаленного входа, вида "ssh User@127.0.0.1", где User - это логин текущего пользователя, 127.0.0.1 - ваш текущий IP адрес.

> Если компьютер подключен к сети напрямую через сетевой кабель, то должен отображаться внешний IP адрес предоставленный провайдером, который можно использовать для удаленного подключения по SSH по сети интернет.
> 
> Если же компьютер подключен к Wi-Fi сети, то будет отображаться внутренний (локальный) IP адрес и доступ будет только по локальной сети, поэтому будет необходимо настроить на роутере перенаправление [22 порта](https://support.apple.com/en-us/HT202944) (ENG/[RUS](https://support.apple.com/ru-ru/HT202944)) на компьютер по его локальному IP адресу.

<a name="ssh-keys-authentication"></a>
## Настройка SSH аутентификации по ключам для подключения к удаленному компьютеру

<a name="ssh-keys-authentication-links"></a>
### Ссылки

1. [Настройка SSH сервера во FreeBSD, конфигурационный файл sshd_config](http://vds-admin.ru/ssh/nastroika-servera-ssh-vo-freebsd-fail-sshdconfig) (RUS).
2. [Command Line SSH restart Mac OSX Mountain Lion](http://superuser.com/questions/478035/command-line-ssh-restart-mac-osx-mountain-lion) (ENG).
3. [Create an SSH Tunnel for MySQL Remote Access](https://www.linode.com/docs/databases/mysql/securely-administer-mysql-with-an-ssh-tunnel) (ENG).

<a name="ssh-keygen"></a>
### Создание SSH-ключей

Создадим публичный и приватный ключи на локальном компьютере, с помощью команды терминала:

	$ ssh-keygen

Далее нам предлагается ввести путь и имя ключа вместо значения по умолчанию `/Users/USER/.ssh/id_rsa` (где `USER` - это имя текущего пользователя), копируем предложенный путь и добавляем к имени ключа имя пользователя:

	: /Users/USER/.ssh/id_rsa_USER

Далее пропускаем ввод парольной фразы, просто нажимаем 2 раза "Enter".

Проверяем что ключи созданы:

	$ ls ~/.ssh -l | grep 'id_rsa_USER'
	id_rsa_USER
	id_rsa_USER.pub

> `~/.ssh` - в домашнем каталоге будет автоматически создан каталог `/.ssh`, в котором будут созданы SSH-ключи.
>
> `id_rsa_USER` - это приватный ключ (всегда хранится на локальном компьютере).
> 
> `id_rsa_user.pub` - это публичный ключ.

Изменим права доступа каталога `~/.ssh`, чтобы доступ к нему имел только текущий пользователь компьютера:

	$ chmod 700 ~/.ssh

<a name="ssh-key-transfer"></a>
### Передача публичного SSH-ключа на удаленный компьютер

Входим на удаленный компьютер:

	$ ssh ROOT@SERVER
	
> `ROOT@SERVER` - имя пользователя и IP/домен удаленного компьютера.	

Создаем каталог `~/.ssh` и файл `~/.ssh/authorized_keys`, если их нет, меняем права доступа и выходим:

	$ ls -a | grep '.ssh'
	$ mkdir ~/.ssh
	$ > authorized_keys
		# или 
		# touch authorized_keys
	$ chmod 700 ~/.ssh
	$ chmod 644 ~/.ssh/authorized_keys
	$ exit

После того как вернулись на локальный компьютер, копируем содержимое публичного ключа на удаленный компьютер в файл `~/.ssh/authorized_keys` (файл для публичных ключей):

	$ scp ~/.ssh/id_rsa_USER.pub ROOT@SERVER:~/.ssh/authorized_keys
	
Затем проверяем вход на удаленный комьютер по публичному ключу, без пароля:
	
	ssh ROOT@SERVER -i .ssh/id_rsa_USER

<a name="sshd-config"></a>	
### Конфигурация SSH-сервера на удаленном комьюетере	

Настройка на удаленной машине конфигурационного файла `/etc/ssh/sshd_config`:

Предварительно создадим копию файла конфигурации:

	$ cd /etc
	$ ls | grep 'ssh'
		ssh
	$ cd ssh
	$ cp sshd_config sshd_config.default

> `ls | grep 'ssh'` - команда поиска файлов и каталогов, содержащих фразу "ssh", потому что на macOS Yosemite файл конфигурации располагался сразу в каталоге `/etc`, в отличие от macOS Mojave и Linux.

Далее открываем файл в для редактирования под root пользователем:

	$ sudo nano sshd_config
		# или
		# sudo vim sshd_config

Закрываем возможность залогиниться как root удаленно, для этого меняем строчку:
	
	#PermitRootLogin yes

На:

	PermitRootLogin no

Разрешаем aутентификацию по публичному ключу - меняем строчку:

	#PubkeyAuthentication yes

На:

	PubkeyAuthentication yes

Запрещаем aутентификацию по паролю и пустым паролям - меняем строчки:

	#PasswordAuthentication no
	#PermitEmptyPasswords no

На:

	PasswordAuthentication no
	PermitEmptyPasswords no

Запрещаем без парольную аутентификация "запрос-ответ"  - меняем строчку:

	#ChallengeResponseAuthentication yes

На:

	ChallengeResponseAuthentication no

Сохраняем изменения и перезапускаем демона SSH.

Перезапуск демона SSH на macOS, путем остановки и повторного запуска:

	sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
	sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist

<a name="ssh-alias"></a>
### Настройка алиаса для команды подключения к удаленному компьютеру по SSH

Вход по SSH на удаленную машину с помощью короткого имени (алиаса, например: `ssh server`) вместо полного адреса, ввода пароля и указания публичного ключа: 
`ssh admin@8.8.8.8`

Заходим через терминал в конфигурационный файл на локальной машине через VIM редактор :

	vim .ssh/config
	Host server
		HostName 8.8.8.8
		User admin
		IdentityFile ~/.ssh/id_rsa_user
		IdentitiesOnly yes

`IdentityFile` - указываем путь публичного ключа.

Можно также поменять порт SSH с 22 на другой, но в видео инструкции не было примера для Mac, и при смене может быть много сложностей, например, на Fedore очень много заморочек.

Очищаем историю команд терминала:

	history -c

Проверяем:

	history

Очищаем экран:

	clear

<a name="ssh-mysql"></a>
## Подключение к MySQL на удаленном компьютере через SSH туннель

	ssh -L 127.0.0.1:33306:127.0.0.1:3307 ssh_server -N

> `127.0.0.1:33306` - IP и порт (любой незанятый, поэтому к обычному порту добавил в начале 3) на локальном компьютере.
> 
> `127.0.0.1:3307` - IP и порт на удаленном компьютере.
> 
> `ssh_server` - aлиас для подключения к серверу по SSH или полный путь `ROOT@SERVER`.

<a name="sftp"></a>
## Обмен файлами с удаленным компьютером Mac через Finder с помощью SFTP

Сначала устанавливаем [../HomeBrew](../HomeBrew/readme.md), затем с его помощью устанавливаем [osxfuse](https://osxfuse.github.io/) и [sshfs](https://github.com/libfuse/sshfs):

	$ brew cask install osxfuse
	$ brew install sshfs

Затем подключаемся к серверу через SFTP:

	$ sshfs -o reconnect username@hostname:/remote/directory/path /local/mount/point -ovolname=NAME

> `-o reconnect` - опция, для активации повторной установке накопителя после приостановки работы  компьютера и т. д.
> `/remote/directory/path` - путь до нужного каталога на удаленном компьютере.
> `/local/mount/point` - путь до директории на локальном компьютере, где будет монтироваться удаленный диск (например: `~/Fuse/ServerName`).
> `-ovolname=` - опция для указания имени удаленного диска (например: ServerName).