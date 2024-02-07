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