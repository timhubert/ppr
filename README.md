# Apache Spark-Projekt PPR

Originale Docker-Images: https://github.com/big-data-europe/docker-spark

Docker-Images wurden in diesem Repository für entsprechende PPR-Projekt-Zwecke angepasst.

## Schritt für Schritt Anleitung:
1. Repository klonen
2. Docker-Images durch Dockerfiles erstellen
   ```
   ./build.sh
   ```
3. Starten der benötigten Docker-Images (Master und einen Worker)
   ```
   docker-compose up -d
   ```
4. http://localhost:8080, um auf die Apache Spark-Übersicht zu gelangen
5. Erstellen eines Docker-Image, um unsere auszuführende Python-Datei dem Spark-Master mitzuteilen.
   ```
   docker build --rm -t spark-app keras/.
   ```
6. Starten des in Schritt 6 erstellten Docker-Images mit dem Namen "my-spark-app".
   ```
   docker run --name my-spark-app -e ENABLE_INIT_DAEMON=false --link spark-master:spark-master -d --net docker-spark_default          spark-app
   ```
   Ist dieser gestartet, wird der Code in der Datei "app.py" dem Spark-Master übermittelt. Dieser zerstückelt den Code (RDD-      Datenstruktur) und verteilt diese auf die beim Master angemeldeten Spark-Worker.

## Worker manuell starten
Den Parameter "--name" im folgenden Befehl anpassen!
```
docker run --name spark-worker-2 --link spark-master:spark-master -e ENABLE_INIT_DAEMON=false -d --net docker-spark_default bde2020/spark-worker:2.4.0-hadoop2.7
```

## Apache Spark-Container neustarten
Es müssen alle unter "docker ps" aufgelisteten Container gestoppt und gelöscht werden (NICHT die Docker-Images).
Ist dies ausgeführt, wird wieder mit Schritt 3 der Spark-Master und Spark-Worker gestartet. Mit Schritt 7 wird die bereits erstellte (Erstellt durch Schritt 6) Spark-App gestartet.

## Fehlerbehebungen
### Beim Start des Docker-Images kommt folgende Fehlermeldung:
```
docker: Error response from daemon: Cannot link to /spark-master, as it does not belong to the default network.
```
Lösung: Beim Start des Docker-Container muss der "--net"-Parameter angegeben werden. Durch "docker network ls" erhält man die erstellten Netzwerke. Alle Spark-Container müssen im selben Netzwerk sein.

### Application wird nicht auf Spark ausgeführt
Kann viele Ursachen haben. Am Besten in der Konsole mit "docker logs my-spark-app" die Ausgabe des App-Containers anzeigen lassen.
