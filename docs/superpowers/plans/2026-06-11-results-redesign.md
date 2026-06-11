# Results Screen Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the generic results screen with a SVG radar chart, Poziom 1–5 maturity cards, and a top-3 priority action list — all inside `renderResults()` in `index.html`, zero new dependencies.

**Architecture:** Single HTML file. All new code lives inside the `Survey` class's `renderResults()` method as `const` helpers. Three new CSS classes (`.radar-wrap`, `.action-card`, `.action-num`) added to the existing `<style>` block. Existing classes (`.results-grid`, `.result-card`, `.progress`, `.button-group`) reused unchanged.

**Tech Stack:** Vanilla JS, inline SVG, existing Quantica CSS tokens (`--quantica-pink`, `--fg-*`, `--bg-*`, `--border-*`, `--radius-*`, `--space-*`, `--font-*`).

---

### Task 1: Add CSS for new components

**Files:**
- Modify: `index.html` — `<style>` block, after `.recommendation-card` rules

- [ ] **Step 1: Add the three new CSS classes**

Find the `.recommendation-card.success` rule (last recommendation rule) and insert after it:

```css
        /* ── Radar chart ─────────────────────────────────────────────────────── */
        .radar-wrap {
            display: flex;
            justify-content: center;
            margin: var(--space-6) 0 var(--space-7);
        }

        .radar-wrap svg {
            overflow: visible;
        }

        /* ── Action cards ────────────────────────────────────────────────────── */
        .action-card {
            display: flex;
            gap: var(--space-5);
            align-items: flex-start;
            padding: var(--space-5);
            background: var(--bg-2);
            border-radius: var(--radius-md);
            border: 1px solid var(--border-1);
            border-left: 3px solid var(--quantica-pink);
            margin: var(--space-3) 0;
        }

        .action-num {
            font-family: var(--font-mono);
            font-size: var(--fs-caption);
            font-weight: 600;
            color: var(--fg-brand);
            min-width: 24px;
            padding-top: 2px;
            flex-shrink: 0;
        }

        .action-body { flex: 1; }

        .action-eyebrow {
            font-size: 11px;
            font-weight: 500;
            text-transform: uppercase;
            letter-spacing: var(--tracking-caps);
            color: var(--fg-brand);
            margin-bottom: var(--space-1);
        }

        .action-text {
            font-size: var(--fs-body-lg);
            line-height: var(--lh-body);
            color: var(--fg-1);
        }
```

- [ ] **Step 2: Verify by opening `index.html` locally**

Open `index.html` directly in a browser (file://). The page should load without console errors. No visual change yet on the results screen — these classes aren't used yet.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add CSS for radar chart and action card components"
```

---

### Task 2: Add `SECTION_KEY` and `ACTIONS` data

**Files:**
- Modify: `index.html` — inside `renderResults()` method of the `Survey` class

- [ ] **Step 1: Locate `renderResults()` in the file**

Find the line:
```js
            renderResults() {
```

Immediately after the opening brace, before the `const sections = ...` line, insert the two data constants below.

- [ ] **Step 2: Insert `SECTION_KEY` and `ACTIONS`**

```js
                const SECTION_KEY = {
                    'Strategia i przywództwo': 'strategia',
                    'Dane i prywatność': 'dane',
                    'Usługi miejskie i przypadki użycia': 'uslugi',
                    'Zespół i partnerstwa': 'zespol',
                    'Etyka i zaufanie mieszkańców': 'etyka'
                };

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

- [ ] **Step 3: Verify no syntax errors**

Open browser console on `index.html`. No errors should appear on load. The results screen (if reached) still shows old output — data is defined but not yet wired up.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add SECTION_KEY and ACTIONS data to renderResults()"
```

---

### Task 3: Add `getLevel()` and `renderRadar()` helpers

**Files:**
- Modify: `index.html` — inside `renderResults()`, after the `ACTIONS` const from Task 2

- [ ] **Step 1: Insert `getLevel()` immediately after the `ACTIONS` closing brace**

```js
                const getLevel = (score) => {
                    if (score <= 20) return 1;
                    if (score <= 40) return 2;
                    if (score <= 60) return 3;
                    if (score <= 80) return 4;
                    return 5;
                };
```

- [ ] **Step 2: Insert `renderRadar()` after `getLevel()`**

```js
                const renderRadar = (categoryScores) => {
                    const cx = 160, cy = 160, maxR = 110;
                    const sectionNames = Object.keys(categoryScores);
                    const n = sectionNames.length;
                    const angles = sectionNames.map((_, i) =>
                        (i * 2 * Math.PI / n) - Math.PI / 2
                    );

                    const SHORT_LABELS = {
                        'Strategia i przywództwo': 'Strategia',
                        'Dane i prywatność': 'Dane',
                        'Usługi miejskie i przypadki użycia': 'Usługi',
                        'Zespół i partnerstwa': 'Zespół',
                        'Etyka i zaufanie mieszkańców': 'Etyka'
                    };

                    // Concentric gridlines at 33%, 66%, 100%
                    const gridLines = [0.33, 0.66, 1.0].map(factor => {
                        const r = factor * maxR;
                        const pts = angles
                            .map(a => `${(cx + r * Math.cos(a)).toFixed(1)},${(cy + r * Math.sin(a)).toFixed(1)}`)
                            .join(' ');
                        return `<polygon points="${pts}" fill="none" stroke="#E6E8EC" stroke-width="1"/>`;
                    }).join('');

                    // Axis spokes
                    const axisLines = angles.map(a =>
                        `<line x1="${cx}" y1="${cy}" x2="${(cx + maxR * Math.cos(a)).toFixed(1)}" y2="${(cy + maxR * Math.sin(a)).toFixed(1)}" stroke="#E6E8EC" stroke-width="1"/>`
                    ).join('');

                    // Score polygon
                    const scorePoints = sectionNames.map((s, i) => {
                        const r = (categoryScores[s] / 100) * maxR;
                        return `${(cx + r * Math.cos(angles[i])).toFixed(1)},${(cy + r * Math.sin(angles[i])).toFixed(1)}`;
                    }).join(' ');

                    // Axis labels — anchored by horizontal direction
                    const labelR = maxR + 20;
                    const axisLabels = sectionNames.map((s, i) => {
                        const x = cx + labelR * Math.cos(angles[i]);
                        const y = cy + labelR * Math.sin(angles[i]);
                        const cosA = Math.cos(angles[i]);
                        const anchor = cosA > 0.1 ? 'start' : cosA < -0.1 ? 'end' : 'middle';
                        const label = SHORT_LABELS[s] || s.split(' ')[0];
                        return `<text x="${x.toFixed(1)}" y="${y.toFixed(1)}" text-anchor="${anchor}" dominant-baseline="middle" font-family="Satoshi,Inter,sans-serif" font-size="11" fill="#6B6B76">${label}</text>`;
                    }).join('');

                    // Overall score in centre
                    const overall = Math.round(
                        Object.values(categoryScores).reduce((a, b) => a + b, 0) / n
                    );

                    return `
                        <div class="radar-wrap">
                            <svg viewBox="0 0 320 320" width="320" height="320" aria-hidden="true">
                                ${gridLines}
                                ${axisLines}
                                <polygon points="${scorePoints}" fill="rgba(196,30,84,0.2)" stroke="#C41E54" stroke-width="2" stroke-linejoin="round"/>
                                ${axisLabels}
                                <text x="160" y="152" text-anchor="middle" font-family="Satoshi,Inter,sans-serif" font-size="28" font-weight="700" fill="#111111">${overall}%</text>
                                <text x="160" y="172" text-anchor="middle" font-family="Geist Mono,monospace" font-size="11" fill="#6B6B76">wynik ogólny</text>
                            </svg>
                        </div>
                    `;
                };
```

- [ ] **Step 3: Quick sanity check in browser console**

Open `index.html`, complete all questions, reach results. Open DevTools console and run:

```js
survey.renderResults();
```

Should return a long HTML string without throwing. (Results screen still shows old layout — `renderResults()` return value not yet updated.)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add getLevel() and renderRadar() helpers inside renderResults()"
```

---

### Task 4: Rewrite the `renderResults()` return value

**Files:**
- Modify: `index.html` — the `return \`...\`` template literal inside `renderResults()`

- [ ] **Step 1: Find the existing return statement**

Locate this line inside `renderResults()`:

```js
                return `
                    <div style="font-size: 12px; font-weight: 500; text-transform: uppercase; letter-spacing: 0.08em; color: var(--fg-brand); margin-bottom: 16px;">Wyniki</div>
                    <h1 class="title">Ocena Gotowości AI</h1>
```

The entire `return \`...\`` block (from `return \`` to the closing backtick) is replaced in the next step.

- [ ] **Step 2: Replace the full return block**

Delete everything from `return \`` to the closing `` ` `` (end of `renderResults()`'s return statement) and replace with:

```js
                // Build category scores map (preserving section order)
                const categoryScores = {};
                sections.forEach(s => { categoryScores[s] = this.calculateScore(s); });

                // Top-3 worst categories for action list
                const sorted = [...sections].sort((a, b) => categoryScores[a] - categoryScores[b]);
                const top3 = sorted.slice(0, 3);

                return `
                    <div style="font-size:11px;font-weight:500;text-transform:uppercase;letter-spacing:var(--tracking-caps);color:var(--fg-brand);margin-bottom:var(--space-3);">Wyniki oceny</div>
                    <h1 class="title">Ocena Gotowości AI<br>dla Samorządów</h1>

                    ${renderRadar(categoryScores)}

                    <h3>Wyniki szczegółowe</h3>
                    <div class="results-grid">
                        ${sections.map(section => {
                            const score = categoryScores[section];
                            const level = getLevel(score);
                            return `
                                <div class="result-card">
                                    <div style="display:flex;justify-content:space-between;align-items:baseline;margin-bottom:var(--space-2);">
                                        <span style="font-size:var(--fs-body);font-weight:500;color:var(--fg-1);">${section}</span>
                                        <span style="font-size:18px;font-weight:700;color:${this.getScoreColor(score)};">${Math.round(score)}%</span>
                                    </div>
                                    <div class="progress">
                                        <div class="progress-bar" style="width:${score}%;background:${this.getScoreColor(score)};"></div>
                                    </div>
                                    <div style="font-family:var(--font-mono);font-size:11px;color:var(--fg-3);margin-top:var(--space-2);">Poziom ${level}</div>
                                </div>
                            `;
                        }).join('')}
                    </div>

                    <h3>3 priorytety działań</h3>
                    ${top3.map((section, i) => {
                        const score = categoryScores[section];
                        const level = getLevel(score);
                        const key = SECTION_KEY[section] || 'strategia';
                        const action = ACTIONS[key][level];
                        const num = String(i + 1).padStart(2, '0');
                        return `
                            <div class="action-card">
                                <div class="action-num">${num}</div>
                                <div class="action-body">
                                    <div class="action-eyebrow">${section}</div>
                                    <div class="action-text">${action}</div>
                                </div>
                            </div>
                        `;
                    }).join('')}

                    <div class="download-buttons" style="margin-top:var(--space-7);">
                        <button class="button icon-button" onclick="survey.downloadJSON()">JSON</button>
                        <button class="button secondary icon-button" onclick="survey.downloadCSV()">CSV</button>
                    </div>
                    <div class="button-group" style="margin-top:var(--space-4);">
                        <button class="button secondary" onclick="survey.showResults=false;survey.showReview=true;survey.render()">← Przejrzyj odpowiedzi</button>
                        <button class="button" onclick="survey.reset()">Rozpocznij ponownie</button>
                    </div>
                `;
```

- [ ] **Step 3: Verify visually in browser — complete flow**

Open `index.html`. Complete all 21 questions (or answer a few and skip to results via console: `survey.showResults=true;survey.render()`).

Check all of the following:

| Check | Expected |
|-------|----------|
| Radar chart visible | Pink polygon on grey gridlines, 5 labels around edge |
| Centre score | Integer `XX%` with `wynik ogólny` below |
| Category cards | 5 cards, each showing score %, Poziom N |
| Action list | Exactly 3 cards, numbered 01/02/03 |
| Action text | Polish text matching the category's level |
| Buttons | JSON, CSV, Przejrzyj odpowiedzi, Rozpocznij ponownie all clickable |
| No JS errors | Browser console clean |

- [ ] **Step 4: Edge case — all scores equal**

In console: set all answers to max score then render:

```js
Object.keys(survey.questions).forEach(s =>
    survey.questions[s].forEach(q => { survey.answers[q.id] = q.scores[0]; })
);
survey.showResults = true;
survey.render();
```

Expected: radar polygon fills most of the chart; all categories show Poziom 4 or 5; action list picks any 3 (arbitrary when scores are tied — order matches section order).

- [ ] **Step 5: Edge case — all scores zero**

```js
survey.answers = {};
survey.showResults = true;
survey.render();
```

Expected: radar polygon collapses to a tiny shape near centre (0% radius = cx,cy points); all cards show `0% · Poziom 1`; actions show the level-1 string for each category.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Rewrite renderResults(): radar chart, Poziom 1-5 cards, top-3 action list"
```

---

### Task 5: Push, open PR, merge

**Files:** none — just git operations

- [ ] **Step 1: Push branch**

```bash
git push origin smart-city-questions
```

- [ ] **Step 2: Create and merge PR**

```bash
gh pr create \
  --title "Results screen: radar chart, maturity levels, priority actions" \
  --body "$(cat <<'EOF'
## Changes

Rewrites \`renderResults()\` in \`index.html\`:

- **SVG radar chart** — 5 axes (one per category), concentric gridlines at 33/66/100%, Quantica pink polygon scaled to scores, overall % in centre
- **Maturity cards** — each category shows score %, a progress bar, and Poziom 1–5 label
- **Top-3 action list** — bottom 3 categories ranked by score, each paired with a concrete Polish next-step from a 5×5 \`ACTIONS\` lookup
- **New CSS** — \`.radar-wrap\`, \`.action-card\`, \`.action-num\`, \`.action-body\`, \`.action-eyebrow\`, \`.action-text\`
- No new dependencies, no new files

Spec: \`docs/superpowers/specs/2026-06-11-results-redesign-design.md\`

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

gh pr merge --merge
```

---

## Self-Review

**Spec coverage check:**

| Spec requirement | Covered by |
|-----------------|------------|
| Radar SVG, 5 axes at 72° | Task 3 `renderRadar()` |
| Gridlines at 33/66/100% | Task 3 `renderRadar()` gridLines |
| Pink fill polygon + stroke | Task 3, `rgba(196,30,84,0.2)` + `#C41E54` |
| Overall score in SVG centre | Task 3, `<text>` at (160,152) and (160,172) |
| Axis short labels | Task 3 `SHORT_LABELS` map |
| Category cards with Poziom 1–5 | Task 4 `sections.map()` block |
| `getLevel()` at 20% intervals | Task 3 |
| Top-3 worst categories | Task 4 `sorted.slice(0,3)` |
| All 25 Polish action strings | Task 2 `ACTIONS` object |
| `SECTION_KEY` mapping | Task 2 |
| `.radar-wrap`, `.action-card`, `.action-num` CSS | Task 1 |
| Reuse `.results-grid`, `.result-card`, `.progress`, `.button-group` | Task 4 |
| No new files, no new dependencies | All tasks — single file only |
| Download buttons retained | Task 4 return block |

**Placeholder scan:** No TBDs, no TODOs, all 25 action strings present, all code complete.

**Type consistency:** `categoryScores` object built in Task 4 matches the `categoryScores` parameter signature expected by `renderRadar()` defined in Task 3. `getLevel()` returns `1`–`5`; `ACTIONS[key][level]` is indexed by those same integers. `SECTION_KEY` keys match the exact Polish section names in `questions.txt`.
