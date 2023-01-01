# Installation FHEM

### Allgemein

FHEM ist eine OpenSource Smarthome Lösung auf Basis von Perl. Hiermit kann man verschiedene Systeme in eine Smarthome Anwendung integrieren.  

Der Einstiegspunkt für die Installation ist <a href="https://fhem.de" target="_blank">https://fhem.de</a>
Hier sind Wiki und Forum sehr hilfreich!

### Abhängigkeiten installieren

Es gibt einige Abhängigkeiten, die vorher installiert werden sollten.

!!! terminal "Terminal"
    <pre>
    sudo apt-get install libdevice-serialport-perl
    sudo apt-get install libio-socket-ssl-perl
    sudo apt-get install libjson-perl
    </pre>

### Installation

Inzwischen ist die Installation vereinfacht worden. Details gibt es hier: <a href="https://debian.fhem.de/" target="_blank">https://debian.fhem.de</a>

!!! terminal "Terminal"
    <pre>
    //Import repository gpg key:
    wget -O- https://debian.fhem.de/archive.key | gpg --dearmor | sudo tee /usr/share/keyrings/debianfhemde-archive-keyring.gpg

    //Add repository to /etc/apt/sources.list - / at the end:
    sudo nano /etc/apt/sources.list
    //  Am Ende: 
    deb [signed-by=/usr/share/keyrings/debianfhemde-archive-keyring.gpg] https://debian.fhem.de/nightly/ /

    //Update your package administration:
    sudo apt-get update

    //Install fhem:
    sudo apt-get install fhem
    </pre>

### Updates

_ACHTUNG!!!_ Vor der Ausführung eines FHEM Updates oder vor der Installation eines CCU Firmware Updates muss der RPC-Server unter Umständen gestoppt werden, falls es zu Problemen kommt, das der rpcserver danach nicht mehr startet.

!!! fhem "FHEM Kommando"
    <pre>set ccu rpcserver on/off</pre>

_ACHTUNG!!!_ Regelmäßig Config Save im Menü im Web klicken, damit für neue Geräte GUIDs angelegt werden. Wichtig für Alexa (Duplikate).

## Backup alte Installation wiederherstellen

Das Hauptverzeichnis der FHEM Installation sollte "_/opt/fhem_" sein.  
Auf dem NAS sollte es ein FHEM Verzeichnis mit der alten Installation geben. Die folgnden Dateien überschreiben bzw. neu einfügen:

* fhem.cfg
* FHEM/99_myUtils.pm
* FHEM/99_regenprognoseUtils.pm
* FHEM/99_rok_*.pm
* Evtl. alexa-fhem.cfg
* rokscripte/*
* log/weekprofile*.cfg
* log/Zisterne*.log
* www/gplot/SVG_FileLog_*.gplot
* Max Thermostate einmal die Temperatur ändern, dann werden sie kurz danach erkannt!
* Max Thermostate müssen manchmal nach Änderungen von Manu auf Auto neu gesetzt werden

## Berechtigungen & Aufräumen

Das eigene script rokscripts/cleanup ausführen. Dadurch werden Berechtigungen gesetzt und debug_info.txt wird geleert und alte log Files werden gelöscht. Für die Details am besten im Script nachschauen, welche Schritte durchgeführt werden.

!!! terminal "Terminal"
    <pre>
    cd /opt/fhem
    bash rokscripts/cleanup
    </pre>

Das Script als cron einplanen. Das bedeutet es wird regelmäßig ein Job gestartet, der das Script ausführt:

!!! terminal "Terminal"
    <pre>
    crontab -e
    </pre>

!!! file "crontab"
    <pre>
    \# Jeden Freitag um 0:30 den Job ausführen
    30 0 * * FRI /bin/bash /opt/fhem/rokscripts/cleanup >/dev/null 2>&1
    
    \# Reboot jeden Sonntag 3 Uhr
    0 3 * * sun /sbin/shutdown -R now
    </pre>

## sendMail

Damit FHEM Mails verschicken kann, muss das Tool sendMail installiert werden:

!!! terminal "Terminal"
    <pre>
    sudo apt-get install sendmail    #reicht nicht für FHEM und ist evtl. gar nicht notwendig!!!
    sudo apt-get install sendemail libio-socket-ssl-perl libnet-ssleay-perl perl
    </pre>

Bei sendEmail handelt es sich nur um ein Script!  

Problem ist, dass sendmail auf CLI Ebene den Absender nicht ändern kann. Das liegt am ssmtp und das ist auch veraltet.  
Daher sollte msmtp verwendet werden! Dazu existiert ein extra Kapitel [SendMail über CLI](raspi.md#sendmail-uber-cli) unter Raspberry Installation.

## KNXD

Für die Anbindung von FHEM an das EIB/KNX Bussystem wird das Tool KNXD benötigt.  
Die KNX Installation im Haus ist über ein Modul im Elektroschrank (BAOS) in das Netzwerk eingebunden (IP 192.168.1.11)
Weitere Infos hier: <a href="https://wiki.fhem.de/wiki/Knxd" target="_blank">https://wiki.fhem.de/wiki/Knxd</a>

??? terminal "Terminal"
    <pre>
    sudo apt-get install knxd knxd-tools
    </pre>

Folgende Konfiguration anpassen:

!!! terminal "Terminal"
    <pre>
    sudo nano /etc/knxd.conf
    </pre>

!!! file "/etc/default/knxd"
    <pre>
    KNXD_OPTS="-f -t1023 -e 0.0.1 -E 0.0.2:8 -c -b ipt:192.168.1.11"
    </pre>

## MQTT

Nicht den internen FHEM MQTT Server aktivieren: Der blockiert FHEM!  
Mosquitto installieren und aktivieren.

!!! terminal "Terminal"
    <pre>
    sudo apt-get install mosquitto mosquitto-clients
    sudo systemctl enable mosquitto.service

    \# MQTT Server Test
    sudo systemctl status mosquitto 

    \# Start und Stop des Servers
    sudo systemctl stop mosquitto 
    sudo systemctl start mosquitto
    </pre>

Monitor Tool für MQTT - MQTT Explorer (Win Installation oder Portable): <a href="http://mqtt-explorer.com/" target="_blank">http://mqtt-explorer.com/</a>

## Homematic

Homematic ist ein eigenständiges System, was über die CCU gesteuert wird. Dieses System kann ebenfalls in FHEM eingebunden werden.  
Wir verwenden es z.B. für die Prüfung, ob die Dachfenster geöffnet sind, oder Heizungsthermostate (höhere Reichweiter als MAX).  
Die CCU ist in unserem Netz unter IP 192.168.1.98 erreichbar.

Die Doku für die Inbetriebname ist hier: <a href="https://wiki.fhem.de/wiki/HomeMatic" target="_blank">https://wiki.fhem.de/wiki/HomeMatic</a>  
Man muss erst das CCU Device definieren und dann sind die Interfaces wichtig! Dadurch werden weitere Devices pro Interface (BidCos-RF, HmIP-RF usw.) angelegt.

!!! terminal "Terminal"
    <pre>
    sudo apt-get update && sudo apt-get install -y librpc-xml-perl
    </pre>

FHEM Homematic Best Practices gibt es hier: <a href="https://wiki.fhem.de/wiki/HMCCU_Best_Practice" target="_blank">https://wiki.fhem.de/wiki/HMCCU_Best_Practice</a>
 
Geräte anlernen – siehe auch vorherigen Link Best Practices:  
Auf das Homematic WebUI gehen und anlernen im Menü und am Device aktivieren. Im Posteingang erscheint dann leicht verzögert die Geräteanlage.
Im Homematic WebUI einen eindeutigen Namen <devicename> ohne Umlaute und ohne Leerzeichen vergeben.  
In FHEM cmd Gerät bekanntmachen und Details einlesen: 

!!! fhem "FHEM Kommandos"
    <pre>
    \# Bekanntmachen der Geräte in FHEM
    get ccu ccuConfig
    \# Geräte auflisten und in Address steht die ID
    get ccu ccuDevices
    \# Details zu einem Gerät lesen - nicht notwendig
    get ccu deviceinfo <ID_aus_ccuDevices>
    \# Device anlegen - Seite neu laden, damit des Device im Dropdown auftaucht
    get ccu createDev <select_from_dropdown_or_devicename_aus_WebUICCU>
    \# WICHTIG! Nach createDev nochmal 
    get ccu ccuConfig
    </pre>

Die Address/ID ist im Homematic WebUI in der Spalte *Address* im Ergebnis von get ccu ccuDevices zu sehen.  
Das ist aber für die Anlage nicht mehr notwendig!

Nun noch die Ergänzungen in der Config:  

!!! fhem "fhem.cfg"
    <pre>
    attr <new_device> ccuflags showMasterReadings,showLinkReadings,showDeviceReadings
    attr <new_device> userattr weekprofile
    attr <new_device> weekprofile Temperatur_Weekprofile:default:<new_device_profil>
    </pre>

WICHTIG! Danach die Readings updaten:

!!! fhem "FHEM Kommandos"
    <pre>
    get <new_device> config
    </pre>

_Probleme:_  
Falls die Readings nicht automatisch aktualisiert werden, muss der rpcserver (Device ccu) wieder gestartet werden.

!!! fhem "FHEM Kommando"
    <pre>set ccu rpcserver on/off</pre>

## Pushover Nachrichten

Hiermit können Nachrichten versendet werden mit Links, Bildern und Ablaufzeit: <a href="https://wiki.fhem.de/wiki/Pushover" target="_blank">https://wiki.fhem.de/wiki/Pushover</a>  
Es muss ein Konto angelegt werden und die App (Android, Apple, Desktop) kostet einmalig 5€.  
Viele Beispiele sind in der <a href="https://fhem.de/commandref_DE.html#Pushover" target="_blank">CommandRef</a>  
Expire geht wohl nur in Verbindung mit retry!  
Über cancel_id kann man evtl. ein Löschen der Nachricht für andere realisieren.

!!! fhem "FHEM Kommando"
    <pre>
    define pushmsg Pushover <TOKEN> <USER>
    set pushover msg Diese Nachricht hat einen Anhang! attachment="demolog/pictures/p1.jpg"
    set pushover msg title="Betreff" message="Nachrichtentext und die Nachricht läuft nach 1 Stunde ab.\n2. Zeile und noch ein Link" url_title="Haussteuerung" action="http://192.168.1.99:8083/fhem?room=Wellness" expire=3600
    set pushover msg Nur an ein Device! device="Rogis-iPhone"
    set pushover msg Tonauswahl sound="falling"  \# https://pushover.net/api#sounds
    </pre>

## Yamaha Boxen

Die MusicCast Boxen von Yahama lassen sich in FHEM einbinden. Dazu ist ein weiteres Perl Paket notwendig:

!!! terminal "Terminal"
    <pre>
    sudo apt-get install -y libnet-upnp-perl
    </pre>

## Telegram

Der Messenger _telegram_ lässt sich in FHEM nutzen zum Senden und Empfangen von Nachrichten.

!!! fhem "fhem.cfg"
    <pre>
    define RogisBot TelegramBot 280347001:AAGJmhGrb8AAJ8U7jqi5NdSP8b1GB_C-ryA
    </pre>

Token verschwindet nach dem Speichern. Es ist also auch wichtig die GUID nach Neustart zu ergänzen und dann speichern!

Damit ein Kontakt angeschrieben werden kann muss der Kontakt (oder aus der Gruppe heraus) @rogisbot angeschrieben werden.  
Danach sollte FHEM an den Kontakt oder in die Gruppe schreiben können.  

!!! fhem "Device RogisBot - Kontakte hinzufügen"
    <pre>
    attr RogisBot allowUnknownContacts 1
    </pre>

Dach wieder zurücksetzen:

!!! fhem "Device RogisBot - Kontakte NICHT hinzufügen"
    <pre>
    attr RogisBot allowUnknownContacts 0
    </pre>

## Conbee II

Mit dem USB Gateway können Aqara, Hue und ähnliche Geräte mit Zigbee Protokoll in FHEM eingebunden werden.  

Eine gute Anleitung gibt es hier: <a href="http://coldcorner.de/2018/04/02/deconz-hue-bridge-auf-dem-raspberry-pi-emulieren" target="_blank">http://coldcorner.de/2018/04/02/deconz-hue-bridge-auf-dem-raspberry-pi-emulieren</a>

Vorab den Zugriff auf die Serial Ports aktivieren:

!!! terminal "Terminal"
    <pre>
    sudo raspi-config
    </pre>

Hier auf Interfacing Options -> Serial und dann mit "_Yes_" bestätigen!

??? terminal "Terminal"
    <pre>
    sudo wget http://deconz.dresden-elektronik.de/raspbian/deconz-latest.deb
    sudo dpkg -i deconz-latest.deb
    sudo apt update
    sudo apt install -f
    _//Autostart einrichten_
    sudo systemctl enable deconz
    </pre>

Hier noch die Anleitung um das ausführbare Programm statt dem Web Interface zu nutzen: <a href="https://phoscon.de/en/conbee2/install#raspbian" target="_blank">https://phoscon.de/en/conbee2/install#raspbian</a>

??? terminal "Terminal"
    <pre>
    sudo gpasswd -a $USER dialout
    wget -O - http://phoscon.de/apt/deconz.pub.key | sudo apt-key add –
    sudo sh -c "echo 'deb http://phoscon.de/apt/deconz \
                $(lsb_release -cs) main' > \
                /etc/apt/sources.list.d/deconz.list"
    sudo apt update
    sudo apt install deconz
    </pre>

Nun sollte das Conbee Gateway (Phoscon oder deCONZ) erreichbar sein über die Raspi IP Adresse: <a href="http://192.168.1.99/pwa/login.html" target="_blank">http://192.168.1.99/pwa/login.html</a>

Beim ersten Start vergeben:

* Gateway Name: Phoscon-GW
* Password: siehe Password Depot

!!! fhem "fhem.cfg"
    <pre>
    define deCONZ HUEBridge ip.vom.deconz.gateway
    attr deCONZ httpUtils 1
    </pre>

!!! fhem "FHEM Kommando"
    <pre>
    set deCONZ active
    </pre>

!!! fhem "Conbee Gateway Web Interface"
    <pre>
    deCONZ App -> Gateway ->Erweitert -> App verbinden
    </pre>

Nun ist das Gateway (hoffentlich) mit FHEM verbunden: siehe deconz Status connected.

Mit "_get deCONZ sensors_" wird die Sensoren Liste angezeigt und mit der ID kann man die Sensoren entsprechend einbinden.

!!! fhem "Zigbee Geräte einbinden"
    <pre>
    get deCONZ sensors
    </pre>

Den Magic Cube muss man 2x pairen, da er für Kippen und Drehen je einen Sensor anlegt. Er erscheint dann nur einmal in der Phoscon App aber bei get sensors tauchen 2 auf.

## Alexa

Alexa kann in FHEM zur Steuerung mit eingebunden werden: <a href="https://wiki.fhem.de/wiki/FHEM_Connector" target="_blank">https://wiki.fhem.de/wiki/FHEM_Connector</a>

sudo apt-get install nodejs npm
sudo npm install -g alexa-fhem
ProxyKey 39415AD3-D9A3F62F7C936B37-B4B4E601A294D088

In FHEM cmd ermitteln mit: get alexa proxyKey

Der ProxyKey muss bei Neuinstallation in <a href="https://alexa.amazon.com" target="_blank">https://alexa.amazon.com</a> neu gesetzt werden durch Skill deaktivieren, wieder aktivieren und anmelden.

Wenn es neue Alexa Geräte in FHEM gibt, dann muss der Alexa Server in FHEM WebUI neu gestartet werden und dann muss man unter <a href="https://alexa.amazon.com" target="_blank">https://alexa.amazon.com</a> neue Smarthome Geräte suchen.

## Amazon Echo Geräte

Hier können mit Hilfe des Moduls echodevice z.B. Texte auf den Echos ausgegeben werden: <a href="https://mwinkler.jimdo.com/smarthome/eigene-module/echodevice/" target="_blank">https://mwinkler.jimdo.com/smarthome/eigene-module/echodevice/</a>
 
!!! terminal "Terminal"
    <pre>
    sudo npm install --prefix /opt/fhem/cache/alexa-cookie alexa-cookie2
    sudo chown -R fhem:  /opt/fhem/cache/alexa-cookie
    </pre>

!!! fhem "fhem.cfg xxx@xxx.xx wirklich so übernehmen!"
    <pre>
    define echodevice echodevice xxx@xxx.xx xxx
    </pre>

!!! fhem "FHEM Kommando"
    <pre>
    set echodevice NPM_login new
    </pre>

Warten bis URL angezeigt wird – evtl. unter: <a href="http://10.10.0.233:3002" target="_blank">http://10.10.0.233:3002</a>
Hier mit Amazon Login einloggen!

Geräte anlegen mit:

!!! fhem "FHEM Kommando"
    <pre>
    set echodevice autocreate_devices
    </pre>

## MAX Gateway

MAX Geräte werden nicht mehr hergestellt. Wir haben evtl. noch Heizungsthermostate im Einsatz (OG Bad und Vincent).  

Falls im Log folgendes auftaucht, obwohl anscheinend verbunden:
CUL_MAX_SendQueueHandler: Missing ack from 1997cb for 0b0e00401234561997cb0068

Achtung!!! Vincent zum Anlernen abschrauben und ins Büro mitnehmen wegen Empfang.

Dann neu anlernen:

* FHEM: set cm pairmode 300 (aktiviert den Modus in FHEM für 5 Minuten)
* Am Thermostat BOOST für mehr als 3 Sekunden drücken. 

Evtl. muss komplett neu angelegt werden

* Geräte aus der cfg auskommentieren
* Batterie raus alle 3 Knöpfe drücken Batterie rein
* Nach Ins autocreate an und mit pairmode und 3 Sek Boost anlernen
* AdA (Adaptierfahrt) per 2x kurz Boost erst nachdem wieder aufgeschraubt 

## Wetterstation WS980Wifi

<a href="../../attachments/Wetterstation_WS980WiFi_Bedienungsanleitung.pdf" target="_blank">Hier ist die Bedienungsanleitung hinterlegt!</a>

Die Verbindung der Wetterstation in das WLAN erfolgt über die App *WS View*.  
Die App funktioniert NUR unter Android!  
Ausserdem muss im Fritz Router das 5 GHz WLAN deaktiviert werden, da sonst die App nicht funktioniert.

## FreeFileSync Backup

Mit FreeFileSync komplettes Verzeichnis /opt/fhem sichern.  
Aber nicht auf das Netzlaufwerk \\192... (beim letzten Mal ging \\192 doch) sondern auf R:\ und vorher verbinden mit pi User.  
Die Verzeichnisse .alexa und .ssh müssen ausgeschlossen werden.  
In FreeFileSync sollte Fehler ignorieren bei Batch ausgewählt sein, sonst gibt es evtl. bei fehlenden Berechtigungen einen Abbruch. Z.B. kann die sync lock Datei nicht geschrieben werden – ist aber kein Problem.

## VSCode einrichten für die FHEM Bearbeitung

Auf dem Raspi muss NodeJS installiert sein!  
VSCode installieren und die Extension Remote SSH.  
STRG+SHIFT+P => neue Remote Verbindung anlegen:  

* pi@192.168.1.nnn
* Fingerprint bestätigen
* Passwort eingeben
* Ordner wählen
* Lokaler VSCode PC: Key in *.pub File in Folder: %userprofile%\\.ssh finden
* Raspi: ~/.ssh/authorized_keys erstellen oder erweitern um den Key: ssh-rsa <key> aus dem .pub File

## Modifikation Modul weekprofile für Homematic

***Der Patch sollte ab jetzt in FHEM enthalten sein!***

Weekprofile unterstützt zwar Homematic, aber da das Homematic Modul die Geräte unterschiedlich behandelt, gibt es wohl keine einheitliche Lösung. Daher muss für meine Heizungsthermostate eine Modifikation vorgenommen werden. Es dürfen im derzeitigen Knop Haus keine Präfixe verwendet werden.  
&rarr; Modul: 98_weekprofile.pm  
&rarr; Routine: Sub weekprofile_sendDevProfile  
&rarr; Im letzten Drittel (hinter Zeile 480)  

??? file "98_weekprofile.pm"
    <pre>
    ...
    } elsif ($type =~ /HMCCU.*/){ 
        $cmd .= "set $device config" if ($type eq "HMCCU_HM");
        $cmd .= "set $device config 1" if ($type eq "HMCCU_IP");
        my $k=0;
        my $dayCnt = scalar(@dayToTransfer);
        my $prefix = weekprofile_get_prefix_HM($device,"ENDTIME_SUNDAY_1",$me);
        <b># no prefix by set see topic,46117.msg1104569.html#msg1104569
        $prefix = "" if ($type eq "HMCCU_HM");</b>
        if (!defined($prefix)) {
        Log3 $me, 3, "$me(sendDevProfile): no prefix found"; 
        $prefix = ""; 
        }
    ...
    </pre>
