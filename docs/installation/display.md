# Raspberry-Pi-Display installieren

Diese Anleitung richtet einen neuen Produktions-Pi reproduzierbar aus einem
versionierten GitHub Release ein. Auf Produktionsgeräten werden weder ein
Git-Checkout noch Poetry oder Docker verwendet. Der Installer legt für jede
Version eine eigene virtuelle Python-Umgebung aus den mitgelieferten ARM64-
Wheels an.

Die Beispielversion ist `2026.07.21-1`. Für eine reale Installation überall die
gewünschte Release-Version einsetzen.

## Systemanforderungen

### Hardware

Unterstütztes Zielsystem:

- Raspberry Pi mit 64-Bit-ARM-Prozessor und Debian-Architektur `arm64`,
- empfohlen Raspberry Pi 4 oder neuer mit mindestens 4 GB RAM,
- mindestens 8 GB freier Speicher,
- zwei HDMI-Ausgänge für getrennte Primary- und Luma-Ausgabe,
- 1920 × 1080 auf beiden Ausgängen,
- Netzwerkzugriff auf Backend, Medien- und Theme-Hosts,
- für öffentliche Releases zusätzlich HTTPS-Zugriff auf GitHub.

Eine kabelgebundene Netzwerkverbindung wird empfohlen. Der Referenz-Pi besitzt
4 GB RAM, zwei angeschlossene 1920×1080-Ausgänge, Ethernet und einen
117-GB-Datenträger.

### Betriebssystem

Verbindliche Referenz ist Raspberry Pi OS beziehungsweise Debian 13 „Trixie“
für ARM64 mit Python 3.13, systemd und KMS-Grafikstack. Verifiziert ist:

```text
Debian GNU/Linux 13.5
Kernel 6.18.34+rpt-rpi-v8
Python 3.13.5
Architektur aarch64 / arm64
```

Nicht unterstützt sind 32-Bit Raspberry Pi OS, `armhf`, Windows, macOS oder
Docker als Produktionssystem. Ebenso nicht unterstützt ist ein Python außerhalb
der virtuellen Umgebung des jeweiligen Releases.

Vorbereitung prüfen:

```bash
dpkg --print-architecture
python3 --version
systemctl --version
df -h /
timedatectl status
```

`dpkg --print-architecture` muss `arm64` liefern. Die Systemzeit muss per NTP
synchronisiert sein, damit HTTPS und zeitbasierte API-Abläufe zuverlässig
funktionieren.

## Grafik und HDMI vorbereiten

In `/boot/firmware/config.txt` müssen diese KMS-Einstellungen vorhanden sein:

```ini
dtoverlay=vc4-kms-v3d
max_framebuffers=2
disable_overscan=1
```

Änderungen daran erfordern einen Neustart. Der Produktionsbetrieb ist für
1920×1080 bei 50 Hz vorgesehen. HDMI-Auflösung und Bildfrequenz werden vom
Betriebssystem konfiguriert, nicht vom Theme. Beide Displays möglichst vor dem
Boot anschließen und danach prüfen:

```bash
ls -l /dev/dri
cat /sys/class/drm/card*-HDMI-A-*/status
cat /sys/class/drm/card*-HDMI-A-*/modes
```

Für beide verwendeten HDMI-Anschlüsse muss der Status `connected` sein und die
Modusliste `1920x1080` enthalten. Primary und Luma sind getrennte Ausgänge; ihre
Zuordnung erfolgt später über `DISPLAY_INDEX` und `LUMA_DISPLAY_INDEX`.
Die Modusliste bestätigt nur die verfügbare Auflösung. Die tatsächlich aktive
Bildfrequenz zusätzlich mit dem Grafikwerkzeug des Betriebssystems prüfen, zum
Beispiel mit `modetest -c`, falls `modetest` installiert ist. Beide Ausgänge
müssen vor dem Dienststart auf 1920×1080 bei 50 Hz stehen.

## Benutzer und Geräterechte

Der Installer und die systemd-Unit erwarten den Benutzer `pi`. Er muss Zugriff
auf Grafik- und Eingabegeräte besitzen:

```bash
id pi
```

Fehlen Gruppen, ergänzen:

```bash
sudo usermod -aG video,render,input pi
```

Danach neu anmelden oder den Pi neu starten, bevor der Display-Dienst getestet
wird.

## Netzwerk prüfen

Der Pi benötigt Namensauflösung, korrekte Systemzeit und HTTPS-Zugriff auf alle
konfigurierten Gegenstellen:

- Backend aus `API_BASE_URL`,
- jeden Host aus `ALLOWED_MEDIA_HOSTS`,
- jeden zusätzlichen Host aus `ALLOWED_THEME_HOSTS`,
- für die Online-Installation `github.com` und dessen Release-Downloads.

Vor der Installation beispielhaft prüfen:

```bash
timedatectl status
getent hosts github.com
curl --fail --show-error https://github.com/ -o /dev/null
```

Nach dem Eintragen von `API_BASE_URL` zusätzlich den öffentlichen Health-Pfad
des Backends abrufen. Medien- und Theme-CDNs müssen aus demselben Netz erreichbar
sein; die Allowlist allein stellt keine Netzwerkverbindung her.

## Release-Inhalt

Ein GitHub Release enthält:

```text
paws-display-VERSION-linux-aarch64.tar.gz
paws-display-VERSION-linux-aarch64.tar.gz.sha256
install.sh
```

Das Archiv enthält Client, lokale Fallback-Themes, systemd-Unit,
Konfigurationsvorlage, gesperrte ARM64-Wheels und `RELEASE.json`. Zusätzlich
liegt die Xorg-Unit `east-display-x.service` bei, die den lokalen X-Server auf
`vt7` startet und beide HDMI-Ausgänge auf 1920×1080 bei 50 Hz setzt. Der
Installer stammt aus `install/install.sh` des Display-Repositories.

## Verzeichnisstruktur

Programmversionen, Gerätekonfiguration und Laufzeitdaten sind getrennt:

```text
/opt/paws-on-stream/
├── releases/
│   ├── VERSION-1/
│   └── VERSION-2/
└── current -> releases/VERSION-2/

/etc/paws-on-stream/
└── display.env

/var/lib/paws-on-stream/
├── messages.db
├── media-cache/
└── theme-cache/

/usr/local/sbin/
└── paws-display-install
```

- `releases/` enthält unveränderliche Programmversionen.
- `current` zeigt atomar auf die aktive Version.
- `display.env` enthält Gerätekonfiguration und Secrets.
- `/var/lib/paws-on-stream` bleibt über Updates und Rollbacks erhalten.
- `paws-display-install` dient anschließend für Status, Update und Rollback.

Ein Release überschreibt niemals Konfiguration, Datenbank, Medien- oder
Theme-Cache.

## Erstinstallation aus einem öffentlichen GitHub Release

Installer herunterladen und vor der Ausführung optional prüfen:

```bash
curl -LO \
  https://github.com/paws-on-stream/display/releases/download/v2026.07.21-1/install.sh
less install.sh
```

Installation starten:

```bash
sudo bash install.sh install --version 2026.07.21-1
```

Der Installer:

1. verlangt Root-Rechte und prüft Debian `arm64` sowie den Benutzer `pi`,
2. installiert die benötigten Debian-Laufzeitpakete,
3. lädt Archiv und Sidecar-Prüfsumme über TLS,
4. validiert SHA-256, Release-Struktur und Version,
5. legt ein neues, unveränderliches Release-Verzeichnis an,
6. erzeugt dessen `.venv` und installiert nur aus dem Wheelhouse,
7. führt als Benutzer `pi` einen Importtest aus,
8. migriert bei Bedarf den alten Laufzeitzustand,
9. installiert Konfigurationsvorlage und systemd-Unit,
10. installiert auch die Xorg-Unit für den lokalen Grafikstack,
11. schaltet `current` atomar um,
12. aktiviert und startet beide systemd-Units,
13. prüft Readiness und stellt bei einem fehlgeschlagenen Update das vorherige
    Release wieder her,
14. räumt standardmäßig ältere Releases bis auf die neuesten drei auf.

Bei einer echten Erstinstallation erzeugt der Installer zunächst
`/etc/paws-on-stream/display.env` und startet aus Sicherheitsgründen noch nicht,
weil Tokens und Gerätewerte leer sind. Das ist erwartetes Verhalten. Die
Xorg-Unit wird trotzdem installiert und beim ersten späteren Start gemeinsam
mit `east-display.service` aktiviert.

## Installation aus einem lokalen Artefakt

Diese Variante wird für private Repositories und Pis ohne GitHub-Zugriff
empfohlen. Bei einer Erstinstallation zusätzlich das `install.sh` desselben
Releases übertragen; bei späteren Updates ist der Installer bereits unter
`/usr/local/sbin` vorhanden:

```bash
scp \
  paws-display-2026.07.21-1-linux-aarch64.tar.gz \
  paws-display-2026.07.21-1-linux-aarch64.tar.gz.sha256 \
  install.sh \
  pi@DISPLAY-IP:/tmp/
```

Auf einem neuen Pi installieren:

```bash
less /tmp/install.sh
sudo bash /tmp/install.sh install \
  --version 2026.07.21-1 \
  --artifact /tmp/paws-display-2026.07.21-1-linux-aarch64.tar.gz
```

Bei einem bereits eingerichteten Pi kann stattdessen die installierte Kopie
verwendet werden:

```bash
sudo /usr/local/sbin/paws-display-install install \
  --version 2026.07.21-1 \
  --artifact /tmp/paws-display-2026.07.21-1-linux-aarch64.tar.gz
```

Liegt die Sidecar-Datei direkt als
`paws-display-…tar.gz.sha256` neben dem Archiv, liest der Installer sie
automatisch. Alternativ den erwarteten Hash explizit übergeben:

```bash
sudo /usr/local/sbin/paws-display-install install \
  --version 2026.07.21-1 \
  --artifact /tmp/paws-display-2026.07.21-1-linux-aarch64.tar.gz \
  --sha256 ERWARTETE_SHA256
```

Die Prüfsummenprüfung niemals umgehen. Für private GitHub Releases unterstützt
der Installer zwar einen temporären `GITHUB_TOKEN`; die lokale Übertragung ist
zu bevorzugen. Tokens nicht in Shell-History, `display.env` oder
Installationsverzeichnissen speichern.

## Installation ohne automatischen Start

Für die erste Hardwareprüfung ausdrücklich ohne Start installieren:

```bash
sudo bash install.sh install \
  --version 2026.07.21-1 \
  --no-start
```

Danach die [Display-Konfiguration](../configuration.md#display-client) bearbeiten:

```bash
sudoedit /etc/paws-on-stream/display.env
sudo chown root:pi /etc/paws-on-stream/display.env
sudo chmod 640 /etc/paws-on-stream/display.env
```

Anschließend starten:

```bash
sudo systemctl enable east-display.service
sudo systemctl start east-display.service
```

## Alte Installation migrieren

Der erste Installerlauf übernimmt vorhandene Daten aus
`/opt/paws_on_stream/display`, sofern das jeweilige neue Ziel noch nicht
existiert:

```text
/opt/paws_on_stream/display/.env
  -> /etc/paws-on-stream/display.env

/opt/paws_on_stream/display/messages_cache.db
  -> /var/lib/paws-on-stream/messages.db

/opt/paws_on_stream/display/media-cache/
  -> /var/lib/paws-on-stream/media-cache/
```

Vorher den alten Dienst stoppen und eine Sicherung von `.env`, Datenbank und
Mediencache erstellen. Der Installer löscht die alte Installation nicht. Er
normalisiert in der übernommenen Konfiguration die Pfade für Datenbank, Medien,
lokale Themes und Theme-Cache auf die neue Struktur.

Die alte Installation erst manuell archivieren oder entfernen, nachdem
Produktionsbetrieb, Neustart und ein Rollback auf dem neuen Layout vollständig
getestet wurden.

## Abnahme nach der Installation

1. [Konfiguration](../configuration.md#display-client) vollständig prüfen.
2. Beide DRM-/HDMI-Ausgänge erneut kontrollieren.
3. Dienst und Readiness prüfen.
4. Aktives Release und `current`-Symlink kontrollieren.
5. Text, statisches und animiertes WebP in beiden Darstellungsmodi testen.
6. Display-Ack im Backend prüfen.
7. Primary- und Luma-Ausgabe vergleichen.
8. Pause, Resume, Clear und Killswitch testen.

```bash
sudo /usr/local/sbin/paws-display-install status
curl http://127.0.0.1:8765/health
curl http://127.0.0.1:8765/readiness
readlink -f /opt/paws-on-stream/current
```

Betrieb, Updates, Rollback und Datensicherung stehen im
[Display-Betriebshandbuch](../operations/display.md).
