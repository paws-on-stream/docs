# Spamfilter

Das Web-Backend bewertet jede neu angelegte Nachricht mit einem Spam-Score von
`0.0` bis `1.0`. Der konfigurierbare Grenzwert liegt standardmäßig bei `0.70`.
Der Filter lehnt Nachrichten nicht automatisch ab: Auffällige Nachrichten
bleiben zur manuellen Prüfung `pending`.

## Pipeline

Die Teilwerte werden in dieser festen Reihenfolge addiert und nach jedem Schritt
auf zwei Nachkommastellen sowie insgesamt auf höchstens `1.0` begrenzt:

| Reihenfolge | Filter | Bedingung | Teilwert |
| --- | --- | --- | --- |
| 1 | LengthFilter | bereinigter Text kürzer als drei Zeichen; reine Mediennachrichten sind ausgenommen | `0.30` |
| 2 | RepeatCharFilter | mindestens sechs gleiche Nicht-Leerzeichen direkt hintereinander | `0.20` |
| 3 | EmojiSpamFilter | mehr als zehn erkannte Emojis | `0.30` |
| 4 | URLFilter | mindestens zwei `http://`-, `https://`- oder `www.`-URLs | `0.20` |
| 5 | RateFilter | mindestens drei vorherige Nachrichten desselben Participants innerhalb von zehn Sekunden | `0.20` |
| 6 | ParticipantSpamHistory | persistente Vorgeschichte gemäß Staffelung unten | `0.00` bis `0.50` |

Sobald die laufende Summe den Grenzwert erreicht oder überschreitet, endet die
Pipeline. Spätere Filter werden dann nicht mehr ausgewertet. Dieses
Short-Circuiting reduziert Datenbankarbeit und sorgt dafür, dass der gespeicherte
Score nicht zwingend der theoretischen Summe aller sechs Filter entspricht.

## Participant-Historie

`Participant.spam_count` zählt bisherige Nachrichten dieses Participants, die
bei ihrer Erstellung den damals gültigen Grenzwert erreicht oder überschritten
haben. Die Historie steuert den letzten Filter:

| `spam_count` | Teilwert |
| --- | --- |
| `0` | `0.00` |
| `1–2` | `0.10` |
| `3–5` | `0.20` |
| `6–9` | `0.30` |
| `10–19` | `0.40` |
| ab `20` | `0.50` |

Scoreberechnung, Speichern der Message und Erhöhen der Historie laufen in einer
Datenbanktransaktion. Der Participant wird dabei für konkurrierende
Nachrichteneingänge gesperrt, damit parallele Requests die Historie nicht
verlieren.

Eine spätere manuelle Aktion **Als Spam ablehnen** erhöht `spam_count` nicht
nochmals. Maßgeblich für die Historie ist die automatische Bewertung beim
Nachrichteneingang.

## Zusammenspiel mit Auto-Approve

```text
Score >= Grenzwert
  → Message bleibt pending
  → spam_count wird um 1 erhöht

Score < Grenzwert und Auto-Approve aktiv
  → Message wird approved

Score < Grenzwert und Auto-Approve inaktiv
  → Message bleibt pending
```

Der Grenzwert ist einschließlich: Ein Score von `0.70` ist bei einem Grenzwert
von `0.70` bereits auffällig. Nur ein echt niedrigerer Score darf automatisch
freigegeben werden.

## Migration bestehender Daten

Die Migrationen stellen frühere ganzzahlige Werte auf den Bereich `0.0` bis
`1.0` um:

- `streaming.0010_normalize_spam_score` teilt bestehende, von Null verschiedene
  Message-Scores durch zehn, begrenzt sie auf den gültigen Bereich und ergänzt
  `spam` als Ablehnungsgrund.
- `core.0012_normalize_spam_threshold` setzt den bisherigen Standardwert `5`
  auf `0.70`; andere Altwerte werden durch zehn geteilt und auf `0.10` bis
  `1.0` begrenzt.

Die Migrationen müssen vor dem Start des aktualisierten Web-Backends ausgeführt
werden. Der normale Produktionsablauf steht unter
[Produktionssystem](installation/production.md#web-backend-ausrollen).
