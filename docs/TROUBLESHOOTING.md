# Troubleshooting-Dokument: PACS DICOM Viewer Projekt

Dieser Abschnitt beschreibt mögliche Probleme, die während der Installation, Konfiguration und Nutzung des PACS DICOM Viewers auftreten können, sowie die entsprechenden effektiven Lösungen basierend auf den tatsächlichen Herausforderungen des Projekts.

## 1. Ansible Playbook Probleme

### Problem 1: SSH-Authentifizierung fehlgeschlagen
**Symptom:** Beim Ausführen eines Playbooks erhalten Sie eine Fehlermeldung wie „Permission denied“ oder „Authentication failed“.

**Lösung:**
1. Erstellen Sie einen neuen SSH-Schlüssel:
   ```
   ssh-keygen -t rsa -b 4096
   ```
2. Kopieren Sie den öffentlichen Schlüssel auf das Managed Node:
   ```
   ssh-copy-id <username>@<IP_Managed_Node>
   ```
3. Überprüfen Sie, ob der Benutzer auf dem Managed Node in der Datei `/etc/ssh/sshd_config` unter `AllowUsers` aufgeführt ist. Starten Sie den SSH-Dienst neu:
   ```
   sudo systemctl restart sshd
   ```

### Problem 2: Fehler beim Aktualisieren der APT-Quellen
**Symptom:** Das Playbook scheitert bei der Aktualisierung des APT-Caches.

**Lösung:**
1. Prüfen Sie die Internetverbindung des Managed Nodes:
   ```
   ping -c 4 8.8.8.8
   ```
2. Aktualisieren Sie die Paketquellen manuell:
   ```
   sudo apt update
   ```
3. Überprüfen Sie, ob Proxy-Einstellungen in `/etc/apt/apt.conf` erforderlich sind und passen Sie diese an.

---

## 2. Probleme bei der Orthanc-Installation

### Problem 1: Orthanc-Dienst startet nicht
**Symptom:** Der Orthanc-Dienst startet nach der Installation nicht oder stürzt sofort ab.

**Lösung:**
1. Analysieren Sie die Orthanc-Logs:
   ```
   sudo journalctl -u orthanc
   ```
2. Validieren Sie die Datei `/etc/orthanc/orthanc.json` mit einem JSON-Validator (z. B. `jq`):
   ```
   jq . /etc/orthanc/orthanc.json
   ```
3. Stellen Sie sicher, dass keine Ports von anderen Anwendungen blockiert werden:
   ```
   sudo netstat -tuln | grep 8042
   ```
4. Starten Sie den Dienst:
   ```
   sudo systemctl restart orthanc
   ```

### Problem 2: Plugins fehlen oder funktionieren nicht
**Symptom:** Funktionen wie DICOMWeb oder MySQL-Unterstützung sind nicht verfügbar.

**Lösung:**
1. Installieren Sie fehlende Plugins:
   ```
   sudo apt install orthanc-dicomweb orthanc-mysql
   ```
2. Aktivieren Sie die Plugins in der Datei `orthanc.json` unter dem Schlüssel `Plugins` und prüfen Sie, ob die Pfade korrekt sind.
3. Testen Sie die Funktionalität des Plugins, z. B.:
   ```
   curl http://localhost:8042/dicom-web
   ```

---

## 3. Konfigurationsprobleme

### Problem 1: Netzwerkprobleme zwischen CT-Scanner und PACS
**Symptom:** Der CT-Scanner kann keine Verbindung zum PACS-Server herstellen.

**Lösung:**
1. Überprüfen Sie die IP-Adresse des PACS-Servers und stellen Sie sicher, dass sie statisch ist.
2. Testen Sie die Verbindung zwischen dem Scanner und dem PACS-Server:
   ```
   telnet <IP_PACS_Server> 11112
   ```
3. Falls der Port blockiert ist, wenden Sie sich an Ihre IT-Abteilung und erstellen Sie ein Ticket, um die entsprechende Firewall-Regel hinzufügen zu lassen.
4. Überprüfen Sie die Konfiguration des AE-Titels auf dem PACS-Server und auf dem Scanner.

### Problem 2: Fehlerhafte JSON-Syntax
**Symptom:** Orthanc gibt eine Fehlermeldung zu ungültigen JSON-Dateien aus.

**Lösung:**
1. Validieren Sie die Datei mit einem Online-JSON-Validator oder lokal mit:
   ```
   jq . /etc/orthanc/orthanc.json
   ```
2. Beheben Sie Syntaxfehler wie fehlende oder zusätzliche Kommata und führen Sie einen Neustart des Dienstes durch.

---

## 4. Probleme mit Weasis DICOM Viewer

### Problem 1: Verbindung zu DICOMWeb nicht möglich
**Symptom:** Weasis kann keine Verbindung zur DICOMWeb-URL herstellen.

**Lösung:**
1. Stellen Sie sicher, dass DICOMWeb auf dem PACS-Server aktiviert ist und die URL korrekt konfiguriert wurde.
2. Testen Sie die Erreichbarkeit der URL:
   ```
   curl http://<IP_PACS_Server>/dicom-web
   ```
3. Aktualisieren Sie die Konfigurationsdatei von Weasis, um die Base64-codierten Anmeldedaten korrekt einzufügen.

### Problem 2: Authentifizierungsfehler
**Symptom:** Weasis zeigt "Authentication failed".

**Lösung:**
1. Generieren Sie die Base64-codierten Anmeldedaten korrekt:
   ```
   echo -n "<username>:<password>" | base64
   ```
2. Prüfen Sie, ob die Benutzeranmeldedaten in der Datei `credentials.json` auf dem PACS-Server registriert sind.
3. Testen Sie die Anmeldung manuell mit einem HTTP-Client (z. B. Postman oder curl).

---

## 5. Allgemeine Fehlerbehebung

### Problem 1: Speichermangel auf dem PACS-Server
**Symptom:** Orthanc stoppt unerwartet oder gibt Warnungen zu geringem Speicherplatz aus.

**Lösung:**
1. Entfernen Sie alte Logs und ungenutzte Daten:
   ```
   sudo rm -rf /var/log/orthanc/*
   ```
2. Überwachen Sie den Speicherplatz mit Tools wie `df -h` und richten Sie Benachrichtigungen ein.

### Problem 2: Benutzerberechtigungen unzureichend
**Symptom:** Zugriff auf PACS-Daten oder Funktionen wird verweigert.

**Lösung:**
1. Prüfen Sie die Berechtigungen der Konfigurationsdateien:
   ```
   ls -l /etc/orthanc
   ```
2. Aktualisieren Sie die Datei `credentials.json`, um fehlende Benutzer hinzuzufügen:
   ```json
   {
      "RegisteredUsers": {
         "admin": "securepassword"
      }
   }
   ```
3. Starten Sie den Dienst, um Änderungen zu übernehmen:
   ```
   sudo systemctl restart orthanc
   ```

Falls weitere Probleme auftreten, konsultieren Sie die Projektunterlagen oder kontaktieren Sie den technischen Support.