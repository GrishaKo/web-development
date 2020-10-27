# Заметки по использованию macOS VPN

- [VPN server on macOS High Sierra & macOS Mojave
](https://softwarerecs.stackexchange.com/questions/50816/vpn-server-on-macos-high-sierra-macos-mojave)
- [macOS Server
Service Migration Guide v1.2](https://developer.apple.com/support/downloads/macOS-Server-Service-Migration-Guide.pdf)

<!--ts-->
  * [Запуск VPN](#load-vpn)
 <!--te-->

<a id="load-vpn"></a>
## Запуск VPN на macOS

Выключаем VPN в приложении macOS Server, если он есть.

Служба VPN сервера macOS использует демон [vpnd](https://www.unix.com/man-page/osx/5/vpnd/) в macOS для предоставления служб L2TP IPSEC VPN. Файл конфигурации `/Library/Preferences/SystemConfiguration/com.apple.RemoteAccessServers.plist`, и его формат определяется на странице руководства vpnd.

В [Terminal](../Terminal/readme.md) переходим в директорию:

	cd /Library/LaunchDaemons
	
Далее создаем файл:

	sudo touch vpn.ppp.l2tp.plist
	
Изменяем права доступа к файлу:

	sudo chown root:wheel ./vpn.ppp.l2tp.plist
	
Открываем созданный файл:

	sudo nano vpn.ppp.l2tp.plist
	
Добавляем следующее содержимое:

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
	“http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	  <dict>
	    <key>Disabled</key>
	    <true/>
	    <key>EnableTransactions</key>
	    <true/>
	    <key>Label</key>
	    <string>vpn.ppp.l2tp</string>
	    <key>KeepAlive</key>
	    <true/>
	    <key>Program</key>
	    <string>/usr/sbin/vpnd</string>
	    <key>ProgramArguments</key>
	    <array>
	      <string>vpnd</string>
	      <string>-x</string>
	      <string>-i</string>
	      <string>com.apple.ppp.l2tp</string>
	    </array>
	    <key>EnableTransactions</key>
	    <false/>
	    <key>EnablePressuredExit</key>
	    <false/>
	  </dict>
	</plist>

Сохраняем изменения с помощью горячих клавиш `Control + o`, подтверждаем имя файла `Enter` и закрываем редактор `Control + q`.

Запускаем и проверяем работу:

	sudo launchctl load -w ./vpn.ppp.l2tp.plist
	launchctl print system/vpn.ppp.l2tp
	
**Текущее управление:**

Настройки можно изменить после настройки vpnd, отредактировав `/Library/Preferences/SystemConfiguration/com.apple.RemoteAccessServers.plist` файл. После внесения изменений вы можете заставить службу перечитать файл конфигурации, выполнив команду `sudo killall -HUP vpnd`.