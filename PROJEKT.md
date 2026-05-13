# Kalorien Tracker – Projektdokumentation

**Stand:** 13. Mai 2026  
**Erstellt mit:** Claude (Anthropic)  
**GitHub:** https://github.com/LeipnitzKarsten/kalorien-tracker  
**Live-URL:** https://leipnitzkarsten.github.io/kalorien-tracker/

---

## Hintergrund

Karsten hat Yazio Pro (59,90 €/Jahr) durch diese eigene PWA ersetzt. Er nutzte von Yazio nur den Kalorienzähler – keine Rezepte, kein Fasten, keine Statistiken. Die App ist vollständig selbstgebaut und kostenlos.

---

## Tagesziele (fest eingestellt)

| Nährwert      | Ziel    |
|---------------|---------|
| Kalorien      | 1.980 kcal |
| Kohlenhydrate | 169 g   |
| Eiweiß        | 169 g   |
| Fett          | 64 g    |

Diese Werte stammen aus Karstens Yazio-Profil (Screenshot vom 13.05.2026) und sind direkt im Code hinterlegt (`const GOAL = { kcal: 1980, kh: 169, ew: 169, fat: 64 }`).

---

## Funktionen

### Was die App macht
- **Tagesübersicht:** Ring-Anzeige mit gegessen / übrig / Tagesziel
- **Makros:** Fortschrittsbalken für KH / Eiweiß / Fett
- **Essen erfassen** auf zwei Wegen:
  - Textsuche → Open Food Facts Datenbank
  - Barcode scannen → iPhone-Kamera → Open Food Facts
- **Drum-Roller:** iOS-typisches Rollrad für Mengenauswahl (1–500 g), synchronisiert mit manuellem Textfeld darunter
- **Häufig verwendet:** Die 8 zuletzt/meistgenutzten Produkte erscheinen direkt beim Öffnen von „Essen hinzufügen" — kein Suchen nötig
- **7 Tage Verlauf:** Tagesnavigation für die letzten 7 Tage (Chips oben)
- **Einträge bearbeiten:** Menge ändern oder Eintrag löschen (nur für den aktuellen Tag)
- **Offline-fähig:** Service Worker cached die App-Dateien; Dateneingabe geht offline, Suche/Barcode brauchen Internet

### Was die App NICHT macht (bewusst weggelassen)
- Mahlzeit-Kategorien (Frühstück / Mittag / Abend)
- Streak / Gamification / Diamanten
- Gewichtstracking
- Health-App-Sync
- Langzeit-Statistiken
- Benutzerkonten / Login
- Kalorienverbrauch durch Sport

---

## Technischer Aufbau

### Dateien
```
Kalorien Tracker/
├── index.html      ← Komplette App (HTML + CSS + JS in einer Datei)
├── manifest.json   ← PWA-Manifest (Homescreen-Icon, Display-Modus)
├── sw.js           ← Service Worker (Offline-Caching)
├── icon.png        ← App-Icon (512×512 px, grün, generiert mit Python)
└── PROJEKT.md      ← Diese Datei
```

### Datenbank / API
- **Open Food Facts** (kostenlos, kein API-Key)
  - Suche: `https://world.openfoodfacts.org/cgi/search.pl?search_terms=...&json=1`
  - Barcode: `https://world.openfoodfacts.org/api/v2/product/{barcode}`
- **html5-qrcode** (CDN) für Barcode-Scan über iPhone-Kamera

### Datenspeicherung (localStorage)
```
kt          → Tageseinträge (7 Tage, dann automatisch gelöscht)
kt_recent   → Häufig verwendete Produkte (6 Monate, max. 50 Einträge)
```

**Struktur `kt`:**
```json
{
  "2026-05-13": [
    {
      "id": "abc123",
      "name": "Alpro Sojadrink",
      "brand": "Alpro",
      "amount": 200,
      "unit": "g",
      "kp": 27,
      "khp": 1.5,
      "ewp": 1.5,
      "fatp": 0.9,
      "kcal": 54,
      "kh": 3.0,
      "ew": 3.0,
      "fat": 1.8
    }
  ]
}
```

**Struktur `kt_recent`:**
```json
[
  {
    "name": "Alpro Sojadrink",
    "brand": "Alpro",
    "kp": 27, "khp": 1.5, "ewp": 1.5, "fatp": 0.9,
    "unit": "g",
    "useCount": 12,
    "lastUsed": "2026-05-13"
  }
]
```

---

## Hosting

- **GitHub Pages** (kostenlos, HTTPS automatisch)
- Repo: `LeipnitzKarsten/kalorien-tracker` (Public)
- Branch: `main`, Pfad: `/`
- Deployment: automatisch bei jedem `git push`

### Update einspielen
```bash
cd "/Users/karstenleipnitz/Library/Mobile Documents/com~apple~CloudDocs/Kalorien Tracker"
git add -A
git commit -m "Beschreibung der Änderung"
git push
```
→ Nach ca. 1–2 Minuten ist die neue Version live.  
→ Auf dem iPhone: Pull-to-Refresh im Safari oder einfach App neu öffnen.

### GitHub-Zugangsdaten (für Pushes)
- Username: `LeipnitzKarsten`
- Remote-URL enthält Token (wird von Claude bei Bedarf neu gesetzt)
- gh CLI liegt unter: `~/bin/gh`

---

## iPhone-Installation

1. Safari öffnen (nicht Chrome)
2. `https://leipnitzkarsten.github.io/kalorien-tracker/` aufrufen
3. Teilen-Symbol → „Zum Home-Bildschirm"
4. Grünes App-Icon erscheint auf dem Homescreen

---

## Bekannte Design-Entscheidungen

### Safari-Menüleiste (Ghost-Button)
Safari auf iPhone hat eine feste Menüleiste unten, die Fixed-Elemente überlagert. Lösung: Unter dem „Essen hinzufügen"-Button liegt ein unsichtbarer Ghost-Block (gleiche Hintergrundfarbe, `z-index` tiefer als Button), der den echten Button nach oben schiebt. Der Button hat `position: relative; z-index: 1`, damit sein Schatten über dem Ghost liegt.

### Drum-Roller
Technisch umgesetzt mit CSS `scroll-snap-type: y mandatory` und `scroll-snap-align: center` auf 500 generierten Div-Elementen (Werte 1–500). Zwei unsichtbare Platzhalter oben/unten sorgen dafür, dass Wert 1 und Wert 500 zentriert angezeigt werden können. Zwei horizontale Linien + Farbverlauf oben/unten erzeugen den iOS-typischen Look.

### Makros im Rollrad
Wert `(v-1) * 44` als `scrollTop` ergibt exakt Wert `v` in der Mitte. Synchronisation: Scrollen → Input-Feld; Tippen ins Input → Roller springt mit.

---

## Offene Ideen / mögliche Erweiterungen
- Wasser-Tracker (separate Zählung, z.B. 2,5 L Tagesziel)
- Eigene Produkte manuell anlegen (ohne Open Food Facts)
- Wochenzusammenfassung (Durchschnitt letzte 7 Tage)
- Dark Mode
