# FHEM Tipps im Betrieb

### Direktbefehle & Tipps

* Neustart: shutdown restart
* Sonnenuntergang testen: {sunset_abs("HORIZON=1.0",0,"16:30","23:00")}  
	Durch das _abs wird die Zeit für den nächsten Tag ermittelt. Wenn der Zeitpunkt rum ist würde ohne _abs Schrott rauskommen.  
	Ebenso geht auch sunrise_abs
* Log File löschen: {unlink ("./log/fhem-2014-07.log")}
* Test notify: Im Web auf Everything klicken und das notify suchen. Draufklicken und dann bei set execNow auswählen und auf set klicken
* Alle Log Files eines Jahres löschen (muss noch getestet werden): {unlink(glob("./log/fhem-2014-*.log")}
* Log Level:
    * In fhem.cfg oben setzen
    * Hoch: attr TRX_0 verbose 5
    * Normal: attr TRX_0 verbose 1
* Readonly für fhem.cfg entfernen: attr WEB editConfig 1
* Version anzeigen: version
* List
    * List <device> - Zeigt die komplette Definition (fürs Forum auch wichtig)
    * List at_end.* DEF -> Zeit alle Geräte die mit at_end anfangen
* Harmony:
    * Alle Geräte definieren in der Config über Kommandozeile: set [harmony_name] autocreate
    * Alle Activies anzeigen: get [harmony_name] activities
    * Alle Gerätebefehle anzeigen: get [harmony_name] deviceCommands
* HTML Seite einbinden
	define Wetter_meteo weblink iframe http://www.meteoblue.com/de_DE/wetter/vorhersage/woche/essen_de_33856  
	attr Wetter_meteo htmlattr width="600" height="400"
* FHEM über URL Steuern: http://192.168.1.99:8083/fhem?cmd.Test=set%20Lampe%20on  
	Wegen CSRF Token so:  
	curl "http://192.168.1.99:8083/fhem?cmd=set%20Arbeit.Licht.Decke%20on&XHR=1&fwcsrf="\`curl -s -D - 'http://192.168.1.99:8083/fhem?XHR=1' | awk '/X-FHEM-csrfToken/{print $2}' | tr -d "\r\n"`
* Reading löschen: deletereading <device\> <reading\> 
* Device umbenennen: rename <device\> <new_device_name\> 
* Device anlegen: define <device>
* Weblink htmlCode editieren
    * list TYPE=weblink
    * Klick auf Wert hinter "name"

### Dämmerung

attr global latitude 51.xxx
attr global longitude 6.xxx

sunset liefert den nächsten sonnenuntergang in stunden - somit > 24 möglich
sunset_abs liefert korrekte uhrzeit (für tests)

Beispiel: Aussenlampe - Steuerung An-/Ausschaltzeit
define AussenlampeAn1 at *{sunset(0,"17:30","22:00")} set SW_Dose_1 on

### Notify testen

* Everthing anklicken
* Dort das notify suchen und draufklicken
* Im set Dropdown ExecNow auswählen 
* Auf set klicken
