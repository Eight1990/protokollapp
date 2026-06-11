# Protokollapp — Übergabe-Protokoll (Zeit ist Geld GmbH)

Web-App zum Erstellen von Wohnungs-Übergabeprotokollen mit Fotos, Unterschriften und PDF-Export.

## Schnellstart

Einfach `index.html` im Browser öffnen — kein Server, kein Build, keine Installation nötig.
Die gesamte App (HTML + CSS + JavaScript) liegt in **einer einzigen Datei**: `index.html`.

## Workflow in der App

1. **Login** (einfacher Login-Screen)
2. **Schritt 1:** Excel-Belegungsdatei hochladen (Häuser/Zimmer/Mieter werden daraus geparst)
3. **Schritt 2:** Haus auswählen
4. **Schritt 3:** Zimmer auswählen
5. **Schritt 4:** Protokoll ausfüllen:
   - Protokolltyp: **Einzug / Auszug / Momentaufnahme** (Radio-Buttons, `name="protocolType"`)
   - Checkliste mit Zustandsbewertung (Neu/Gebraucht/Abgenutzt/Beschädigt …)
   - Fotos pro Checklisten-Punkt (Datei-Upload oder Kamera)
   - Bemerkungen
   - Zwei Unterschriften (Canvas: Prüfer/Vermieter und Mieter/Mitarbeiter)
   - PDF erzeugen und herunterladen

## Technik

| Bereich | Lösung |
|---|---|
| Excel-Import | SheetJS (`xlsx` via CDN) |
| PDF-Erzeugung | jsPDF 2.5.1 (via CDN) — direkt gezeichnet, **ohne** html2canvas (ist zwar eingebunden, wird aber nicht genutzt) |
| Persistenz | `localStorage` (nur Benutzer, Excel-Daten, Haus-/Zimmer-Auswahl — **keine Fotos!**) |
| State | Globales Objekt `appState` (user, houses, selectedHouse, selectedRoom, excelData, signatures, fotos) |
| Design | ZiG-Branding: Grün `#C5D32A`, Grau `#4D4D4D`, Logos als Base64 eingebettet |

## Wichtige Funktionen (in `index.html`)

| Funktion | Zweck |
|---|---|
| `handleExcelUpload()` | Excel parsen, Häuser/Zimmer extrahieren |
| `goToStep(n)` | Navigation zwischen den 4 Schritten |
| `resizeImageDataUrl(dataUrl)` | Foto auf max. 2560 px lange Kante skalieren, JPEG 90 % (Konstanten `FOTO_MAX_DIM`, `FOTO_JPEG_QUALITY`) |
| `handleFotoUpload(e)` | Datei-Upload → FileReader → Kompression → `appState.fotos[itemId]` |
| `startWebcam()` / `capturePhoto()` | Kamera-Modal, Aufnahme ebenfalls auf 2560 px / 90 % begrenzt |
| `deleteFoto(itemId, fotoId)` | Foto aus State und Vorschau entfernen |
| `generatePDF()` | Protokoll-Objekt zusammenbauen, Validierung (beide Unterschriften Pflicht) |
| `renderPDFPreview(protocol)` | PDF mit jsPDF zeichnen: Titel = Protokolltyp, Info-Tabelle, farbige Checkliste, Unterschriften, dann **ein Foto pro Seite** |
| `resetProtocolState()` | Formular für nächstes Protokoll leeren |
| `saveToLocalStorage()` / `loadFromLocalStorage()` | Metadaten persistieren (keine Fotos) |

## Foto-Handling (wichtig für Qualität!)

- Fotos werden beim Erfassen auf **max. 2560 px lange Kante** skaliert und als **JPEG mit Qualität 0.90** gespeichert — das ergibt ≈ 360 DPI auf voller PDF-Seitenbreite (Schäden bleiben beim Zoomen scharf), hält aber den Speicher bei 10–15 Fotos stabil (~0,5–1,5 MB statt 4–8 MB pro Foto).
- Jedes Foto speichert `{id, data (Base64-JPEG), width, height, timestamp}` in `appState.fotos[itemId]`.
- Im PDF wird jedes Foto **proportional eingepasst** (kein Verzerren), zentriert, eine A4-Seite pro Foto. `width`/`height` aus dem Foto-Objekt; Fallback: `pdf.getImageProperties()`.
- **Anforderung:** Die Foto-Qualität im fertigen PDF muss immer sehr gut bleiben (Schäden im Detail erkennbar) — bei Änderungen an `FOTO_MAX_DIM`/`FOTO_JPEG_QUALITY` nicht unter diese Werte gehen.

## Bekannte Einschränkungen

- **Fotos liegen nur im RAM** — bei Seiten-Reload vor dem PDF-Export gehen sie verloren. (Mögliche spätere Verbesserung: IndexedDB.)
- `localStorage`-Limit ~5–10 MB, wird hauptsächlich von den Excel-Daten belegt.
- PDF-Erzeugung läuft synchron (Modal „PDF wird generiert…" blockiert Doppelklicks).

## Testen nach Änderungen

1. `index.html` im Browser öffnen, kompletten Workflow durchspielen.
2. 12–15 große Fotos (Handy-Auflösung) hochladen → App darf nicht einfrieren.
3. PDF erzeugen: Titel = gewählter Protokolltyp, Fotos unverzerrt und beim Zoomen scharf, eine Seite pro Foto.
4. Alle drei Protokolltypen gegenprüfen (Einzug, Auszug, Momentaufnahme).
