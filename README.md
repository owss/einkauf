# Einkaufsliste · Список покупок

Kleine Web-App ohne Server: abends Mengen eintragen, Liste als Link/QR-Code ans Handy schicken, beim Einkauf abhaken. Funktioniert nach dem ersten Öffnen auch offline.

## Dateien

| Datei | Zweck |
|---|---|
| `index.html` | die ganze App (HTML, CSS, JS) |
| `catalog.json` | Warengruppen, Artikel (3 Sprachvarianten), Einheiten |
| `stores.json` | Reihenfolge der Warengruppen je Laden |
| `qrcode.js` | QR-Code-Generator (Kazuhiko Arase, MIT-Lizenz, siehe `THIRD_PARTY_LICENSES.md`) |
| `THIRD_PARTY_LICENSES.md` | Lizenztexte der mitgelieferten Fremdbibliotheken |
| `.nojekyll` | GitHub Pages veröffentlicht die Dateien direkt, ohne Jekyll-Build |
| `robots.txt` | hält Suchmaschinen fern (zusätzlich `noindex` in `index.html`) |
| `manifest.json`, `sw.js`, `icon-*.png` | Installation auf dem Startbildschirm, Offline-Cache |

## Lokal testen

```bash
cd einkaufsapp
python -m http.server 8000
# → http://localhost:8000
```

Direkt per Doppelklick (`file://`) geht es nicht, weil der Browser dann die JSON-Dateien nicht lädt.

## Auf GitHub Pages veröffentlichen

1. Auf github.com ein neues Repository anlegen, z. B. `einkauf`, Sichtbarkeit **Public** (Pages ist mit einem Free-Account nur für öffentliche Repos verfügbar).
2. Alle Dateien aus diesem Ordner hochladen: *Add file → Upload files* → Commit.
   Oder per Git:
   ```bash
   git init && git add . && git commit -m "Einkaufsliste"
   git branch -M main
   git remote add origin https://github.com/<user>/einkauf.git
   git push -u origin main
   ```
3. *Settings → Pages → Build and deployment*: Source **Deploy from a branch**, Branch **main**, Ordner **/ (root)** → *Save*.
4. Nach ca. einer Minute läuft die App unter `https://<user>.github.io/einkauf/`.
5. Auf den Handys einmal öffnen und *Zum Startbildschirm hinzufügen* (Android: Menü ⋮, iPhone: Teilen-Symbol).

Öffentlich ist nur der Katalog. Was ihr tatsächlich einkauft, steht ausschließlich im geteilten Link.

## Bedienung

- **Eintragen:** Der aktuelle Artikel ist aufgeklappt. Menge per −/+ oder Eingabe, Einheit wählen, *Weiter* springt zum nächsten. *Nicht nötig* setzt die Menge auf 0. Jeder Artikel lässt sich jederzeit wieder antippen und ändern. Der Entwurf bleibt auf dem Gerät gespeichert.
- **+ Sonstiges:** einmaliger Artikel, der nicht im Katalog steht.
- **Senden:** WhatsApp-Knöpfe für gespeicherte Empfänger öffnen den Chat mit fertig ausgefüllter Nachricht – nur noch auf *Senden* tippen. *WhatsApp: Chat auswählen …* öffnet die Auswahl (auch Gruppen). Darunter QR-Code, Link und *Teilen …* für andere Wege.
- **Empfänger:** im Sende-Dialog unter *Empfänger bearbeiten*. Name und Handynummer werden nur auf dem jeweiligen Gerät gespeichert (nicht im Repo). Nummern ohne Ländervorwahl werden als deutsche Nummern behandelt (`0170…` → `+49170…`).
- **Zu zweit eintragen:** A schickt B den Link → B öffnet ihn, *Liste bearbeiten*, ergänzt, sendet den neuen Link weiter.
- **Einkaufen:** Nur Artikel mit Menge > 0 erscheinen. Laden oben wählen (sortiert die Warengruppen), antippen hakt ab. Häkchen bleiben auf dem Handy gespeichert, auch ohne Netz.
- **Sprache:** *Ориг.* (Wortlaut wie auf dem Papierzettel), *RU* (korrektes Russisch), *DE*.

Tipp: Links aus Messengern öffnen sich oft im eingebauten Browser der App. Für zuverlässig gespeicherte Häkchen lieber *Im Browser öffnen* wählen.

## Katalog pflegen (`catalog.json`)

Direkt auf GitHub bearbeiten (Stift-Symbol) → *Commit changes*. Wegen des Offline-Caches sind Änderungen spätestens beim zweiten Öffnen der App sichtbar.

Ein Artikel:

```json
{ "id": 112, "group": "putzen", "unit": "stk", "names": { "ru_orig": "Батарейки", "ru": "Батарейки", "de": "Batterien" } }
```

- **`id` niemals wiederverwenden oder ändern.** Die Links speichern nur die ID. Neue Artikel bekommen die nächste freie Zahl; gelöschte IDs bleiben frei.
- `group`: eine der IDs aus `groups` (`obst`, `fleisch`, `brot`, `getraenke`, `molkerei`, `tk`, `nudeln`, `konserven`, `backen`, `gewuerze`, `suess`, `snacks`, `pflege`, `putzen`, `papier`).
- `unit`: Standardeinheit, eine der IDs aus `units` (`stk`, `kg`, `g`, `l`, `pkg`, `fl`, `glas`, `dose`, `bund`). `step` bei einer Einheit bestimmt, um wie viel +/− ändert.
- `names`: `ru_orig`, `ru`, `de`. Fehlt `ru_orig`, wird `ru` angezeigt.
- Die Reihenfolge der Artikel in der Datei ist die Reihenfolge beim Eintragen.

## Läden (`stores.json`)

`order` ist die Reihenfolge der Warengruppen beim Durchlaufen des Ladens. Nicht genannte Gruppen kommen in Katalog-Reihenfolge ans Ende. Ein weiterer Laden ist einfach ein weiterer Eintrag.

## Link-Format

```
…/#v1_<id>-<menge>[-<einheit>]_…_x-<name>-<menge>-<einheit>-<gruppe>
```

Beispiel: `#v1_1-1.5_30-2_4-1-kg_x-Kerzen-2-stk-putzen` — Einheit nur, wenn sie von der Standardeinheit abweicht; `x-…` ist ein einmaliger Artikel (Name URL-kodiert, `-` und `_` maskiert).

## Aktualisieren der App selbst

Bei Änderungen an `index.html` reicht ebenfalls ein Commit. Wer ganz sicher sofort die neue Version will, erhöht in `sw.js` die Zeile `const CACHE = 'einkauf-v1';` auf `v2`.
