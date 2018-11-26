# Заметки по использованию Terminal

[MacOS Terminal](https://support.apple.com/ru-ru/guide/terminal/welcome/mac) - встроенное приложение в macOS для работы с командной строкой.

<!--ts-->
<!--te-->

## Ссылки

1. [Описание команд терминала в macOS от A до Z](http://osxh.ru/content/spisok-terminalnyh-komand-os-x).

## Настройки

### Добавление alias для команд терминала
	
Например, можно установить команду, которая будет открывать указанный после нее файл в редакторе кода [TextWrangler](https://itunes.apple.com/ru/app/textwrangler/id404010395?mt=12) (нет автосохранения), для этого необходимо отредактировать в домашнем каталоге файл ~/.bash_profile и добавить alias "code" для команды открытия редактора:

	$ cd ~/
	$ nano ~/.bash_profile
		# вставляем строку
	alias code='open -a /Applications/TextWrangler.app'
		# сохраняем, выходим и выполняем	
	$ source ~/.bash_profile
	
Теперь используя команду code, можно открывать текстовые файлы из терминала:

	$ code ~/.bash_profile

Также можно назначать alias для уже существующей команды, изменяя ее поведение по умолчанию, например, добавив в ~/.bash_profile alias для команды "ls":

	alias ls='ls -GFh'

Можно добавлять alias для нескольких команд сразу, разделяя их точкой с запятой, например, команда отображения/скрытия скрытых файлов:

	alias showfiles='defaults write com.apple.finder AppleShowAllFiles YES; killall Finder /System/Library/CoreServices/Finder.app'
	alias hidefiles='defaults write com.apple.finder AppleShowAllFiles NO; killall Finder /System/Library/CoreServices/Finder.app'