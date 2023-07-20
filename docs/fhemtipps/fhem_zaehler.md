# FHEM Gaszähler und Wasserzähler

Die mechanischen Zähler können per Kamera ausgelesen werden. Sogar mit den kleinen roten Zeigern für die Nachkommastellen.
Passende Gehäuse gibt es bei Thingiverse.

### ESP Hardware & Software installieren

* ESP32 mit Kamera OV2640
* Klebestelle an der Linse wegkratzen und mit 2 Zangen die Linse um 1/4 Drehung gegen den Uhrzeigersinn (am Anfang kurz schwer bis der Leim sich komplett löst)
* Links: 
    * Hauptsseite: <a href="https://github.com/jomjol/AI-on-the-edge-device" target="_blank">https://github.com/jomjol/AI-on-the-edge-device</a>
    * Installation: <a href="https://jomjol.github.io/AI-on-the-edge-device-docs/Installation/" target="_blank">https://jomjol.github.io/AI-on-the-edge-device-docs/Installation/</a> 
    * Firmware: <a href="https://github.com/jomjol/AI-on-the-edge-device/releases" target="_blank">https://github.com/jomjol/AI-on-the-edge-device/releases</a>  
    * Update über Menü und vorher zip runterladen: System -> OTA Update
* Firmware am besten direkt über Web Installer installieren. Danach die Folgeschritte für SD Card durchführen.
* SD Card (2GB) auf FAT32 formatieren
* Folder sd-card: Dateien auf Root der SD Card kopieren
* wlan.ini editieren und WLAN Informationen eintragen: ssid, WLAN Passwort, IP, Gateway, Netmask
* Karte in den ESP und Reboot

### Setup

* Dem Setup Prozess auf der Hauptseite folgen
* Neues Referenzbild anlegen. Prüfen, ob Bild scharf - sonst Linse justieren.
* Alignment Marks: Hier ist es wichtig einen statischen Bereich des Bildes zu definieren. Anhand dessen wird vor der Erkennung das Bild positioniert
* Bereiche für die Werte festlegen. Gegebenenfalls auch Dezimalstellen angeben (z.B. -2)
* In der Config MaxRateValue mit 1 angeben. Wenn zu klein kommen immer Fehlermeldungen, dass die letzte Änderungen zu hoch sei.
* MQTT URL und Port eintragen und mit einem Monitor Tool am Ende prüfen, ob was ankommt. 