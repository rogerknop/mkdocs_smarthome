# VPN

Entweder über die Fritzbox direkt den Wireguard VPN Zugang einrichten, oder einen Raspi als VPN Server nutzen.

## Wireguard VPN über Fritzbox

OFFEN - Wireguard wird erst mit dem FritzOS Update Ende 22 unterstützt!

## Wireguard VPN über Raspberry

### Raspberry Server installieren und einrichten

Die Installation erfolgt sehr einfach über das Tool PiVPN (https://www.pivpn.io/).

!!! terminal "Terminal"
    <pre>
    curl -L https://install.pivpn.io | bash
    </pre>

Die Konfiguration mit den Defaultwerten bestätigen.  
Die statische IP und die DynDNS URL muss während der Konfiguration eingetragen werden.

Die Konfiguration kann bearbeitet werden mit:

!!! terminal "Terminal"
    <pre>
    sudo nano /etc/pivpn/wireguard/setupVars.conf
    </pre>

Ein neues Device legt man an über ():

!!! terminal "Terminal"
    <pre>
    pivpn add
    \# Zum Kopieren
    sudo ls /home/pi/configs
    sudo more /home/pi/configs/[Name der Datei]
    </pre>

Es sollte ein sprechender Name vergeben werden, damit man leicht die Dateien wieder löschen kann, bzw. auch für die Weitergabe einfach identifizieren kann.

Die Dateien werden im Ordner /home/pi/configs erzeugt.

Man kann einen QR Code für die Installation auf den jeweiligen Clients erzeugen, oder die Dateien per Mail senden, wenn [SendMail über CLI](raspi.md#sendmail-uber-cli) eingerichtet:

!!! terminal "Terminal"
    <pre>
    \# QR Code erstellen
    pivpn -qr
    \# Über Index Device auswählen und ein Foto machen

    \# Status Monitoring
    pivpn -c

    \# Problem, wenn HTTP Seiten nicht gehen - iptables masquerade rule set
    pivpn -d

    \# Conf an Client mailen
    sudo mpack -s "PC Roger" ~/configs/roger_pc.conf roger@dieknops.de
    </pre>

Die Dateien können auch auf das Device (Windows, Android, iOS etc.) übertragen werden und können in den jeweiligen Wireguard Client importiert werden.

## Router einrichten für VPN auf dem Raspberry

Auf der Seite https://www.spdyn.de kann man einen DynDNS Weiterleitung einrichten - z.B. praxisbous.spdns.org.  
Diese URL muss im Router als DynDNS eingetragen werden.  
Als Custom URL für das Update der IP muss folgende URL verwendet werden: https://update.spdyn.de/nic/update?hostname=<domain>&myip=<ipaddr>&user=roger.knop@sap.com&pass=<password>

Falls es mit dem Router Probleme gibt, kann das Update auch über ein Bash Script auf dem Raspi erfolgen, was im Cron eingeplant wird.  
Die eigene IP ermitteln über:

!!! terminal "Terminal"
    <pre>
    curl -4 -s checkip.spdyn.de
    \# returns 165.1.191.121
    \# ODER
    curl https://api.ipify.org?format=json
    \# returns {"ip":"165.1.191.121"}
    </pre>

Das Update Bash Script könnte wie folgt aussehen (läuft auf dem Praxis Raspi):

??? file "update_public_ip.sh - Per User"
    <pre>
    \#!/bin/bash

    my_host="deinhost.spdns.eu"
    user="username"
    pass="passwort"
    datum=\`date +%A,%d.%m.%Y_%H:%M\`

    \# externe ip holen
    ip=\`curl -4 -s checkip.spdyn.de\`

    \# das file .ext_ip4 merkt sich die aktuelle ip
    if [ -f /root/.ext_ip4 ]
    then
    alt_ip=\`cat /root/.ext_ip4\`
    if [ "$alt_ip" == "$ip" ]
    then
      \# ip unverändert, abbruch
      logger "ip4 unverändert am $datum"
      exit 1
    else
      echo "ip4 verändert, wird aktualisiert am $datum"
      \# aktualisierung, hier könnte man den Rückgabewert good? auswerten
      logger \`curl -u $user:$pass -4 -s "https://update.spdyn.de/nic/update?hostname=$my_host&myip=$ip"\`
      \#ip merken
      echo $ip > /root/.ext_ip4
    fi
    else
      \# erster lauf, keine .ext_ip4 da…
      echo $ip > /root/.ext_ip4
      logger \`curl -u $user:$pass -4 -s "https://update.spdyn.de/nic/update?hostname=$my_host&myip=$ip"\`
    fi    
    </pre>

Oder ein Script, was über den Token und nicht über den User geht:

??? file "update_public_ip.sh - Per Token"
    <pre>
    \#!/bin/bash

    \# registered domains. might be the same for v4 and v6
    DOMAINV4=foobar.my-router.de
    DOMAINV6=${DOMAINV4}

    \# generated token for each domain
    TOKENV4=abcd-abcd-abcd
    TOKENV6=abcd-abcd-abcd

    \# get own IPs to feed them back to spdyn. See http://wiki.securepoint.de/index.php/SPDyn/meineIP
    MYIPV4=$(curl -s http://checkip4.spdyn.de/)
    MYIPV6=$(curl -s http://checkip6.spdyn.de/)

    \# update spdyn records if IPs were fetched
    if [ -n "${MYIPV4}" ];
    then
        echo "updating v4 to: ${MYIPV4}"
        curl -u "${DOMAINV4}:${TOKENV4}" "https://update.spdyn.de/nic/update?hostname=${DOMAINV4}&myip=${MYIPV4}"
    else
        echo "error retrieving v4"
    fi

    if [ -n "${MYIPV6}" ];
    then
        echo "updating v6 to: ${MYIPV6}"
        curl -u "${DOMAINV6}:${TOKENV6}" "https://update.spdyn.de/nic/update?hostname=${DOMAINV6}&myip=${MYIPV6}"
    else
        echo "error retrieving v6"
    fi
    </pre>

Weiterhin muss eine Portweiterleitung für die statische IP des Raspi eingetragen werden. Default Port ist 51820 (intern und extern verwenden).