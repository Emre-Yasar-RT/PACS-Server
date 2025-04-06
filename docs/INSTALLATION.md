# **Anleitung Installation & Konfiguration PACS DICOM Viewer**

Diese Anleitung beschreibt die Installation, Konfiguration und Nutzung eines PACS DICOM Viewers mithilfe von Ansible und Orthanc sowie die Einbindung eines CT-Scanners und die Nutzung von Weasis.

## **1. Control Node: Einrichtung des Techniker-Arbeitsgeräts**

### **1.1 Ansible installieren**
#### **1.1.1 Systemaktualisierung:**
- Stellen Sie sicher, dass das Betriebssystem auf dem neuesten Stand ist.

#### **1.1.2 Installation von Ansible:**
- Installieren Sie Ansible über das Paketverwaltungssystem des Betriebssystems.

### **1.2 SSH-Schlüssel erstellen und kopieren**
- Generieren Sie ein SSH-Schlüsselpaar, falls noch nicht vorhanden. Dies ermöglicht eine sichere Verbindung zu den Managed Nodes.
- Kopieren Sie den öffentlichen Schlüssel auf das Zielsystem (Managed Node), um passwortlose SSH-Verbindungen zu ermöglichen.

> **Hinweis:** Notieren Sie sich die IP-Adresse und den Benutzernamen des Managed Nodes für spätere Schritte.

---

## **2. Erstellung von Ansible Playbooks**

### **2.1 Playbook: `install_orthanc.yml`**
Erstellen Sie ein Playbook zur automatisierten Installation von Orthanc PACS und den dazugehörigen Plugins. Stellen Sie sicher, dass alle notwendigen Konfigurationsdateien (`orthanc.json`, `credentials.json`, `dicomweb.json` und `orthanc.service`) im Verzeichnis `files` vorliegen und vor der Ausführung an Ihre Anforderungen angepasst wurden.

Das Playbook sollte folgende Aufgaben enthalten:
- Aktualisierung der Paketquellen.
- Installation von Orthanc und den benötigten Plugins.
- Bereitstellung der vorbereiteten Konfigurationsdateien aus dem Verzeichnis `files`.
- Neustart des Orthanc-Dienstes, um Änderungen zu übernehmen.

> **Hinweis:** Passen Sie die bereitgestellten Konfigurationsdateien vor der Ausführung des Playbooks entsprechend Ihrer Anforderungen an.

### **2.2 Playbook: `uninstall_orthanc.yml` (optional)**
Optional kann ein weiteres Playbook erstellt werden, um Orthanc und alle zugehörigen Plugins rückstandslos zu deinstallieren. Dieses Playbook sollte die Entfernung von Softwarepaketen und Konfigurationsdateien sowie die Bereinigung von Log- und Datenverzeichnissen umfassen.

> **Hinweis:** Speichern Sie die Playbooks in einem zentralen Verzeichnis, das für die Verwaltung von Ansible-Projekten vorgesehen ist.

---

## **3. Erstellung und Konfiguration der Datei `orthanc.service`**

Vor der Ausführung des Playbooks muss sichergestellt sein, dass die Datei `orthanc.service` korrekt konfiguriert ist und im Verzeichnis `files` bereitliegt. Diese Datei stellt sicher, dass der Orthanc-Dienst mit den richtigen Benutzerrechten und Einstellungen ausgeführt wird.

Die Datei `orthanc.service` muss so konfiguriert werden, dass der Dienst unter dem Benutzer `root` ausgeführt wird. Dies ist erforderlich, da der Standard-Port 80 für Orthanc Root-Rechte voraussetzt. Wenn der Benutzer in der Konfiguration auf `orthanc` gesetzt ist, wird der Dienst nicht ordnungsgemäss gestartet.

> **Hinweis:** Stellen Sie sicher, dass die Systemd-Daemon nach der Erstellung oder Bearbeitung der Datei aktualisiert wird, damit die Änderungen wirksam werden.

---

## **4. Ausführung der Ansible Playbooks**

Um ein Playbook auszuführen, verwenden Sie den Befehl `ansible-playbook`. Stellen Sie sicher, dass folgende Bedingungen erfüllt sind:
- Die Playbook-Dateien (`install_orthanc.yml` und `inventory.ini`) befinden sich im selben Verzeichnis.
- Die vorbereiteten Konfigurationsdateien wurden im Verzeichnis `files` an Ihre Anforderungen angepasst.
- Die `inventory.ini` enthält die korrekten Angaben zu den Managed Nodes.

> **Wichtig:** Die Ausführung des Playbooks erfordert erhöhte Berechtigungen (z. B. durch Angabe eines sudo-Passworts).

---

## **5. Managed Node: Überprüfung von Orthanc**

Da die Konfigurationsdateien vorab angepasst und bereitgestellt wurden, sind keine weiteren manuellen Änderungen erforderlich. Das Playbook übernimmt die Bereitstellung und Konfiguration automatisch.

> **Hinweis:** Sollte dennoch eine Anpassung notwendig sein, öffnen Sie die Orthanc-Konfigurationsdateien (`orthanc.json`, `dicomweb.json`, `credentials.json`) direkt auf dem Managed Node und starten Sie den Dienst neu.

### **5.1 Statusüberprüfung:**
- Überprüfen Sie, ob der Dienst ordnungsgemäss läuft:
  ```
  sudo systemctl status orthanc
  ```

---

## **6. Einbindung eines CT-Scanners**

### **6.1 Gerät einschalten:**
- Stellen Sie sicher, dass der CT-Scanner betriebsbereit ist.

### **6.2 Netzwerkverbindung herstellen:**
- Verbinden Sie den Scanner mit dem Netzwerk, idealerweise über DHCP.

### **6.3 Gerätekonfiguration:**
- Legen Sie die folgenden Parameter fest:
    - **AE-Titel:** ORTHANC
    - **IP-Adresse:** Dynamisch zugewiesen (DHCP)
    - **Port:** 11112

> **Hinweis:** Prüfen Sie die Netzwerkverbindung und die Erreichbarkeit des PACS-Servers vom Scanner aus.

---

## **7. Client: Weasis DICOM Viewer**

### **7.1 Installation von Weasis**
Installieren Sie den Weasis DICOM Viewer auf dem Client-System. Weasis bietet eine benutzerfreundliche Oberfläche zur Visualisierung von DICOM-Daten.

### **7.2 Konfiguration des DICOMWeb-Nodes**
- Öffnen Sie Weasis und navigieren Sie zu den Einstellungen für DICOMWeb.
- Fügen Sie einen neuen DICOMWeb-Node mit folgenden Parametern hinzu:
    - **Typ:** DICOMWeb
    - **URL:** `http://<IP_PACS_Server>/dicom-web`

### **7.3 Autorisierung hinzufügen:**
- Konvertieren Sie die Anmeldedaten (Benutzername und Passwort) in ein Base64-codiertes Format.
- Fügen Sie den Header `Authorization: Basic <Base64-Code>` in den DICOMWeb-Einstellungen hinzu.

> **Hinweis:** Stellen Sie sicher, dass der PACS-Server erreichbar ist und Weasis korrekt mit diesem kommunizieren kann.

---

## **Zusätzliche Hinweise**
- **Dokumentation:** Bewahren Sie eine Kopie der Playbooks und Konfigurationsdateien sicher auf, um sie bei zukünftigen Änderungen oder Fehlerbehebungen schnell zur Hand zu haben.
- **Fehlerbehebung:** Überprüfen Sie bei Problemen die Log-Dateien auf dem PACS-Server und Client.

