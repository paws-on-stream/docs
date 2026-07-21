# Betriebs-Runbook

Dieses Runbook enthält wiederkehrende Produktionsabläufe. Einmalige Installation
und sämtliche Variablendefinitionen stehen bewusst nur unter
[Installation](../installation/production.md) und
[Konfiguration](../configuration.md).

## Vor dem Event

1. Aktuellen Backup- und Restore-Test bestätigen.
2. Web, Bot und alle Displays auf erwartete unveränderliche Image-/Commit-Stände
   bringen.
3. Health, Readiness und Metriken aller Dienste prüfen.
4. Reg- und Event-Sync kontrolliert ausführen.
5. Event, Zeitraum, `allow_messages`, effektiven Display-Modus, Bot-Status,
   Auto-Approve und Spam-Grenzwert prüfen.
6. Jedes Display im Dashboard auf Aktivstatus und aktuellen Heartbeat prüfen.
7. Testnachrichten für Text, Bild, GIF und Sticker senden, moderieren, anzeigen
   und im Display-Log bestätigen.
8. Regieaktionen einschließlich Killswitch testen und wieder zurücksetzen.

## Während des Events

- Pending-Queue und „Approved, not shown“ beobachten.
- Bot-, Web- und Display-Health überwachen.
- Datenbank-, Medien-Volume- und Retry-Queue-Auslastung beobachten.
- Bei problematischen Inhalten zuerst den lokalen Killswitch verwenden und
  anschließend gemäß [Regie-Handbuch](../handbook/control.md) vorgehen.
- Konfigurationsänderungen mit Uhrzeit und verantwortlicher Person protokollieren.

## Deployment

1. Neues Image mit unveränderlichem Tag bereitstellen.
2. Datenbank und Medien-Volume gemeinsam sichern.
3. Migrationsjob mit dem neuen Tag ausführen.
4. Erfolgreichen Job und Datenbankschema prüfen.
5. Web ausrollen, dann Bot und CronJobs.
6. Health, Readiness, Metrics, Login und API-Verbindungen prüfen.
7. Displays einzeln aktualisieren und je Gerät eine Testnachricht bestätigen.

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
Vorschau, Backup und erwartete Mengen prüfen.

## Backup und Restore

- PostgreSQL mit `pg_dump` sichern.
- Medien-PVC zum selben logischen Zeitpunkt sichern.
- Backup-ID, Schema-/Image-Version und Zeitpunkt gemeinsam protokollieren.
- Restore zuerst in einer separaten Datenbank und einem separaten Volume testen.
- Nach dem Restore Medienabruf, Login, Participant-Beziehungen und Display-Logs
  stichprobenartig prüfen.

## Rollback

1. Vorherige unveränderliche Image-Tags für Deployment und CronJobs einsetzen.
2. Dienste neu ausrollen und Health prüfen.
3. Migrationen nur rückwärts ausführen, wenn sie nachweislich reversibel sind.
4. Bei inkompatiblen Schemaänderungen Datenbank und Medien-Volume gemeinsam aus
   demselben Wiederherstellungspunkt restaurieren.
5. Bot-Retry-Queue erst wieder abarbeiten lassen, wenn das Backend konsistent ist.

## Nach dem Event

1. `allow_messages` deaktivieren oder Bot-Status auf `offline` setzen.
2. Displays kontrolliert leeren und stoppen, falls sie abgebaut werden.
3. Abschlussbackup von Datenbank und Medien erstellen.
4. Pending-, Retry- und Display-Ack-Rückstände dokumentieren.
5. Logs und Metriken für die Nachbereitung sichern.
6. Cleanup erst nach bestätigter Aufbewahrungsfrist ausführen.
