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
    \# Kann keine neuen Pakete installieren und keine alten deinstallieren
    sudo apt full-upgrade
    \# Kann neue installieren und alte deinstallieren UND neue Kernel Version
    sudo apt-get dist-upgrade
    \# Simulation für dist-upgrade
    sudo apt-get --simulate dist-upgrade
    </pre>
 
### Probleme analysieren & Resource Monitoring

!!! terminal "Terminal"
    <pre>
    dmesg
    more /var/log/messages
    more /var/log/syslog
    more /var/log/kern.log
    </pre>

Gesammelte Infos auf der folgenden <a href="https://elinux.org/R-Pi_Troubleshooting" target="_blank">Wiki Seite</a>!

Tools, die im Terminal verwendbar sind: 

!!! terminal "Terminal"
    <pre>
    \# Ähnlich Taskmanager in Windows MIT Interaktion - Sortierung per Klick
    sudo apt-get install htop   -   Oft schon vorinstalliert!
    htop
    \# Ähnlich Taskmanager in Windows aber ohne Interaktion sortiert nach Memory oder CPU
    top -o %MEM oder %CPU
    \# Memory in MB und alle 2 Sekunden auffrischen
    free -m -s 2
    </pre>

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

Seit bookwarm Release wird der Network Manager verwendet:

!!! terminal "Terminal"
    <pre>
    sudo nmcli con show
    sudo nmcli con mod "Wired connection 1" ipv4.addresses "192.168.1.88/24"
    sudo nmcli con mod "Wired connection 1" ipv4.gateway "192.168.1.1"
    sudo nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8,192.168.1.1"
    sudo nmcli con mod "Wired connection 1" ipv4.method manual
    sudo nmcli con mod "Wired connection 1" connection.autoconnect yes
    \# Ohne Reboot aktivieren
    sudo nmcli con up "Wired connection 1"
    </pre>

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
    //falls leer wird der system hostname verwendet
    //also nicht notwendig

    interface eth0 ODER wlan0
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

Geht inzwischen auch über Desktop und über den Imager

<a href="http://www.netzmafia.de/skripten/hardware/RasPi/RasPi_Network.html" target="_blank">http://www.netzmafia.de/skripten/hardware/RasPi/RasPi_Network.html</a>

Die Netzwerkkonfiguration editieren wie in [Kapitel _Statische IP vergeben_](#statische-ip-vergeben):

!!! terminal "Terminal" 
    <pre>
    //Alte Versionen:
    sudo nano /etc/dhcpcd.conf
    /etc/network/interfaces
    </pre>

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

### SSH

##### SSH aktivieren

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

##### SSH ohne Passwortabfrage für Zugriff von außerhalb

Um den Zugriff von außerhalb des lokalen Netzes zu ermöglichen sollte aus Sicherheitsgründen die Passwortabfrage deaktivert werden und der Zugriff ausschließlich über Private und Public Key erfolgen.

Dazu wird auf dem Client Rechner (z.B. Windows Bash) ein Schlüsselpaar erzeugt:
!!! terminal "Terminal"
    <pre>
    \# Unbedingt Passphrase vergeben, da sonst Hacker leichtes Spiel haben!
    ssh-keygen -t ed25519 -f /.ssh/dispi.ed25519 -C "Dispi Schlüssel"
    </pre>

Dabei gibt -t ed25519 ein modernes Verfahren für die Verschlüsselung an, die viel kürzer als die RSA Schlüssel sind.  
Die erzeugte Datei mit einem sprechenden Namen belegen, so dass sie einfach wiedererkannt werden kann.  
Der Kommentar hinter -C ist hilfreich für Tools, die den Public Key anzeigen.

Der Public Key muss nun auf dem Server in ~/.ssh/authorhized_keys eingetragen werden:
!!! terminal "Terminal"
    <pre>
    \# Datei anlegen und Berechtigung setzen
    (umask 077 && test -d ~/.ssh || mkdir ~/.ssh)
    (umask 077 && touch ~/.ssh/authorized_keys)

    \# Kompletten Inhalt von dispi.ed25519.pub (Wichtig pub!!!) in die Datei authorized key in eine eigene Zeile kopieren und speichern
    sudo nano ~/.ssh/authorized_keys

    \# Auf dem Client testen (z.B. Windows Bash), ob Anmeldung nur mit Passphrase funktioniert
    ssh -i ~/.ssh/dispi.ed25519 192.168.1.101
    </pre>

In der Datei /etc/ssh/ssh_config mit sudo nano folgende Einträge auf no setzen:
!!! file "/etc/ssh/ssh_config"
    <pre>
    PasswordAuthentication no
    ChallengeResponseAuthentication no
    </pre>

Danach den SSH Dienst neustarten, damit die Änderungen aktiv werden:
!!! terminal "Terminal"
    <pre>
    systemctl restart ssh
    </pre>

***Achtung!!!*** 
Der private Key muss dann unbedingt gesichert werden, da sonst bei Verlust kein Zugriff mehr auf den Server möglich ist!

Der vereinfachte Zugriff auf den Server kann über Config Dateien erfolgen. Hierzu auf dem Client die Datei ~/.ssh/config anlegen.  
Mehrere Hosts sind möglich!
!!! file "%UserProfile%/.ssh/config"
    <pre>
    Host dispi
        HostName 192.168.1.101
        User pi
        ForwardAgent yes
        IdentityFile ~/.ssh/dispi.ed25519
    </pre>

Hierbei kann man auch mit Platzhaltern arbeiten, wie z.B. *Host \*.local* für alle Server im Netzwerk.

Die SSH Verbindungen können ggf. auch mit einem FIDO2 Schlüssel gesichert werden. Hierzu wird ein Schlüssel mit ed25519-sk erstellt (Suche Internet).

##### SSH ohne Passwortabfrage in VSCode

Dies ist die Variante mit existierenden Keys ohne Deaktivierung der Passwort Eingabe und ohne Passphrase.  
Daher auch weniger Sicherheit!  

Es müssen zuerst auf dem Remote Rechner (Windows) im Homeverzeichnis die Private und Public Key Dateien ~/.ssh/id_rsa und ~/.ssh/id_rsa.pub angelegt werden (sshgen)  
Dann muss auf dem Server das .ssh Verzeichnis mit der Datei authorized_key angelegt werden:

Berechtigung und Anlage auf dem Raspi: siehe vorheriges Kapitel!

In die Datei authorized_keys muss der Inhalt der Datei .ssh/id_rsa.pub kopiert werden.

Bei einem neuen Raspi mit gleicher IP muss auf dem PC die Einträge für die IP aus .ssh/known_hosts gelöscht werden!

### Zeitzone

Geht auch über raspi-config.

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

ACHTUNG!!! Unter Bookworm war am Anfang ein Fehler, dass beim Reboot kein mount durchgeführt wird und immer auch ein Hinweis beim Mountbefehl.  
Hier ist ein Script zur Behebung: https://forums.raspberrypi.com/viewtopic.php?t=362156#p2172794  
Hat nicht richtig funktioniert, bzw. weiss ich nicht genau, ob es was bewirkt hat.  
In fstab habe ich x-systemd.automount rausgeschmissen.

!!! terminal "Terminal"
    <pre>
    sudo mkdir /kino
    mkdir /home/pi/nas
    sudo nano /etc/fstab
    </pre>
!!! file "/etc/hostname"
    <pre>
    \# ACHTUNG!!! Evtl. ohne x-systemd.automount ab Bookworm
    //192.168.1.111/roger    /home/pi/nas    cifs    defaults,user=admin,password=NAS_PASSWORT,gid=pi,uid=pi,x-systemd.automount,x-systemd.requires=network-online.target,rw    0    0
    </pre>
_Anmerkung:_ Die x-systemd Sachen sind notwendig, dass mount beim Booten erst nach Netzwerkverbindung versucht wird.  
*gid* und *uid* sind User und Gruppe für das Verzeichnes bzgl. Zugriff. 

!!! terminal "Terminal: _fstab neu einlesen_"
    <pre>sudo mount -a </pre>

Ist ein Tunnel aufgebaut, dann kann man auch direkt ein Remote NAS mounten:

!!! terminal "Terminal"
    <pre>
    sudo mount -t cifs -o username=admin,password=[Password],uid=pi,gid=pi //192.168.2.33/Mitarbeiterdokumente /home/pi/nas_praxis
    </pre>

### SendMail über CLI

msmtp unterstütz im Vergleich zu ssmtp über mehr Sicherheit und ermöglicht den Sender zu definieren.

!!! terminal "Terminal"
    <pre>
    sudo apt-get install msmtp msmtp-mta mailutils
    sudo nano /etc/msmtprc
    \# Content siehe unten
    sudo chown root:msmtp /etc/msmtprc
    sudo chmod 640 /etc/msmtprc
    </pre>

??? file "/etc/msmtprc"
    <pre>
    \# A system wide configuration file is optional.
    \# If it exists, it usually defines a default account.
    \# This allows msmtp to be used like /usr/sbin/sendmail.
    account roger@dieknops.de

    \# The SMTP smarthost
    host mail.dieknops.de

    set_from_header on

    \# Use TLS on port 587
    port 587
    tls on
    tls_starttls on
    tls_certcheck off

    from roger@dieknops.de
    auth on
    user roger@dieknops.de
    password <password\>

    \# Set a default account
    account default : roger@dieknops.de

    \# Map local users to mail addresses (for crontab)
    aliases /etc/aliases
    </pre>    

Das Versenden einer Mail über CLI erfolgt dann über:

!!! terminal "Terminal"
    <pre>
    echo "Hello world email body!" | mail -s "Test Subject" roger.knop@sap.com
    </pre>

Um eine Datei zu versenden ist zusätzlich mpack erforderlich:
!!! terminal "Terminal"
    <pre>
    sudo apt-get install mpack
    \# Versenden mit
    sudo mpack -s "Test Subject" /etc/msmtprc roger.knop@sap.com
    </pre>

### OneDrive

Es ist möglich (auch ohne Desktop) OneDrive auf dem Raspi zu installieren.  
Alle Infos in GitHub unter <a href="https://github.com/abraunegg/onedrive" target="_blank">https://github.com/abraunegg/onedrive</a>

***Achtung!*** Den URL Login nicht auf dem Firmenrechner und nicht auf dem IPad ausführen, sondern in einem anderen PC im Incognito Modus!

!!! terminal "Terminal"
    <pre>
    sudo apt install onedrive
    onedrive
    \# URL in einen Browser kopieren, anmelden und dann die URL der leeren Seite als Response einfügen

    \# Verzeichnisse definieren, die gesynct werden sollen: z.B. RaspiShare
    \# Im Online muss unter geteilt über die 3 Punkte das Verzeichnis in eigene Dateien aufgenommen werden
    nano .config/onedrive/sync_list
    </pre>

Vorgehensweise zum Austausch sollte sein:

* Ein Verzeichnis in sync_list eintragen: z.B. RaspiShare (dran denken über Online OneDrive das Verzeichnis in eigene Dateien aufnehmen)
* Sync ausführen: onedrive --synchronize
* Änderungen durchführen und ggf. wieder Sync ausführen 

CLI Options:

* --synchronize --verbose --dry-run => Es werden keine Dateien kopiert oder gelöscht - nur erweiterte (verbose) Anzeige
* --synchronize => Normaler Sync
* --display-config => Konfiguration - Kann überschrieben werden
* --resync => Sync, wenn keine Änderung war
* --resync => Sync, wenn keine Änderung war

Den Service kann man einplanen, aber aus Performancegründen würde ich empfehlen nur bei Bedarf zu synchen und auch nur ein Verzeichnis!

!!! terminal "Terminal"
    <pre>
    \# Als Service - NICHT NOTWENDIG - Performance??? - Nur, wenn wirklich notwendig
    systemctl enable onedrive@pi.service
    systemctl start onedrive@pi.service
    \# Logs
    journalctl --unit=onedrive@pi -f
    \# Deaktivieren mit stop und disable
    </pre>

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
    curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
    //Update Package Manager auf eine spezifische Node Version
    curl -fsSL https://deb.nodesource.com/setup_15.x | sudo -E bash -
    sudo apt-get install -y nodejs
    </pre>

NPM wird automatisch installiert zusammen mit den neuen NodeJS Versionen.  
*Achtung* node -v kann erst ausgeführt werden, wenn ein neues Terminal geöffnet wird (PATH).

### Autostart eines NodeJS Scripts

Autostart eines eigenen NodeJS Web Servers über das Tool pm2.  
Startet nach Neustart und nach kill -9 erfolgt ein Restart des Scripts.  

***ACHTUNG!!!*** Damit pm2 beim Booten ein Netzwerk hat muss in raspi-config unter System Options -> Network at Boot YES ausgewählt werden!

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

    //Wichtig bei Erstinstallation und pm2 ohne sudo nutzen
    sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u pi --hp /home/pi --wait-ip
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

  * Mit sudo davor ist eine andere Liste! Für fancontrol.js siehe sudo pm2 l 
  * pm2 l => listet aktive nodejs jobs mit id
  * pm2 stop [id] => stoppt aktiven job mit der id
  * pm2 start [id] => startet aktiven job mit der id
  * pm2 start test.js --cron "*/15 * * * *" --no-autorestart => Startet node test.js alle 15 Minuten / no-autorestart ist WICHTIG!!!
  * pm2 startup, start und save siehe Internet mit Logging OneNote
  * pm2 save ist wichtig! Sonst werden keine Änderungen (z.B. del usw.) übernommen
  * Ventilator muss über sudo gestartet werden, da gpio sudo Rechte benötigt

Mit Log bei Problemen: pm2 start --log /smartdisplay/pmlog.txt server/index.js -- 8000

***Achtung!*** Ports unter 1024 gehen nicht

Ports für lokale Server unter 1024 aktivieren: sudo setcap 'cap_net_bind_service=+ep' \`which node\` (ACHTUNG Backslash nur für md Anzeige)

Debugging:

  * pm2 del 0 => Webserver Port 8000 löschen
  * npm run debug => Webserver im Debug Mode auf 3000 öffnen
  * Browser auf Dispi (VNC) öffnen
  * F12 und grünes NodeJS Icon klicken
  * Methode this.globals.waitForDebugSession() verwenden, um auf den Debugger zu warten

### RSYNC Synchronisation

Über ssh kann man rsync auch nutzen, was hilfreich ist, wenn man einen VPN Tunnel aufbaut.
rsync -avzhe ssh backup.tar.gz root@192.168.0.141:/backups/

Der Fortschritt bei rsync wird über --info=progress2 angezeigt.  
Hierbei zeigt ir-chk=1029/30621 den Fortschritt des Incremental Scans an. 1029 von 30621 Dateien sind noch zu prüfen. 
Am Anfang steigen beide Zahlen stetig.
Am Ende ändert sich die Anzeige in to-chk, wenn keine Incremental Scans mehr kommen. Dann nimmt nur noch die erste Zahl ab.

#### Vorbereitung

1. Tunnel konfigurieren
2. mkdir /home/pi/nas_praxis
3. Lokales nas Backup Folder mounten unter /home/pi/remote_backups

!!! file "/etc/fstab"
    <pre>
    \# ACHTUNG!!! Evtl. ohne x-systemd.automount ab Bookworm
    //192.168.1.111/remote_backups    /home/pi/remote_backups    cifs    defaults,user=admin,password=[Password],gid=pi,uid=pi,x-systemd.automount,x-systemd.requires=network-online.target,rw    0   >
    </pre>

#### Script Erstellung

!!! file "~/backup_scripts/backup_praxis_mitarbeiterdocs.sh"
    <pre>
    \#!/bin/bash

    sudo umount /home/pi/nas_praxis

    echo "***********************************************************"
    echo "BACKUP Praxis Admin"

    \#VPN Tunnel aufbauen
    echo ""
    echo "-----> Tunnel aufbauen"
    wg-quick up vpn_praxis

    \#Remote NAS mounten
    echo ""
    echo "-----> Remote NAS mounten"
    sudo mount -t cifs -o username=admin,password=,uid=pi,gid=pi //192.168.2.33/Mitarbeiterdokumente /home/pi/nas_praxis

    \#Remote Verzeichnis spiegeln
    echo ""
    echo "-----> Remote Verzeichnis spiegeln"
    rsync -zar --delete --info=progress2 --stats /home/pi/nas_praxis/ /home/pi/remote_backups/praxis/mitarbeiterdokumente

    \#Remote NAS unmounten
    echo ""
    echo "-----> Remote NAS unmounten"
    sleep 20
    sudo umount /home/pi/nas_praxis

    \#VPN Tunnel schliessen
    echo ""
    echo "-----> Tunnel schliessen"
    wg-quick down vpn_praxis

    echo ""
    echo "BACKUP ENDE"
    echo "***********************************************************"
    </pre>

Mit chmod +x ~/backup_scripts/backup_praxis_mitarbeiterdocs.sh ausführbar machen!  
Und dann die Scripte nach Wunsch im cron einplanen.

Zum Beispiel die Praxis Sicherung jeden Freitag 22 Uhr mit einem separaten Logfile mit Timestamp im Format 2023-12-29_22:00

!!! file "crontab -e"
    <pre>
    \# Praxis Mitarbeiterdokumente Sicherung: Jeden Freitag 22 Uhr
    00 22 * * 5 /bin/bash /home/pi/backup_script/backup_praxis_mitarbeiterdocs.sh > /home/pi/backup_script/logs/praxis_backup_\`date +\%Y-\%m-\%d_\%H:\%M\`.log 2>&1
    \#
    \# Helmi Bidirect Sicherung: Jeden 1. Dienstag im Monat um 22 Uhr
    \#0 22 1-7 * 2 /bin/bash /home/pi/backup_script/backup_helmi_bidirect.sh > /home/pi/backup_script/logs/helmi_backup_\`date +\%Y-\%m-\%d_\%H:\%M\`.log 2>&1
    \#
    \# Helmi Bidirect Sicherung: Alle 2 Wochen am Dienstag um 22 Uhr
    0 22 * * Tue [ $(expr $(date +%W) % 2) -eq 1 ] && /bin/bash /home/pi/backup_script/backup_helmi_bidirect.sh > /home/pi/backup_script/logs/helmi_backup_\`date +\%Y-\%m-\%d_\%H:\%M\`.log 2>&1
    </pre>

### Bash History Search 

Es ist besser das Tool hstr zu installieren mit Alias hh.  
Über die Pfeiltasten auswählen und mit TAB übernehmen zum Ändern, oder mit ENTER um direkt auszuführen.

!!! terminal "Terminal"
    <pre>
    sudo apt-get install hstr
    \# Konfiguration in .bashrc übernehmen (z.B. Alias hh)
    hstr --show-configuration >> ~/.bashrc
    \# Neue Bash Shell öffnen
    hh [Suchstring]
    </pre>

Default mäßig kommt eine Suche mit.  
Damit die Forwardsuche funktioniert vorher ```stty -ixon``` eingeben!

!!! terminal "Terminal"
    <pre>
    CTRL-r
    Suchstring
    CTRL-r weiter zurück suchen
    \# Wieder forward Suche
    CTRL-s
    </pre>

### Hilfreiche Befehle

| Befehl | Beschreibung  |
| --- | --- |
| which | Pfad eines Programms ermitteln |
| chmod -R u+w <file/path> | Datei/Pfad User darf auch schreiben (oder 755 Muster) / -R recursive |
| chown -R usr:grp <file/path> | Datei/Pfad Owner User usr und Gruppe grp setzen |
| usermod -aG grp usr | Fügt den User usr zur Gruppe grp hinzu |
| gpasswd -d usr grp | Löscht den User usr von der Gruppe grp |
| df -h | Speicherplatz auf den Laufwerken anzeigen |
| sudo du -xh / &#124; grep -P "G\t" | Speicherplatz der größten Folder |
| ncdu /folder | Treesize für Folder |
| ls -al /folder --block-size=M oder G | Folder in MB oder GB anzeigen |
| history \| grep [Suchbegriff] | History nach bestimmten Wort durchsuchen |

### SD Karte vergrößern

* Win32 Diskimager Tool verwenden und von SD Karte Image auf PC erstellen
* Neue größer Karte mit dem Image erstellen
* Raspi mit der neuen Karten booten und prüfen. Achtung! df liefert noch die alte Größe
* Partition erweitern mit ```sudo raspi-config --expand-rootfs``` 

### Backup Image erstellen

Als Voraussetzung muss das NAS gemounted sein. 

Zuerst muss die Partition ermittelt werden:

!!! terminal "Terminal"
    <pre>
    sudo fdisk -l
    sudo fdisk /dev/mmbcblk0p2
    Key p => Listet die Partitionen (siehe ganz oben)
    Key q => Beenden
    </pre>

Dann kann mit der Partition die Image Erstellung getriggert werden (if=Partition / of=Pfad&File des zu erstellenden Images):

!!! terminal "Terminal"
    <pre>sudo dd if=/dev/mmcblk0p2 of=/nas/smartdisplay_backup_sdcard/raspi_image.img bs=1M</pre>

Dies kann über eine Stunde dauern!

### Chromium installieren bzw. deinstallieren

Chromium Browser installieren bzw. deinstallieren.

!!! terminal "Terminal - Chromium installieren"
    <pre>sudo apt install chromium-browser -y</pre>

!!! terminal "Terminal - Chromium deinstallieren"
    <pre>
    sudo apt-get remove chromium-browser
    sudo dpkg -r chromium-browser
    sudo dpkg --purge chromium-browser
    sudo rm -Rf /etc/chromium-browser
    sudo rm -Rf ~/.config/chromium
    </pre>

### Editor nano installieren

Dieser Terminal Editor ist einfacher zu bedienen, als der VI, aber trotzdem noch relativ rudimentär.

!!! terminal "Terminal"
    <pre>sudo apt-get install nano</pre>

Für ALT Taste in Kitty/Putty das Keyboard unter "Terminal - Keyboard - Function Keys" auf "xterm*" ändern.

Nano Cheatssheet: <a href="https://www.nano-editor.org/dist/latest/cheatsheet.html" target="_blank">https://www.nano-editor.org/dist/latest/cheatsheet.html</a>


File handling

* Ctrl+S  Save current file
* Ctrl+O	Offer to write file ("Save as")
* Ctrl+R	Insert a file into current one
* Ctrl+X	Close buffer, exit from nano

Editing

* Ctrl+K   	Cut current line into cutbuffer
* Alt+6	Copy current line into cutbuffer
* Ctrl+U	Paste contents of cutbuffer
* Alt+T	Cut until end of buffer
* Ctrl+]	Complete current word
* Alt+3	Comment/uncomment line/region
* Alt+U	Undo last action
* Alt+E	Redo last undone action

Search and replace

* Ctrl+Q   	Start backward search
* Ctrl+W	Start forward search
* Alt+Q	Find next occurrence backward
* Alt+W	Find next occurrence forward
* Alt+R	Start a replacing session

Deletion

* Ctrl+H	Delete character before cursor      
* Ctrl+D	Delete character under cursor
* Alt+Bsp	Delete word to the left
* Ctrl+Del   	Delete word to the right
* Alt+Del	Delete current line

Operations

* Ctrl+T   	Execute some command
* Ctrl+J	Justify paragraph or region
* Alt+J	Justify entire buffer
* Alt+B	Run a syntax check
* Alt+F	Run a formatter/fixer/arranger
* Alt+:	Start/stop recording of macro
* Alt+;	Replay macro

Moving around

* Ctrl+B   	One character backward
* Ctrl+F	One character forward
* Ctrl+←	One word backward
* Ctrl+→	One word forward
* Ctrl+A	To start of line
* Ctrl+E	To end of line
* Ctrl+P	One line up
* Ctrl+N	One line down
* Ctrl+↑	To previous block
* Ctrl+↓	To next block
* Ctrl+Y	One page up
* Ctrl+V	One page down
* Alt+\	To top of buffer
* Alt+/	To end of buffer

Special movement

* Alt+G    	Go to specified line
* Alt+]	Go to complementary bracket
* Alt+↑	Scroll viewport up
* Alt+↓	Scroll viewport down
* Alt+<	Switch to preceding buffer
* Alt+>	Switch to succeeding buffer

Information

* Ctrl+C   	Report cursor position
* Alt+D	Report line/word/character count
* Ctrl+G	Display help text

Various

* Alt+A	Turn the mark on/off
* Tab	Indent marked region
* Shift+Tab   	Unindent marked region
* Alt+V	Enter next keystroke verbatim
* Alt+N	Turn line numbers on/off
* Alt+P	Turn visible whitespace on/off
* Alt+X	Hide or unhide the help lines
* Ctrl+L	Refresh the screen

### Deutsche Tastatur im Terminal

Ohne Desktop kann man auch im Terminal die deutsche Tastatur einrichten.

!!! terminal "Terminal"
    <pre>
    sudo dpkg-reconfigure keyboard-configuration
    </pre>
Dann "_Generische Tastatur mit 105 Tasten (Intl)_" auswählen.
Anschließend "_German_" und dann alles bestätigen.
