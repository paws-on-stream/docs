# Display-Themes

Diese Seite ist die zentrale Referenz für das Theme-System des Display-Clients
und des Web-Monitors. Es gibt zwei gültige Quellen für Themes:

- lokale Fallback-Themes im Display-Repository, weiterhin im Schema v2,
- serververwaltete Themes im Schema v3 mit Manifest und Asset-Dateien.

Der aktive Theme-Name bleibt `overlay_theme`. Der Display-Client und der
Browser-Monitor lösen denselben Theme-Slug auf, aber nicht auf dieselbe Weise:
der Display-Client nutzt lokale Fallback-Dateien oder zentrale v3-Versionen,
der Browser lädt die zentrale Darstellung über das Web-Backend.

## Lokale Fallback-Themes

Lokale Themes liegen im aktiven Release unter
`display_client/assets/themes/<name>/theme.json`.

```text
display_client/assets/themes/east13/
├── theme.json
├── chat-top.png
├── chat-middle.png
└── chat-bottom.png
```

Das Schema v2 bleibt für die mitgelieferten Themes erhalten. Wenn eine
serververwaltete Version nicht verfügbar ist oder beim Laden fehlschlägt, fällt
der Client auf die letzte gültige zentrale Version, dann auf lokale `east13`
und schließlich auf lokales `east-default` zurück.

## Serververwaltete Themes

Schema v3 wird im Web-Backend unter `core/themes/` verwaltet. Admins können dort
Theme-ZIPs importieren, Versionen aktivieren und nicht aktive Uploads löschen.
Der zugehörige API-Vertrag ist:

```text
GET /api/v1/themes/<slug>/
GET /api/v1/themes/<slug>/<version>/assets/<asset_id>/
```

Der Display-Client erhält das Manifest mit `overlay_theme_version` und
`overlay_theme_manifest_url` oder alternativ mit
`overlay_theme_package: {"version": "...", "manifest_url": "..."}`. Manifest
und Assets werden per HTTP geladen, geprüft und erst danach aktiviert. Für
zentrale Themes sind Frame-Assets Pflicht.

Der Browser-Monitor erhält dieselbe Theme-Information im Feed und lädt Assets
über die Monitor-Route:

```text
GET /monitor/themes/<slug>/<version>/assets/<asset_id>/
```

## Referenzprofil

`broadcast-1080p50` beschreibt die logische Renderfläche 1920 × 1080 bei
50 Hz. Der tatsächliche HDMI-Modus wird vom Betriebssystem gesetzt und nicht
vom Theme. Themes bestimmen also weder Auflösung noch Bildfrequenz.

## EAST-Frame

EAST verwendet `segmented_vertical`:

1. Top und Bottom behalten ihre native Höhe und werden horizontal skaliert.
2. Middle wird horizontal skaliert und vertikal gekachelt.
3. Der Content wird mit dem im Theme definierten Padding eingesetzt.
4. Danach wird die zusammengesetzte Bubble mit `chat.scale` skaliert.

Das ist bewusst kein Nine-Slice-Verfahren.

## Template

`chat.template.elements` ist eine geordnete Liste. Erlaubt sind ausschließlich:

- `display_name`
- `content`
- `media`
- `sticker_emoji`

Jedes Element verweist auf einen Eintrag aus `chat.styles`. Leere Felder werden
nicht gerendert. Freies HTML, Jinja, Python, JSONQ oder andere ausführbare
Ausdrücke sind nicht erlaubt.

Die v2-Kurzform `margin_bottom` bleibt in v3 erhalten. Clients dürfen sie
intern in ein vollständiges Margin-Modell normalisieren.

## Sicherheit

- Theme- und Asset-IDs sind auf kleine ASCII-Slugs begrenzt.
- Asset-Pfade müssen einfache Dateinamen im Theme-Ordner sein.
- Zunächst werden ausschließlich PNG-Theme-Assets akzeptiert.
- Tatsächliches Format, Maße, Alpha und SHA-256 werden geprüft.
- Ein Manifest ist auf 256 KiB, ein Asset auf 5 MiB und ein Theme auf 32
  Assets begrenzt.
- Ein Template darf höchstens 16 Elemente enthalten.
- Eine unbekannte Schema-Hauptversion wird nicht aktiviert.

Nachrichtenmedien gehören nicht zum Theme-Paket. Sie werden weiterhin über die
separate, ausschließlich WebP-basierte Medienpipeline ausgeliefert.

## Cache-Verhalten des Display-Clients

Der Client lädt neue zentrale Themes zunächst vollständig in ein separates
Verzeichnis und validiert sie dort. Erst danach wird die Version atomar
aktiviert. Die Fallback-Reihenfolge lautet:

1. vollständig validierte zentrale Version,
2. zuletzt gültige gecachte Version desselben Themes,
3. mitgeliefertes lokales `east13`,
4. mitgeliefertes lokales `east-default`.

Ein Download- oder Validierungsfehler darf das aktive Theme nicht verändern.

## Dashboard-Verwaltung

Nur Admins können `/core/themes/` öffnen. Die Verwaltung unterstützt:

- Import eines vollständigen ZIP-Pakets,
- mehrere unveränderliche Versionen pro Theme-Slug,
- Aktivierung einer hochgeladenen oder eingebauten Version,
- gemeinsame Aktivierung für Display-Client und Web-Vorschau,
- Löschen nicht aktiver hochgeladener Versionen,
- unveränderliche eingebaute Fallback-Themes.

Das ZIP enthält `theme.json` und exakt die dort deklarierten Dateien direkt im
Wurzelverzeichnis. Verzeichnisse, Symlinks, verschlüsselte Einträge, doppelte
Dateinamen und nicht deklarierte Dateien werden abgelehnt. Das Paket ist auf
10 MiB komprimiert und 24 MiB entpackt begrenzt.
