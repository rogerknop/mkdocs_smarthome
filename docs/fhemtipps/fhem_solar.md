# FHEM Solar

### ToDo

* Device als Dummy und dann beim Change eigene Sachen programmieren, falls notwendig. Achtung der Status muss stimmen: https://forum.fhem.de/index.php/topic,117864.msg1237769.html#msg1237769
* Vorschlagswerte von SolarForecast auslesen: https://forum.fhem.de/index.php/topic,117864.msg1234358/topicseen.html#msg1234358

### Übersicht

PDF ABLEGEN!

* Für DWD: sudo apt-get install libxml-libxml-perl
* Für DWD: sudo apt-get install libdatetime-perl
* Für SMAEM: sudo apt-get install libio-socket-multicast-perl
* Für SMAInverter: sudo apt-get install libio-socket-perl
* Für AI Support: sudo apt-get install libai-decisiontree-perl
* Feste IPs:
    * Home Manager -> Download Tool: SMA Home Manager Assistent
    * Wechselrichter (Netz & Akku) über die Webseite (IP im Router suchen): Geräteparamter & Parameter bearbeiten -> Anlagenkommunikation -> Speedwire: Automatische Konfiguration eingeschaltet NEIN & IP & Gateway usw. Speichern
* DWD installieren und Station wählen: https://www.dwd.de/DE/derdwd/messnetz/bodenbeobachtung/messnetzkarte_boden.pdf - 10704=Berus
    * Set Attr forecastProperties Rad1h,TTT,Neff,R101,ww,SunUp,SunRise,SunSet
    * Set Attr forecastResolution 1
* Homemanager wird in FHEM automatisch erkannt: define SMAHomeManager SMAEM
* SMAInverter Wechselrichter: define Tripower6  SMAInverter <Benutzer PW> 192.168.1.65
* SMAInverter Wechselrichter: define BoyStorage  SMAInverter <Benutzer PW> 192.168.1.64
* DWD zuordnen
* set pvCorrectionFactor_Auto für DWD auf on_complex_ai setzen 
* SolarForecast
    * "wget -qO ./FHEM/76_SolarForecast.pm https://svn.fhem.de/fhem/trunk/fhem/contrib/DS_Starter/76_SolarForecast.pm"
    * FHEM Command Line: help SoloarForecast de => Hilfe
    * define SolarForecast SolarForecast
    * currentInverterDev: Tripower6 pv=SPOT_PACTOT:kW etotal=SPOT_ETODAY:W capacity=6000
    * currentMeterDev: SMAHomeManager gcon=SMAEM3011953121_Bezug_Wirkleistung:W contotal=BezWirkZaehler:kWh gfeedin=SMAEM3011953121_Einspeisung_Wirkleistung:W feedtotal=SMAEM3011953121_Einspeisung_Wirkleistung_Zaehler:kWh
    * currentBatteryDev: BoyStorage pin=POWER_IN:W pout=POWER_OUT:W intotal=SPOT_ETOTAL:Wh charge=ChargeStatus
    * inverterStrings: Westdach
    * moduleDirection Westdach=W
    * moduleDirection Westdach=W
    * moduleTiltAngle Westdach=30
    * pvCorrectionFactor_Auto on
    * Attr flowGraphicSize 600

### Problem, wenn der SMA Homemanager keine Werte mehr liefert

Wenn das device SMAHomeManager keine aktuellen Readings mehr bekommt, dann gibt es ein Problem mit den Multicast Messages.

Simpel erklärt: SMA-HM/EM erzeugt die Daten und sendet diese dann pauschal ins interne Netzwerk (Eigentlich an die hinterlegte IP-Adresse - dies ist aber hier eine "öffentliche" Multicastadresse 239.12.255.254).

Alle anderen (die Multicast empfangen möchten) lauschen auf dieser Mulicastadresse und können dies nun empfangen - wenn dafür was "kommt'.

Wenn irgend ein Gerät (im internen IP-Netz) dann dies nicht mehr weiterleitet - weil es dies nicht kann oder defekt ist -, wird der Datensatz nur bis zu diesem Gerät (Switch/Hub/Repaeter/DLAN ect.) ankommen, aber kann (s.v.) von diesem Gerät aus nicht mehr weiter zw. durchgeleitet werden und das Gerät dahinter empfängt nichts.

<a href="https://forum.fhem.de/index.php?topic=51569.1005" target="_blank">https://forum.fhem.de/index.php?topic=51569.1005</a>

#### Manuelle Prüfungen & Testscript

* Ist die IP (192.168.1.66) vom Raspi erreichbar?
* Funktioniert die App und das Portal noch? https://www.sunnyportal.com/
* SMAEM Testscript testen (geht auch auf beliebigem Raspi):
    * Evtl. vorher Multicast Package installieren: 
        * https://github.com/kettenbach-it/FHEM-SMA-Speedwire
        * sudo apt-get install libio-socket-multicast-perl
    * FHEM stoppen: sudo systemctl stop fhem (falls fhem Raspi)
    * Unter rokscripts das Tool starten: perl rokscripts/smaem_test.pl
    * Prüfen, ob Werte ankommen
    * FHEM starten: sudo systemctl start fhem (falls fhem Raspi)

#### Lösungsansätze

!!! success "Evtl. Erfolgreich!"
    Letztendlich hat wahrscheinlich der Router Neustart die Lösung gebracht!

##### Versuch 1: SMA Home Manager neu starten

Allerdings prüfen. Es kann sein, dass kurz danach wieder keine Daten aktualisiert werden

* Portalseite -> Konfiguration Geräteübersicht
* Eigenschaften Icon des SMA Home Manager anklicken
* Ganz unten auf "Bearbeiten" klicken
* Oben Radiobutton "Erweiterte Konfiguration wählen"
* Bei "Produktgruppe: Sunny Home Manager" auf "Neustart" klicken

!!! warning "Warnung"
    Nach 10 Minuten evtl. wieder kein Empfang!

##### Versuch 2: Wenn es die Multicast Messages immer wieder ausbleiben, dann kann man die Raspi IP lokal im Sunny Portal eintragen:

* Portalseite -> Konfiguration Geräteübersicht
* Eigenschaften Icon des SMA Home Manager anklicken
* Ganz unten auf "Bearbeiten" klicken
* Oben Radiobutton "Erweiterte Konfiguration wählen"
* Ganz unten "Direkte Zähler Kommunikation" - Hier die IP vom FHEM Raspi eintragen

!!! warning "Warnung"
    Dadurch wird die Standard IP 239.12.255.254 nicht mehr versorgt => Kein Laden mehr der Batterie !
