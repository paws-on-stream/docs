# Display-Betrieb und Wiederherstellung

Diese Seite beschreibt den laufenden Betrieb eines über ein versioniertes
Release installierten Raspberry-Pi-Displays. Die einmalige Vorbereitung steht
unter [Raspberry-Pi-Display installieren](../installation/display.md).

## Service steuern

```bash
# Status
sudo systemctl status east-display-x.service
sudo systemctl status east-display.service

# Start und Stop
sudo systemctl start east-display-x.service
sudo systemctl stop east-display-x.service
sudo systemctl start east-display.service
sudo systemctl stop east-display.service

# Neustart
sudo systemctl restart east-display-x.service
sudo systemctl restart east-display.service

# Autostart
sudo systemctl enable east-display-x.service
sudo systemctl enable east-display.service

# Laufende Logs
sudo journalctl -u east-display-x.service -f
sudo journalctl -u east-display.service -f

# Letzte 100 Einträge
sudo journalctl -u east-display-x.service -n 100 --no-pager
sudo journalctl -u east-display.service -n 100 --no-pager
```

`east-display-x.service` startet den lokalen Xorg-Server, setzt die beiden
HDMI-Ausgänge und muss funktionieren, bevor der eigentliche Client sinnvoll
starten kann. `east-display.service` startet anschließend die Python-App.

Die Unit startet direkt:

```text
/opt/paws-on-stream/current/.venv/bin/python -m display_client.main
```

Das Arbeitsverzeichnis ist `/opt/paws-on-stream/current`, die Konfiguration
wird aus `/etc/paws-on-stream/display.env` geladen und der Prozess läuft als
Benutzer und Gruppe `pi`. Poetry ist nicht installiert und nicht am Dienststart
beteiligt.

## Health und Readiness

Bei aktivierter lokaler Regie-API:

```bash
curl http://127.0.0.1:8765/health
curl http://127.0.0.1:8765/readiness
```

Beide Antworten enthalten die relevanten Laufzeitangaben:

| Feld | Bedeutung |
| --- | --- |
| `status` | `ok` bei Health, `ready` bei erfolgreicher Readiness |
| `renderer_ready` | Pygame-Textbackend ist verwendbar |
| `theme` | Name des aktuell aktiven Themes |
| `outputs` | Anzahl der geöffneten Ausgänge |
| `mode` | `running`, `paused` oder `killswitched` |
| `pending_acks` | lokal wartende Display-Bestätigungen, maximal bis 1000 gezählt |

Readiness antwortet mit `503`, wenn `renderer_ready` falsch ist. Bei aktiviertem
Luma-Modus sollte `outputs` den Wert `2` haben. Ein wachsender Wert bei
`pending_acks` weist meist auf ein Backend-, Netzwerk- oder Tokenproblem hin.

Ist `REGIE_API_ENABLED=false`, gibt es diese HTTP-Prüfung nicht. Installer und
Updates prüfen dann nach drei Sekunden, ob der systemd-Dienst weiterhin aktiv
ist.

## Aktive Version prüfen

```bash
sudo /usr/local/sbin/paws-display-install status
readlink -f /opt/paws-on-stream/current
ls -la /opt/paws-on-stream/releases
```

`status` zeigt den aufgelösten Releasepfad und anschließend den systemd-Status.

## Update installieren

Ein Update ist die Installation einer neuen Version:

```bash
sudo /usr/local/sbin/paws-display-install install \
  --version 2026.08.02-1
```

Das aktive Release bleibt während Download, Prüfsummenprüfung, Entpacken,
Wheel-Installation und Importtest in Betrieb. Erst danach wechselt `current`
atomar auf die neue Version und der Dienst wird neu gestartet.

Bei fehlgeschlagener Readiness stoppt der Installer den Dienst, stellt den
vorherigen Symlink wieder her und versucht, den vorherigen Dienst neu zu
starten. War vorher kein Release installiert, gibt es kein automatisches Ziel
für diese Wiederherstellung.

Standardmäßig werden die drei zuletzt installierten Release-Verzeichnisse
behalten. Für einen einzelnen Lauf kann die Anzahl geändert werden:

```bash
sudo /usr/local/sbin/paws-display-install install \
  --version 2026.08.02-1 \
  --keep-releases 5
```

Nach jedem Update Version, Readiness, beide Ausgänge, Theme, Testnachricht und
Ack prüfen. Konfiguration und Laufzeitdaten bleiben unverändert.

## Manuelles Rollback

Zuerst verfügbare Ziele prüfen:

```bash
ls -la /opt/paws-on-stream/releases
readlink -f /opt/paws-on-stream/current
```

Dann auf eine installierte Version zurückschalten:

```bash
sudo /usr/local/sbin/paws-display-install rollback 2026.07.21-1
```

Der Installer schaltet `current` atomar um, startet den Dienst neu und prüft
Readiness. Schlägt diese Prüfung beim manuellen Rollback fehl, bleibt das
gewählte Ziel aktiv; der Befehl stellt nicht automatisch wieder die zuvor aktive
Version her. In diesem Fall Logs prüfen und gezielt auf ein bekannt
funktionierendes Release zurückrollen.

Ein Rollback ändert `/etc/paws-on-stream/display.env` und
`/var/lib/paws-on-stream` nicht. Alte Programmversionen müssen deshalb mit dem
aktuellen persistenten Datenformat kompatibel bleiben.

## Was gesichert werden muss

Releases sind aus veröffentlichten Artefakten wiederherstellbar. Regelmäßig zu
sichern sind dagegen:

```text
/etc/paws-on-stream/display.env
/var/lib/paws-on-stream/messages.db
/var/lib/paws-on-stream/media-cache/
/var/lib/paws-on-stream/theme-cache/
```

`display.env` enthält Secrets und muss verschlüsselt gespeichert und übertragen
werden. Vor einer konsistenten manuellen Sicherung den Dienst stoppen:

```bash
sudo systemctl stop east-display.service
sudo tar -C / -czf /tmp/paws-display-state.tar.gz \
  etc/paws-on-stream \
  var/lib/paws-on-stream
sudo chmod 600 /tmp/paws-display-state.tar.gz
sudo systemctl start east-display.service
```

Die Sicherungsdatei anschließend auf einen geschützten Verwaltungsrechner
übertragen und vom Pi entfernen. Backupdatum, aktive Release-Version und
Prüfsumme gemeinsam dokumentieren.

## Wiederherstellung auf einem neuen Pi

1. Betriebssystem, KMS, Benutzergruppen und HDMI gemäß Installationsanleitung
   vorbereiten.
2. Dieselbe oder eine nachweislich kompatible Release-Version mit `--no-start`
   installieren.
3. Die Sicherung sicher nach `/tmp/` übertragen.
4. Den Dienst gestoppt lassen und Inhalt der Sicherung vor dem Entpacken prüfen.
5. Konfiguration und Laufzeitdaten wiederherstellen.
6. Eigentümer und Rechte korrigieren.
7. Dienst starten und die vollständige Abnahme durchführen.

```bash
tar -tzf /tmp/paws-display-state.tar.gz
sudo systemctl stop east-display.service
sudo tar -C / -xzf /tmp/paws-display-state.tar.gz
sudo chown root:pi /etc/paws-on-stream/display.env
sudo chmod 640 /etc/paws-on-stream/display.env
sudo chown -R pi:pi /var/lib/paws-on-stream
sudo systemctl enable east-display.service
sudo systemctl start east-display.service
```

Das Entpacken überschreibt gleichnamige Konfigurations- und Zustandsdateien.
Deshalb ausschließlich eine geprüfte Sicherung auf dem vorbereiteten Ziel
verwenden. Danach Health, Readiness, Theme, zwei Ausgänge, Backend-Registrierung
und Ack-Rückstände kontrollieren.

## Release erstellen

Dieser Abschnitt gilt für Entwickler und Release-Verantwortliche, nicht für den
Produktions-Pi. Ein versionierter Tag startet den Workflow:

```bash
git tag v2026.07.21-1
git push origin v2026.07.21-1
```

`.github/workflows/release.yml` läuft auf einem ARM64-Runner und:

1. installiert Python 3.13 und Poetry samt Export-Plugin,
2. installiert Quell- und Entwicklungsabhängigkeiten,
3. führt Ruff-, Format- und Pytest-Prüfungen aus,
4. exportiert die gesperrten Hauptabhängigkeiten,
5. lädt ausschließlich binäre ARM64-Wheels,
6. baut mit `scripts/build_release.sh` das Archiv und seine SHA-256-Datei,
7. veröffentlicht Archiv, Prüfsumme und `install/install.sh` im GitHub Release.

Lokaler Strukturtest ohne Wheel-Download:

```bash
SKIP_WHEELS=true scripts/build_release.sh test-1
```

Dabei ist `requirements.txt` absichtlich leer. Ein auf macOS oder ohne
ARM64-Wheelhouse gebautes Archiv ist nur ein Strukturtest und darf nicht auf
einem Produktions-Pi installiert werden.
