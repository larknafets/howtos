# SOLECTRUS mit evcc verbinden

## Was ist evcc?

[evcc](https://www.evcc.io/) ist ein Energie-Management-System mit Fokus auf Elektromobilität. Die Software steuert deine Wallbox oder Schaltsteckdose. Um intelligente Entscheidungen zu treffen, kommuniziert es auch mit deinem Fahrzeug, Wechselrichter oder Hausspeicher.

## Voraussetzungen
1. Bestehende und eingerichtete evcc Instanz
2. MQTT Broker

> Die Einrichtung von evcc und SELECTRUS sind nicht Gegenstand dieser Anleitung.

## Links
* [evcc](https://www.evcc.io/)
* [evcc Dokumentation](https://docs.evcc.io/docs/Home)
* [SOLECTRUS](https://www.solectrus.de/)
* [SOLECTRUS Dokumentation](https://docs.solectrus.de/)
* [MQTT Explorer (Mac, Windows, Linux)](https://mqtt-explorer.com/)
* [SOLECTRUS-Konfigurator](https://configurator.solectrus.de/)

## evcc Konfiguration für MQTT

Damit SOLECTRUS Daten von evcc erhält, kann der Umweg über einen MQTT Broker genommen werden, welcher die Daten "zwischenspeichert".

Der MQTT Broker kann entweder über die <b>Konfigurationsoberläche</b> oder direkt im der <b>YAML-Datei</b> hinzugefügt werden.

### Konfigurationsoberfläche

In der <b>Konfigurationsoberfläche</b> kann die Einstellung im Burgermenüunkt "Konfiguration" ganz unten bei "Integrationen" vorgenommen werden. Hier einfach bei der Integration "MQTT" den Button für die EInstellungen anwählen und die folgenden Infos eintragen:
* Broker: `<url/ip>:1883`
* Thema: `evcc`
* Benutzer: `user` (optional)
* Passwort: `password` (optional)

### Alternativ: YAML-Datei

Wenn die Einstellung direkt im der<b>YAML-Datei</b> vorgenommen werden soll, muss die `evcc.yaml` um die folgenden Zeilen erweitert werden:

```
# mqtt message broker
mqtt:
  broker: <url/ip>:1883
  topic: evcc
  user: <user>
  password: <password>
```

> Es ist darauf zu achten, dass die Werte richtig eingerückt sind.

`url/ip` muss durch die IP Adresse oder Domain (URL) des MQTT Brokers ersetzt werden. Der Port ist i.d.R. `1883`, bei SSL-Verschlüsselung `8883`. `user` und `password` sind die Zugangsdaten, die beim MQTT Broker eingerichtet wurden. Diese können auch leer sein, wenn kein Benutzer angelegt wurde.

Danach muss evcc einmal neu gestartet werden. Über Client Programme, kann man prüfen, ob der MQTT Broker Daten erhält.

## SOLECTRUS Konfiguration

Am einfachsten ist die Konfiguration über den [SOLECTRUS-Konfigurator](https://configurator.solectrus.de/). Hier läßt sich von vornherein anwählen, dass der MQTT-Konnektor genutzt werden soll und relevanten Daten werden abgefragt.

Für evcc sind die folgenden Topics einzutragen:
* Erzeugung: `evcc/site/pvPower`
* Hausverbrauch: `evcc/site/homePower`
* Netzbezug-/einspeisung: `evcc/site/grid/power`
* Netz: Vorzeichenbehandlung: <b>Positiv ist Netzbezug , Negativ ist Einspeisung</b>
* Einspeisebegrenzung: <b><i>leer</i></b>
* Batterie: Beladung / -entladung: `evcc/site/batteryPower`
* Batterie: Vorzeichenbehandlung: <b>Positiv ist Entladung , Negativ ist Beladung</b>
* Batterie-Ladestand: `evcc/site/batterySoc`
* Geräte- oder Gehäusetemperatur: <b><i>leer</i></b>
* Wallbox: `evcc/loadpoints/1/chargePower`
* Wallbox im Hausverbrauch enthalten? <b>Der Verbrauch der Wallbox ist nicht im Hausverbrauch enthalten</b>
* Wallbox: Auto verbunden: `evcc/loadpoints/1/connected`
* E-Auto: Batterie-Ladestand: `evcc/loadpoints/1/vehicleSoc`
* Systemstatus: `evcc/status`
* Systemstatus ist OK: <b><i>leer</i></b>

> SOLECTRUS unterstützt nur eine Wallbox und einen E-Auto Ladestand. Hier konfiguriert wird der E-Auto Ladenstand, der angezeigt wird, wenn ein E-Auto mit evcc / der Wallbox verbunden ist.
>
> Wer erweiterte Kenntnisse besitzt, kann sich aber die MQTT Topics aus anderen Quellen zusammenstellen. Es ist auch mögich die evcc Wallboxverbräuche zu addieren, wenn man mehrere Wallboxen hat.

In der `.env` Datei sieht das dann folgendermaßen aus:

```
...

##################################################################
###                      Sensor mapping                        ###
##################################################################

INFLUX_SENSOR_INVERTER_POWER=pv:inverter_power
INFLUX_SENSOR_HOUSE_POWER=pv:house_power
INFLUX_SENSOR_GRID_IMPORT_POWER=pv:grid_import_power
INFLUX_SENSOR_GRID_EXPORT_POWER=pv:grid_export_power
INFLUX_SENSOR_BATTERY_CHARGING_POWER=pv:battery_charging_power
INFLUX_SENSOR_BATTERY_DISCHARGING_POWER=pv:battery_discharging_power
INFLUX_SENSOR_BATTERY_SOC=pv:battery_soc
INFLUX_SENSOR_WALLBOX_POWER=pv:wallbox_power
INFLUX_SENSOR_WALLBOX_CAR_CONNECTED=pv:car_connected
INFLUX_SENSOR_SYSTEM_STATUS=pv:system_status
INFLUX_SENSOR_CAR_BATTERY_SOC=car:battery_soc

...

##################################################################
###                       MQTT Collector                       ###
##################################################################

##### MQTT Broker credentials
MQTT_HOST=<ip>
MQTT_PORT=1883
MQTT_SSL=false
MQTT_USERNAME=<usernam>
MQTT_PASSWORD=<password>

##### Mappings

# Inverter/PV power
MAPPING_0_TOPIC=evcc/site/pvPower
MAPPING_0_MEASUREMENT=pv
MAPPING_0_FIELD=inverter_power
MAPPING_0_TYPE=integer
# House power
MAPPING_1_TOPIC=evcc/site/homePower
MAPPING_1_MEASUREMENT=pv
MAPPING_1_FIELD=house_power
MAPPING_1_TYPE=integer
# Grid power
MAPPING_2_TOPIC=evcc/site/grid/power
MAPPING_2_MEASUREMENT_POSITIVE=pv
MAPPING_2_MEASUREMENT_NEGATIVE=pv
MAPPING_2_FIELD_POSITIVE=grid_import_power
MAPPING_2_FIELD_NEGATIVE=grid_export_power
MAPPING_2_TYPE=integer
# Battery power
MAPPING_3_TOPIC=evcc/site/batteryPower
MAPPING_3_MEASUREMENT_POSITIVE=pv
MAPPING_3_MEASUREMENT_NEGATIVE=pv
MAPPING_3_FIELD_POSITIVE=battery_discharging_power
MAPPING_3_FIELD_NEGATIVE=battery_charging_power
MAPPING_3_TYPE=integer
# Battery SOC
MAPPING_4_TOPIC=evcc/site/batterySoc
MAPPING_4_MEASUREMENT=pv
MAPPING_4_FIELD=battery_soc
MAPPING_4_TYPE=float
# Wallbox
MAPPING_5_TOPIC=evcc/loadpoints/1/chargePower
MAPPING_5_MEASUREMENT=pv
MAPPING_5_FIELD=wallbox_power
MAPPING_5_TYPE=integer
# System status
MAPPING_6_TOPIC=evcc/status
MAPPING_6_MEASUREMENT=pv
MAPPING_6_FIELD=system_status
MAPPING_6_TYPE=string
# Wallbox: car connected
MAPPING_7_TOPIC=evcc/loadpoints/1/connected
MAPPING_7_MEASUREMENT=pv
MAPPING_7_FIELD=car_connected
MAPPING_7_TYPE=string
# Car battery
MAPPING_8_TOPIC=evcc/loadpoints/1/vehicleSoc
MAPPING_8_MEASUREMENT=car
MAPPING_8_FIELD=battery_soc
MAPPING_8_TYPE=float

...
```

Wenn die Einstellungen nachträglich vorgenommen werden, muss der MQTT-Connector in der `docker-compose.yml`/`compose.yml` hinzugefügt werden.

Die Änderungen sind nach einem Neustart von SELECTRUS aktiv.

Sollten Informationen nicht angezeigt werden, sollte das Log der Docker Komponenten auf Hinweise geprüft werden.

Weitere Informationen zur MQTT-Anbindung ist in der [SOLECTRUS Dokumentation für den MQTT-Collector](https://docs.solectrus.de/referenz/mqtt-collector/) zu finden.