# Implementation Plan

- [ ] 1. Write bug condition exploration test
  - **Property 1: Bug Condition** - Broken or Unstable Widgets Present in README
  - **CRITICAL**: This test MUST FAIL on unfixed code — failure confirms the bug exists
  - **DO NOT attempt to fix the test or the code when it fails**
  - **NOTE**: This test encodes the expected behavior — it will validate the fix when it passes after implementation
  - **GOAL**: Surface counterexamples that demonstrate each of the six buggy widget patterns
  - **Scoped PBT Approach**: For deterministic bugs (static file content), scope the property to the six concrete failing patterns; read `README.md` and assert each condition
  - Test that `README.md` does NOT contain `demolab.com` (from Bug Condition: `X.service CONTAINS "demolab.com"`)
  - Test that `README.md` does NOT contain `streak-stats` (from Bug Condition: `X.widgetType = "streak-stats"`)
  - Test that the Tech Stack section does NOT contain `img.shields.io/badge` outside of the social badge `<div align="center">` header block (from Bug Condition: `X.widgetType = "tech-stack-badges" AND X.format = "shields-text"`)
  - Test that `README.md` contains BOTH `github-readme-stats.vercel.app/api?username=` AND `top-langs` (from Bug Condition: `X.widgetType = "stats-section" AND X.cards < 2`)
  - Test that the trophy URL includes `theme=tokyonight`, `no-frame=true`, and `no-bg=true` (from Bug Condition: `X.widgetType = "trophy" AND X.theme = UNSET`)
  - Test that the `<picture>` element does NOT contain `.gif` or `ocean` (from Bug Condition: `X.widgetType = "snake-gif-ocean"`)
  - Run test on UNFIXED code (the original README before any fixes were applied)
  - **EXPECTED OUTCOME**: Test FAILS (this is correct — it proves each of the six bugs exists in the unfixed state)
  - Document counterexamples found (e.g., `demolab.com` URL present at line N, trophy URL missing `theme=tokyonight`)
  - Mark task complete when test is written, run against unfixed code, and failures are documented
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_

- [ ] 2. Write preservation property tests (BEFORE implementing fix)
  - **Property 2: Preservation** - Non-Buggy Content Unchanged After Fix
  - **IMPORTANT**: Follow observation-first methodology
  - Observe: "About Me" section text, code block, and bullet list are present on unfixed code — record exact line ranges
  - Observe: LinkedIn, Pinterest, GitHub `shields.io` badge URLs and hrefs are present on unfixed code — record exact values
  - Observe: `komarev.com/ghpvc` profile views badge URL is present (once added it is stable content) — record exact URL
  - Observe: `<!-- RECENT-REPOS:START -->` / `<!-- RECENT-REPOS:END -->` and `<!-- WEEKLY-ACTIVITY:START -->` / `<!-- WEEKLY-ACTIVITY:END -->` comment blocks and their contents are intact
  - Observe: Snake `<picture>` element contains dark SVG source pointing to `github-contribution-grid-snake-dark.svg` and light SVG source pointing to `github-contribution-grid-snake.svg` from the `output` branch
  - Observe: `./pages.md` link in "Free Dev Resources" section is present and unchanged
  - Write property-based tests: for all sections where `isBugCondition(X)` returns `false`, assert the markdown content matches the observed baseline
  - Property-based approach: generate variations of the six buggy widgets and assert that swapping them in/out does NOT alter any of the above preserved sections
  - Run tests on UNFIXED code
  - **EXPECTED OUTCOME**: Tests PASS (confirms baseline behavior to preserve)
  - Mark task complete when tests are written, run, and passing on unfixed code
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6_

- [ ] 3. Fix for broken and unstable GitHub profile README widgets

  - [ ] 3.1 Remove demolab.com streak-stats widget and add komarev.com profile view counter
    - Locate and remove any `<img>` or `[![...]]` tag referencing `streak-stats.demolab.com`
    - Add `[![Profile Views](https://komarev.com/ghpvc/?username=SebastianMartinezLesmes&style=for-the-badge&color=6E40C9&label=PROFILE+VIEWS)](https://github.com/SebastianMartinezLesmes)` to the social badges block in `<div align="center">`
    - Verify no `demolab.com` URLs remain anywhere in the file
    - _Bug_Condition: `isBugCondition(X)` where `X.service CONTAINS "demolab.com"` OR `X.widgetType = "streak-stats"`_
    - _Expected_Behavior: widget loads reliably; `result.service = "komarev.com/ghpvc"`; `result.loads = true`_
    - _Preservation: LinkedIn, Pinterest, GitHub badge links and their hrefs must remain byte-identical_
    - _Requirements: 1.1, 1.2, 2.1, 2.2, 3.2_

  - [ ] 3.2 Replace shields.io tech-stack text badges with skillicons.dev icon rows
    - Remove all `![...](https://img.shields.io/badge/...)` tags from the `## 🛠️ Tech Stack` section
    - Add four `skillicons.dev` rows grouped by category:
      - Frontend: `[![My Skills](https://skillicons.dev/icons?i=js,ts,html,css,sass)](https://skillicons.dev)`
      - Frameworks & Libraries: `[![My Skills](https://skillicons.dev/icons?i=react,angular,ionic)](https://skillicons.dev)`
      - Backend & Data: `[![My Skills](https://skillicons.dev/icons?i=python,fastapi,mysql,mongodb)](https://skillicons.dev)`
      - Tools & Environment: `[![My Skills](https://skillicons.dev/icons?i=git,github,vscode,postman)](https://skillicons.dev)`
    - Confirm no `img.shields.io/badge` URLs remain inside the Tech Stack section
    - _Bug_Condition: `isBugCondition(X)` where `X.widgetType = "tech-stack-badges" AND X.format = "shields-text"`_
    - _Expected_Behavior: Tech Stack renders as SVG icon grid; `result.service = "skillicons.dev"`; `result.visuallyCorrect = true`_
    - _Preservation: Social badge links (LinkedIn, Pinterest, GitHub, Profile Views) and pages.md badge remain unchanged_
    - _Requirements: 1.3, 2.3, 3.2, 3.5_

  - [ ] 3.3 Add main GitHub stats card to the stats section
    - In `## 📊 GitHub Stats`, add above the existing Top Languages card:
      ```
      <img src="https://github-readme-stats.vercel.app/api?username=SebastianMartinezLesmes&show_icons=true&theme=tokyonight&hide_border=true&include_all_commits=true&count_private=true" alt="GitHub Stats" />
      ```
    - Confirm the section now contains both `api?username=` (main stats) and `api/top-langs` (languages), in that order
    - _Bug_Condition: `isBugCondition(X)` where `X.widgetType = "stats-section" AND X.cards < 2`_
    - _Expected_Behavior: `result.cards = 2`; main stats card precedes top languages card; both use `theme=tokyonight` and `hide_border=true`_
    - _Preservation: Top languages card URL and parameters remain byte-identical_
    - _Requirements: 1.4, 2.4, 3.1_

  - [ ] 3.4 Add explicit theme parameters to GitHub trophies widget
    - Locate the `github-profile-trophy.vercel.app/?username=SebastianMartinezLesmes` URL
    - Replace with: `https://github-profile-trophy.vercel.app/?username=SebastianMartinezLesmes&theme=tokyonight&no-frame=true&no-bg=true&margin-w=6&row=1`
    - Confirm the URL includes all four parameters: `theme=tokyonight`, `no-frame=true`, `no-bg=true`, `margin-w=6`, `row=1`
    - _Bug_Condition: `isBugCondition(X)` where `X.widgetType = "trophy" AND X.theme = UNSET`_
    - _Expected_Behavior: Trophies render with dark background matching tokyonight palette; `result.visuallyCorrect = true`_
    - _Preservation: Trophy section heading and surrounding markdown structure remain unchanged_
    - _Requirements: 1.5, 2.5_

  - [ ] 3.5 Remove ocean GIF snake variant from the contribution snake picture element
    - Locate the `<picture>` element in `## 🐍 Contribution Snake`
    - Remove any `<source>` or `<img>` tag referencing a `.gif` or `ocean` variant
    - Confirm the `<picture>` element contains exactly:
      - One `<source media="(prefers-color-scheme: dark)">` → `github-contribution-grid-snake-dark.svg`
      - One `<source media="(prefers-color-scheme: light)">` → `github-contribution-grid-snake.svg`
      - One `<img>` fallback → `github-contribution-grid-snake.svg`
    - _Bug_Condition: `isBugCondition(X)` where `X.widgetType = "snake-gif-ocean"`_
    - _Expected_Behavior: No `.gif` reference in snake section; page weight reduced; SVG variants preserved_
    - _Preservation: Both SVG `<source>` tags and the `<img>` fallback point to the same `output` branch raw URLs as before_
    - _Requirements: 1.6, 2.6, 3.4_

  - [ ] 3.6 Verify bug condition exploration test now passes
    - **Property 1: Expected Behavior** - All Six Buggy Widget Patterns Absent from Fixed README
    - **IMPORTANT**: Re-run the SAME test from task 1 — do NOT write a new test
    - The test from task 1 encodes the expected behavior for all six bug conditions
    - When this test passes, it confirms all six `isBugCondition(X) = true` cases are resolved
    - Run bug condition exploration test from step 1 against the fixed `README.md`
    - **EXPECTED OUTCOME**: Test PASSES (confirms all six bugs are fixed)
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6_

  - [ ] 3.7 Verify preservation tests still pass
    - **Property 2: Preservation** - All Non-Buggy Content Unchanged
    - **IMPORTANT**: Re-run the SAME tests from task 2 — do NOT write new tests
    - Run preservation property tests from step 2 against the fixed `README.md`
    - **EXPECTED OUTCOME**: Tests PASS (confirms no regressions in About Me, social links, repo table, snake SVGs, pages.md)
    - Confirm all six preservation requirements (3.1–3.6) still hold after the fix

- [ ] 4. Checkpoint — Ensure all tests pass
  - Run the full exploration test suite (Property 1) and preservation test suite (Property 2) against the final `README.md`
  - Confirm zero matches for: `demolab.com`, `streak-stats`, `img.shields.io/badge` in Tech Stack section, `.gif`/`ocean` in snake section
  - Confirm positive matches for: `komarev.com/ghpvc`, `skillicons.dev` rows (4 total), `github-readme-stats.vercel.app/api?` (main stats card), `top-langs` (languages card), `github-profile-trophy` URL with `theme=tokyonight`
  - Confirm preservation: About Me text, LinkedIn/Pinterest/GitHub badge URLs, `<!-- RECENT-REPOS:START/END -->` blocks, `<!-- WEEKLY-ACTIVITY:START/END -->` blocks, snake SVG `<source>` URLs, `./pages.md` link all unchanged
  - Ensure all tests pass; ask the user if questions arise
