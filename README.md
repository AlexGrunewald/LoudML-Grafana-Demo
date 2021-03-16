# LoudML-Grafana-Demo
Diese Demo Andwendung verbindet Loud ML für das Anwenden von Machine Learning auf Metrik basierte Daten und die Integration von Loud ML in eine Grafana Oberfläche.

# Installation

```bash
docker compose -f docker-compose.yml up
```

Nachdem alle Container hochgefahren wurden, müssen noch zusätzlich noch zwei kleine Anpassungen vorgenommen werden, damit die Demo Anwendung funktioniert.

## Aktualisieren der Python Bibliotheken auf dem Loud ML Server

```bash
docker exec -it -u 0 loudml  bash
```
Mit diesem Befehl gelangt man auf die Shell des Loud ML Containers. Nun führt man noch den nachfolgenden Befehl aus, um die Bibliotheken zu aktualisieren. Der Loud ML Container benötigt hierbei keinen Neustart, nach dem Update.

```bash
apt-get update && apt-get install -y python3-pip python3-setuptools python3-dev && apt-get install -y --no-install-recommends build-essential gcc git && apt-get purge -y
```

## Hinzufügen einer Output Datenbank in die InfluxDB

Damit die von Loud ML erzeugten Daten auch in der InfluxDB abgespeichert werden, muss im nächsten Schritt noch mittels Curl eine weitere Datenbank angelegt werden. Hierzu kann der nachfolgende Befehl einfach kopiert und eingefügt werden.

```bash
curl -XPOST 'http://0.0.0.0:8086/query' --data-urlencode 'q=CREATE DATABASE "mydb"'
```

# Benutzung

Nachdem die Installation erfolgreich abgeschlossen wurde, kann damit begonnen werden die Demo Umgebung einzurichten. 
Hierzu öffnet man die Grafana Weboberfläche, welche unter <http://0.0.0.0:3000/> erreichbar sein sollte. Die Login Credentials lauten admin/admin.
Nun kann man unter den Configuration die InfluxDB als Data Source hinzufügen mithilfe der folgenden Werte: 
```bash
URL: http://influxdb:8086
Database: _internal
```
Nachdem die Datenbank eingebunden ist, kann man unter Dashboards sich ein neues Panel anlegen, welchen den nachfolgenden Query umsetzt:

```bash
SELECT mean("Alloc") FROM "monitor"."runtime" WHERE $timeFilter GROUP BY time(1m) fill(previous)
```
Anschließend kann das Dashboard gespeichert werden. Nun kann der Loud-ML Server als Data Source hinzugefügt werden. Hierzu sollte zuvor das Loud-ML Plugin unter Configuration->Plugins aktiviert werden. Ist dies geschehen, kann der Loud-ML Server unter Configuration, wie zuvor auch schon die InfluxDB eingebunden werden. Hierzu fügt man einfach die URL ein, unter welcher der Loud ML Server erreichbar ist. Im Fall dieser Demo Umgebung ist dieser unter der folgenden Adresse für Grafana erreichbar:

```bash
URL: http://loudml:8077/
```
Nachdem dies geschehen ist, geht man zurück auf das zuvor erstelle Panel. Nun sollte, wenn man dieses editieren will, die Option "Loud ML Graph" unter Visualisierungen verfügbar sein. Wählt man diese aus, muss anschließedn noch noch unter Display das Input- und Output-Bucket ausgewählt werden. Im falle dieser Demo, ist das Input-Bucket "influxdb" und das Output-Bucket "loudml". Beide Buckets wurden automatisch durch die Config.yml in Loud ML erstellt und müssen nicht händdisch konfiguriert werden. 

Nachdem diese Einstellungen vorgenommen wurden, sollte das Panel wie in der nachfolgenden Grafik aussehen:
![Loud ML Graph Panel](https://github.com/AlexGrunewald/LoudML-Grafana-Demo/blob/main/img/loudml_dashboard_plugin.png "Loud ML Graph Panel")

Nun kann im nächsten Schritt das Machine Learning Modell trainiert werden, indem die Schaltfläche "Create Baseline" auf dem Panel geklickt wird. Der Loud ML Server erstellt nun eine Baseline für die gefragten Daten. Dies kann einige Zeit in Anspruch nehmen.

```bash
**Note:** Sollte das erstllen einer Baseline fehlschlagen, kann das auf die fehlenden Datenpunkte zurückzuführen sein. Hierzu muss die Demo Umgebung erst eine Weile laufen, damit genügend Daten in der Datenbank vorhanden sind, damit der Algorithmus genügend Ansatzpunkte hat.
```

Wenn das trainieren einer Baseline reibungslos abgelaufen ist, kann der Algorithmus zur Anomalieerkennung durch ein einfaches drücken des "Play-Buttons" in der Panel Oberfläche aktiv geschaltet werden. Anschließend muss dann die Datenbank, in welcher Loud ML die Metriken, die der Algorithmus erzeugt, speichert, als Datenquelle hinzugefügt werden. Dies funktioniert analog zum hinzufügen der ersten InfluxDB Quelle mit folgenden Parametern:

```bash
URL: http://influxdb:8086
Database: loudml
```
Im letzten Schritt kann dann das Panel noch angepasst werden, um die Machine Learning Daten als Overlay anzeigen zu lassen. Hierzu kann das Panel wie in folgender Grafik angepasst werden:
![Loud ML Graph Panel](https://github.com/AlexGrunewald/LoudML-Grafana-Demo/blob/main/img/Screenshot%202021-01-29%20110451.png "Loud ML Graph Panel")

Nun ist man in der Lage die Machine Learning Daten auf einem Blick als Overlay anzeigen zu lassen.

# Hinzufügen von Annotationen

Um Annotationen hinzuzufügen, muss zunächst eine weitere Datenquelle angelegt werden. Loud ML speichert die Anomalien standartmäßig in der Datenbank Chronograf, weshalb die Parameter für die Annotation-Quelle folgendermaßen aussieht:

```bash
URL: http://influxdb:8086
Database: chronograf
```
Nachdem diese hinzugefügt wurde, kann unter den Dashboard Settings die Option Annotations ausgewählt und ein neuer Annotations-Query angelegt werden. Als Datenquelle gibt man hier dann die zuvor angelegt Quelle an, sowie den folgenden Query:

```bash
SELECT "text" FROM "autogen"."annotations" WHERE $timeFilter
```
Anschließend speichert man alles und die von Loud ML gefundenen Anomalien sollten nun als Annotation am entsprechenden Graphen angezeigt werden.

# Future Work
- Eine Anleitung wie Annotations zum Panel hinzugefügt werden
- Ein weiteres Machine Learning Modell neben dem Donut Modell in Loud ML implementieren
