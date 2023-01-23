# Installation Grandstream SIP Telefonanlage HT818

### Fragen

* Türstelle 2 SIP Nebenstellen? Tür und Türstelle
* Mehrere Anschlüsse an einer Nebenstelle? Da maximal nur 8 SIP in Fritzbox

### Allgemein

Es gibt bei Amazon eine Rezension mit Infos: <a href="https://www.amazon.de/gp/customer-reviews/R2N6H554EPYHSZ/ref=cm_cr_dp_d_rvw_ttl?ie=UTF8&ASIN=B07B6TL7N6" target="_blank">Rezension G. Hopf</a>  
Weiterhin eine Internetseite mit Details von Steffen Winkler: <a href="https://www.steffen-winkler.de/voip.html" target="_blank">Steffen Winklers HT8xx Tipps</a>

### Installations Schritte

Bei Änderungen im WebUI rechtzeitig auf Apply klicken, sonst sind die Änderungen wegen Timeout weg. Ausserdem bevor man den Tab wechselt.

* *ACHTUNG!!!* Netzwerk nicht an den WAN Port sondern an den rechten Port mit dem Internet Icon.
* Im Router nach der IP suchen und öffnen
* Standard User admin / admin
* Basic Settings:
  * DHCP hostname: HT818 (danach evtl. im Router das Device zurücksetzen, damit nach Neuanmeldung der Name gezogen wird)
  * Time Zone GMT+1 Berlin auswählen
* Advanced Settings:
  * Admin Passwort setzen
  * Dial Tone: f1=425@-17,c=0/0;
  * Busy Tone: f1=425@-21,c=480/480;
  * Automatic Reboot: Yes, reboot every week at day 0
* Profile 1
  * Primary SIP Server: 192.168.1.1
  * Use # as Dial Key: No
  * Disable # as Redial Key: Yes
  * Dial Plan: { x+ | \+x+ | *x+ | *xx*x+ | x+*x+*x+*x+ | x+*x+*x+*x+#x+ | **x+ | *12x#x+ }
  * Preferred Vocoder: Erst PCMA, dann PCMU
  * SLIC Setting: GERMANY
  * Caller ID Scheme: ETSI-FSK during ringing
  * Ring Frequency: 25 Hz
  * Enable High Ring Power: Yes

### Firmware Upgrade

Lokaler Upload geht nicht!  
Evtl. gehen die Default Einstellungen für den Firmware Server Path: fm.grandstream.com/gs  
Ansonsten geht auch eine Lösung über einen lokalen Raspberry: 192.168.1.101:8000  
Dort auf dem Dispi war dann im Ordner /smartdisplay/web die Datei: -rwxr-xr-x 1 pi pi ht818fw.bin