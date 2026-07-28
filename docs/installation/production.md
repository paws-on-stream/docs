# Produktionssystem

Diese Seite beschreibt den produktiven Betrieb von Web und Bot. Die
Raspberry-Pi-spezifische Display-Installation und der laufende Display-Betrieb
sind auf den eigenen Seiten dokumentiert.

## Images

Veröffentlichte Images:

```text
ghcr.io/paws-on-stream/web
ghcr.io/paws-on-stream/bot
```

In Produktion einen unveränderlichen `sha-<commit>`- oder Release-Tag verwenden,
nicht `latest`. Private GHCR-Pakete benötigen ein `imagePullSecret`.

## Web-Backend ausrollen

1. PostgreSQL und ein persistentes Medien-Volume bereitstellen.
2. Web-Secrets gemäß [Konfiguration](../configuration.md#web-backend) anlegen.
3. Den Migrationsjob mit dem neuen Image ausführen und seinen Erfolg prüfen.
4. Web-Deployment ausrollen.
5. Sync- und Cleanup-CronJobs ausrollen.
6. Health, Readiness und Metrics prüfen.

```bash
curl -f https://<web-domain>/api/v1/health/
```

Readiness und Metrics sind absichtlich nicht öffentlich erreichbar. Kubernetes
prüft Readiness über den internen Service; Prometheus ruft Metrics ebenfalls
intern ab. Der öffentliche Healthcheck bleibt für externe Verfügbarkeitschecks
vorgesehen.

Backup und Restore sind im aktuellen Projektumfang nicht vorgesehen.

## Bot ausrollen

Für Produktion wird der Webhook-Modus verwendet. Neben Token und Backend-URL
sind eine öffentliche HTTPS-URL und ein zufälliges Webhook-Secret erforderlich.

Der Bot stellt bereit:

```text
POST /webhook
GET  /health
GET  /readiness
GET  /metrics
```

Das Verzeichnis `/tmp/stickers-cache` sollte persistent sein, da es Bot-Status
und Retry-Queue enthält. Die Webhook-URL muss auf `/webhook` routen; Telegram
sendet das konfigurierte Secret im Header
`X-Telegram-Bot-Api-Secret-Token`.

## Telegram-Login aktivieren

1. Telegram OIDC bei BotFather konfigurieren.
2. `https://<web-domain>/auth/callback/` exakt als Callback freigeben.
3. Client-ID und Client-Secret im Web-Deployment setzen.
4. Mindestens eine numerische Telegram-ID über
   `TELEGRAM_AUTH_BOOTSTRAP_IDS` als ersten Admin zulassen.
5. Nach dem ersten erfolgreichen Login weitere Zugänge im Dashboard verwalten
   und die Bootstrap-Liste anschließend leeren.

## Display anbinden

Das Pi-Setup steht in [Raspberry-Pi-Display](display.md). Für jedes Gerät eine
stabile `DISPLAY_DEVICE_ID` vergeben und denselben `DISPLAY_API_AUTH_TOKEN` wie
im Web-Backend als `API_DISPLAY_TOKEN` setzen.

Der vollständige Ausroll-, Prüf- und Rollback-Ablauf steht im
[Betriebs-Runbook](../operations/runbook.md).
