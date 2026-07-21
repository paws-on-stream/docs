# Paws on Stream – Dokumentation

Dieses Repository enthält das Bedienhandbuch, die Installationsanleitungen und
die technische Referenz für Web-Backend, Telegram-Bot und Raspberry-Pi-Display.

Die Dokumentation folgt dem Prinzip „eine Information, eine Quelle“:

- Bedienabläufe stehen unter **Handbuch**.
- Installationsschritte stehen unter **Installation**.
- Schnittstellen und Konfigurationswerte stehen unter **Technische Referenz**.
- Wiederkehrende Betriebsaufgaben stehen unter **Betrieb**.

Der schnellste Einstieg ist die Startseite unter
[docs/index.md](docs/index.md). Die wichtigsten Arbeitsbereiche sind:

- [Telegram-Bot verwenden](docs/handbook/bot.md)
- [Moderation](docs/handbook/moderation.md)
- [Rollen und Anmeldung](docs/handbook/access.md)
- [Ein Event vorbereiten](docs/handbook/events.md)
- [Regie und Displaysteuerung](docs/handbook/control.md)
- [Passives Browser-Display](docs/handbook/monitor.md)
- [Raspberry-Pi-Display installieren](docs/installation/display.md)
- [Display-Betrieb und Wiederherstellung](docs/operations/display.md)
- [Konfiguration](docs/configuration.md)
- [Display-Themes](docs/themes.md)
- [HTTP-API](docs/api.md)
- [Datenmodell](docs/data-model.md)

## Lokale Vorschau

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/mkdocs serve
```

## Build

```bash
.venv/bin/mkdocs build --strict
```
