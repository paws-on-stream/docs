# Events und Einstellungen

Das aktive Event bestimmt, ob Nachrichten angenommen werden und wie sie
angezeigt werden. Es kann höchstens ein Event gleichzeitig aktiv sein.

## Event vorbereiten

1. Unter **Events** ein Event anlegen oder ein synchronisiertes Event öffnen.
2. Start und Ende prüfen.
3. `allow_messages` aktivieren, wenn der Bot Nachrichten annehmen soll.
4. Optional den Display-Modus setzen. Bei einem laufenden, aktiven Event mit
   `allow_messages` ist dieser Wert der effektive Modus für die Bot-Regeln. Ein
   leerer Wert fällt auf den globalen Modus zurück. Der Pi-Display-Client liest
   für sein Rendering weiterhin die globalen Settings; die eventbezogene
   Scroll-Geschwindigkeit wird derzeit nicht angewendet.
5. Das Event aktivieren. Ein zuvor aktives Event wird dabei deaktiviert.

Wenn die globale Option `require_event_active` aktiv ist, nimmt das Backend nur
dann Nachrichten an, wenn das aktive Event innerhalb seines Zeitfensters liegt
und `allow_messages` gesetzt ist.

## Globale Einstellungen

Unter **Einstellungen** werden unter anderem gesteuert:

- Bot-Status `online`, `offline` oder `maintenance`,
- maximale Nachrichtenlänge und API-Rate-Limit,
- automatische Freigabe und Spam-Grenzwert zwischen `0.10` und `1.00`,
- Theme, Schriftgröße, Anzeigedauer und Display-Modus,
- Reg- und Event-Synchronisierung.

Die Bedeutung aller Felder steht zentral im Abschnitt
[Laufzeiteinstellungen](../configuration.md#laufzeiteinstellungen-im-dashboard).

## Externe Events synchronisieren

Die Event-API und der jq-Filter werden in den globalen Einstellungen gepflegt.
Der Filter muss eine JSON-Liste liefern. Der Sync aktualisiert Name, Beginn und
Ende anhand der externen ID; lokale Felder wie Aktivstatus, `allow_messages` und
Display-Modus bleiben erhalten.

Der regelmäßige und der manuelle Aufruf stehen im [Runbook](../operations/runbook.md#synchronisierung).

## Vor dem Einlass prüfen

- genau das gewünschte Event ist aktiv,
- Beginn, Ende und Zeitzone stimmen,
- `allow_messages` und `require_event_active` haben den gewünschten Wert,
- Bot-Status ist `online`,
- globale Theme- und Anzeigeeinstellungen sind korrekt,
- mindestens ein Display ist aktiv und meldet sich regelmäßig.
