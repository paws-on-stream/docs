# Web-Anwendung

## Aufgabe

Die Web-Anwendung ist das Backend und die Moderationsoberfläche.

## Hauptbereiche

- Dashboard
- Nachrichtenverwaltung
- Teilnehmerverwaltung
- Eventverwaltung
- Settings
- API für Bot und Displays

## Nachrichten-Lifecycle

1. Nachricht kommt vom Bot an.
2. Web validiert Participant, Event und Medien.
3. Nachricht wird gespeichert.
4. Moderation kann sie freigeben, ablehnen oder löschen.
5. Freigegebene Nachrichten können an Displays weitergegeben werden.

## Medien

Medien werden als eigene Assets gespeichert.

Unterstützt:

- Photos
- GIFs / Animationen
- Sticker

## Reg-Status

Der Check-in-Status eines Participants kann:

- direkt über die API abgefragt werden
- oder über den lokalen DB-Stand bereitgestellt werden, wenn der Upstream nicht erreichbar ist

## UI

Wichtige Ansichten:

- Nachrichtenliste mit Filtern
- Message-Detail
- Participant-Detail
- Event-Listen und -Details

