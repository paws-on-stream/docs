# Rollen und Anmeldung

Das Dashboard ist nicht öffentlich. Jede Ansicht erfordert eine aktive
Staff-Sitzung. Bot und Display melden sich nicht am Dashboard an, sondern
verwenden eigene API-Tokens.

## Rollen

| Rolle | Rechte |
| --- | --- |
| Staff | Nachrichten moderieren sowie Teilnehmende, Events, Einstellungen, Geräte und Logs bearbeiten |
| Admin | Alle Staff-Rechte plus Zugänge und passiven Web-Display-Link verwalten |

Die Autorisierung eines Telegram-Zugangs basiert immer auf der numerischen
Telegram-ID. Ein Telegram-Benutzername ist nur eine Anzeigeinformation.

## Anmelden

1. `/auth/login/` öffnen.
2. Entweder ein vorhandenes Django-Staff-Konto verwenden oder die Anmeldung mit
   Telegram starten.
3. Beim ersten Telegram-Login wird ein inaktiver Zugangsantrag angelegt. Bis zur
   Freigabe erscheint die Ablehnungsseite.
4. Nach der Freigabe erneut anmelden.

## Zugang freigeben

Diese Aktion erfordert die Rolle Admin.

1. Im Dashboard **Zugänge** öffnen.
2. Den Antrag anhand der numerischen Telegram-ID prüfen.
3. Staff oder Admin auswählen und den Zugang aktivieren.

Wird ein Zugang deaktiviert, endet eine bestehende Sitzung bei der nächsten
Anfrage. Für den ersten Admin kann beim Deployment einmalig
`TELEGRAM_AUTH_BOOTSTRAP_IDS` gesetzt werden. Danach werden Zugänge im Dashboard
verwaltet; Details stehen in der [Konfigurationsreferenz](../configuration.md#web-backend).
