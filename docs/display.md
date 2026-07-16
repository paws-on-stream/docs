# Display

## Aufgabe

Der Display-Client läuft auf Raspberry Pi und zeigt freigegebene Nachrichten live als Overlay an.

## Hauptfunktionen

- Polling von freigegebenen Nachrichten über die Web-API
- Anzeige-Feedback ans Backend (`displayed`)
- Chat-Modus (eine Nachricht gleichzeitig)
- Ticker-Modus (laufender Text mit optionalem Frame)
- Lokale Regie-API (`killswitch`, `pause`, `resume`, `clear`)
- Lokaler Message-/Media-Cache für robusten Betrieb

## Wichtige API-Endpunkte

- `GET /api/v1/messages/display/`
- `POST /api/v1/messages/{id}/displayed/`
- `GET /api/v1/settings/1/`
- `POST /api/v1/devices/register/`

## Theming

Themes liegen unter `display_client/assets/themes/*.json`.

### Chat/Bubble

- `frame.top_image`, `frame.middle_tile_image`, `frame.bottom_image`
- `message_position.x/y`
- `bubble_scale`

### Ticker

- `ticker.position.x/y`
- `ticker.width`
- `ticker.padding_x`, `ticker.padding_y`
- `ticker.scale`
- `ticker.separator`
- optional `ticker.frame.top_image/middle_tile_image/bottom_image`

Wenn `ticker.frame` fehlt, wird `ticker_color` als Hintergrund verwendet.

## Betrieb auf dem Pi

- Empfohlen: Raspberry Pi OS Lite (64-bit)
- Python: 3.12 oder 3.13
- Start als systemd-Service (`install/east-display.service`)
- `.env` für API-URL/Token und Laufzeitkonfiguration nutzen

## Neuinstallation (reproduzierbar)

Die folgenden Schritte sind für eine vollständige Neuinstallation auf einem Pi ausgelegt.

### 1. Systempakete installieren

```bash
sudo apt update
sudo apt install -y \
  git curl build-essential pkg-config \
  python3.13 python3.13-venv python3.13-dev \
  libsdl2-dev libsdl2-image-dev libsdl2-ttf-dev \
  libdrm-dev libgbm-dev
```

Wenn auf deinem Pi Python 3.12 statt 3.13 genutzt wird, ersetze `python3.13*` entsprechend.

### 2. Projekt holen (neu) oder aktualisieren (bestehend)

```bash
sudo mkdir -p /opt/paws_on_stream
sudo chown -R pi:pi /opt/paws_on_stream
cd /opt/paws_on_stream

# Neu:
git clone git@github.com:paws-on-stream/display.git

# Oder Update:
cd /opt/paws_on_stream/display
git pull
```

### 3. Poetry installieren und Environment aufsetzen

```bash
cd /opt/paws_on_stream/display
curl -sSL https://install.python-poetry.org | python3 -
export PATH="$HOME/.local/bin:$PATH"

# gewünschte Python-Version binden
poetry env use /usr/bin/python3.13
poetry install --with dev
```

### 4. Pygame explizit aus Source bauen

Damit SDL/KMS/Freetype sauber zur Pi-Umgebung passt:

```bash
cd /opt/paws_on_stream/display
poetry run pip uninstall -y pygame
poetry run pip install --no-binary=pygame pygame==2.6.1
```

### 5. `.env` konfigurieren

```bash
cd /opt/paws_on_stream/display
cp .env_example .env
# dann API_BASE_URL, API_DISPLAY_TOKEN, DISPLAY_DEVICE_ID, KILLSWITCH_TOKEN etc. eintragen
```

### 6. systemd-Service installieren

```bash
sudo cp /opt/paws_on_stream/display/install/east-display.service /etc/systemd/system/east-display.service
sudo systemctl daemon-reload
sudo systemctl enable east-display
sudo systemctl restart east-display
```

### 7. Lauf prüfen

```bash
systemctl status east-display --no-pager
journalctl -u east-display -b -n 200 --no-pager
```

### 8. Update-Prozedur (später)

```bash
cd /opt/paws_on_stream/display
git pull
export PATH="$HOME/.local/bin:$PATH"
poetry install --with dev
poetry run pip install --no-binary=pygame --force-reinstall pygame==2.6.1
sudo systemctl restart east-display
```

## Häufige Probleme

### Theme lädt nicht

- JSON auf Syntaxfehler prüfen
- Asset-Pfade relativ zum Projektverzeichnis prüfen

### Pygame startet nicht / Font-Modul fehlt

- Python-Version prüfen: empfohlen **3.12 oder 3.13**
- Falls Fehler wie `font module not available` auftreten:
  - sicherstellen, dass keine Projektdatei einen Stdlib-Namen überschreibt (z. B. `types.py`)
  - virtuelle Umgebung neu erstellen und Abhängigkeiten neu installieren
- Bei `offscreen`-Rendering auf dem Pi:
  - Service-Environment auf echten Video-Backend-Betrieb prüfen
  - sicherstellen, dass `SDL_VIDEODRIVER` nicht auf `offscreen` gesetzt ist

### Nur Konsole sichtbar

- systemd-Umgebung und Video-Backend prüfen
- sicherstellen, dass der Prozess nicht im `offscreen`-Modus läuft

### Nachrichten doppelt

- Display-Feedback (`displayed`) und Deduplizierung im Client prüfen
