# Regie und Displaysteuerung

Jeder Display-Client bietet lokal eine Regie-API. Sie wirkt unmittelbar auf
dieses Gerät und verwendet den Header `X-Killswitch-Token`.

| Aktion | Wirkung |
| --- | --- |
| Pause | friert die Verarbeitung neuer Inhalte ein |
| Resume | setzt die Verarbeitung nach einer Pause fort |
| Clear | leert sichtbare Inhalte sowie lokalen Message- und Mediencache |
| Killswitch | leert die Anzeige sofort und sperrt die weitere Ausgabe |
| Killswitch reset | hebt die Sperre wieder auf |

Beispiel für ein Display unter `display-ost:8765`, wenn die Regie-API bewusst an
eine vom Regienetz erreichbare Adresse gebunden wurde:

```bash
curl -X POST \
  -H "X-Killswitch-Token: $KILLSWITCH_TOKEN" \
  http://display-ost:8765/api/pause
```

Die weiteren Pfade sind:

```text
POST /api/resume
POST /api/clear
POST /api/killswitch
POST /api/killswitch/reset
GET  /health
GET  /readiness
```

`health` und `readiness` benötigen keinen Regie-Token. Der Killswitch wird lokal
persistiert und als Display-Event an das Backend nachgeliefert, sobald dieses
erreichbar ist.

## Sicherer Ablauf bei problematischen Inhalten

1. Killswitch für das betroffene Display auslösen.
2. Problematische Nachrichten im Dashboard identifizieren.
3. Anzeige und Cache mit Clear leeren, falls nötig.
4. Ursache beheben und Display-Health prüfen.
5. Killswitch zurücksetzen und anschließend Resume ausführen.

Die Produktionsvorlage bindet die Regie-API sicher an `127.0.0.1:8765`. Für
Remote-Regie muss `REGIE_API_HOST` bewusst angepasst und der Zugriff per
Firewall auf das Regienetz begrenzt werden. Ein leerer oder beispielhafter
`KILLSWITCH_TOKEN` darf nicht produktiv verwendet werden.

## Passives Web-Display

Das Web-Backend bietet unter `/monitor/` zusätzlich eine browserbasierte
Anzeige. Sie besitzt keinen Killswitch und keine Geräte-Acks. Einrichtung,
Zugangslink und genaues Anzeigeverhalten stehen im eigenen Handbuch für das
[passive Browser-Display](monitor.md).
