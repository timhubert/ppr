# Paralleles Training neuronaler Netzwerke  
Projekt für Vorlesung Parallele Prozesse. 

## Fragestellung
Beim Training von Neuronalen Netzwerken müssen sehr viele Gleitkommaoperationen ausgeführt werden. Durch den Einsatz von entsprechender Bibliotheken, wie TensorFlow kann die Berechnungen in ein Berechnungsgraphenüberführt werden. Somit lassen sich die Berechnungen gut parallelisieren. Diese Nebenläufigkeit beschränkt sich allerdings nur auf eng gekoppelte Multiprozessor-Systeme. Eine Verteilung über mehrere unabhängie Rechner erfolgt nicht. 
Dieses Projekt zeigt eine Möglichkeit, wie dies möglich ist. Des Weiteren soll ein Blick auf den Aufwand im Bezug zu dem Nutzen geworfen werden.

## Apache Spark-Projekt PPR

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


## Ergebnisse

### Generell
Mit diesem Projekt konnte gezeigt werden, dass eine Möglichkeit besteht, Neuronale Netzwerke gut skalierbar parallel zu trainieren. [Apache Spark](https://spark.apache.org/) wurde genutzt um eine Cluster-Computing-Infrastruktur bereitzustellen. Die Verteilung der Infrastruktur erfolgt mit dem Sysem Docker, sodass die Anwendung gut skaliert und leicht auf andere Systeme portiert werden kann.  
Wir nutzen das bekannte [Keras](https://keras.io/), was eine Deep-Learning-Bibliothek ist, mit der man komfortabel Neuronale Netzwerke erzeugen und trainieren kann. Das mit Keras erstellte Modell konnte mit Hilfe der Python-Bibliothek [Distributed Keras](https://joerihermans.com/work/distributed-keras/) in kleine parallel ausführbare Einheiten zerlegt werden. Das training erfolgt mit einem von dist-keras mitgeliefertem Optimierer. Dieser trainiert in kleinen Batches verteilt und fügt die Zwischenergebnisse immer wieder zentral zusammen.

#### Verteilung
Die von uns hier vorgestellte Lösung ist auf eine sehr große Verteilung ausgelegt. Da wir zum Testen nur zwei Worker-Container starten, die auch noch auf einem lokalen PC laufen, konnte keine Zeit für das Training eingespart werden. Das Zerschneiden und starten der einzelnen Berechnungspakete (RDD), der Netzwerkverkehr und Apache Spark selbst erzeugen so viel Overhead, dass die Ausführung um ein zehnfaches so lange dauerte, wie ohne Cluster (Apache Spark) auf einem einzelnen PC.
Der Aufwand lohnt sich erst, wenn ein hoch optimiertes Spark-Cluster in einem Rechenzentrum mit mehreren PCs genutzt wird und damit sehr große Mengen verarbeitet werden.

#### Kosten-Nutzen
Der Aufwand zum Erzeugen und konfigurieren der Infrastruktur war sehr hoch. Das von big-data-europa bereitgestellte Repository mit der Spark-Docker-Images war für unsere Anforderungen nicht lauffähig und musste weitgehen angepasst werden. Auch bis eine passende Bibliothek zum Verteilen des Keras-Modells gefunden war, mussten mehrere Projekte und Frameworks ausprobiert werden. Schlussendlich lohnte der Aufwand zum Trainieren von wenigen Daten nicht. Auch, weil kein großes Cluster  zum Ausführen und eine sehr große Datenmenge zum Trainieren vorhanden war.
Als Alternative bietet sich an, nicht verteilt zu rechnen, sondern im Shared-Memory-Verfahren die Berechnung auf einer Grafikkarte zu tätigen. Diese ist auf Gleitkommaoperationen spezialisiert und bringt eine deutliche Performance-Steigerung. Tensorflow unterstützt dies bereits. 
