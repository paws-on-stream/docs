# API

## Basis

`/api/v1/`

## Bot-Integration

### `POST /api/v1/media/upload/`

Lädt Medien hoch und speichert sie als `MediaAsset`.

### `POST /api/v1/message/`

Übergibt eine neue Nachricht aus dem Bot an das Backend.

### `GET /api/v1/health/`

Health-Check für Bot und Monitoring.

## Participant-Status

### `GET /api/v1/participants/{telegram_id}/check_status/`

Liefert den Check-in-Status; fällt bei Upstream-Problemen auf lokale DB-Daten zurück.

## Moderation

### Messages

- `pending`
- `display`
- `{id}/approve`
- `{id}/reject`
- `{id}/displayed`

### Participants

- `participant/{telegram_id}/ban/`
- `participant/{telegram_id}/mute/`

