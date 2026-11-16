# GitHub Profile README Bugfix Design

## Overview

The GitHub profile README for `SebastianMartinezLesmes` references six widget configurations that are either broken, visually outdated, or structurally incomplete. This design formalizes each defect as a component of the Bug Condition `C(X)`, defines the expected correct behavior `P(result)`, documents preservation requirements, and provides the exact replacement services with their parameters.

The fix strategy is **surgical replacement**: only the six identified widgets are touched. All prose, structure, links, and the contribution snake workflow remain unchanged.

---

## Glossary

- **Bug_Condition (C)**: A function that returns `true` for any README widget instance that is broken, unstable, visually inconsistent, or missing. Defined formally in the Bug Details section.
- **Property (P)**: The desired rendered state for any widget where `C(X)` is true — it must load reliably, render visually correctly, and use a stable service endpoint.
- **Preservation**: All README content where `C(X)` is false must render identically before and after the fix.
- **ReadmeWidget**: A logical unit of the README that depends on an external service URL (image src, badge href, etc.).
- **demolab.com**: Third-party hosting provider for `streak-stats` and `readme-typing-svg` services known for frequent downtime and rate limiting.
- **skillicons.dev**: A stable CDN-backed service providing SVG skill icons via a query-string list of technology slugs.
- **github-readme-stats.vercel.app**: The canonical Vercel deployment of the `anuraghazra/github-readme-stats` project, used for stats cards and top-language cards.
- **komarev.com/ghpvc**: A widely-used, stable profile view counter service with badge-style output.
- **tokyonight**: The chosen visual theme — dark background, purple/blue/teal accent palette — applied consistently across all stats widgets.

---

## Bug Details

### Bug Condition

The bug manifests when any of six specific widget types are present in the README. Each maps to a distinct failure mode: service unavailability, missing content, visual regression, or unnecessary page weight.

**Formal Specification:**

```
FUNCTION isBugCondition(X)
  INPUT: X of type ReadmeWidget
  OUTPUT: boolean

  RETURN (X.service CONTAINS "demolab.com")
      OR (X.widgetType = "streak-stats")
      OR (X.widgetType = "tech-stack-badges" AND X.format = "shields-text")
      OR (X.widgetType = "stats-section" AND X.cards < 2)
      OR (X.widgetType = "trophy" AND X.explicitTheme = UNSET)
      OR (X.widgetType = "snake-ocean-gif")
END FUNCTION
```

### Examples

| Widget | Current (Buggy) | Expected (Fixed) |
|--------|-----------------|------------------|
| Streak stats | `https://streak-stats.demolab.com/?user=SebastianMartinezLesmes` — renders broken image or 503 | Profile view counter from `komarev.com/ghpvc` — loads reliably |
| Typing SVG | `https://readme-typing-svg.demolab.com?...` — intermittent downtime | Static `<h1>`/`<h3>` heading in HTML — no external dependency |
| Tech stack | `![Python](https://img.shields.io/badge/python-3670A0?...)` text badges × N | `[![My Skills](https://skillicons.dev/icons?i=js,ts,...)]` icon rows |
| Stats section | Only Top Languages card visible | Main stats card + Top Languages card both present |
| Trophies | `github-profile-trophy.vercel.app/?username=...` — no theme set, inconsistent render | Same URL + `&theme=tokyonight&no-frame=true&no-bg=true&margin-w=6&row=1` |
| Snake animation | Three `<source>` / `<img>` tags including ocean GIF variant | Two `<source>` tags (dark SVG + light SVG) + one fallback `<img>` |

---

## Expected Behavior

### Preservation Requirements

The following behaviors must remain **byte-for-byte structurally identical** (same markdown, same URLs) after the fix:

**Unchanged Behaviors:**
- The "About Me" section — personal bio, code block with current focus, and bulleted project summary
- All social/professional badge links (LinkedIn, Pinterest, GitHub)
- The "Recent Repositories" table managed by the GitHub Actions workflow (the `<!-- RECENT-REPOS:START -->` / `<!-- RECENT-REPOS:END -->` and `<!-- WEEKLY-ACTIVITY:START -->` / `<!-- WEEKLY-ACTIVITY:END -->` comment blocks and their contents must not be modified)
- The contribution snake `<picture>` element — already uses the correct dark/light SVG sources; only the ocean GIF variant (if present) is removed
- The "Free Dev Resources" section and its link to `./pages.md`
- The overall section order and `---` horizontal rule dividers
- The English language of all prose

**Scope:**
All `ReadmeWidget` instances where `isBugCondition(X)` returns `false` — including the LinkedIn/Pinterest/GitHub badges, the `komarev.com` profile view counter badge (once added, it is a stable widget and must be preserved by any future fix), the snake SVG sources, and the repo table — must be completely unaffected by this fix.

---

## Hypothesized Root Cause

Each of the six defects has a distinct root cause:

1. **Service instability (demolab.com)**: Both `streak-stats.demolab.com` and `readme-typing-svg.demolab.com` are community-hosted services with no SLA. They experience rate limiting and downtime that GitHub's CDN cache cannot always absorb, resulting in broken image badges.

2. **Missing stats card**: The README was authored with only the Top Languages card. The main stats card (`/api?username=...`) was never added, leaving the stats section structurally incomplete.

3. **Visual inconsistency in tech stack**: The original tech stack used `shields.io` text badges. These render as small pill-shaped text labels — functional but visually inconsistent with modern profile READMEs that use icon grids. No rendering error occurs, but the UX is inferior.

4. **Trophy widget missing explicit theming**: The `github-profile-trophy.vercel.app` service defaults to a light theme. Without `theme=tokyonight` explicitly set, the widget renders with a white background that clashes with the dark theme of the rest of the stats section.

5. **Ocean GIF snake variant**: A heavy animated GIF was included alongside SVG variants. Since the `<picture>` element's `<source>` tags correctly serve the SVG for dark/light preference, the GIF is never displayed in supported browsers but still increases page weight if fetched as the default `<img>` fallback.

---

## Correctness Properties

Property 1: Bug Condition — Broken Widgets Replaced with Stable Alternatives

_For any_ `ReadmeWidget` instance `X` where `isBugCondition(X)` returns `true`, the fixed README SHALL render that widget using a stable, reliably available service with correct visual parameters, such that the widget loads without error on a standard GitHub profile page visit.

**Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5, 2.6**

Property 2: Preservation — Non-Buggy Content Unchanged

_For any_ `ReadmeWidget` instance `X` where `isBugCondition(X)` returns `false`, the fixed README SHALL contain exactly the same markdown as the original README for that widget, preserving all prose, social links, repository tables, snake animations, and resource links.

**Validates: Requirements 3.1, 3.2, 3.3, 3.4, 3.5, 3.6**

---

## Fix Implementation

### Design Decisions

| Decision | Chosen Approach | Alternatives Considered | Rationale |
|----------|-----------------|------------------------|-----------|
| Replace streak-stats | `komarev.com/ghpvc` profile view counter | Keep streak-stats with a different host; remove entirely | `komarev.com` is one of the most stable and widely-used badge services; profile view count is a meaningful replacement metric |
| Replace typing SVG | Static `<h1>`/`<h3>` header in HTML | `typing-svg.skywillow.vercel.app`; `capsule-render.vercel.app` | Static HTML has zero external dependencies and zero failure modes; visual impact from centered name + role subtitle is equivalent |
| Tech stack icons | `skillicons.dev` icon rows grouped by category | `devicons`, `simple-icons` via shields.io, `badges.pages.dev` | `skillicons.dev` is CDN-backed, supports the exact tech stack needed, and uses a clean uniform icon style |
| Stats section | Add main stats card above existing Top Languages card | Use a combined layout | Two separate cards allow independent loading; main stats card placed first for visual hierarchy |
| Trophy theme | Add `theme=tokyonight&no-frame=true&no-bg=true` | Wrap in dark-mode `<picture>` source | Explicit parameters are the simplest, most portable fix |
| Snake variants | Remove ocean GIF, keep dark + light SVG | Keep all three | SVG is already served correctly by `<picture>`; GIF adds weight with no visual benefit |

### Visual Theme Specification

All stats widgets use a consistent parameter set:

```
theme=tokyonight
hide_border=true
```

The `tokyonight` theme uses:
- Background: `#1a1b27`
- Text: `#a9b1d6`
- Accent/icons: `#7aa2f7` (blue), `#bb9af7` (purple), `#73daca` (teal)

`skillicons.dev` icons use their native brand colors on a transparent background, which renders correctly on both GitHub dark and light modes.

### README Section Structure (After Fix)

```
<div align="center">
  [Static header: # Hi, I'm Sebastián Martínez 👋]
  [Subtitle: ### Junior Web Developer · Angular / Ionic / React · Python Automation]
  [Social badges: LinkedIn | Pinterest | GitHub | Profile Views counter]
</div>

---
## 💫 About Me
[Unchanged]

---
## 🛠️ Tech Stack
### Frontend        → skillicons.dev: js,ts,html,css,sass
### Frameworks      → skillicons.dev: react,angular,ionic
### Backend & Data  → skillicons.dev: python,fastapi,mysql,mongodb
### Tools           → skillicons.dev: git,github,vscode,postman

---
## 📊 GitHub Stats
[Main stats card: github-readme-stats.vercel.app/api?username=...]
[Top languages:   github-readme-stats.vercel.app/api/top-langs?username=...]

---
## 🏆 GitHub Trophies
[github-profile-trophy.vercel.app?...&theme=tokyonight&no-frame=true&no-bg=true]

---
## 🐍 Contribution Snake
[<picture> with dark SVG + light SVG sources only]

---
## 📦 Recent Repositories
[Unchanged — managed by GitHub Actions]

---
## 📄 Free Dev Resources
[Unchanged]
```

### Changes Required

**File:** `README.md`

**Change 1 — Remove demolab typing SVG, replace with static header:**

Remove:
```html
<img src="https://readme-typing-svg.demolab.com?font=..." .../>
```
Replace with:
```html
# Hi, I'm Sebastián Martínez 👋

### Junior Web Developer · Angular / Ionic / React · Python Automation
```
(Already present in the current README — verify demolab tag is absent.)

**Change 2 — Replace streak stats with profile view counter:**

Remove any `streak-stats.demolab.com` `<img>` tag.
Add to the social badges block:
```markdown
[![Profile Views](https://komarev.com/ghpvc/?username=SebastianMartinezLesmes&style=for-the-badge&color=6E40C9&label=PROFILE+VIEWS)](https://github.com/SebastianMartinezLesmes)
```
(Already present in the current README — verify streak-stats tag is absent.)

**Change 3 — Replace shields.io text badges with skillicons.dev icon rows:**

Remove all `shields.io` badge `<img>` tags from the Tech Stack section.
Replace with four `skillicons.dev` rows:
```markdown
[![My Skills](https://skillicons.dev/icons?i=js,ts,html,css,sass)](https://skillicons.dev)
[![My Skills](https://skillicons.dev/icons?i=react,angular,ionic)](https://skillicons.dev)
[![My Skills](https://skillicons.dev/icons?i=python,fastapi,mysql,mongodb)](https://skillicons.dev)
[![My Skills](https://skillicons.dev/icons?i=git,github,vscode,postman)](https://skillicons.dev)
```
(Already present in the current README — verify no shields.io tech badges remain.)

**Change 4 — Add main stats card:**

In the `## 📊 GitHub Stats` section, add above the Top Languages card:
```markdown
<img src="https://github-readme-stats.vercel.app/api?username=SebastianMartinezLesmes&show_icons=true&theme=tokyonight&hide_border=true&include_all_commits=true&count_private=true" alt="GitHub Stats" />
```
(Already present in the current README — verify it precedes the top-langs card.)

**Change 5 — Add explicit theme parameters to trophies:**

Replace:
```markdown
![Trophies](https://github-profile-trophy.vercel.app/?username=SebastianMartinezLesmes)
```
With:
```markdown
![Trophies](https://github-profile-trophy.vercel.app/?username=SebastianMartinezLesmes&theme=tokyonight&no-frame=true&no-bg=true&margin-w=6&row=1)
```
(Already present in the current README — verify all parameters are present.)

**Change 6 — Remove ocean GIF snake variant:**

The `<picture>` element must contain exactly:
- One `<source>` for `prefers-color-scheme: dark` → `github-contribution-grid-snake-dark.svg`
- One `<source>` for `prefers-color-scheme: light` → `github-contribution-grid-snake.svg`
- One `<img>` fallback → `github-contribution-grid-snake.svg`

Any `<source>` or `<img>` referencing a `.gif` ocean variant must be removed.
(Already absent in the current README — verify no GIF reference exists.)

---

## Testing Strategy

### Validation Approach

The testing strategy follows a two-phase approach: first verify the current README already satisfies the requirements (since it appears partially or fully fixed), then confirm preservation of all unchanged sections.

### Exploratory Bug Condition Checking

**Goal:** Surface any remaining `isBugCondition(X) = true` instances in the live README before declaring the fix complete. Confirm or refute whether each of the six defects has been resolved.

**Test Plan:** Perform a structured read of `README.md`, searching for each buggy pattern. Run on the CURRENT file to detect any remaining defects.

**Test Cases:**

1. **demolab.com Scan** — grep for `demolab.com` in `README.md`; expect zero matches (will fail if any demolab URL remains)
2. **streak-stats Scan** — grep for `streak-stats` in `README.md`; expect zero matches
3. **shields.io Tech Badge Scan** — grep for `img.shields.io/badge` in the Tech Stack section; expect zero matches (skillicons.dev rows should be the only tech stack images)
4. **Stats Card Count** — count `github-readme-stats.vercel.app/api?` occurrences; expect exactly 1 (main stats card) plus 1 `top-langs` = 2 total
5. **Trophy Parameters** — search for `github-profile-trophy`; expect the URL to include `theme=tokyonight`, `no-frame=true`, `no-bg=true`
6. **Ocean GIF Scan** — grep for `.gif` or `ocean` in the snake section; expect zero matches

**Expected Counterexamples (if any defect remains):**
- A demolab URL still present in the header or elsewhere
- `shields.io` badge URLs in the tech stack section
- Trophy URL missing theme or frame parameters
- Ocean GIF `<source>` tag still present

### Fix Checking

**Goal:** Verify that for all widgets where `isBugCondition(X)` was `true`, the fixed README now satisfies `P(result)`.

**Pseudocode:**
```
FOR ALL X WHERE isBugCondition(X) DO
  result := renderWidget_fixed(X)
  ASSERT result.service != "demolab.com"
       AND result.loads = true
       AND result.visuallyCorrect = true
       AND result.usesStableEndpoint = true
END FOR
```

### Preservation Checking

**Goal:** Verify that all content where `isBugCondition(X)` is `false` is identical between original and fixed README.

**Pseudocode:**
```
FOR ALL X WHERE NOT isBugCondition(X) DO
  ASSERT renderWidget_original(X) = renderWidget_fixed(X)
END FOR
```

**Testing Approach:** Property-based testing is recommended for preservation checking because it systematically covers the full set of unchanged sections rather than relying on manually chosen spot checks.

**Test Cases:**

1. **About Me Preservation** — verify the bio text, code block, and bullet list are byte-identical to the original
2. **Social Links Preservation** — verify all three badge links (LinkedIn, Pinterest, GitHub) and their hrefs are unchanged
3. **Profile Views Badge Preservation** — once added, verify the `komarev.com/ghpvc` badge URL and parameters are unchanged
4. **Repo Table Preservation** — verify the `<!-- RECENT-REPOS:START/END -->` and `<!-- WEEKLY-ACTIVITY:START/END -->` comment blocks and their contents are intact
5. **Snake SVG Preservation** — verify the dark and light SVG `<source>` tags point to the correct raw GitHub URLs
6. **Free Dev Resources Preservation** — verify the `./pages.md` link is unchanged

### Unit Tests

- Scan README for each of the six buggy patterns (`demolab.com`, `streak-stats`, `shields.io` tech badges, missing stats card, trophy without theme, ocean GIF)
- Verify each replacement URL includes all required parameters
- Verify the `<picture>` element has exactly 2 `<source>` tags and 1 `<img>` tag

### Property-Based Tests

- Generate variations of the README with random permutations of the unchanged sections and assert they are unaffected by the fix logic
- For any widget not matching `isBugCondition`, assert its URL, alt text, and surrounding markdown are preserved across a generated set of README states

### Integration Tests

- Load the raw GitHub README URL and visually inspect each widget renders (manual verification against live GitHub profile)
- Verify the contribution snake SVGs are served from the `output` branch of the repository
- Verify the `komarev.com` counter increments on repeated profile visits
- Verify the trophies render with a dark background consistent with the `tokyonight` theme
