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
    </pre>

!!! terminal "Terminal - Docker-Compose Installation"
    <pre>
    \# Python3 mit Abhängigkeiten
    sudo apt install libffi-dev libssl-dev python3-dev python3 python3-pip
    \# Docker-compose
    sudo pip3 install docker-compose

    \# Reboot Raspi
    reboot raspi now

    \# Testen, ob Dienst aktiviert
    docker run hello-world
    \# Ansonsten aktivieren
    sudo systemctl enable docker
    </pre>

## Kommandos

* docker-compose up --build => Docker Container bauen und starten
* docker-compose down => Docker Container beenden
* docker ps => Listet alle laufenden Container mit ID und Namen
* docker exec -it docker-praxman_db_1  bash => Öffnet Terminal im Container
* docker images => listet alle verfügbaren images
* docker container list => Listet alle Container
* docker volume list => Listet alle Volumes
* docker rm [volume|container|image] -f (=force) id => Löscht entsprechendes Objekt

## Docker Lösungen

### Portainer

!!! terminal "Terminal"
    <pre>
    docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
    </pre>

Beim ersten Start User anlegen (Passwort muss mindestens 12 Zeichen lang sein): pi / temppassword  

Dann in den Settings:

* Session Liftime: 1 year
* Password rule: 6 characters

Im Account oben rechts das Passwort ändern, so dass Raspi Login und Portainer Login identisch.

Settings URL anpassen: Environment -> local -> Public IP: 192.168.x.x

### Paperless

#### Installation

https://docs.paperless-ngx.com/setup/

Auf der Setup Seite wird PostgeSQL empfohlen.

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

##### Folder Struktur

!!! terminal "Terminal"
    <pre>
    docker compose down
    docker-compose.yml anpassen:
    \# Volume Export auf NAS umstellen
    volumes:
        - /home/pi/nas/paperless/export:/usr/src/paperless/export
    \# Verzeichnisstruktur und Dateinamen anpassen
    environment:
        PAPERLESS_FILENAME_FORMAT: "{created_year}/{correspondent}/{created_year}-{created_month}-{created_day}-{asn}-{title}"
    docker compose up -d
    </pre>

Falls Filename Format geändert wurde - bestehenden Datenbestand anpassen:

!!! terminal "Terminal"
    <pre>
    docker compose -f /home/pi/paperless-ngx/docker-compose.yml exec -T webserver document_renamer
    </pre>

##### Inbox Tag erstellen: ToDo

##### User Anzeigename und Mail von pi anpassen

#### Backup

Verzeichnis ../export ist auf dem NAS unter (nas/paperless/export).  
Dadurch entsteht auch eine Sicherung auf Helmis NAS.

!!! terminal "Terminal"
    <pre>
    \# Manuell
    docker compose exec -T webserver document_exporter ../export
    \# Cron Job 
    \# Jeden Montag Dienstag Morgen 3 Uhr
    0 3 * * Tue docker compose -f /home/pi/paperless-ngx/docker-compose.yml exec -T webserver document_exporter ../export/
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

Evtl. Integeration in Paperless: https://github.com/paperless-ngx/paperless-ngx/discussions/4588

#### ToDo

* Export / Import prüfen
* Alle Dokumente eines Jahres löschen ohne ASN - geht das?

### Kopia

![](../drawio/backup.drawio)

Verzeichnisse in /home/pi anlegen (nas=NAS Mount & tmp lokal auf dem Raspi):

* nas/kopia/config
* nas/kopia/cache
* nas/kopia/logs
* nas/kopia/backupdata
* tmp/snapshot

In backupdata stehen letztendlich die Backups!

!!! file "/home/pi/nas/kopia/docker-compose.yml"
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
