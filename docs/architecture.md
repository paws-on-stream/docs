# Architektur

## Gesamtbild

Die Anwendung ist in zwei getrennte Repositories aufgeteilt:

- `paws_on_stream_bot`
- `paws_on_stream_web`

Der Bot spricht ausschließlich mit dem Web-Backend über HTTP.

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

## Wichtige Design-Entscheidungen

- Das Web ist die Quelle der Wahrheit für gespeicherte Nachrichten.
- Der Bot ist möglichst schlank und hält Logik für Telegram-spezifische Fälle.
- Medien werden als eigene Assets gespeichert und mit Messages verknüpft.

