# Results Screen Redesign — Design Spec

**Date:** 2026-06-11  
**Repo:** workszop/aiready  
**Scope:** `renderResults()` inside `index.html` — no new files, no new dependencies

---

## Context

The current results screen shows a large percentage score and a simple grid of category cards with progress bars. It lacks visual impact, doesn't communicate maturity level, and provides only generic recommendations. This redesign replaces it with a radar chart, EU-style maturity levels (Poziom 1–5), and a focused top-3 action list.

Polish is the primary language. All new strings are Polish only — English translation is out of scope for this spec.

---

## Screen Layout (top to bottom)

```
┌─────────────────────────────────────────────────┐
│  [eyebrow] Wyniki oceny                         │
│  [h1] Ocena Gotowości AI dla Samorządów         │
│                                                 │
│  ┌─────────────────────────────────────────┐    │
│  │          SVG Radar Chart                │    │
│  │  (5 axes, pink polygon, score centre)   │    │
│  └─────────────────────────────────────────┘    │
│                                                 │
│  [eyebrow section] Wyniki szczegółowe           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ...   │
│  │ Strategia│ │  Dane &  │ │ Usługi   │        │
│  │ Poziom 3 │ │Prywatność│ │ miejskie │        │
│  │   72%    │ │ Poziom 2 │ │ Poziom 1 │        │
│  └──────────┘ └──────────┘ └──────────┘        │
│                                                 │
│  [eyebrow section] 3 priorytety działań         │
│  ┌─────────────────────────────────────────┐    │
│  │ 01  [KATEGORIA]                         │    │
│  │     Konkretny następny krok             │    │
│  ├─────────────────────────────────────────┤    │
│  │ 02  [KATEGORIA]  ...                    │    │
│  ├─────────────────────────────────────────┤    │
│  │ 03  [KATEGORIA]  ...                    │    │
│  └─────────────────────────────────────────┘    │
│                                                 │
│  [JSON]  [CSV]          [Rozpocznij ponownie]   │
└─────────────────────────────────────────────────┘
```

---

## Component 1 — Radar Chart (SVG)

**Geometry:**
- Canvas: 320×320px inline SVG, `viewBox="0 0 320 320"`, centre at (160, 160)
- 5 axes at 72° intervals, starting from top (−90°): Strategia, Dane, Usługi, Zespół, Etyka
- Max radius: 110px (represents 100%)
- Three concentric gridlines at 33%, 66%, 100% radius — `--border-1` stroke, no fill

**Score polygon:**
- Each category score maps linearly: `r = (score / 100) * 110`
- Point coordinates: `x = cx + r * cos(angle)`, `y = cy + r * sin(angle)`
- Fill: `--quantica-pink` at 20% opacity (`rgba(196,30,84,0.2)`)
- Stroke: `--quantica-pink`, 2px, no dash

**Axis labels:**
- Short label per axis (truncated category name, ≤12 chars)
- `font-family: --font-sans`, `font-size: 11px`, `fill: --fg-3`
- Positioned 18px beyond the outer gridline along each axis direction

**Centre score:**
- Overall score (average of 5 categories, rounded to integer) rendered as `XX%`
- `font-size: 28px`, `font-weight: 700`, `fill: --fg-1`, text-anchor: middle
- Below it: `font-size: 11px`, `fill: --fg-3`, text `"wynik ogólny"`

**Axis short labels (in SVG):**

| Category | Short label |
|----------|-------------|
| Strategia i przywództwo | Strategia |
| Dane i prywatność | Dane |
| Usługi miejskie i przypadki użycia | Usługi |
| Zespół i partnerstwa | Zespół |
| Etyka i zaufanie mieszkańców | Etyka |

The mapping is derived from the first word(s) of each section key from `this.questions`.

---

## Component 2 — Maturity Scale

Five levels, no names — number only.

| Score | Poziom |
|-------|--------|
| 0–20% | Poziom 1 |
| 21–40% | Poziom 2 |
| 41–60% | Poziom 3 |
| 61–80% | Poziom 4 |
| 81–100% | Poziom 5 |

**Category card content:**
- Section name (full)
- Score percentage (right-aligned, bold, color from `getScoreColor()`)
- Thin progress bar (`--grad-magenta` fill at the score %)
- `Poziom N` label below — `font-family: --font-mono`, `font-size: 11px`, `color: --fg-3`

Cards rendered in a `results-grid` (existing CSS: `repeat(auto-fit, minmax(280px, 1fr))`).

---

## Component 3 — Top 3 Priority Actions

**Algorithm:**
1. Compute score for all 5 categories
2. Sort ascending by score
3. Take the bottom 3 (worst performers)
4. For each: look up `ACTIONS[categoryKey][level]` where `level = getLevel(score)`
5. Render as a numbered list (01, 02, 03) with category as eyebrow and action as body text

**`getLevel(score)` function:**
```js
function getLevel(score) {
    if (score <= 20) return 1;
    if (score <= 40) return 2;
    if (score <= 60) return 3;
    if (score <= 80) return 4;
    return 5;
}
```

**Category key mapping** (object keys match `Object.keys(this.questions)`):

| Section name | ACTIONS key |
|-------------|-------------|
| Strategia i przywództwo | `strategia` |
| Dane i prywatność | `dane` |
| Usługi miejskie i przypadki użycia | `uslugi` |
| Zespół i partnerstwa | `zespol` |
| Etyka i zaufanie mieszkańców | `etyka` |

The mapping is a hardcoded lookup object `SECTION_KEY` inside `renderResults()`.

**Action strings — `ACTIONS` object (25 strings):**

```js
const ACTIONS = {
    strategia: {
        1: "Powołaj zespół roboczy ds. AI i wyznacz lidera cyfryzacji odpowiedzialnego za koordynację działań.",
        2: "Opracuj formalny dokument strategii AI i przedstaw go władzom miasta do akceptacji.",
        3: "Włącz strategię AI do wieloletniego planu finansowego i powiąż z mierzalnymi wskaźnikami KPI.",
        4: "Dołącz do krajowej lub unijnej sieci smart city i aktywnie dziel się doświadczeniami.",
        5: "Opublikuj otwarte sprawozdanie z realizacji strategii AI i buduj pozycję miasta jako punktu referencyjnego."
    },
    dane: {
        1: "Przeprowadź audyt dostępnych danych miejskich i zidentyfikuj kluczowe luki w jakości i dostępności.",
        2: "Opracuj politykę zarządzania danymi i wdróż procedury ochrony danych zgodne z RODO.",
        3: "Stwórz centralny katalog danych miejskich i uruchom platformę wymiany danych między wydziałami.",
        4: "Wdróż mechanizmy zarządzania jakością danych i przygotuj zbiory do zasilania systemów AI.",
        5: "Otwórz wybrane zbiory jako open data i nawiąż współpracę z uczelniami w zakresie ich analizy."
    },
    uslugi: {
        1: "Zidentyfikuj 2–3 usługi o największym potencjale automatyzacji i opracuj wstępne analizy możliwości.",
        2: "Uruchom pierwszy projekt pilotażowy AI z mierzalnymi celami i dedykowanym budżetem.",
        3: "Opracuj mapę drogową wdrożeń AI dla kolejnych usług i określ priorytety na podstawie analizy wartości.",
        4: "Wdróż monitoring efektywności systemów AI i prowadź regularne przeglądy wyników z interesariuszami.",
        5: "Standaryzuj procesy wdrożeń i udostępnij wypracowane rozwiązania innym miastom."
    },
    zespol: {
        1: "Wyznacz osobę odpowiedzialną za projekty cyfrowe i zapewnij jej podstawowe szkolenie z zakresu AI.",
        2: "Opracuj plan rozwoju kompetencji AI dla pracowników i nawiąż kontakt z lokalną uczelnią lub hubem technologicznym.",
        3: "Sformalizuj partnerstwa z dostawcami AI i określ kryteria oceny technologii w zamówieniach publicznych.",
        4: "Zbuduj interdyscyplinarny zespół ds. innowacji łączący IT, prawo i kompetencje merytoryczne wydziałów.",
        5: "Ustanów miejskie centrum kompetencji AI i uczestnictw aktywnie w europejskich sieciach wymiany wiedzy."
    },
    etyka: {
        1: "Zinwentaryzuj systemy AI stosowane w mieście i oceń ich wpływ na prawa mieszkańców.",
        2: "Opracuj politykę przejrzystości AI i poinformuj mieszkańców o sposobach wykorzystania ich danych.",
        3: "Wdróż procedury audytu algorytmicznego dla systemów podejmujących decyzje dotyczące mieszkańców.",
        4: "Sklasyfikuj systemy AI zgodnie z unijnym Aktem o AI i dostosuj procesy do wymogów wysokiego ryzyka.",
        5: "Powołaj radę ds. etyki AI z udziałem mieszkańców jako stały organ doradczy miasta."
    }
};
```

**Action card visual:**
- Left border `3px --quantica-pink`
- Monospace number `01` / `02` / `03` in `--fg-brand`
- Category name as eyebrow (`text-transform: uppercase`, `--fg-brand`)
- Action text in `--fg-1`, `font-size: --fs-body-lg`
- Background `--bg-2`, `border-radius: --radius-md`

---

## Implementation Notes

**All changes confined to `renderResults()` inside `index.html`.** No new files. No new script tags.

Two helper functions added as `const` inside `renderResults()`:
- `getLevel(score)` — maps 0–100 to 1–5
- `renderRadar(scores)` — receives `{ categoryName: score }` object, returns SVG string

The `ACTIONS` and `SECTION_KEY` objects are also declared as `const` inside `renderResults()`.

**Existing CSS classes reused:** `.results-grid`, `.result-card`, `.progress`, `.progress-bar`, `.button-group`, `.button`, `.download-buttons`

**New CSS classes needed** (to be added to the `<style>` block):
- `.radar-wrap` — centers the SVG, adds `margin-bottom: --space-7`
- `.action-card` — action item card (left border, padding, flex layout)
- `.action-num` — monospace `01`/`02`/`03` label

---

## Out of Scope

- English translations of new strings
- Email results
- Presentation mode
- Print/PDF layout
- Saving results history
