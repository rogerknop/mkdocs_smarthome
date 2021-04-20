# ESP Tipps

### Kabel

Die Kabeldicke 22 AWG (0,32mm<sup>2</sup>) ist optimal zum Löten.

Steckverbinder gibt es unter dem Schlagwort JST zu finden. Die funktionieren auch wie die PSK Stecker mit den PINs (Crimpzange).

### Access Point

<a href="https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/soft-access-point-examples.html" target="_blank">https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/soft-access-point-examples.html</a>

Es ist möglich den ESP als Soft Access Point aufzusetzen über:

???+ file ""
    <pre>WiFi.softAP("ESPsoftAP_01", "pass-to-soft-AP");</pre>

### Zusatzantenne

<a href="https://www.stall.biz/project/esp8266-esp12-mit-externer-wlan-antenne-fuer-wiffi-und-co" target="_blank">https://www.stall.biz/project/esp8266-esp12-mit-externer-wlan-antenne-fuer-wiffi-und-co</a>

__Achtung!!!__ Es gibt 2 Antennensteckerarten: _RP-SMA_ und _SMA_. Beim Kaufen aufpassen dass Stecker und Antenne vom gleichen Typ sind.

##### NodeMCU Typ 1

![NodeMCU Wifi Typ 1](../../img/esp/wifi_antenne_typ1_1.jpg)

![NodeMCU Wifi Typ 1](../../img/esp/wifi_antenne_typ1_2.jpg)

![NodeMCU Wifi Typ 1](../../img/esp/wifi_antenne_typ1_3.jpg)

##### NodeMCU Typ 2

![NodeMCU Wifi Typ 2](../../img/esp/wifi_antenne_typ2_1.jpg)

![NodeMCU Wifi Typ 2](../../img/esp/wifi_antenne_typ2_2.jpg)

![NodeMCU Wifi Typ 2](../../img/esp/wifi_antenne_typ2_3.jpg)

