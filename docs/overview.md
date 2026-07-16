# Überblick

## Ziele

- Nachrichten aus Telegram strukturiert ins Backend bringen
- Moderation über Web-UI ermöglichen
- Event- und Teilnehmerstatus zentral verwalten
- Medien wie Bilder, GIFs und Sticker sauber behandeln

## Komponenten

| Komponente | Aufgabe |
| --- | --- |
| Bot | Telegram-Integration, Validierung, Upload |
| Display | Rendering auf Raspberry Pi, Polling, Display-Feedback, Regie-API |
| Web | API, Backend, Moderation, Dashboard |
| Reg-Sync | Abgleich von Check-in/Registrierungsstatus |

## Zentrale Flows

1. Nutzer sendet Nachricht an den Bot.
2. Bot prüft Status, Check-in und Rate-Limits.
3. Medien werden ggf. konvertiert und an das Web hochgeladen.
4. Bot sendet Message-Payload an das Web.
5. Web speichert, validiert und stellt die Nachricht im Moderations-Workflow bereit.
6. Display pollt freigegebene Nachrichten und sendet `displayed`-Feedback zurück.
