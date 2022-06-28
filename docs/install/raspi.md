# Installation Raspberry

### Allgemein

Alle Informationen zum Raspberry Pi findet man hier: <a href="https://www.raspberrypi.org" target="_blank">https://www.raspberrypi.org</a>  
Inzwischen gibt es ein Tool auf der Seite, das man herunterlädt und ausführt.  

Dieses Tool erstellt ein Raspberry Pi Image auf einer SD Card, das dann in den Raspberry Pi eingesteckt wird und das Betriebssystem sollte dann starten.  

Für den ersten Start eine Tastatur Maus und Bildschirm an den Raspi hängen und anschalten.  
Es kann auch ein WLAN beim Start aktivieren (Google Suche) oder ein Netzwerkkabel angklemmen und starten. 
Dann findet man die IP im Router und kann direkt auf den Raspi zugreifen. Allerdings dann ohne Desktop Windows Unterstützung. Dann geht nur Kommondozeilenebene im Terminal.  

Im Betrieb erfolgt der Zugriff auf den Raspi über SSH mit dem Tool Putty und der entsprechenden IP Adresse. Dabei muss der Raspi nur per WLAN oder LAN Kabel im Netz hängen.   

Das folgende Problem taucht nur auf, wenn die Tastatur noch nicht geändert ist.
User: pi / Passwort: raspberry (Achtung – evtl. wg. Tastatur: raspberrz) => ändern in (siehe Password Depot) mit passwd

### Installation aktualisieren

Man kann die Standardinstallation aktualisieren. Das bedeutet nicht, dass alle Installationen auch aktualisiert werden! Zum Beispiel KNXD muss manuell aktualisiert werden.  
Fast alle Befehle müssen als Root User auf Unix Systemen ausgeführt werden. Dies erfolgt über den Zusatz "_sudo_" vor den einzelnen Befehlen. Es gibt aber auch Ausnahmen.

!!! terminal "Terminal"
    <pre>
    sudo apt-get update
    sudo apt full-upgrade
    sudo apt-get dist-upgrade
    </pre>
 
### Probleme analysieren

!!! terminal "Terminal"
    <pre>
    dmesg
    more /var/log/messages
    more /var/log/syslog
    more /var/log/kern.log
    </pre>

Gesammelte Infos auf der folgenden <a href="https://elinux.org/R-Pi_Troubleshooting" target="_blank">Wiki Seite</a>!

 
### Konfigurationsmenü  

!!! terminal "Terminal"
    <pre>sudo raspi-config 
    //Bzw. im Desktop
    startx</pre>
Hostname / Autostart startx / Zeitzone / Locale / Tastatur / SSH & VNC – unter Interfacing Options usw.
Hostname trotzdem auch wie in [Kapitel _Statische IP vergeben_](#statische-ip-vergeben) beschrieben hostname in dhcpcd.conf anpassen.  
Warten auf Netzwerk beim booten

### IP anzeigen

Ermitteln der aktuellen IP im Netzwerk.

!!! terminal "Terminal"
    <pre>ifconfig</pre>

### Statische IP vergeben

Geht inzwischen auch über Desktop! Rechtsklick auf die Netzwerkverbindung oben rechts und dann zu Einstellungen.

Damit der Raspi im Netzwerk immer mit der gleichen IP angesprochen werden kann, muss eine statische IP vergeben werden. Der Router vergibt sonst unter Umständen nach einer gewissen Zeit eine neue IP.  
Im Router muss ein Bereich für feste/ statische IPs eingetragen werden, der nicht über DNS vergeben wird. Aus diesem Bereich muss die statische IP gewählt werden.  
_Achtung!_ In raspi-config -> Network Options -> network interface names muss predictable interface names disabled sein! Könnte veraltet sein.

In diesem Beispiel wird die statische IP des Raspis auf 192.168.1.99 gesetzt.

!!! terminal "Terminal" 
    <pre>sudo nano /etc/dhcpcd.conf
    
    //Alte Versionen:
    /etc/network/interfaces)</pre>

??? file "/etc/dhcpcd.conf"
    <pre>
    hostname
    berry

    interface
    static ip_address=192.168.1.99/24
    static routers=192.168.1.1
    static domain_name_servers=192.168.1.1 8.8.8.8
    </pre>

### IPv6 deaktivieren

Geht inzwischen auch über Desktop!

Keine Ahnung mehr, wo das Probleme verursacht hat!?

!!! terminal "Terminal"
    <pre>sudo nano /etc/sysctl.conf</pre>

!!! file "/etc/dhcpcd.conf"
    <pre>net.ipv6.conf.all.disable_ipv6=1</pre>

### WLAN

Geht inzwischen auch über Desktop!

<a href="http://www.netzmafia.de/skripten/hardware/RasPi/RasPi_Network.html" target="_blank">http://www.netzmafia.de/skripten/hardware/RasPi/RasPi_Network.html</a>

Die Netzwerkkonfiguration editieren wie in [Kapitel _Statische IP vergeben_](#statische-ip-vergeben):

!!! terminal "Terminal" 
    <pre>sudo nano /etc/dhcpcd.conf
    
    //Alte Versionen:
    /etc/network/interfaces)</pre>

??? file "/etc/dhcpcd.conf - Dynamische IP"
    <pre>
    auto lo
    iface lo inet loopback
    iface eth0 inet dhcp
    auto wlan0
    iface wlan0 inet dhcp
    allow-hotplug wlan0
    wpa-ap-scan 1
    wpa-scan-ssid 1
    wpa-ssid "NAME-DES-WLAN"
    wpa-psk "DER-GEHEIME-WLAN-KEY"
    </pre>

??? file "/etc/dhcpcd.conf - Statische IP"
    <pre>
    auto lo
    iface lo inet loopback
    iface eth0 inet static
    address 172.20.0.2
    netmask 255.255.255.0
    gateway 172.20.0.1
    auto wlan0
    allow-hotplug wlan0
    iface wlan0 inet static
    address 192.168.1.99
    netmask 255.255.255.0
    gateway 192.168.1.1
    wpa-ap-scan 1
    wpa-scan-ssid 1
    wpa-ssid "NAME-DES-WLAN"
    wpa-psk "DER-GEHEIME-WLAN-KEY"
    </pre>

!!! terminal "Terminal"
    <pre>
    //Änderungen dhcpcd.conf aktivieren:
    /etc/init.d/networking restart
    </pre>

Zum Testen der Verbindung _ifconfig_ oder _ping google.com_ im Terminal eingeben.

**Problem!** Netzwerkverbindung steht ping 8.8.8.8 geht, aber ping google.com geht nicht
In _/etc/resolv.conf_ gleiche IP wie das Gateway eintragen:

!!! terminal "Terminal"
    <pre>
    sudo nano /etc/resolv.conf
    </pre>

### SSH aktivieren

Damit der SSH Zugriff über das Tool Putty innerhalb des Netzwerks funktioniert, muss auf dem Raspi SSH aktiviert werden.

Geht auch über Desktop in der Konfiguration!
!!! terminal "Terminal"
    <pre>
    sudo apt-get install ssh 
    sudo /etc/init.d/ssh start
    </pre>
~~sudo update-rc.d ssh defaults~~ Autostart beim Booten (alt) => gemäß [Kapitel _Konfigurationsmenü_](#konfigurationsmenu) Interfaces aktivieren

***ACHTUNG!***
Falls die IP schon mal von einem PC aus per SSH verbunden wurde per Putty oder VSCode, dann die IP Einträge aus der Datei **"%UserProfile%/.ssh/known_hosts"** entfernen

### SSH per Private und Public Key => keine Passwortabfrage in VSCode
Es müssen zuerst auf dem Remote Rechner (Windows) im Homeverzeichnis die Dateien ~/.ssh/id_rsa und ~/.ssh/id_rsa.pub angelegt werden (sshgen)  
Dann muss auf dem Server das .ssh Verzeichnis mit der Datei authorized_key angelegt werden:

```
(umask 077 && test -d ~/.ssh || mkdir ~/.ssh)
(umask 077 && touch ~/.ssh/authorized_keys)
```
In die Datei authorized_keys muss der Inhalt der Datei .ssh/id_rsa.pub kopiert werden.


### Zeitzone

!!! terminal "Terminal"
    <pre>sudo dpkg-reconfigure tzdata</pre>
Europe und Berlin auswählen

### Zeitserver

Die Einrichtung des Zeitservers stellt die aktuelle Uhrzeit sicher.

!!! terminal "Terminal"
    <pre>
    sudo apt-get install ntpdate
    sudo ntpdate -u de.pool.ntp.org
    </pre>

### Rechnername ändern

Den aktuellen Namen des Raspberrys kann man anpassen. In diesem beispiel auf "_berry_"

!!! terminal "Terminal"
    <pre>sudo nano /etc/hosts</pre>
!!! file "/etc/hosts"
    <pre>127.0.0.1 berry</pre>

!!! terminal "Terminal"
    <pre>sudo nano /etc/hostname</pre>

!!! file "/etc/hostname"
    <pre>berry</pre>

!!! terminal "Terminal"
    <pre>sudo hostname -F /etc/hostname</pre>

### Samba installieren

Der Samba Server aktiviert die Möglichkeit, dass innerhalb des Netzwerks auf bestimmte Verzeichnisse des Raspis zugegriffen werden kann.  
Dadurch kann man die FHEM Konfiguration in die automatische Sicherung mit einbinden.

!!! terminal "Terminal"
    <pre>
    sudo apt-get install samba cifs-utils
    sudo nano /etc/samba/smb.conf
    </pre>

??? file "/etc/samba/smb.conf - FHEM erst nach der FHEM Installation ergänzen!"
    <pre>
    workgroup = PRIVAT
    wins support = yes

    \#Am Ende ergänzen
    [fhem]
    path = /opt/fhem
    public = yes
    writeable = yes
    read only = no
    </pre>

!!! terminal "Terminal"
    <pre>
    //smb.conf testen:
    testparm

    //Änderungen smb.conf aktivieren (start oder stop geht auch)
    sudo service smbd restart
    </pre>

### Festplatte mounten

Mit Mounten kann man andere Netzwerklaufwerke in den Raspi einbinden, damit man darauf zugreifen kann.  
Somit kann man z.B. auf die FHEM Sicherung im NAS oder auf andere Daten im Netzwerk zugreifen.

!!! terminal "Terminal"
    <pre>
    sudo mkdir /kino
    sudo mkdir /nas
    sudo nano /etc/fstab
    </pre>
!!! file "/etc/hostname"
    <pre>
    //192.168.1.111/kino    /kino    cifs    defaults,user=admin,password=admin,rw    0    0
    //192.168.1.111/roger    /nas    cifs    defaults,user=admin,password=<NAS_PASSWORT>,x-systemd.automount,x-systemd.requires=network-online.target,rw    0    0
    </pre>
_Anmerkung:_ Die x-systemd Sachen sind notwendig, dass mount beim Booten erst nach Netzwerkverbindung versucht wird.

!!! terminal "Terminal: _fstab neu einlesen_"
    <pre>sudo mount -a </pre>

### NodeJS & NPM installieren

<a href="https://github.com/nodesource/distributions/blob/master/README.md" target="_blank">Installations Infos</a>

<a href="https://github.com/nodesource/distributions/tree/master/deb" target="_blank">Alle verfügbaren Versionen</a>

Das NodeJS Paket ist nodejs und NICHT node!

Evtl. muss vorher das node Tool entfernt werden:

!!! terminal "Terminal"
    <pre>sudo apt purge node</pre>

!!! terminal "Terminal"
    <pre>
    //Update Package Manager auf die aktuellste Node Version
    curl -sL https://deb.nodesource.com/setup_current.x | sudo -E bash -
    //Update Package Manager auf eine spezifische Node Version
    curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
    sudo apt-get install -y nodejs
    </pre>

NPM wird automatisch installiert zusammen mit den neuen NodeJS Versionen.  
*Achtung* node -v kann erst ausgeführt werden, wenn ein neues Terminal geöffnet wird (PATH).

### Autostart eines NodeJS Scripts

Autostart eines eigenen NodeJS Web Servers über das Tool pm2.  
Startet nach Neustart und nach kill -9 erfolgt ein Restart des Scripts.  

!!! terminal "Terminal"
    <pre>
    sudo npm install -g pm2
    pm2 startup
    //Es wird ein Kommando erzeugt und das kopieren und ausführen
    cd /smartdisplay
    pm2 start server/index.js -- --port=8000
    //Mit Log bei Problemen: pm2 start --log /smartdisplay/pmlog.txt server/index.js -- 8000
    //Alle 15 Minuten (no-autorestart ist Wichtig): pm2 start twitterbot/bot.js --cron "\*/15 \* \* \* \*" --no-autorestart
    pm2 save
    </pre>

***Achtung!*** Ports unter 1024 gehen nicht! Und es muss "Warten auf Netzwerk beim Booten" aktiviert sein!    

Man kann die Permissions aktivieren über:

!!! terminal "Terminal"
    <pre>
    sudo setcap 'cap_net_bind_service=+ep' \`which node\`
    </pre>

pm2 Befehle (komplett über pm2 -h):

| Befehl | Beschreibung  |
| --- | --- |
| l | Listet alle Prozesse |
| del <nr> | Löscht Prozess Nr <nr> |
| save | Speichert den aktuellen Stand ab |

### Hilfreiche Befehle

| Befehl | Beschreibung  |
| --- | --- |
| which | Pfad eines Programms ermitteln |
| chmod -R u+w <file/path> | Datei/Pfad User darf auch schreiben (oder 755 Muster) / -R recursive |
| chown -R usr:grp <file/path> | Datei/Pfad Owner User usr und Gruppe grp setzen |
| usermod -aG grp usr | Fügt den User usr zur Gruppe grp hinzu |
| gpasswd -d usr grp | Löscht den User usr von der Gruppe grp |
| df -h | Speicherplatz auf den Laufwerken anzeigen |

### Backup Image erstellen

Als Voraussetzung muss das NAS gemounted sein. 

Zuerst muss die Partition ermittelt werden:

!!! terminal "Terminal - Chromium installieren"
    <pre>sudo fdisk /dev/mmbcblk0
    Key p => Listet die Partitionen (siehe ganz oben)
    Key q => Beenden</pre>

Dann kann mit der Partition die Image Erstellung getriggert werden (if=Partition / of=Pfad&File des zu erstellenden Images):

!!! terminal "Terminal - Chromium installieren"
    <pre>sudo dd if=/dev/mmcblk0 of=/nas/smartdisplay_backup_sdcard/raspi_image.img bs=1M</pre>

Dies kann über eine Stunde dauern!

### Chromium installieren bzw. deinstallieren

Chromium Browser installieren bzw. deinstallieren.

!!! terminal "Terminal - Chromium installieren"
    <pre>sudo apt install chromium-browser -y</pre>

!!! terminal "Terminal - Chromium deinstallieren"
    <pre>
    sudo apt-get remove chromium-browser
    sudo dpkg -r chromium-browser
    sudo rm -Rf /etc/chromium-browser
    sudo rm -Rf ~/.config/chromium
    </pre>

### Editor nano installieren

Dieser Terminal Editor ist einfacher zu bedienen, als der VI, aber trotzdem noch relativ rudimentär.

!!! terminal "Terminal"
    <pre>sudo apt-get install nano</pre>

### Deutsche Tastatur im Terminal

Ohne Desktop kann man auch im Terminal die deutsche Tastatur einrichten.

!!! terminal "Terminal"
    <pre>
    sudo dpkg-reconfigure keyboard-configuration
    </pre>
Dann "_Generische Tastatur mit 105 Tasten (Intl)_" auswählen.
Anschließend "_German_" und dann alles bestätigen.
