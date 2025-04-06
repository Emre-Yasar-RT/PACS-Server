# PACS-Server und Integration von CT-Scannern

## Zielsetzung

Das Ziel dieses Projekts war es, einen PACS-Server einzurichten und zwei CT-Scanner in das Netzwerk der Fachhochschule Muttenz zu integrieren. Dadurch sollte ermöglicht werden, auf DICOM-Bilder direkt zuzugreifen, ohne sie vorher über einen USB-Stick auf den Server mit der Brainlab-Software importieren zu müssen. Dies spart Zeit, reduziert Fehlerquellen und optimiert die Arbeitsabläufe.

## Vorgehen

### 1. Analyse und Gespräche

- **Projektaufnahme:** Das Projekt wurde mit einem Kickoff-Meeting eröffnet und alle nötigen Anforderungen für das Projekt mitgeteilt
- **Netzwerk-Team:** Es wurden Gespräche mit dem Netzwerk-Team geführt, um die notwendigen Ports für den Zugriff der CT-Geräte, des PACS-Servers und des Brainlab Systems, zu öffnen.
- **Brainlab-Software:** Ein weiteres Gespräch fand mit dem Verantwortlichen der Brainlab-Software statt, um sicherzustellen, dass diese auf das PACS-System zugreifen kann.

### 2. Anforderungen

| **Anforderung**                                                                |                                                **Dokument**                                                 |  **Status**   |                                                                                             **Bemerkungen**                                                                                              |
|--------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------:|:-------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| Einbindung der CT-Scanner (C-Boden und O-ARM) in das FHNW Muttenz Netzwerk     |                                                                                                             |    erfüllt    |              Durch die Ermittlung der MAC-Adressen der CT-Scanner und der Hilfe des Netzwerk-Teams konnten die CT-Scanner erfolgreich in das Netzwerk der FHNW Muttenz eingebunden werden.               |
| Erfolgreiche Kommunikation zwischen CT-Scanner und PACS-Server                 |                                                                                                             |    erfüllt    |                                                                                                                                                                                                          |
| Integration der UID der FHNW Muttenz in die beiden CT-Scanner                  |                                                                                                             | nicht erfüllt |             Nicht ohne weiteres machbar aber evt. mit technischem Support von Leuag AG ([Tel.: 041 618 81 00]() / [E-Mail: ziehm_service@leuag.ch]()), welcher kostenpflichtig ist, möglich              |
| Erstellung einer Netzwerkarchitektur                                           |               [Netzwerkarchitektur](netzwerkarchitektur/netzwerkarchitektur_pacs_system.pdf)                |    erfüllt    |                                                                   Dieses Dokument wurde mit Absprache von Thomas Morgenthaler erstellt                                                                   |
| Erstellung Dokumentation                                                       |                                     [Dieses Dokument](DOCUMENTATION.md)                                     |    erfüllt    |                                                                                                                                                                                                          |
| Erstellung einer Anleitung für die Serverkonfiguration                         |                                        [Anleitung](INSTALLATION.md)                                         |    erfüllt    |                                                                                                                                                                                                          |
| Erstellung eines Szenarios (Brainlab System) welches das PACS-System verwendet |                    [Dokument im Original](brainlab/pr_stereotaxy_instructions-final.pdf)                    | nicht erfüllt | Konnte nicht ausführlich getestet werden, um ein Dokument zu erstellen. <br/>(**NICHT GETESTET:** Möglich wäre ein Download eines DICOMZIP oder DICOMDIR, um es in der Brainlab Software zu importieren) |
| Erstellung Reflexionen                                                         | [Emre Yasar](reflexionen/reflexion_emre_yasar.md) / [Nicola Bodmer](reflexionen/reflexion_nicola_bodmer.md) |    erfüllt    |                                                                                                                                                                                                          |

### 3. Eingebundene Geräte

| **Hostname** | **Bezeichnung** | **IP-Adresse** |  **MAC-Adressen**  |  **Hinterlegte UID**  |  **Zu integrierende UID**  |
|:------------:|:---------------:|:--------------:|:------------------:|:---------------------:|:--------------------------:|
|   v001489    |   PACS-Server   |  10.xx.x.xxx   |   nicht benötigt   |           -           |             -              |
| MU15NS33060  | Brainlab-Server |  10.xx.xx.xxx  |   nicht benötigt   |           -           |             -              |
| MU15NU33019  |     C-Bogen     |  10.xx.xx.xx   | 00:03:2D:16:09:2B  | 2.16.840.1.113669.632 | 1.2.826.0.1.3680043.10.609 |
| MU15NU33020  |      O-ARM      |  10.xx.xx.xx   | 00:01:29:97:C3:52  |  1.2.826.0.1.3680043  | 1.2.826.0.1.3680043.10.609 |


### 4. Evaluation von PACS-Systemen

Zwei Open-Source PACS-Systeme wurden evaluiert: **Dicoogle** und **Orthanc**. Nach einer Gegenüberstellung entschieden wir uns für Orthanc, da es einfacher zu konfigurieren war.

| **Kriterium**                     | **Dicoogle**                              | **Orthanc**                        |
|-----------------------------------|-------------------------------------------|------------------------------------|
| **Einfachheit der Konfiguration** | Komplex, erfordert umfassende Anpassungen | Sehr einfach, gut dokumentiert     |
| **Unterstützte Protokolle**       | DICOM, DICOMWeb                           | DICOM, DICOMWeb                    |
| **Benutzerfreundlichkeit**        | Moderat, technisch orientiert             | Hoch, intuitiv                     |
| **Erweiterbarkeit**               | Plugins müssen manuell integriert werden  | Plugins sind leicht konfigurierbar |
| **Community-Support**             | Klein                                     | Gross                               |

### 5. Automatisierung mit Ansible

Ein **Ansible-Playbook** wurde entwickelt, um die Einrichtung von Orthanc zu automatisieren. Dabei wurden folgende Dateien für die Konfiguration eingebunden:

- `orthanc.json`: Hauptkonfigurationsdatei für Orthanc
- `credentials.json`: Benutzerauthentifizierung
- `dicomweb.json`: Konfiguration für DICOMWeb
- `orthanc.service`: Service-Definition für Systemd

Die Verwendung des Playbooks ermöglicht eine reproduzierbare und effiziente Installation von Orthanc.

### 6. Tests der DICOM-Datenübertragung

- Es wurde getestet, ob die CT-Scanner DICOM-Bilder erfolgreich an den PACS-Server senden können.
- Der Weasis DICOM Viewer wurde als Testumgebung eingesetzt, um die Bilder vom PACS-Server mit einem **C-GET** zu importieren und zu betrachten.

### 7. Einrichtung von Weasis DICOM Viewer

Zur Nutzung des Weasis DICOM Viewers wurden folgende Schritte durchgeführt:

1. Angabe von **AE-Titel**, **IP-Adresse** und **Port**.
2. Anpassung der Datei `orthanc.json`, um einen entsprechenden Eintrag unter `DicomModalities` zu erstellen.
3. Einrichtung eines DICOMWeb-Servers unter Orthanc:
    - Start eines passwortgeschützten DICOMWeb-Servers.
    - Erstellung eines Web-Nodes im Weasis Viewer.
    - Eintrag eines HTTP-Headers mit **Base64-Authentifizierung**, um den Zugriff auf den Server zu ermöglichen.

### 8. Technische Einschränkungen

#### Anpassung der UID der CT-Scanner

Eine weitere Anforderung war die Änderung der UID der beiden CT-Scanner. Nach Kontaktaufnahme mit dem technischen Support von Leuag Ag wurde festgestellt, dass dies nicht ohne weiteres möglich ist.

#### Integration der Brainlab-Software

Es war vorgesehen, die Brainlab-Software so einzurichten, dass sie direkt auf den PACS-Server zugreifen kann. Dabei traten Probleme auf:
- Die Brainlab-Software führt einen **C-MOVE** durch, was zu einem Fehler führte.
- Aufgrund fehlendem Zugang zur Software und zeitlichen Einschränkungen konnte dieses Problem nicht abschliessend analysiert und gelöst werden, daher gehen wir davon aus, dass es unter Umständen trotzem möglich sein könnte über 

## Fazit

Das Projekt hat gezeigt, dass die Integration eines PACS-Servers in das Netzwerk der Fachhochschule Muttenz und die Anbindung von CT-Scannern erfolgreich umgesetzt werden kann. Die Nutzung von Orthanc und die Automatisierung mit Ansible erwiesen sich als effizient und praktikabel.

### Noch offene Punkte

- Lösung des **C-MOVE**-Problems in der Brainlab-Software.
- Mögliche Alternativen zur Anpassung der UID der CT-Scanner.

Die Dokumentation und das entwickelte Ansible-Playbook bieten eine solide Grundlage für zukünftige Projekte in diesem Bereich.