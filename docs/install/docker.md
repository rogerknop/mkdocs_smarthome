# Docker

## Installation 

!!! terminal "Terminal - Docker Installation"
    <pre>
    \# Install Script installieren
    curl -fsSL https://get.docker.com -o get-docker.sh
    \# You can run the script with the --dry-run option to learn what steps the script will run when invoked
    sudo sh ./get-docker.sh --dry-run
    \# Install docker
    sudo sh ./get-docker.sh
    \# pi User der Docker Gruppe hinzufügen
    sudo usermod -aG docker pi
    \# Reboot Raspi
    sudo reboot now
    \# Testen, ob Dienst aktiviert
    docker run hello-world
    \# Ansonsten aktivieren
    sudo systemctl enable docker
    </pre>

Das Programm docker compose wird nun über docker compose aufgerufen!

## Kommandos

* docker compose up --build -d => Docker Container bauen und mit -d im Hintergrund starten
* docker compose down => Docker Container beenden
* docker ps => Listet alle laufenden Container mit ID und Namen
* docker exec -it docker-praxman_db_1  bash => Öffnet Terminal im Container
* docker images => listet alle verfügbaren images
* docker container list => Listet alle Container
* docker volume list => Listet alle Volumes
* docker rm [volume|container|image] -f (=force) id => Löscht entsprechendes Objekt
* docker compose down --rmi all => Löscht auch den Cache!

## Docker Lösungen

### Portainer

!!! terminal "Terminal"
    <pre>
    docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
    </pre>

Beim ersten Start User anlegen (Passwort muss mindestens 12 Zeichen lang sein): pi / temppassword  

Dann in den Settings:

* Session Lifetime: 1 year
* Password rule: 6 characters

Im Account oben rechts das Passwort ändern, so dass Raspi Login und Portainer Login identisch.

Settings URL anpassen: Environment -> local -> Public IP: 192.168.x.x

### Paperless

#### Installation

https://docs.paperless-ngx.com/setup/

Auf der Setup Seite wird PostgeSQL empfohlen.

***ACHTUNG!!!*** Wenn auf dem NAS schon ein paperless Verzeichnis existiert, dann dieses in paperless-old umbenennen, um später den export Folder von dort verwenden für den Import.

Setup Script in /home/pi ausführen:

!!! terminal "Terminal"
    <pre>
    bash -c "$(curl --location --silent --show-error https://raw.githubusercontent.com/paperless-ngx/paperless-ngx/main/install-paperless-ngx.sh)"
    </pre>

Default Werte nutzen im Setup wie z.B. DB (Postgres) - ausser:

* OCR language: deu
* User ID: unverändert auf 1000
* User Group: unverändert auf 1000
* Folder
    * Consume: /home/pi/nas/paperless/consume
    * Media: /home/pi/nas/paperless/media
    * Data: /home/pi/nas/paperless/data
    * DB: ENTER Default - nix ändern wegen Berechtigungsproblemen
* User: pi
* Passowort: [RaspiPasswort]

Falls Probleme mit Sonderzeichen im Public Key, dann Setup erneut ausführen.

#### Anpassungen

Erst nach den Anpassungen den Docker starten!

##### Folder Struktur

!!! terminal "Terminal"
    <pre>
    docker compose down
    \# ---------------------------------
    docker compose.yml anpassen:
    \# Volume Export auf NAS umstellen & Startup Script für ZXING Aktivierung Workaround (siehe Extra Kapitel)
    volumes:
        - /home/pi/nas/paperless/export:/usr/src/paperless/export
        - /home/pi/paperless-ngx/scripts:/custom-cont-init.d:ro
    \# ---------------------------------
    docker compose.env anpassen:
    \# Falls consume Folder auf NAS muss Polling aktiviert werden (alle 60 Sek.)
    PAPERLESS_CONSUMER_POLLING=60
    \# Ordnerstruktur, Barcode ASN und Memory Einstellungen
    PAPERLESS_FILENAME_FORMAT: "{created_year}/{correspondent}/{created_year}-{created_month}-{created_day}-{asn}-{title}"
    PAPERLESS_CONSUMER_ENABLE_BARCODES=true # enable search for barcodes
    PAPERLESS_CONSUMER_ENABLE_ASN_BARCODE=true # enable setting ASN by ASN barcodes
    \# Scanner Software für Barcodes, die auch die kleinen QR Codes der Aufkleber kann
    PAPERLESS_CONSUMER_BARCODE_SCANNER=ZXING
    \# Ab hier nicht notwendig - war zu Performance Tests
    PAPERLESS_TASK_WORKERS=2
    PAPERLESS_THREADS_PER_WORKER=1
    PAPERLESS_WEBSERVER_WORKERS=1
    \# ---------------------------------
    </pre>

Noch NICHT den Docker starten!

Falls Filename Format geändert wurde - bestehenden Datenbestand nach den Anpassungen und Docker Start anpassen:

!!! terminal "Terminal"
    <pre>
    docker compose -f /home/pi/paperless-ngx/docker compose.yml exec -T webserver document_renamer
    </pre>

***Achtung!!!*** Erst den folgenden Workaround für ZXING einbauen und dann erst starten mit:
!!! terminal "Terminal"
    <pre>
    docker compose up --build -d
    </pre>

##### Workaround ZXING Scanner

ZXING Scanner ermöglicht die Nutzung der kleinen Aufkleber QR Codes für die ASN.  
Der Workaround verlängert den Start Prozess erheblich!  
Das if im Script bewirkt leider nichts. Es muss bei jedem start installiert werden.

!!! terminal "Terminal"
    <pre>
    cd /home/pi/paperless-ngx
    mkdir scripts
    sudo chown -R root:root scripts
    sudo nano startup_install_zxing.sh
    </pre>

!!! file "scripts/startup_install_zxing.sh"
    <pre>
    \#!/bin/bash
    if pip show zxing-cpp > /dev/null 2>&1; then
        echo "Paket ZXING ist bereits installiert!"
    else
        echo "Paket ZXING wird installiert..."
        apt-get update
        apt-get install -y build-essential libssl-dev cmake
        \# Installiere zxing-cpp mit pip - DAUERT SEHR LANGE!
        pip install zxing-cpp
        echo "Paket ZXING ist nun installiert. Weiter geht's..."
    fi
    </pre>

#### Backup

Verzeichnis ../export ist auf dem NAS unter (nas/paperless/export).  
Dadurch entsteht auch eine Sicherung auf Helmis NAS.

!!! terminal "Terminal"
    <pre>
    \# Manuell
    docker compose exec -T webserver document_exporter ../export
    \# Cron Job 
    \# Jeden Montag Dienstag Morgen 3 Uhr
    0 3 * * Tue docker compose -f /home/pi/paperless-ngx/docker compose.yml exec -T webserver document_exporter ../export/
    </pre>

#### Ablauf

* App: QuickScan mit WebDAV - Evtl. reicht auch die Paperless App
* App: Paperless für Apple und Android
* Inbox Dokumente Prüfen (z.B. Typ, Korrespondent, Datum, Tags) - Am Ende Inbox Tag entfernen
* Alte Docs aufräumen, über Dokumente Filter: ASN (leer/nicht leer), Jahr vor n, Alle auswählen, Löschen

#### QR Codes und ASN

* AVERY Zweckform L4731
* Online Creator: https://tobiasmaier.info/asn-qr-code-label-generator/
* CLI Generator installieren: pip install paperless-asn-qr-codes
* CLI Generator verwenden:
    * paperless-asn-qr-codes -h => Parameter StartASN und Output pdf
    * paperless-asn-qr-codes 1 paperless.pdf
    ACHTUNG! Beim PDF Druck darauf achten, dass keine Größenanpassung durchgeführt wird. Also "Actual Size" verwenden.

Evtl. Integeration in Paperless: https://github.com/paperless-ngx/paperless-ngx/discussions/4588

### Kopia

![](../drawio/backup.drawio)

Verzeichnisse in /home/pi anlegen (nas=NAS Mount & tmp lokal auf dem Raspi):

* nas/kopia/config
* nas/kopia/cache
* nas/kopia/logs
* nas/kopia/backupdata
* tmp/snapshot

In backupdata stehen letztendlich die Backups!

!!! file "/home/pi/nas/kopia/docker compose.yml"
    <pre>
    version: '3.7'
    services:
    kopia:
        image: 'kopia/kopia:latest'
        container_name: kopia
        hostname: kopiapi
        restart: unless-stopped
        ports:
            - 51515:51515
        command:
            - server
            - start
            - --disable-csrf-token-checks
            - --insecure
            - --address=0.0.0.0:51515
            - --server-username=pi
            - --server-password=[RASPI PASSWORT]
        environment:
            # Set repository password
            KOPIA_PASSWORD: "[RASPI PASSWORT]"
            USER: "pi"
            TZ: "Europe/Berlin"
        volumes:
        # ! Base mounts
        - /home/pi/nas/kopia/config:/app/config
        - /home/pi/nas/kopia/cache:/app/cache
        - /home/pi/nas/kopia/logs:/app/logs
        # ! Backup Target mount
        - /home/pi/nas/kopia/backupdata:/data
        # ! Backup Source mounts
        - /home/pi/nas:/nas
        - /home/pi/remote_backups:/remote_backups
    </pre>

!!! terminal "Terminal"
    <pre>
    docker compose up --build
    </pre>

Repository: 

* Folder: /data
* Advanced User: pi
* Repo Passwort = [RASPI PASSWORT]
* Name: Rogis Backup

Sicherung erstellen => Policies mit Folder und Button "Set Policy":

* /remote_backups/praxis/mitarbeiterdokumente
* /nas/Sicherung/wiebke/Themen

Einplanung Job => Scheduling - Cron Expressions: 0 6 * * 4 #Jeden Donnerstag 6 Uhr

Mit den folgenden Retentions (wieviel aufheben): 

* Latest: 5
* Monthly: 6
* Annual: 2
* Rest: 0


## PraxMan (lokal)

* Praxman Docker readme.md in git
* Nach docker compose DB in Adminer neu anlegen und import dump.sql ausführen
* DB Import und Export geht auch über Seite!
* DB Docker User und PW: root
