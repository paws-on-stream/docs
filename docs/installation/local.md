# Lokale Entwicklung

Diese Anleitung startet Web, Bot und Display aus den drei benachbarten
Repositories. Für reine Backend-Arbeit reichen Web und die dortigen Tests;
Display und Bot sind dann optional.

## Voraussetzungen

- Python 3.12 oder 3.13
- Poetry
- Node.js und npm
- Git
- ein Telegram-Bot-Token, wenn der echte Bot getestet wird

Erwartete Verzeichnisstruktur:

```text
workspace/
├── web/
├── bot/
├── display/
└── paws-on-stream-docs/
```

## 1. Web-Backend

```bash
cd web
cp .env_example .env
poetry install
npm ci
npm run build
poetry run python paws_on_stream_web/manage.py migrate
poetry run python paws_on_stream_web/manage.py runserver
```

Für eine lokale SQLite-Instanz genügen beispielsweise:

```dotenv
DEBUG=True
SECRET_KEY=local-development-only
DATABASE_URL=sqlite:///db.sqlite3
API_AUTH_TOKEN=local-shared-token
BOT_API_AUTH_TOKEN=local-bot-token
DISPLAY_API_AUTH_TOKEN=local-display-token
ALLOWED_HOSTS=localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=http://localhost:8000
```

Migrationen und Tests:

```bash
poetry run python paws_on_stream_web/manage.py test
poetry run ruff check paws_on_stream_web
```

## 2. Telegram-Bot

```bash
cd bot
poetry install --with dev
TELEGRAM_BOT_TOKEN=<telegram-token> \
API_BASE_URL=http://127.0.0.1:8000 \
API_AUTH_TOKEN=local-bot-token \
ADMIN_TELEGRAM_IDS=<deine-telegram-id> \
poetry run python -m paws_on_stream_bot.main --polling
```

Ohne Web-Backend kann `python mock_api.py` im Bot-Repo verwendet werden. Der
Bot zeigt dann mit `API_BASE_URL=http://localhost:9000` auf das Mock-Backend.

Tests:

```bash
TELEGRAM_BOT_TOKEN=dummy poetry run pytest -v
poetry run ruff check .
poetry run ruff format --check .
```

## 3. Display-Client

Im Display-Repo fehlt derzeit eine versionierte `.env_example`. Deshalb die
Datei `.env` selbst mit mindestens folgenden Werten anlegen:

```dotenv
API_BASE_URL=http://127.0.0.1:8000
API_DISPLAY_TOKEN=local-display-token
DISPLAY_DEVICE_ID=display-dev
DISPLAY_HOSTNAME=localhost
DISPLAY_LOCATION=Entwicklung
KILLSWITCH_TOKEN=local-killswitch-token
ALLOWED_MEDIA_HOSTS=127.0.0.1,localhost
LUMA_MODE=False
```

Dann installieren und im Fenstermodus starten:

```bash
cd display
poetry install --with dev
poetry run python -m display_client.main --dev-mode --no-luma-mode
```

Tests:

```bash
poetry run pytest -q
poetry run ruff check .
poetry run ruff format --check .
```

Alle Konfigurationswerte und Defaults stehen in der
[Konfigurationsreferenz](../configuration.md).
