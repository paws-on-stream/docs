# Datenmodell

## Kernobjekte

| Objekt | Zweck |
| --- | --- |
| Participant | Telegram-Nutzer mit Check-in- und Moderationsstatus |
| Event | Laufende oder geplante Convention-/Event-Session |
| Message | Nachricht eines Participants |
| MediaAsset | Persistierte Mediendatei vom Bot |
| DisplayDevice | Zielsystem für die Anzeige |
| DisplayLog | Protokoll der ausgespielten Messages |

## Beziehung Message → MediaAsset

Eine Message kann optional auf ein `MediaAsset` verweisen.

Das ist wichtig, damit:

- die Datei unabhängig von der Message gespeichert wird
- Medien sauber wiederverwendet und angezeigt werden können
- Upload-Fehler nicht zu inkonsistenten URLs führen

