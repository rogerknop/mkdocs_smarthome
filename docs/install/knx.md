# KNX

Rollladen in ETS5

Die Rückmeldung muss eingerichtet werden, damit der Rollladen mit Prozent gesteuert werden kann.  
Folgende Schritte wurden im Jalousie Aktor 1.1.7 durchgeführt:

* Kanal Parameter Statusmeldung & Kalibrierung entsperren
* Antrieb Faktor Laufzeit Höhe Messen runter / 100ms
* Antrieb Faktor Laufzeitzuschlag Messen hoch Diff zu runter / 100ms
* Position Höhe & Rückmeldung Höhe unterschiedliche Nummern 811 und 821
* Beide auf L & S (lesen und senden) setzen
* Programmieren - Applikationsprogramm
