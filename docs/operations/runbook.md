# Betriebs-Runbook

Dieses Runbook enthält wiederkehrende Produktionsabläufe. Einmalige Installation
und sämtliche Variablendefinitionen stehen bewusst nur unter
[Installation](../installation/production.md) und
[Konfiguration](../configuration.md).

## Vor dem Event

1. Web, Bot und alle Displays auf erwartete unveränderliche Image-/Commit-Stände
   bringen.
2. Health, Readiness und Metriken aller Dienste prüfen.
3. Reg- und Event-Sync kontrolliert ausführen.
4. Event, Zeitraum, `allow_messages`, effektiven Display-Modus, Bot-Status,
   Auto-Approve und Spam-Grenzwert prüfen.
5. Jedes Display im Dashboard auf Aktivstatus und aktuellen Heartbeat prüfen.
6. Testnachrichten für Text, Bild, GIF und Sticker senden, moderieren, anzeigen
   und im Display-Log bestätigen.
7. Regieaktionen einschließlich Killswitch testen und wieder zurücksetzen.

## Während des Events

- Pending-Queue und „Approved, not shown“ beobachten.
- Bot-, Web- und Display-Health überwachen.
- Datenbank-, Medien-Volume- und Retry-Queue-Auslastung beobachten.
- Bei problematischen Inhalten zuerst den lokalen Killswitch verwenden und
  anschließend gemäß [Regie-Handbuch](../handbook/control.md) vorgehen.
- Konfigurationsänderungen mit Uhrzeit und verantwortlicher Person protokollieren.

## Deployment

1. Neues Image mit unveränderlichem Tag bereitstellen.
2. Migrationsjob mit dem neuen Tag ausführen.
3. Erfolgreichen Job und Datenbankschema prüfen.
4. Web ausrollen, dann Bot und CronJobs.
5. Health, Readiness, Metrics, Login und API-Verbindungen prüfen.
6. Displays einzeln aktualisieren und je Gerät eine Testnachricht bestätigen.

Nie mehrere risikoreiche Komponenten gleichzeitig aktualisieren, wenn dadurch
kein funktionierender Nachrichtenpfad mehr zum Vergleich verfügbar wäre.

## Synchronisierung

Participant-Status manuell synchronisieren:

```bash
python paws_on_stream_web/manage.py sync_reg_status --workers 8
```

Der Standard sind acht, maximal sind sechzehn Worker. Der Sync berücksichtigt
`status_check_interval`; Einzelfehler brechen den Batch nicht ab.

Events synchronisieren:

```bash
python paws_on_stream_web/manage.py sync_events
```

Beide Befehle verwenden eine Datenbank-Lease gegen überlappende Läufe. Die
Kubernetes-Beispiele planen Reg-Sync minütlich und Event-Sync alle fünf Minuten.

## Alte Daten bereinigen

Ohne `--execute` ist der Cleanup nur eine Vorschau:

```bash
python paws_on_stream_web/manage.py cleanup_old_data
python paws_on_stream_web/manage.py cleanup_old_data --execute
```

Der geplante Job entfernt standardmäßig Nachrichten nach 30 Tagen und danach
nicht mehr referenzierte Medien nach sieben Tagen. Vor manueller Ausführung
Vorschau und erwartete Mengen prüfen.

## Backup und Restore

Backup und Restore sind im aktuellen Projektumfang nicht vorgesehen.

## Rollback

1. Vorherige unveränderliche Image-Tags für Deployment und CronJobs einsetzen.
2. Dienste neu ausrollen und Health prüfen.
3. Migrationen nur rückwärts ausführen, wenn sie nachweislich reversibel sind.
4. Bei inkompatiblen Schemaänderungen keine Rückwärtsmigration erzwingen,
   sondern eine vorwärtskompatible Korrektur bereitstellen.
5. Bot-Retry-Queue erst wieder abarbeiten lassen, wenn das Backend konsistent ist.

## Nach dem Event

1. `allow_messages` deaktivieren oder Bot-Status auf `offline` setzen.
2. Displays kontrolliert leeren und stoppen, falls sie abgebaut werden.
3. Pending-, Retry- und Display-Ack-Rückstände dokumentieren.
4. Logs und Metriken für die Nachbereitung sichern.
5. Cleanup erst nach bestätigter Aufbewahrungsfrist ausführen.
