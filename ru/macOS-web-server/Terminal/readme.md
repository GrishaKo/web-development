# Заметки по использованию macOS Terminal

[MacOS Terminal](https://support.apple.com/guide/terminal/welcome/mac) (ENG/[RUS](https://support.apple.com/ru-ru/guide/terminal/welcome/mac)) - встроенное приложение в macOS для работы с командной строкой.

<!--ts-->
   * [Заметки по использованию macOS Terminal](#заметки-по-использованию-macos-terminal)
      * [Ссылки](#ссылки)
      * [Настройки](#настройки)
         * [Добавление алиасов для команд терминала](#добавление-алиасов-для-команд-терминала)
<!--te-->

<a id="links"></a>
## Ссылки

1. [An A-Z Index of the Apple macOS command line](https://ss64.com/osx/) (ENG).
2. [Описание команд терминала в macOS от A до Z](http://osxh.ru/content/spisok-terminalnyh-komand-os-x) (RUS).

<a id="configuration"></a>
## Настройки

<a id="alias"></a>
### Добавление алиасов для команд терминала
	
Например, можно установить собственную команду, которая будет открывать указанные файл в редакторе TextEdit, для этого необходимо отредактировать в домашнем каталоге файл `~/.bash_profile` и добавить алиас с именем `code` для команды открытия редактора:

	$ nano ~/.bash_profile
		# вставляем строку
	alias code='open -a /Applications/TextEdit.app'
		# сохраняем, выходим и выполняем	
	$ source ~/.bash_profile

> Следует учесть, что по умолчанию TextEdit заменяет программистские кавычки на "Smart кавычки", что можно изменить в настройках редактора.
	
Теперь используя команду `code`, можно открывать текстовые файлы из терминала в редакторе TextEdit, например:

	$ code ~/.bash_profile

Также можно добавлять алиас для уже существующей команды, изменяя ее поведение по умолчанию, например, добавив в `~/.bash_profile` алиас для команды `ls`:

	alias ls='ls -GFh'

Можно добавлять алиас для нескольких команд сразу, разделяя их точкой с запятой, например, для команд отображения/скрытия скрытых файлов:

	alias showfiles='defaults write com.apple.finder AppleShowAllFiles YES; killall Finder /System/Library/CoreServices/Finder.app'
	alias hidefiles='defaults write com.apple.finder AppleShowAllFiles NO; killall Finder /System/Library/CoreServices/Finder.app'