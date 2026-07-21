# Datenmodell

Alle zentralen fachlichen Daten liegen im Web-Backend.

## Beziehungen

```text
Participant 1 ─── * Message * ─── 0..1 Event
                         │
                         └────── 0..1 MediaAsset

Message 1 ─── * DisplayLog * ─── 1 DisplayDevice
DisplayDevice 1 ─── * DisplayEvent

TelegramAccess ─── 0..1 Django User
WebDisplayAccess            Settings            SyncLock
DisplayThemeVersion ─── * DisplayThemeAsset
```

## Kernobjekte

| Objekt | Zweck und wichtige Felder |
| --- | --- |
| Participant | Telegram-ID, Reg-ID, Anzeigename, Check-in, Ban, Mute und persistenter `spam_count` |
| Event | externe ID, Zeitraum, Aktivstatus, Nachrichtenfreigabe und optionale Display-Overrides |
| Message | UUID, Inhalt, Participant, Event, Medium, Spam-Score von 0,0 bis 1,0, Moderationsstatus und Zeitstempel |
| MediaAsset | validierte WebP-Datei samt Format-, Größen-, Animations- und SHA-256-Metadaten |
| DisplayDevice | stabile Geräte-ID, Hostname, Ort, Aktivstatus und letzter Kontakt |
| DisplayLog | gerätespezifische Bestätigung einer Message einschließlich tatsächlicher Anzeigedauer |
| DisplayEvent | persistiertes Regieereignis wie Killswitch, Pause, Resume oder Clear |
| Settings | Singleton mit globalen Laufzeit-, Sync-, Moderations- und Displayeinstellungen |
| TelegramAccess | numerische Telegram-Identität, Aktivstatus und Rolle für das Dashboard |
| WebDisplayAccess | Singleton für Digest, Generation und Aktivstatus des passiven Browser-Displays |
| DisplayThemeVersion | hochgeladene oder eingebaute Theme-Version mit Manifest und Aktivstatus |
| DisplayThemeAsset | deklarierte Theme-Datei mit Pfad, SHA-256 und Content-Type |
| SyncLock | Datenbank-Lease gegen parallel laufende Sync-Jobs |

## Zustände und Regeln

Eine Message besitzt genau einen Moderationsstatus:

```text
pending ── approve ──► approved
   │
   └──── reject ─────► rejected
```

Nur `pending` kann über die API freigegeben oder abgelehnt werden. Ein
Display-Ack erzeugt oder aktualisiert einen `DisplayLog` und setzt beim ersten
Ack `Message.displayed_at`; er ändert `approved` nicht.

Weitere Invarianten:

- Es gibt höchstens ein aktives Event.
- `ends_at` liegt nach `starts_at`.
- Die numerische Telegram-ID eines Participants ist eindeutig.
- Eine Message verweist optional auf genau ein MediaAsset.
- Für Mediennachrichten ist ein MediaAsset des passenden Typs Pflicht.
- Pro Kombination aus Message und Display ist der DisplayLog logisch eindeutig.
- Ein Check-in-Override überschreibt den synchronisierten Check-in-Wert.
- Erreicht eine neue Message den Spam-Grenzwert, bleibt sie `pending` und der
  `spam_count` ihres Participants wird atomar erhöht.
- Pro Theme-Slug kann genau eine hochgeladene Version `is_current=True` sein.
- Ein aktives Theme in `Settings.overlay_theme` kann auf eine eingebaute oder
  hochgeladene Version zeigen; die dazugehörige `version` und `manifest` werden
  zur Laufzeit im API-Response ergänzt.

Berechnung und Gewichtung des Scores stehen in der
[Spamfilter-Referenz](spam-filter.md).
