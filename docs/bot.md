# Bot

## Aufgabe

Der Bot ist die Telegram-Schnittstelle von Paws on Stream.

## Hauptfunktionen

- Nachrichteneingang aus Telegram
- Validierung gegen Reg- und Check-in-Status
- Rate-Limit pro User
- Upload von Medien an das Backend
- Übergabe von Messages an die Web-API

## Verarbeitungs-Pipeline

1. Bot-Status prüfen
2. Reg-/Check-in-Status prüfen
3. Rate-Limit prüfen
4. Medien erkennen
5. Medien ggf. konvertieren
6. Medien hochladen
7. Nachricht an das Backend senden
8. Backend-Antwort an Nutzer zurückmelden

## Unterstützte Inhalte

- Text
- Foto
- GIF / Animation
- Sticker

## Rejection-Verhalten

Der Bot lehnt Nachrichten ab, wenn z. B.:

- der Bot offline ist
- der Nutzer nicht registriert ist
- der Nutzer nicht eingecheckt ist
- das Rate-Limit überschritten ist
- Medien nicht verarbeitet werden können
- das Backend die Nachricht ablehnt

## Konfiguration

Wichtige Variablen:

- `TELEGRAM_BOT_TOKEN`
- `API_BASE_URL`
- `API_AUTH_TOKEN`
- `ADMIN_TELEGRAM_IDS`
- `REG_API_KEY`

## Betrieb

- **Polling** für lokale Entwicklung
- **Webhook** für Produktion

