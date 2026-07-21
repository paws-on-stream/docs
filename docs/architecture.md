# Architektur

Paws on Stream besteht aus drei getrennt deploybaren Anwendungen. Web ist die
zentrale Quelle der Wahrheit; Bot und Display halten nur den für robusten
Betrieb erforderlichen lokalen Zustand.

## Komponenten

| Komponente | Verantwortung | Lokaler Zustand |
| --- | --- | --- |
| Web | API, Datenbank, Dashboard, passives Browser-Display, Moderation, Reg- und Event-Sync | Datenbank und Medien-Dateisystem |
| Bot | Telegram-Updates, Vorprüfung, Mediennormalisierung, Nutzerfeedback | Reg-Cache, Bot-Status und persistente Retry-Queue |
| Display | Polling, Rendering, Theme, Regie-API und Display-Acks | SQLite-, Medien-, Settings- und Ack-Cache |

Bot und Display kommunizieren per HTTP ausschließlich mit Web. Telegram spricht
im Polling- oder Webhook-Modus mit dem Bot. Die lokale Regie spricht direkt mit
dem jeweiligen Display.

## Nachrichtenfluss

```text
Telegram
   │
   ▼
  Bot ── Participant-Check ──► Web ──► Reg-System
   │
   ├── Medium normalisieren und hochladen
   └── Nachricht anlegen
                                │
                                ▼
                             pending
                                │
                      Spamfilter/Moderation/Auto-Approve
                           ┌────┴─────┐
                           ▼          ▼
                       rejected    approved
                                      │
                                  Display-Poll
                                      │
                                      ▼
                                  Rendering
                                      │
                                  Display-Ack
                                      │
                                      ▼
                                  DisplayLog
```

## Medienfluss

Der Bot lädt Telegram-Medien herunter und normalisiert unterstützte Formate zu
statischem oder animiertem WebP. Web validiert Inhalt und Grenzwerte, speichert
ein `MediaAsset` und liefert dessen ID zurück. Erst danach legt der Bot die
`Message` mit `media_asset_id` an. Direkte externe Medien- oder Video-URLs sind
nicht Teil des Vertrags.

## Spamprüfung

Vor dem Speichern bewertet Web jede Message mit der festen Spamfilter-Pipeline.
Scoreberechnung, Message-Erstellung und Aktualisierung der Participant-Historie
laufen atomar. Auffällige Nachrichten werden nicht verworfen, sondern gezielt
von Auto-Approve ausgeschlossen und der Moderation als `pending` vorgelegt. Die
verbindlichen Regeln stehen in der [Spamfilter-Referenz](spam-filter.md).

## Mehrere Displays

Jedes Display sendet seine `X-Device-ID`. Das Backend filtert Nachrichten
heraus, die für dieses Gerät bereits in einem `DisplayLog` bestätigt wurden.
Der Status der Nachricht bleibt `approved`; dadurch kann jedes Gerät dieselbe
Nachricht genau einmal abholen und bestätigen.

## Ausfallverhalten

- Der Bot cached Participant-Status kurzzeitig und stellt fehlgeschlagene
  Message-POSTs in eine persistente JSONL-Retry-Queue.
- Das Display cached Nachrichten, Medien, den letzten gültigen Settings-Stand
  und noch nicht bestätigte Acks.
- Das Web fällt beim Participant-Einzelcheck auf vorhandene lokale Daten zurück,
  wenn das Reg-System nicht erreichbar ist. Unbekannte IDs können ohne Upstream
  nicht verifiziert werden.
- Datenbank und Medienbestand des Web-Backends müssen als konsistentes Paar
  gesichert werden.

Konkrete Fehlerbilder stehen im [Troubleshooting](troubleshooting.md).
