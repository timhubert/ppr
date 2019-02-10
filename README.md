# Paralleles Training neuronaler Netzwerke  
Projekt für Vorlesung Parallele Prozesse. 

## Fragestellung
Beim Training von Neuronalen Netzwerken müssen sehr viele Gleitkommaoperationen ausgeführt werden. Durch den Einsatz von entsprechender Bibliotheken, wie TensorFlow kann die die Berechnungen in ein Berechnungsgraphenüberführt werden. Somit lassen sich die Berechnungen gut parallelisieren. Diese Nebenläufigkeit beschränkt sich allerdings nur aufauf eng gekoppelte Multiprozessor Systeme. Eine Verteilung über mehrere unabhänige Rechner erfolgt nicht. 
Dieses Projekt soll eine Möglichkeit zeigen wie dies möglich ist. Des weiteren soll der Aufwand im Bezug zu dem Nutzen gestellt werden.

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

#### Generelles
Mit diesem Projekt konnte gezeigt werden, dass eine Möglichkeit besteht Neuronale Netzwerke gut skalierbar parallel zu trainieren. [Apache Spark](https://spark.apache.org/) wurde genutzt um eine Cluster-Computing-Infrastruktur bereit zu stellen. Diese wurde mithilfe von Docker-Containern aufgebaut, sodass sie gut skaliert und auf andere Systeme portiert werden kann.  
Wir nutzen das bekannte [Keras](https://keras.io/), was eine Deep-Learning-Bibliothek ist, mit der man komfortabel Neuronale Netzwerke erzeugen und trainieren kann. Das mit Keras erstellte Modell konnte mit Hilfe der Python-Bibliothek [Distributed Keras](https://joerihermans.com/work/distributed-keras/) in kleine parallel ausführbare Einheiten zerlegt werden. Das training erfolgt mit einem von dist-keras mitgeliefertem Optimierer. Dieser trainiert in kleinen Batches verteilt und fügt die Zwischenergebnisse immer wieder zentral zusammen.

#### Verteilung
Die von uns hier vorgestellte Lösung ist auf eine sehr große Verteilung ausgelegt. Da wir zum test nur zwei Worker-Container starten, die auch noch auf einem lokalen PC laufen, konnte keine Zeit für das Training eingespart werden. Das zerschneiden und starten der einzelnen Berechnungspakete, der Netzwerkverkehr und Apache Spark selbst erzeugen so viel Overhead dass die Ausführung um ein Zehnfaches so lang dauerte, wie ohne cluster auf einem einzelnen PC.
Der Aufwand lohnt erst wenn ein hoch optimiertes Spark-Cluster in einem Rechenzentrum mit mehren PCs genutzt wird. 

#### Kosten-Nutzen
Der Aufwand zum erzeugen und konfigurieren war sehr hoch. Das Repository mit der Spark-Docker-Installation war nicht lauffähig und musste weitgehen angepasst werden. Auch bis eine Passende Bibliothek zum verteilen des Keras-Modells gefunden war mussten mehrere Projekte ausprobiert werden. Der Aufwand lohnte also nicht, auch deswegen nicht weil kein großes Cluster zum ausführen vorhanden war.
Als Alternative bietet es sich an nicht verteilt zu rechnen, sondern im Shared-Memory-Verfahren die Berechnung auf einer Grafikkarte zu machen. Diese ist auf Gleitkommaoperationen spezialisiert und bringt eine deutliche Performance-Steigerung. Tensorflow unterstützt dies bereits. 
