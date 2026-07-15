# Troubleshooting

## Bot sendet nichts ans Backend

Prüfen:

- `API_BASE_URL`
- `API_AUTH_TOKEN`
- ob das Backend läuft
- ob der Nutzer eingecheckt und nicht gemutet ist

## Medien werden nicht angezeigt

Prüfen:

- ob das Asset wirklich unter `/media/` gespeichert wurde
- ob die Datei-Erweiterung zum Inhalt passt (`.mp4`, `.webm`, `.gif`)
- ob die URL nicht doppelt mit `//` aufgebaut wurde

## GitHub Pages baut nicht

Prüfen:

- ob `mkdocs.yml` valide ist
- ob `requirements.txt` installiert werden kann
- ob `mkdocs build --strict` Fehler meldet

