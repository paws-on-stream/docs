# Passives Browser-Display

Das Web-Backend stellt unter `/monitor/` eine responsive Anzeige für einen
normalen Browser bereit. Sie eignet sich als passiver Monitor oder als schnell
einrichtbare zusätzliche Ausgabe, ersetzt aber kein registriertes
Raspberry-Pi-Display. Die Anzeige verwendet dasselbe Theme-Schema v3 wie der
Display-Client, aber mit serverseitig gelieferten Theme-Assets.

## Öffnen

Angemeldete Staff- und Admin-Konten verwenden oben rechts den Button
**Web Display öffnen**. Ihre Staff-Sitzung berechtigt den Feed unmittelbar.

Für ein nicht angemeldetes Anzeigegerät erzeugt ein Admin unter
`/core/web-display/` einen öffentlichen Link:

1. **Link erzeugen** oder **Link rotieren** wählen.
2. Den Link sofort kopieren; der Klartext-Token wird nur in dieser Antwort
   angezeigt.
3. Den Link im vorgesehenen Browser öffnen.
4. Bei Verlust oder unerwünschter Weitergabe den Link rotieren oder widerrufen.

Staff kann den Monitor öffnen, aber öffentliche Links weder erzeugen noch
widerrufen.

Die Linkverwaltung liegt unter `/core/web-display/` und ist auf Admins
beschränkt. Dort wird nur der SHA-256-Digest des Tokens gespeichert; der
Klartext-Token erscheint ausschließlich bei der Generierung.

## Nachrichten und Modi

Beim ersten Feed-Aufruf liefert das Backend nur die zuletzt freigegebene
Nachricht. Danach folgen neue Freigaben aufsteigend nach Freigabezeit und UUID.
Eine Feed-Seite enthält höchstens 20 Nachrichten; signierte Cursor verhindern
Verluste bei identischen Zeitstempeln.

Der Browser unterstützt:

- Chat-Modus mit einer Nachricht für `display_duration_sec`,
- Crawling-Modus mit einer aus Fensterbreite, Inhalt und `scroll_speed_px`
  berechneten Laufzeit,
- Text, Fotos, statisches und animiertes WebP sowie Sticker-Fallback als Emoji,
- responsive Größen; `prefers-reduced-motion` deaktiviert derzeit nur den
  Fade-in im Chat-Modus, nicht die notwendige Tickerbewegung,
- begrenzte Formatierung für fett, kursiv, Code und durchgestrichenen Text.

Namen und Inhalte werden über DOM-Textknoten beziehungsweise `textContent`
eingefügt. Telegram-Text wird nicht als HTML ausgeführt.

## Dynamische Einstellungen

Jede Feed-Antwort liefert die aktuellen globalen Displaysettings. Ohne Neuladen
übernimmt der Browser:

| Einstellung | Wirkung |
| --- | --- |
| `display_mode` | Chat oder Crawling; ein Moduswechsel leert die aktuelle Bühne |
| `display_duration_sec` | Dauer einer Chat-Nachricht |
| `scroll_speed_px` | Geschwindigkeit im Crawling-Modus |
| `overlay_font_size` | Basisschriftgröße, mindestens 12 Pixel |
| `overlay_theme` | aktive Theme-Slug; steuert die geladene Theme-Version |

Wenn das Backend eine serververwaltete Theme-Version aktiviert hat, lädt der
Monitor die passenden Manifest- und Asset-URLs aus dem Feed und rendert das
Theme-Schema v3 browserseitig. Gibt es nur lokale Fallback-Themes, zeigt der
Monitor weiterhin die passende Darstellung ohne zusätzlichen Upload.

## Polling und Fehlerverhalten

Das Backend empfiehlt standardmäßig drei Sekunden bis zum nächsten Poll. Nach
Fehlern verwendet der Browser exponentiellen Backoff mit 6, 12, 24 und maximal
48 Sekunden. Ein `401` blendet statt der Bühne den Zugangshinweis ein. Ein
erfolgreicher Feed-Aufruf setzt den Backoff zurück.

Der Browser hält Queue und Cursor nur im Arbeitsspeicher. Ein Neuladen beginnt
wieder mit der neuesten genehmigten Nachricht.

## Sicherheitsmodell

- Der Klartext-Token steht im URL-Fragment und wird deshalb nicht an den Server
  oder als Referrer gesendet.
- JavaScript entfernt das Fragment vor dem Austausch aus der Adresszeile.
- In der Datenbank liegt nur der SHA-256-Digest des Tokens.
- Der Austausch liefert ein signiertes `HttpOnly`-Cookie mit `SameSite=Strict`,
  Pfad `/monitor/` und maximal 30 Tagen Laufzeit.
- In Produktion besitzt das Cookie zusätzlich das `Secure`-Attribut.
- Rotation ändert die Generation und invalidiert bestehende öffentliche Cookies.
- Widerruf leert den Digest und deaktiviert den öffentlichen Zugriff.
- Nach zehn fehlgeschlagenen Austauschversuchen pro erkannter IP innerhalb des
  Cachefensters antwortet der Endpunkt mit `429`.
- Monitor-Antworten verwenden `Cache-Control: no-store`, `Referrer-Policy:
  no-referrer` und eine restriktive Content Security Policy.

Der Link ist ein Zugangsschlüssel. Er gehört nicht in Tickets, Chatverläufe,
Screenshots oder Versionsverwaltung.

## Bewusst passiver Betrieb

Der Browser registriert kein `DisplayDevice`, sendet keine Acknowledgements und
erzeugt keine `DisplayLog`-Einträge. Eine dort sichtbare Nachricht erhält daher
kein `displayed_at`. Für nachweisbare gerätespezifische Ausspielung muss der
Raspberry-Pi-Client verwendet werden.

## Deployment

Vor der ersten Nutzung muss die Migration `core.0008_webdisplayaccess`
angewendet und der produktive CSS-/JavaScript-Build im Image enthalten sein.
Nach dem Deployment Staff-Zugriff, öffentlichen Link, Rotation, Widerruf, beide
Anzeigemodi, Theme-Ladevorgang und mindestens ein animiertes WebP testen.
