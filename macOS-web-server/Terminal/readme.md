# Заметки по использованию macOS Terminal

[MacOS Terminal](https://support.apple.com/ru-ru/guide/terminal/welcome/mac) - встроенное приложение в macOS для работы с командной строкой.

<!--ts-->
<!--te-->

## Ссылки

1. [Описание команд терминала в macOS от A до Z](http://osxh.ru/content/spisok-terminalnyh-komand-os-x).

## Настройки

### Добавление алиасов для команд терминала
	
Например, можно установить собственную команду, которая будет открывать указанные файл в редакторе TextEdit, для этого необходимо отредактировать в домашнем каталоге файл ~/.bash_profile и добавить алиас с именем "code" для команды открытия редактора:

	$ nano ~/.bash_profile
		# вставляем строку
	alias code='open -a /Applications/TextEdit.app'
		# сохраняем, выходим и выполняем	
	$ source ~/.bash_profile
	
Теперь используя команду code, можно открывать текстовые файлы из терминала в редакторе TextEdit, например:

	$ code ~/.bash_profile

Также можно добавлять алиас для уже существующей команды, изменяя ее поведение по умолчанию, например, добавив в ~/.bash_profile алиас для команды "ls":

	alias ls='ls -GFh'

Можно добавлять алиас для нескольких команд сразу, разделяя их точкой с запятой, например, для команд отображения/скрытия скрытых файлов:

	alias showfiles='defaults write com.apple.finder AppleShowAllFiles YES; killall Finder /System/Library/CoreServices/Finder.app'
	alias hidefiles='defaults write com.apple.finder AppleShowAllFiles NO; killall Finder /System/Library/CoreServices/Finder.app'