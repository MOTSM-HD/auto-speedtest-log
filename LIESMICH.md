# auto-speedtest-log
Einen automatischen Internet Speedtest durchführen, die Daten loggen und alle 24 Stunden ein Diagramm erstellen.

Die folgende Anleitung erstellt einen automatisierten Ablauf der in festgelegten Abständen einen internet Speedtest mit www.speedtest.net durchführt. Die erhaltenen Daten werden in eine CSV Datei geschrieben und einmal am Tag für die letzten 24 Stunden in ein Diagramm exportiert

Folge diesen Schritten zum Einrichten:

## 0) Vorraussetzung: Laufender Linux-Rechner mit installierter python-Unterstützung (sollte bei den meisten Distributionen enthalten sein) und bestehender Internetverbindung
Bei mir läuft es auf einem RaspberryPi mit Raspbian 10 Buster als Betriebssystem. Das eigene Betriebssystem kann man erfahren mit:
```
cat /etc/os-release
```
Erstellen eines Arbeitsverzeichnisses
```
mkdir /path/to/speedtest
```
In dieses Verzeichnis wechseln
```
cd /path/to/speedtest
```
Im Rest dieser Anleitung wird angenommen, dass man sich in diesem Verzeichnis befindet.  
## 1) speedtest.py installieren
Dieses Projekt benutzt speedtest.py von https://github.com/sivel/speedtest-cli . Man kann es herunterladen und ausführbar machen mit:
```
wget https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
sudo chmod +x speedtest.py
```
Zum Testen kann man es ausführen: 
```
./speedtest.py
```
Die Ausgabe sollte etwa so aussehen:
```
Retrieving speedtest.net configuration...
Testing from [provider] (your.IP)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by [server hosting] (location) [distance km]: 26.317 ms
Testing download speed................................................................................
Download: 9.79 Mbit/s
Testing upload speed................................................................................................
Upload: 5.43 Mbit/s
```
## 2) Logfile vorbereiten
```
./speedtest.py --csv-header > speedtest.log
```
Erzeugt das logfile `speedtest.log` und schreibt die Spaltenüberschriften in die erste Zeile.
```
cat speedtest.log
Server ID,Sponsor,Server Name,Timestamp,Distance,Ping,Download,Upload,Share,IP Address
```
Den Speedtest-log testen mit: _(das dauert einen Moment)_
```
./speedtest.py --csv >> speedtest.log
```
dann mit `cat` den logfile anzeigen lassen:
```
cat speedtest.log
Server ID,Sponsor,Server Name,Timestamp,Distance,Ping,Download,Upload,Share,IP Address
10010,[sponsor.url],[location],2019-08-02T19:51:52.311122Z,13.089912652947309,28.738,12271838.6438877,5826553.658862884,,IP.##.##.##
```
Das ist der Anfang des logfiles. __Achtung: der timestamp in den Logfiles ist in [UTC](https://de.wikipedia.org/wiki/Koordinierte_Weltzeit)!__

Zum Leeren des logfiles einfach mit den csv-headern überschreiben: `./speedtest.py --csv-header > speedtest.log`. _(Achtung: `>` überschreibt den gesamten Inhalt der Datei, `>>` hängt eine Zeile an die bestehende Datei hinten an.)_
## 3) gnuplot und das gnuplot-script installieren
In Debian-basierten Systemen (wie raspbian) installiert man gnuplot mit:
```
sudo apt-get install gnuplot
```
Herunterladen des vorbereiteten gnuplot-scripts: 
```
wget https://raw.githubusercontent.com/MOTSM-HD/auto-speedtest-log/master/gnuplot_speedtest
```
das script muss entsprechend den eigenen Anforderungen angepasst werden, insbesondere das Arbeitsverzeichnis (working directory). Englische Kommentare sind zur Erläuterung in dem script vorhanden. 

## 4) Zeitgesteuerte Aufgaben mit cron einrichten
Man sollte die crontab immer mit `crontab -e` bearbeiten. Auf diese Art werden Änderungen nach dem Speichern direkt aktiv ohne dass man cron neu starten muss. Jede cron-Aufgabe (job) ist eine eigene Zeile in der crontab. _Die crontab kann leer sein zu Beginn!_
Wir benötigen die folgenden zwei cronjobs:
```
*/30 * * * * /path/to/speedtest/speedtest.py --csv >> /path/to/speedtest/speedtest.log
5 2 * * * gnuplot /path/to/speedtest/gnuplot_speedtest
```
Der erste führt den Speedtest aus und hängt die Ergebnisse an den Logfile an. Das ganze passiert nach dieser Konfiguration zwei mal pro Stunde, genauer: *in jeder Stunde um 00 und 30 Minuten".

Der zweite führt einmal pro Tag gnuplot mit dem script aus. Nach dieser Konfiguration um 02:05 Uhr. Als Ergebnis wird ein Diagramm für die letzten 24 Stunden als .png Datei ausgegeben.

Zum Testen ob der Speedtest von cron gestartet wird kann man den Intervall verkürzen:
```
*/5 * * * * /path/to/speedtest/speedtest.py --csv >> /path/to/speedtest/speedtest.log
```
So wird der Speedtest alle 5 Minuten ausgeführt. Mit `stat speedtest.log` kann man nachsehen wann die Datei zuletzt bearbeitet wurde.
