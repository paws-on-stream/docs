# HTTP-API

Die zentrale API liegt unter `/api/v1/`. Mit Ausnahme von Health und Readiness
ist der Header `X-API-Token` Pflicht.

## Tokens und Bereiche

| Token | Erlaubter Bereich |
| --- | --- |
| global | alle API-Endpunkte |
| Bot | Message-Erstellung, Media-Upload, Participant-Check, Ban und Mute |
| Display | Display-Poll, Settings lesen, Device registrieren, Ack und Display-Event |

Ein fehlender oder falscher Token ergibt `403`. Validierungsfehler werden als
JSON mit `400`, unbekannte Objekte mit `404` zurückgegeben.

## System

| Methode und Pfad | Auth | Zweck |
| --- | --- | --- |
| `GET /api/v1/health/` | öffentlich | Liveness und Datenbankstatus |
| `GET /api/v1/readiness/` | öffentlich | Readiness und Datenbankstatus |
| `GET /metrics/` | außerhalb `/api/` | Prometheus-Metriken |

## Bot-Vertrag

### Participant prüfen

```http
POST /api/v1/participants/{telegram_id}/check_status/
X-API-Token: <bot-token>
```

Der Aufruf synchronisiert mit dem Reg-System und kann deshalb Daten verändern.
Er liefert `201`, wenn der Participant neu angelegt wurde, sonst `200`. Bei
Upstream-Problemen werden für bekannte IDs lokale Daten mit `fallback: true`
geliefert. Unbekannte IDs ergeben je nach Upstream-Antwort `404` oder `502`.

### Medium hochladen

```http
POST /api/v1/media/upload/
Content-Type: multipart/form-data
X-API-Token: <bot-token>
```

Felder:

| Feld | Bedeutung |
| --- | --- |
| `file` | validiertes statisches oder animiertes WebP |
| `media_type` | `photo`, `gif` oder `sticker` |
| `telegram_file_id` | Telegram-Datei-ID |
| `telegram_file_unique_id` | eindeutige Telegram-Datei-ID |
| `sticker_emoji` | optionales Sticker-Emoji |

Grenzen: 10 MB, 1280 × 1280 Pixel, 150 Frames und 10 Sekunden. Erfolgreiche
Uploads liefern `media_asset_id`, eine stabile URL, Format-, Größen-,
Animations- und SHA-256-Metadaten. Bereits bekannter Inhalt liefert `200`, neu
gespeicherter Inhalt `201`; eine kollidierende Unique-ID liefert `409`.

### Nachricht anlegen

```http
POST /api/v1/message/
Content-Type: application/json
X-API-Token: <bot-token>
```

Beispiel:

```json
{
  "telegram_id": 123456789,
  "display_name": "Flauschpfote",
  "content": "Hallo zusammen!",
  "media_type": "text",
  "media_asset_id": null,
  "sticker_emoji": ""
}
```

Für `photo`, `gif` und `sticker` ist eine zuvor erzeugte, typgleiche
`media_asset_id` Pflicht. Das Backend bereinigt Inhalt, prüft Participant,
Check-in, Ban, Mute, Bot- und Eventstatus und berechnet den Spam-Score. Der
Response enthält `spam_score` als Zahl von `0.0` bis `1.0`. Das Feld ist
serververwaltet; ein vom Client gesendeter Wert wird verworfen.

Ein Score am oder über dem konfigurierten Grenzwert führt nicht zu einem
API-Fehler und nicht zu automatischer Ablehnung. Die Message wird mit `201`
angelegt und bleibt `pending`.

## Display-Vertrag

| Methode und Pfad | Zweck |
| --- | --- |
| `POST /api/v1/devices/register/` | Gerät anlegen oder Heartbeat aktualisieren |
| `GET /api/v1/settings/1/` | aktuelle Render-Einstellungen |
| `GET /api/v1/themes/<slug>/` | Theme-Manifest eines aktiven Themes |
| `GET /api/v1/themes/<slug>/<version>/assets/<asset_id>/` | einzelnes Theme-Asset |
| `GET /api/v1/messages/display/` | für dieses Gerät noch nicht bestätigte Messages |
| `POST /api/v1/messages/{id}/displayed/` | Anzeige gerätespezifisch bestätigen |
| `POST /api/v1/events/killswitch/` | lokales Regieereignis persistieren |

Beim Poll sendet das Display zusätzlich `X-Device-ID`. Optionale Queryparameter
sind `since` als zeitzonenbehafteter ISO-Zeitpunkt und `limit` zwischen 1 und
100. Das Ack erwartet `device_id` und eine nicht negative
`display_duration_actual`.

Ein Display-Event erwartet:

```json
{
  "device_id": "display-ost",
  "event_type": "killswitch",
  "occurred_at": "2026-07-21T20:15:00+02:00"
}
```

Erlaubte Eventtypen sind `killswitch`, `pause`, `resume` und `clear`.

Die Theme-Endpunkte werden mit dem Display-Token gelesen. Sie liefern lokale
Fallback-Themes oder serververwaltete Versionen im Schema v3. Der
Browser-Monitor greift auf dieselben Inhalte über `/monitor/themes/...` zu,
aber mit seiner eigenen Monitor-Authentifizierung.

## Effektiver Display-Modus

```http
GET /api/v1/settings/effective-display-mode/
X-API-Token: <bot-token>
```

Der Bot-Token darf gezielt diesen Settings-Endpunkt lesen, aber nicht die
allgemeinen Settings. Die Antwort enthält:

```json
{
  "display_mode": "crawling",
  "source": "event",
  "event_id": 42
}
```

Ein laufendes, aktives Event mit `allow_messages` und gesetztem Modus gewinnt.
Andernfalls liefert die API den globalen Modus mit `source: "global"`. Der Bot
verwendet das Ergebnis, um im Crawling-Modus Medien abzulehnen.

## Verwaltungs-API

Der globale Token kann die DRF-Ressourcen `events`, `messages`, `participants`,
`settings`, `devices` und `logs` unter `/api/v1/` verwenden. Wichtige Actions:

```text
GET  /api/v1/messages/pending/
POST /api/v1/messages/{id}/approve/
POST /api/v1/messages/{id}/reject/
GET  /api/v1/messages/kpis/
POST /api/v1/participant/{telegram_id}/ban/
POST /api/v1/participant/{telegram_id}/mute/
```

Reject akzeptiert `rejection_reason`; `spam` ist ein gültiger Ablehnungsgrund.
Mute verlangt eine positive Ganzzahl `minutes`. Das Dashboard verwendet für menschliche Bedienung eigene
sessiongeschützte HTML-Routen und nicht diese statischen Tokens.

## Browser-Display-Endpunkte

Das passive Web-Display liegt bewusst außerhalb von `/api/v1/`:

```text
GET  /monitor/
POST /monitor/access/
GET  /monitor/feed/
```

Staff-Sitzungen dürfen den Feed direkt lesen. Anonyme Displays tauschen einmalig
den vom Admin erzeugten Token gegen ein signiertes HTTP-only-Cookie. Der Feed
verwendet signierte Cursor und liefert freigegebene Messages sowie globale
Displaysettings. Bedienung und Widerruf stehen im
[Monitor-Handbuch](handbook/monitor.md).

## Theme-Verwaltung

Admins verwalten Themes über die HTML-Oberfläche unter `/core/themes/`. Dort
lassen sich ZIP-Pakete importieren, Versionen aktivieren und nicht aktive
Uploads löschen. Die aktive Version bestimmt die Daten, die Display-Client und
Browser-Monitor laden.
