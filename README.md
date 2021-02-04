# LoudML-Grafana-Demo
Diese Demo Andwendung verbindet Loud ML für das Anwenden von Machine Learning auf Metrik basierte Daten und die Integration von Loud ML in eine Grafana Oberfläche.

## Installation

```bash
docker compose -f docker-compose.yml up
```

Nachdem alle Container hochgefahren wurden, müssen noch zusätzlich noch zwei kleine Anpassungen vorgenommen werden, damit die Demo Anwendung funktioniert.

### Aktualisieren der Python Bibliotheken auf dem Loud ML Server

```bash
docker exec -it -u 0 loudml  bash
```
Mit diesem Befehl gelangt man auf die Shell des Loud ML Containers. Nun führt man noch den nachfolgenden Befehl aus, um die Bibliotheken zu aktualisieren. Der Loud ML Container benötigt hierbei keinen Neustart, nach dem Update.

```bash
apt-get update && apt-get install -y python3-pip python3-setuptools python3-dev && apt-get install -y --no-install-recommends build-essential gcc git && apt-get purge -y
´´´

