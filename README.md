# Voraussetzungen 
NodeJs und npm und Powerpoint oder Libreoffice müssen installiert sein.

# Setup / Installation
Die Dokumentation wurde mit MkDocs erstellt: https://www.mkdocs.org/  
Dieses Framework erstellt Internetseite auf Basis von Konfiguration und Markdown Files.

Python & pip Installieren: https://www.python.org/downloads/  
Prüfen, ob Phython im Path steht: python --version & pip --version

Hier die Schritte für die Installation. Der letzte git Befehl bewirkt, dass die lokalen Einstellungen nicht gepusht werden
WICHTIG! Shell muss als Admin ausgeführt werden:
```
pip install --upgrade pip
pip install mkdocs
pip install mkdocs-material
pip install mkdocs-material-extensions  //-----> Icons Übersicht https://petershaggynoble.github.io/MDI-Sandbox/
pip install pymdown-extensions  //-----> Extensions e.g. Strikethrough, Latex Formeln usw.
pip install toc //-----> Creates Permalinks or Anchors for header
pip install mkdocs-drawio-file
git clone https://github.com/rogerknop/mkdocs_smarthome.git
npm install
git update-index assume-unchanged config/local.json
```

Auf neuem Rechner .ssh/id_rsa und .pub erstellen.
Inhalt von .pub auf FTP Server in .ssh/authorized_keys am Ende in neuer Zeile ergänzen. 

# Konfiguration
Die Default Einstellungen sind in config/default.json definiert.  
Bei Bedarf können einzelne Attribute in der Datei config/local.json überschrieben werden.

# User htpasswd

https://www.ruhr-uni-bochum.de/system/tools/htpasswd
GEHT NICHT! - https://htpasswdgenerator.de/

Passwort generieren und in docs/install/htpasswd eintragen.

# Python Probleme
* Falls Berechtigungsfehler: pip Befehle um --user erweitern
* Gegebenenfalls Verzeichnis site-packages in %appdata%\Python\Python37\ löschen

# Verfügbare npm scripts
```
npm start oder npm run serve => Localhost wird gestartet
npm run build => site Folder generieren
npm run deploy => Build und Deploy
npm run convert => Powerpoint Slides in JPG exportieren
npm run convertdeploy => Volles Programm: Convert, Build und Deploy
```

# Slides in Bilder konvertieren - IN ZUKUNFT NUR NOCH DrawIO!
Es sollte pro PPTX nur ein Slide existieren, damit beide Konvertierungsvarianten verwendet werden können.

## PPTX convert
Es werden all Slides aus den in oriDocs gefundenen PPTX mit einem PowerShell Script exportiert.  
Evtl. muss im Windows die Berechtigung für das Ausführen von PowerShell Scripts erlaubt werden:  
* set-executionpolicy remotesigned - nur Remote muss signiert sein (besser und sicherer)
* set-executionpolicy Unrestricted - ALLE dürfen ausführen. Sehr unsicher!

## LibreOffice convert
LibreOffice installieren und Pfad im Script convert.js prüfen.  
Mit LibreOffice kann immer nur ein Slide kopiert werden und in deploy.js muss entsprechend der Funktionsaufruf angepasst werden.