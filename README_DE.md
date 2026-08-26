<div align="center">

# AIpolloSync: Remote P2P-Medienserver & Streaming-Skill für Hermes

[English](README.md) | [简体中文](README_ZH.md) | [日本語](README_JA.md) | [Deutsch](README_DE.md) | [Español](README_ES.md)

</div>

---

## 📖 Übersicht & Kernfunktionalität

**AIpolloSync** ist ein persönlicher Medien-Fernzugriffsdienst. Er startet einen lokalen Flask + WebRTC-Medienserver, baut einen ausgehenden FRP-Tunnel für den öffentlichen Internetzugang auf und stellt Ihre lokalen Videodateien über eine WhatsApp-integrierte KI-Agenten-Schnittstelle bereit.

**Kernwert**: Greifen Sie jederzeit und überall über WhatsApp mit Hermes auf Ihre entfernte Medienbibliothek zu. Medienwiedergabelisten unterstützen die Wiedergabe mit dem AIpollo Player.

---

## ⚙️ Systemanforderungen

* **Hermes**: Hermes ist lokal installiert und einsatzbereit.
* **Firewall- / Antivirus-Freigabe**: Fügen Sie `frpc.exe` als Ausnahme in Windows Defender oder Ihrer Sicherheitssoftware hinzu.

---

## 🚀 Schritt-für-Schritt Installationsanleitung

1. **Installation**: Laden Sie den `AIpolloSync`-Skill über HermesHub herunter und installieren Sie ihn.
2. **Medienordner anlegen**: Erstellen Sie im Skill-Verzeichnis einen Ordner namens `videos` und hinterlegen Sie dort die MP4-Videodateien.
3. **WhatsApp konfigurieren**: Richten Sie die WhatsApp-Schnittstelle in Hermes ein.
4. **Skill ausführen**: Starten Sie den Skill.
5. **LLM-gesteuerte Interaktion**: Chatten Sie natürlich mit der KI in Ihrem Kanal. Zum Beispiel:
   - *"Zeig mir meine Videoliste."*
   - *"Habe ich einen Film zum Anschauen?"*
   - *"Spiel das Video mit der Katze ab."*
6. **Wiedergabe starten**: Klicken Sie auf den generierten Link in der Antwort, um Ihr Video abzuspielen.

---

## 🔒 Sicherheits- und Netzwerkoffenlegung

### Kritisch: Der FRP-Tunnel setzt lokale Dienste dem öffentlichen Internet aus

Dieser Skill lädt beim Start **automatisch** den **FRP (Fast Reverse Proxy)-Client (`frpc`)** herunter und führt ihn aus. Die `frpc`-Binärdatei wird von GitHub Releases bezogen und stellt einen ausgehenden Tunnel zu einem entfernten FRP-Server (`129.213.174.213:7000`) her, der wiederum Ihren lokalen Mediendienst (Port 8000) über eine `*.yunfrp.net`-Subdomain dem **öffentlichen Internet** zugänglich macht.

**Dies vergrößert Ihre Angriffsfläche erheblich.** Jeder, der die öffentliche Subdomain kennt oder entdeckt, kann versuchen, auf Ihre Mediendateien und den Flask-Dienst auf Ihrem Rechner zuzugreifen.

### 1. Automatisches Tunnelverhalten (Kein Opt-in)

- **Automatisch beim Start**: Der FRP-Tunnel startet automatisch, wenn `scripts/media_server_flask.py` ausgeführt wird. Es gibt keine Eingabeaufforderung, keine Bestätigung und keine Umgebungsvariablen-Steuerung.
- **Binär-Download**: Beim ersten Start wird `frpc.exe` automatisch von GitHub (`fatedier/frp` Releases) heruntergeladen. Eine Internetverbindung ist erforderlich.
- **Keine Firewall-Änderungen**: Der Tunnel ist nur ausgehend; es müssen keine eingehenden Ports in der Firewall geöffnet werden.

### 2. Lieferkettenrisiko: Ausführung heruntergeladener Binärdateien

- Der Skill lädt eine native Binärdatei (`frpc.exe`) von GitHub Releases herunter und führt sie aus. Eine Kompromittierung des GitHub-Repositorys, des Release-Artefakts oder des Netzwerktransports (MITM) könnte zu **beliebiger Codeausführung** auf Ihrem Host mit denselben Rechten wie der Python-Prozess führen.
- **SHA256-Prüfsummen-Verifikation**: Der Code enthält fest codierte SHA256-Prüfsummen sowohl für das Zip-Archiv als auch für die extrahierte `frpc.exe`-Binärdatei (Version `0.65.0`). Der Download wird abgelehnt, wenn eine Prüfsumme nicht übereinstimmt. Dies schützt vor Transportmanipulation und beschädigten Downloads, **schützt jedoch nicht vor einer Kompromittierung des vorgelagerten GitHub-Repositorys oder Releases**.
- **Versionssperre**: Die FRP-Version ist auf `0.65.0` festgelegt. Ein Upgrade erfordert eine Codeänderung und erneute SHA256-Überprüfung. Dies verhindert stille Upgrades auf potenziell kompromittierte neuere Versionen.

### 3. Authentifizierungsstatus

- **Keine Authentifizierung implementiert**: Der Flask-Server verfügt derzeit über **keine HTTP Basic Auth, keinen Token-Mechanismus und keine Zugriffskontrolle**. Alle API-Routen und Medienendpunkte sind für jeden öffentlich zugänglich, der den Server erreicht — sei es über LAN oder den FRP-Tunnel.
- **Risiko**: Ein nicht authentifizierter Dritter, der die `*.yunfrp.net`-Subdomain entdeckt, kann Mediendateien auf Ihrem Rechner auflisten und herunterladen.

### 4. Vertrauen in den Remote-Server

- Der FRP-Server unter `129.213.174.213:7000` ist ein Drittanbieter-Relay. Der gesamte Datenverkehr zwischen dem öffentlichen Internet und Ihrem lokalen Dienst läuft über diesen Server.
- Der FRP-Tunnel arbeitet im HTTP-Modus (keine TLS-Terminierung durch den FRP-Server).
- Sie müssen darauf vertrauen, dass der Betreiber dieses FRP-Servers Ihren Datenverkehr nicht inspiziert, protokolliert oder manipuliert.

---

## 🛡️ Sicherheitsempfehlungen

* **Dedizierte Hardware / Virtuelle Maschine**: Für maximale Sicherheit empfehlen wir, diesen Dienst auf einem separaten Server (z. B. Homelab/NAS) oder in einer isolierten virtuellen Maschine (VM) zu betreiben.
* **Regelmäßige Updates**: Halten Sie Ihr Betriebssystem und Ihre Hermes-Umgebung stets auf dem neuesten Stand.

---

## 💻 Plattformkompatibilität

* **Aktuell unterstützt**: Windows (x64)
* **In Entwicklung**: Linux / macOS Support folgt in Kürze.

*Bei Fragen oder Problemen öffnen Sie bitte ein GitHub-Issue. Vielen Dank für Ihr Vertrauen!*