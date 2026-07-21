# Paws on Stream

Paws on Stream bringt Nachrichten aus Telegram kontrolliert auf eine
Veranstaltungsanzeige. Der Telegram-Bot nimmt Inhalte entgegen, das Web-Backend
prüft und speichert sie, Moderierende entscheiden über die Freigabe und ein
oder mehrere Display-Clients zeigen freigegebene Nachrichten an.

## Schnellstart

| Ziel | Einstieg |
| --- | --- |
| Telegram-Bot bedienen | [Telegram-Bot verwenden](handbook/bot.md) |
| Nachrichten moderieren | [Moderation](handbook/moderation.md) |
| Zugänge verwalten | [Rollen und Anmeldung](handbook/access.md) |
| Events und Displaymodus | [Events und Einstellungen](handbook/events.md) |
| Regie ausführen | [Regie und Displaysteuerung](handbook/control.md) |
| Browser-Monitor einsetzen | [Passives Browser-Display](handbook/monitor.md) |
| Lokale Entwicklung | [Lokale Entwicklung](installation/local.md) |
| Produktionssystem | [Produktionssystem](installation/production.md) |
| Raspberry Pi installieren | [Raspberry-Pi-Display](installation/display.md) |
| Display betreiben | [Display-Betrieb und Wiederherstellung](operations/display.md) |
| Wiederkehrende Abläufe | [Betriebs-Runbook](operations/runbook.md) |
| Themes verwalten | [Display-Themes](themes.md) |
| Fehler suchen | [Troubleshooting](troubleshooting.md) |
| API integrieren | [HTTP-API](api.md) |

## Komponenten

- **Web** ist die zentrale Quelle der Wahrheit für Events, Teilnehmende,
  Nachrichten, Einstellungen, Themes und Anzeigeprotokolle.
- **Bot** ist die Telegram-Schnittstelle. Er prüft den Teilnehmerstatus,
  normalisiert Medien und übermittelt Nachrichten an Web.
- **Display** pollt freigegebene Nachrichten, rendert sie und bestätigt die
  Anzeige gerätespezifisch.

Die Details und Datenflüsse stehen in der [Architektur](architecture.md).
