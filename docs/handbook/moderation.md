# Nachrichten moderieren

Neue Nachrichten landen normalerweise mit dem Status `pending` in der
Nachrichtenliste. Nur `approved`-Nachrichten werden von Displays abgeholt.

## Nachricht prüfen

1. **Nachrichten** im Dashboard öffnen.
2. Nach Status oder Medientyp filtern; bei Bedarf nach Name, Telegram-ID oder
   Inhalt suchen.
3. Die Detailansicht öffnen und Text, Medium, Participant und Event prüfen.
4. Die Nachricht freigeben oder mit einem Ablehnungsgrund ablehnen.

Mehrere ausstehende Nachrichten lassen sich in der Liste gemeinsam freigeben,
ablehnen oder löschen. Nur `pending`-Nachrichten können regulär freigegeben oder
abgelehnt werden.

## Status verstehen

| Anzeige | Bedeutung |
| --- | --- |
| Pending | wartet auf Moderation |
| Approved | für Displays freigegeben |
| Rejected | endgültig abgelehnt |
| Shown | mindestens ein Display hat die Anzeige bestätigt |
| Approved, not shown | freigegeben, aber noch von keinem Display bestätigt |

Ein Display-Ack verändert den Moderationsstatus nicht. Die Nachricht bleibt
`approved`, damit jedes aktive Display sie unabhängig abholen kann. Die
gerätespezifische Historie steht in den Display-Logs.

## Automatische Freigabe

Jede neue Nachricht erhält einen Spam-Score zwischen `0.0` und `1.0`. Erreicht
oder überschreitet der Score den konfigurierten Grenzwert, bleibt die Nachricht
immer `pending` und wird im Dashboard hervorgehoben. Sie wird nicht automatisch
abgelehnt.

Ist `auto_approve` aktiviert, gibt das Backend nur Nachrichten mit einem echt
niedrigeren Score automatisch frei. In Detail- und Listenansicht ist der Score
sichtbar; die Liste kann danach sortiert werden. Auffällige Pending-Nachrichten
lassen sich einzeln oder gesammelt mit **Als Spam ablehnen** auf `rejected` und
Ablehnungsgrund `spam` setzen.

Filterregeln, Gewichtung und Teilnehmerhistorie stehen ausschließlich in der
[Spamfilter-Referenz](../spam-filter.md).

## Teilnehmende moderieren

In der Participant-Detailansicht können Staff-Mitglieder:

- einen Participant bannen oder entbannen,
- zeitlich begrenzt muten,
- den Check-in-Status lokal überschreiben,
- Nachrichten und letzten Statusabgleich einsehen.

Gebannte, aktuell gemutete oder nicht eingecheckte Teilnehmende können keine
neuen Nachrichten einstellen. Der lokale Check-in-Override hat Vorrang vor dem
synchronisierten Wert.

## Akzeptierte Inhalte

Unterstützt werden Text, Fotos, GIFs beziehungsweise Telegram-Animationen und
Sticker. Medien werden vor dem Upload in statisches oder animiertes WebP
normalisiert. Videos, Video Notes, Audio, Voice, Dokumente und andere
Telegram-Inhaltstypen werden vom Bot abgelehnt.
