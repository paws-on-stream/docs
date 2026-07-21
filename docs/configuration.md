# Konfiguration

Secrets gehören ausschließlich in Umgebungsvariablen oder einen Secret Store,
nicht ins Repository. Bot- und Display-Tokens sollten getrennt sein; der globale
API-Token besitzt weitergehende Rechte.

## Web-Backend

| Variable | Pflicht/Default | Zweck |
| --- | --- | --- |
| `SECRET_KEY` | Pflicht | Django-Signatursecret |
| `DEBUG` | `False` | Debugmodus; nie produktiv aktivieren |
| `DATABASE_URL` | `sqlite:///db.sqlite3` | Datenbankverbindung |
| `ALLOWED_HOSTS` | `testserver` | kommaseparierte erlaubte Hosts |
| `CSRF_TRUSTED_ORIGINS` | leer | kommaseparierte vollständige Origins |
| `API_AUTH_TOKEN` | leer | globaler API-Token mit Zugriff auf alle `/api/`-Pfade |
| `BOT_API_AUTH_TOKEN` | Wert von `API_AUTH_TOKEN` | auf Bot-Endpunkte begrenzter Token |
| `DISPLAY_API_AUTH_TOKEN` | Wert von `API_AUTH_TOKEN` | auf Display-Endpunkte begrenzter Token |
| `TELEGRAM_OIDC_CLIENT_ID` | leer | Telegram-OIDC-Client-ID |
| `TELEGRAM_OIDC_CLIENT_SECRET` | leer | Telegram-OIDC-Secret |
| `TELEGRAM_AUTH_BOOTSTRAP_IDS` | leer | kommaseparierte Telegram-IDs initialer Admins |
| `SESSION_COOKIE_SECURE` | `not DEBUG` | Cookies nur über HTTPS |
| `CSRF_COOKIE_SECURE` | `not DEBUG` | CSRF-Cookie nur über HTTPS |
| `SECURE_SSL_REDIRECT` | `False` | HTTP auf HTTPS umleiten |
| `SECURE_HSTS_SECONDS` | `0` | HSTS-Laufzeit; erst nach HTTPS-Prüfung erhöhen |
| `SECURE_HSTS_PRELOAD` | `False` | HSTS-Preload aktivieren |

Ein leerer API-Token authentifiziert keine Anfrage. Bot und Display senden den
jeweiligen Wert im Header `X-API-Token`.

## Telegram-Bot

| Variable | Pflicht/Default | Zweck |
| --- | --- | --- |
| `TELEGRAM_BOT_TOKEN` | Pflicht | Token von BotFather |
| `TELEGRAM_WEBHOOK_URL` | leer | öffentliche Basis-URL im Webhook-Modus |
| `TELEGRAM_WEBHOOK_SECRET` | leer | Secret zur Prüfung eingehender Webhooks |
| `API_BASE_URL` | `http://localhost:8000` | Basis-URL des Web-Backends |
| `API_AUTH_TOKEN` | leer | muss dem Bot-Token des Web-Backends entsprechen |
| `ADMIN_TELEGRAM_IDS` | leer | kommaseparierte IDs für Bot-Adminbefehle |
| `REG_CHECK_INTERVAL` | `300` | Gültigkeit eines Participant-Cacheeintrags in Sekunden |
| `REG_CACHE_MAX_STALE` | `900` | maximal nutzbares Cachealter bei API-Ausfall |
| `MESSAGE_RATE_LIMIT` | `10` | Nachrichten pro Telegram-ID und Minute im Bot-Prozess |
| `MEDIA_MAX_DOWNLOAD_BYTES` | `10485760` | maximales Telegram-Downloadvolumen |
| `RETRY_INTERVAL` | `10` | Abstand der Retry-Queue-Läufe in Sekunden |
| `RETRY_QUEUE_FILE` | `/tmp/stickers-cache/message-retry.jsonl` | persistente Message-Retry-Queue |
| `BOT_STATUS_FILE` | `/tmp/stickers-cache/bot-status` | lokale Statusdatei |
| `WEBHOOK_MAX_BODY_BYTES` | `1048576` | maximale Webhook-Requestgröße |
| `DISPLAY_MODE_CACHE_TTL` | `10` | Cache-Dauer des effektiven Display-Modus in Sekunden |
| `DISPLAY_MODE_CACHE_MAX_STALE` | `60` | maximal nutzbares Modus-Cachealter bei API-Ausfall |

`TELEGRAM_WEBHOOK_URL` wird vom Bot zu `<url>/webhook` ergänzt. Im
Webhook-Modus müssen URL und Secret gesetzt sein.

## Display-Client

Die Produktionskonfiguration liegt ausschließlich in
`/etc/paws-on-stream/display.env`. Der Installer erzeugt sie aus der
versionierten Vorlage, überschreibt sie bei Updates aber nicht. Empfohlene
Rechte:

```bash
sudo chown root:pi /etc/paws-on-stream/display.env
sudo chmod 640 /etc/paws-on-stream/display.env
```

| Variable | Produktionsvorlage | Zweck |
| --- | --- | --- |
| `API_BASE_URL` | `http://localhost:8000` | Basis-URL des Web-Backends; produktiv HTTPS-URL setzen |
| `API_DISPLAY_TOKEN` | leer, Pflicht | muss dem Display-Token des Web-Backends entsprechen |
| `DISPLAY_DEVICE_ID` | leer, Pflicht | stabile und eindeutige Geräte-ID, zum Beispiel `paws-display-01` |
| `DISPLAY_HOSTNAME` | leer | im Backend angezeigter Hostname, zum Beispiel `paws-on-stream-display-1` |
| `DISPLAY_LOCATION` | leer | verständlicher Standort, zum Beispiel `east-stage` |
| `POLL_INTERVAL_SEC` | `3` | reguläres Nachrichten-Polling in Sekunden |
| `SETTINGS_REFRESH_SEC` | `15` | Intervall für Backend-Settings und zentrales Theme |
| `CACHE_DB_PATH` | `/var/lib/paws-on-stream/messages.db` | persistente SQLite-Datenbank einschließlich Ack-Queue |
| `MEDIA_CACHE_DIR` | `/var/lib/paws-on-stream/media-cache` | persistenter Mediencache |
| `THEME_CACHE_DIR` | `/var/lib/paws-on-stream/theme-cache` | persistente letzte gültige zentrale Themes |
| `THEME_DIR` | `/opt/paws-on-stream/current/display_client/assets/themes` | lokale Fallback-Themes des aktiven Releases |
| `ALLOWED_MEDIA_HOSTS` | leer | kommaseparierte zusätzliche Hosts für Medien-Downloads |
| `ALLOWED_THEME_HOSTS` | leer | kommaseparierte zusätzliche Hosts für Theme-Manifeste und Assets |
| `RENDER_FPS` | `50` | Renderbildrate; gültig sind 1 bis 240 |
| `DISPLAY_INDEX` | `0` | SDL-Index der Primary-Ausgabe |
| `LUMA_DISPLAY_INDEX` | `1` | SDL-Index der getrennten Luma-Ausgabe |
| `LUMA_MODE` | `true` | zweiten synchronen Ausgang aktivieren |
| `REGIE_API_ENABLED` | `true` | lokale Regie-, Health- und Readiness-API aktivieren |
| `REGIE_API_HOST` | `127.0.0.1` | Bind-Adresse; Standard erlaubt nur lokale Zugriffe |
| `REGIE_API_PORT` | `8765` | Port der lokalen API |
| `KILLSWITCH_TOKEN` | leer, Pflicht für Regieaktionen | Secret im Header `X-Killswitch-Token` |

Beispiel für die gerätespezifischen Werte:

```dotenv
API_BASE_URL=https://backend.example
API_DISPLAY_TOKEN=<secret>
DISPLAY_DEVICE_ID=paws-display-01
DISPLAY_HOSTNAME=paws-on-stream-display-1
DISPLAY_LOCATION=east-stage

ALLOWED_MEDIA_HOSTS=media.example
ALLOWED_THEME_HOSTS=themes.example

DISPLAY_INDEX=0
LUMA_DISPLAY_INDEX=1
LUMA_MODE=true
KILLSWITCH_TOKEN=<anderes-secret>
```

Der Host aus `API_BASE_URL` ist für zentrale Themes automatisch erlaubt.
Zusätzliche Medien- und Theme-Hosts müssen explizit in der jeweils passenden
Allowlist stehen. Redirects beim Theme-Download werden abgelehnt.

`API_DISPLAY_TOKEN` und `KILLSWITCH_TOKEN` sind Secrets. Sie dürfen nicht in Git,
Tickets, Shell-History oder Release-Verzeichnissen landen. Persistente Pfade
nicht auf ein Release-Verzeichnis umbiegen; sonst gehen Daten bei Update oder
Rollback verloren.

## Laufzeiteinstellungen im Dashboard

| Feld | Default | Wirkung |
| --- | --- | --- |
| `rate_limit_per_minute` | `10` | API-Requests pro identifiziertem Teilnehmer und Minute |
| `max_message_length` | `4096` | maximale Länge nach serverseitiger Bereinigung |
| `bot_status` | `online` | Backend-Annahme neuer Nachrichten |
| `auto_approve` | `False` | automatische Freigabe geeigneter Nachrichten |
| `spam_threshold` | `0.70` | Grenzwert von `0.10` bis `1.00`; auffällige Messages bleiben pending und sind von Auto-Approve ausgeschlossen |
| `overlay_theme` | `east13` | vom Display zu ladender Theme-Slug; kann auf eine eingebaute oder aktivierte Upload-Version zeigen |
| `overlay_font_size` | `24` | globale Schriftgrößenüberschreibung |
| `display_duration_sec` | `8` | Anzeigedauer im Chat-Modus |
| `display_mode` | `chat` | `chat` oder `crawling` |
| `scroll_speed_px` | `3` | Pixel pro Frame im Crawling-Modus |
| `reg_api_url` / `reg_api_key` | leer | externe Participant-Quelle |
| `event_api_url` / `event_api_jsonq_filter` | leer | externe Event-Quelle und jq-Transformation |
| `status_check_interval` | `300` | Mindestabstand regulärer Reg-Syncs |
| `require_event_active` | `True` | aktives, laufendes Event für Messages verlangen |

Der effektive Display-Modus stammt von einem laufenden, aktiven Event mit
`allow_messages`, sofern dort ein Modus gesetzt ist; sonst gilt der globale
Wert. Der Bot verwendet ihn, um im Crawling-Modus nur Text anzunehmen. Der
aktuelle Pi-Display-Client liest für das Rendering weiterhin die globalen
Settings; Theme-Version und Manifest-URL werden vom Web-Backend separat an die
Renderer-Clients ausgeliefert.
