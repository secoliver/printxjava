# Dokumentation: GitHub Self-Hosted Runner Setup

## 1. Instanz-Details
* **Server-IP:** `192.168.10.54`
* **Repository:** `secoliver/printxjava`
* **Runner-Name:** `ulrich-dev-runner`
* **Agent-ID:** `2`
* **Pool:** `Default` (poolId `1`)
* **Labels:** `self-hosted`, `linux`, `ulrich-dev`

## 2. Runner-Konfiguration (`.runner`)
Der registrierte Runner schreibt seine Laufzeit-Config beim `./config.sh`-Durchlauf. Ist-Stand:

```json
{
  "agentId": 2,
  "agentName": "ulrich-dev-runner",
  "poolId": 1,
  "poolName": "Default",
  "serverUrl": "https://pipelinesghubeus21.actions.githubusercontent.com/qZNNhsY5VaA9eHnYRFqX5bfgD0Fk7nvQE8DygAKxWiTYy54N01/",
  "gitHubUrl": "https://github.com/secoliver/printxjava",
  "workFolder": "_work",
  "useV2Flow": true,
  "serverUrlV2": "https://broker.actions.githubusercontent.com/"
}
```

*Hinweis:* Änderungen an `agentName` oder Labels erfordern `./config.sh remove` gefolgt von einer erneuten `./config.sh`-Registrierung — die Datei darf nicht manuell editiert werden.

## 3. Pfade & Verzeichnisse
* **Installations-Ordner:** `/home/<user>/actions-runner/`
* **Arbeitsverzeichnis (Work folder):** `/home/<user>/actions-runner/_work/`
    * *Hinweis:* Hier speichert der Runner die temporären Build-Daten und checkt den Code aus. Pro Repository entsteht ein Unterordner `_work/printxjava/printxjava/`, in dem die `docker-compose.yml` aus dem Checkout liegt — der Deploy-Job nutzt diese Kopie.

## 4. Konfiguration & Installation
Der Runner wurde mit den folgenden Schritten auf dem Server `192.168.10.54` eingerichtet:

1. **Registrierung:**
   Ausführung von `./config.sh` mit der Zuweisung zum Repository `secoliver/printxjava` (Registrierungstoken aus GitHub → Settings → Actions → Runners → *New self-hosted runner*).
2. **Labels:**
   Während `./config.sh` wurde neben den Default-Labels zusätzlich `ulrich-dev` gesetzt, um die Development-Umgebung eindeutig im Workflow adressieren zu können.
3. **Persistenz:**
   Einrichtung als System-Service (`systemd`), damit der Runner nach einem Reboot automatisch startet.
4. **Docker-Zugriff:**
   Der Runner-User ist in der `docker`-Gruppe (`sudo usermod -aG docker <user>`), sodass der Deploy-Job den Docker-Socket ohne `sudo` nutzen kann.

## 5. Management-Befehle
Befehle auszuführen im Verzeichnis `~/actions-runner/`:

| Aktion | Befehl |
| :--- | :--- |
| **Status prüfen** | `sudo ./svc.sh status` |
| **Dienst starten** | `sudo ./svc.sh start` |
| **Dienst stoppen** | `sudo ./svc.sh stop` |
| **Dienst deinstallieren** | `sudo ./svc.sh uninstall` |
| **Runner entregistrieren** | `./config.sh remove --token <token>` |

## 6. Workflow-Integration
In der GitHub Actions YAML-Datei muss folgender Identifier für den Deploy-Job genutzt werden:

```yaml
jobs:
  deploy:
    runs-on: [self-hosted, linux, ulrich-dev]
    steps:
      - uses: actions/checkout@v4
```

Der Build-Job läuft weiterhin auf `ubuntu-latest` (GitHub-hosted), nur der Deploy-Schritt nutzt den self-hosted Runner — siehe `.github/workflows/build-deploy.yml`.
