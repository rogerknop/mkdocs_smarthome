# WSL - Windows Subsystem Linux

## Installation und Start

* wsl --set-default docker_data => wichtig damit Docker Desktop noch geht
* wsl install -d Ubuntu-22.04 => Installiert ein System
* wsl -l => Zeigt alle installierten Distributionen
* wsl --list --online => Zeigt die Online verfügbaren Distributionen
* wsl -d Ubuntu-22.04 => Login in Distribution Ubuntu

Unter Windows Start Ubuntu suchen und an Start pinnen.

Damit wsl beim Schliessen des Fensters nicht runterfährt und auch direkt mit Putty verwendet werden kann, muss folgender Befehlt in die Autostart:   
wsl -d Ubuntu-22.04 --exec dbus-launch true

## SSH

* sudo service ssh status => Prüfen, ob ssh installiert
* sudo apt install openssh-server
* sudo service ssh start => Einmalig starten
* sudo systemctl enable ssh => Autostart Service

Falls der Service nicht läuft: sudo systemctl status

* wsl --update
* sudo -e /etc/wsl.conf  
  Add the following:  
  [boot]  
  systemd=true
* wsl --shutdown
* wsl neu starten
* Status erneut testen: sudo systemctl status

## Sonstige Installationen

### Git

Damit git in WSL sich die access tokens merken kann:
!!! terminal "Terminal"
    <pre>
    git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
    </pre>

## Ansible - Admin Automatisierung

