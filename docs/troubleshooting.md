# Troubleshooting

Zuerst die betroffene Verbindung eingrenzen: Telegram → Bot, Bot → Web,
Web → Datenbank beziehungsweise Upstream oder Web → Display.

## Bot reagiert nicht

1. Im Webhook-Modus `/health`, `/readiness` und `/metrics` prüfen.
2. `TELEGRAM_BOT_TOKEN`, Webhook-URL und Webhook-Secret kontrollieren.
3. Prüfen, ob Telegram auf die öffentliche URL mit `/webhook` zeigen kann.
4. Bot-Logs nach `403`, ungültigen Updates oder Backend-Fehlern durchsuchen.

## Bot lehnt alle Nachrichten ab

- `bot_status` im Dashboard muss `online` sein.
- Bei `require_event_active` muss ein laufendes, aktives Event mit
  `allow_messages` existieren.
- Participant darf weder gebannt noch aktuell gemutet sein.
- effektiver Check-in und Reg-API-Verfügbarkeit prüfen.
- `API_AUTH_TOKEN` des Bots muss `BOT_API_AUTH_TOKEN` im Web entsprechen.

## Erwartete Nachricht wird als Spam markiert

- Spam-Score und aktuellen Grenzwert in der Message-Detailansicht vergleichen.
- Bei zeitnahen Nachrichten desselben Participants den RateFilter beachten.
- `spam_count` in der Participant-Detailansicht prüfen; die Vorgeschichte fließt
  als letzter Filter ein.
- Der Score ist additiv und die Pipeline endet bereits beim Grenzwert. Deshalb
  müssen nicht alle Filter zum gespeicherten Score beigetragen haben.
- Grenzwert nur bewusst ändern: Er beeinflusst neue Bewertungen und
  Auto-Approve, berechnet bestehende Messages aber nicht neu.

Die exakten Grenzen und Gewichte stehen in der
[Spamfilter-Referenz](spam-filter.md).

## Nachrichten bleiben in der Retry-Queue

Backend-URL, DNS/TLS, Bot-Token und Web-Health prüfen. Danach Logs beobachten;
der Worker versucht die Zustellung im konfigurierten `RETRY_INTERVAL` erneut.
Die Datei unter `RETRY_QUEUE_FILE` nicht während des laufenden Prozesses
manuell bearbeiten.

## Medium wird abgelehnt oder nicht angezeigt

- Bot benötigt FFmpeg mit `libwebp` und `libwebp_anim`.
- Web akzeptiert nur echtes `image/webp` innerhalb der dokumentierten Grenzen.
- Medien-Volume und `/media/media_assets/...` prüfen.
- `ALLOWED_MEDIA_HOSTS` des Displays muss den Host der gelieferten URL enthalten.
- Display-Journal auf Download-, Größen-, Frame- oder Animationsfehler prüfen.

## Display erhält keine Nachrichten

1. Web-Health und Netzwerkverbindung vom Pi prüfen.
2. `API_DISPLAY_TOKEN` mit `DISPLAY_API_AUTH_TOKEN` vergleichen.
3. Im Dashboard kontrollieren, ob das Gerät aktiv und `last_seen` aktuell ist.
4. Prüfen, ob überhaupt `approved`-Nachrichten existieren.
5. Display-Zustand prüfen: nicht pausiert und nicht killswitched.
6. Beachten, dass bereits für dieses Gerät bestätigte Messages nicht erneut
   gepollt werden.

## Display zeigt Konsole oder schwarzes Bild

- Kabel und beide Displays möglichst vor dem Boot verbinden.
- KMS-Einstellungen in `/boot/firmware/config.txt` prüfen.
- DRM-Verbindungen und angebotene Modi prüfen:

```bash
ls -l /dev/dri
cat /sys/class/drm/card*-HDMI-A-*/status
cat /sys/class/drm/card*-HDMI-A-*/modes
id pi
```

- Beide Anschlüsse müssen `connected` und `1920x1080` melden.
- Benutzer `pi` muss den Gruppen `video`, `render` und `input` angehören.
- `DISPLAY_INDEX`, `LUMA_DISPLAY_INDEX` und `LUMA_MODE` prüfen.
- `east-display-x.service` muss laufen; dort wird `Xorg :0` mit `xrandr`
  auf `HDMI-1` und `HDMI-2` auf 1920×1080@50 gesetzt.
- Falls der X-Server nicht hochkommt, zusätzlich die Logs der Xorg-Unit prüfen:

```bash
sudo systemctl status east-display-x.service
sudo journalctl -u east-display-x.service -n 100 --no-pager
```

- SDL darf nicht versehentlich mit `SDL_VIDEODRIVER=offscreen` starten.
- Die Unit muss direkt
  `/opt/paws-on-stream/current/.venv/bin/python` aufrufen; Poetry gehört nicht
  auf das Produktionsgerät.

## Display-Dienst startet nicht

```bash
sudo systemctl status east-display-x.service
sudo systemctl status east-display.service
sudo journalctl -u east-display-x.service -n 100 --no-pager
sudo journalctl -u east-display.service -n 100 --no-pager
readlink -f /opt/paws-on-stream/current
```

Prüfen, ob der `current`-Symlink auf ein vorhandenes Release zeigt,
`/etc/paws-on-stream/display.env` lesbar ist und Python unter
`current/.venv/bin/python` existiert. Danach API-URL, Tokens, Gerätezugriffe,
Xorg-Start und Rendererfehler im Journal prüfen.

## Installer meldet einen Prüfsummenfehler

- Archiv und `.sha256` müssen aus demselben Release stammen.
- Die Sidecar-Datei muss exakt `<archiv>.sha256` heißen.
- Unvollständigen Download beziehungsweise Übertragung löschen und beide Dateien
  erneut beziehen.
- Bei explizitem `--sha256` genau 64 hexadezimale Zeichen übergeben.
- Die Prüfsummenprüfung niemals umgehen.

## Display-Readiness schlägt fehl

```bash
curl -v http://127.0.0.1:8765/readiness
sudo journalctl -u east-display.service -n 100 --no-pager
```

`renderer_ready`, aktives `theme`, `outputs`, `mode` und `pending_acks` prüfen.
Bei einem regulären Update stellt der Installer nach fehlgeschlagener Readiness
automatisch das vorige Release wieder her. Bei einer Erstinstallation ohne
voriges Release und bei einem manuellen Rollback ist eine gezielte Korrektur
beziehungsweise ein weiterer Rollback nötig.

## Theme lädt nicht

- Zentrale und lokale Dateien prüfen:

```bash
find /var/lib/paws-on-stream/theme-cache -maxdepth 3 -type f
find /opt/paws-on-stream/current/display_client/assets/themes -maxdepth 3 -type f
```

- `ALLOWED_THEME_HOSTS`, Backend-URL, TLS und Theme-Metadaten prüfen.
- Assetpfade beachten Groß-/Kleinschreibung; Redirects werden abgelehnt.
- Die Fallback-Reihenfolge ist: vollständig validiertes zentrales Theme, letzte
  gültige Cache-Version desselben Themes, lokales `east13`, lokales
  `east-default`.
- Ein Releasewechsel darf `/var/lib/paws-on-stream/theme-cache` nicht löschen.
- Bei zentralen Themes zusätzlich `/core/themes/` und die Theme-API-URLs
  kontrollieren; der Browser-Monitor nutzt dieselben Versionen über
  `/monitor/themes/...`.

Die kanonische Struktur steht unter [Display-Themes](themes.md).

## Telegram-Login funktioniert nicht

- Client-ID und Client-Secret müssen gesetzt sein.
- Callback muss exakt `https://<domain>/auth/callback/` entsprechen.
- Reverse Proxy muss das ursprüngliche HTTPS-Schema weitergeben.
- Bei `/auth/denied/` numerische Telegram-ID, Aktivstatus und Rolle prüfen.

## Browser-Monitor zeigt „Zugriff erforderlich“

- Angemeldet prüfen, ob das Konto weiterhin `is_staff` besitzt.
- Bei öffentlichem Zugriff den vollständigen Link einschließlich Fragment nach
  `#` verwenden; ein normaler Aufruf von `/monitor/` autorisiert nicht.
- Link kann rotiert oder widerrufen worden sein. Admin-Status unter
  `/core/web-display/` prüfen und bei Bedarf einen neuen Link erzeugen.
- Nach vielen falschen Tokens das Rate-Limit-Fenster abwarten.
- Browser muss Cookies für dieselbe Site erlauben; in Produktion ist HTTPS wegen
  des `Secure`-Cookies erforderlich.

## Browser-Monitor zeigt keine neue Freigabe

- `/monitor/feed/` in den Browser-Entwicklertools auf `401`, `400` oder `5xx`
  prüfen.
- Bei Fehlern bis zu 48 Sekunden Backoff berücksichtigen.
- Nur Messages mit `approved_at` werden geliefert; Status und Zeitstempel im
  Dashboard prüfen.
- Ein Neuladen zeigt zunächst nur die neueste freigegebene Nachricht und setzt
  die lokale Queue zurück.
- Fehlende Themes deuten meist auf fehlende Monitor-Auth, eine nicht aktivierte
  v3-Version oder defekte Theme-Assets hin.

## Sync schlägt fehl

- Reg-Sync: URL, API-Key, Upstream-Antwort und Teilnehmer-ID prüfen.
- Event-Sync: URL und jq-Filter prüfen; das Ergebnis muss eine JSON-Liste sein.
- Ein bestehender `SyncLock` kann einen überlappenden Lauf verhindern.
- Fehler einzelner Participants stoppen den Batch nicht; vollständiges Job-Log
  prüfen.

## Web meldet `503`

`/api/v1/health/` prüft die Datenbank. `DATABASE_URL`, PostgreSQL-Erreichbarkeit,
Credentials und Migrationen kontrollieren. Bei fehlenden Medien zusätzlich das
PVC prüfen; ein Medienfehler allein erklärt den DB-Health-Status nicht.

## Dokumentation baut nicht

```bash
.venv/bin/mkdocs build --strict
```

Die erste Warnung beheben: Strict-Modus behandelt auch nicht aufgelöste Links
und Navigationsprobleme als Buildfehler.
