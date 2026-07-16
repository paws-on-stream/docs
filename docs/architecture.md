# Architektur

## Gesamtbild

Die Anwendung ist in drei getrennte Repositories aufgeteilt:

- `paws_on_stream_bot`
- `paws_on_stream_display`
- `paws_on_stream_web`

Bot und Display sprechen ausschließlich mit dem Web-Backend über HTTP.

## Datenfluss

```text
Telegram -> Bot -> Web API -> Datenbank -> Moderations-UI -> Anzeige
```

## Verantwortlichkeiten

### Bot

- empfängt Telegram-Updates
- erkennt Text, Bilder, GIFs, Sticker und Videos
- konvertiert Sticker, wenn möglich
- lädt Medien hoch
- übergibt Messages an die Web-API
- informiert Nutzer über Ablehnungen

### Web

- speichert Events, Participants, Messages und MediaAssets
- prüft Moderationsregeln
- stellt Dashboard und Detailansichten bereit
- liefert API-Endpunkte für Bot und Display-Systeme

### Display

- pollt freigegebene Nachrichten (`GET /api/v1/messages/display/`)
- sendet Anzeige-Feedback (`POST /api/v1/messages/{id}/displayed/`)
- rendert Chat- und Ticker-Modus mit Theme-System
- bietet lokale Regie-API (`killswitch`, `pause`, `resume`, `clear`)

## Wichtige Design-Entscheidungen

- Das Web ist die Quelle der Wahrheit für gespeicherte Nachrichten.
- Der Bot ist möglichst schlank und hält Logik für Telegram-spezifische Fälle.
- Medien werden als eigene Assets gespeichert und mit Messages verknüpft.
