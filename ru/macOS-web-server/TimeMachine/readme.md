# Заметки по использованию macOS Time Machine

[Time Machine](https://support.apple.com/en-us/HT201250) (ENG/[RUS](https://support.apple.com/ru-ru/HT201250)) - встроенное macOS средство резервного копирования.

<!--ts-->
  * [Подключение сетевого диска для резервных копий Time Machine](#подключение-сетевого-диска-для-резервных-копий-time-machine)
  * [Использование компьютера Mac в качестве хранилища резервных копий Time Machine](#использование-компьютера-mac-в-качестве-хранилища-резервных-копий-time-machine)
 <!--te-->

<a id="connect-network-drive"></a>
## Подключение сетевого диска для резервных копий Time Machine

Сначала необходимо помощью [../#подключиться к сетевому диску с](../FileSharing/readme.md#connect-to-server), затем переходим в "Time Machine", нажимаем "Выбрать резервный диск" (Select Backup Disk) и в списке выбираем подключенный сетевой диск.

Если в Time Machine нет в списке сетевого диска, то может помочь утилита командной строки `tmutil`, например:

	$ sudo tmutil setdestination -ap smb:/USER@IP/Backups
	
> Также c помощью этой команды можно подключить образ диска с резервной копией Time Machine, указав путь до него через пробел сразу после команды "tmutil setdestination".
	
Посмотреть подключенные диски: 
	
	$ tmutil destinationinfo

Подробнее об утилите `tmutil`:

1. [Control Time Machine from the command line](https://www.macworld.com/article/2033804/control-time-machine-from-the-command-line.html) (ENG).
2. [Time Machine utility](https://ss64.com/osx/tmutil.html) (ENG).
3. [Утилита Time Machine](http://osxh.ru/command/tmutil-terminal-time-machine) (RUS). 

<a id="file-sharing"></a>
## Использование компьютера Mac в качестве хранилища резервных копий Time Machine

Сначала необходимо [../#включить общий доступ к папке](../FileSharing/readme.md#enable-file-sharing), которая будет использоваться для резервных копий Time Machine и затем настроить папку для резервных копий:

1. В списке общих папок справа кликаем, удерживая нажатой клавишу Control, каталог, который требуется использовать для резервных копий Time Machine. 
2. В открывшемся контекстном меню выбираем "Дополнительные параметры".
3. В диалоговом окне "Дополнительные параметры" устанавливаем флажок "Использовать как папку для резервного копирования Time Machine".