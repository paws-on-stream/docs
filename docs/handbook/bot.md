# Telegram-Bot verwenden

Teilnehmende schreiben dem Paws-on-Stream-Bot direkt in Telegram. Vor jeder
Nachricht prüft der Bot seinen lokalen Status, Registrierung und Check-in. Das
Backend prüft anschließend seine eigenen Moderations-, Participant- und
Eventregeln.

## Befehle für Teilnehmende

| Befehl | Wirkung |
| --- | --- |
| `/start` | Registrierung und Check-in prüfen |
| `/status` | lokalen Bot-, Registrierungs- und Check-in-Status anzeigen |
| `/events` | Events vom Backend abfragen |
| `/help` | Kurzhilfe anzeigen |

Unterstützte Nachrichtenarten und Moderationsstatus stehen im
[Moderationshandbuch](moderation.md).

`/help` zeigt für normale Teilnehmende nur die Standardbefehle. Für Telegram-
Admins werden die Adminbefehle zusätzlich angehängt.

Vor Mediennachrichten fragt der Bot den effektiven Display-Modus beim Backend
ab. Im Crawling-Modus sind nur Textnachrichten erlaubt. Der Modus wird kurz
gecached; ist die API nicht erreichbar, darf der Bot den letzten gültigen Wert
bis zum konfigurierten Maximalalter verwenden. Ohne sicheren Wert lehnt er die
Mediennachricht ab.

Der aktuelle Web-Token-Scope erlaubt einem getrennten Bot-Token keinen Zugriff
auf `/api/v1/events/`. `/events` funktioniert deshalb derzeit nur, wenn der Bot
den globalen API-Token verwendet. Das ist eine bekannte Abweichung vom
Least-Privilege-Konzept und sollte nicht durch unnötig breite Tokenvergabe
kaschiert werden.

## Adminbefehle

Nur Telegram-IDs aus `ADMIN_TELEGRAM_IDS` dürfen diese Befehle verwenden:

| Befehl | Wirkung |
| --- | --- |
| `/ban <telegram_id>` | Participant im Web-Backend bannen |
| `/mute <telegram_id> <minuten>` | Participant zeitlich begrenzt muten |
| `/online` | lokalen Bot-Prozess auf online setzen |
| `/offline` | lokale Annahme neuer Nachrichten stoppen |
| `/maintenance` | lokalen Bot-Prozess in Wartung setzen |

Die drei Statusbefehle ändern die persistierte Bot-Statusdatei des laufenden
Bots. Sie ändern nicht das gleichnamige Feld im Web-Dashboard. Für eine
vollständige Sperre deshalb lokalen Bot-Status und Web-Einstellung gemeinsam
prüfen.

## Typische Ablehnungen

Der Bot informiert den Absender, wenn Registrierung oder Check-in fehlen, ein
Rate-Limit greift, der Bot nicht online ist, der Inhalt nicht unterstützt wird
oder Backend beziehungsweise Statusdienst nicht verfügbar sind. Eine technisch
angenommene Nachricht kann anschließend weiterhin auf Moderation warten.
