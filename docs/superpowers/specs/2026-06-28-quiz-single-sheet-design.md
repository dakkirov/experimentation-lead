# Diagnostic quiz redesign — "Single-Sheet Scorecard"

**Date:** 2026-06-28
**Status:** approved direction (mockup `_quiz-a.html`), pending spec sign-off
**Scope:** UX/layout refactor of the trustworthiness diagnostic on `index.html`. No change to scoring logic, questions, copy meaning, or result content.

## Goal
Make the quiz easier and more space-efficient. Replace the tall progressive-reveal panel (one question at a time, collapsing summaries, mixed input widgets) with one compact "lab-intake" ledger: all five questions visible at once, answered in any order, one submit reveals the score in the same card.

## What stays untouched
- The five questions, their options/values, and all client-side scoring: the `state` object, `computeScore`, `generateRisks`, `getVerdict`, the count-up animation, the verdict-tag severity classes, and the mailto "scorecard" prefill.
- The section header (eyebrow, headline "Are your tests *lying to you?*", sub, meta) and the `<form id="diag-form">` / `data-q`/`data-v` hooks the handler reads.

## Layout (desktop)
- The panel body becomes `.diag-sheet`, a 2-column grid: `grid-template-columns:1.35fr 1fr; column-gap:32px; row-gap:22px`. A 1px iron-tint left border on the right column is the ledger "spine".
- **Left column:** Q1 tool, Q3 peeking, Q5 review.
- **Right column:** Q2 volume, Q4 multiple-comparisons, then a 1px rule, then the submit pill + tally.
- Every question is a numbered label (PT Serif italic ochre `01`–`05` + Inter 600 label) over a ~44px control band, so the two columns align as a ledger. Target ~600–640px tall, submit visible without scrolling.

## Controls (one unified, square family)
All controls share the brand's square corners and a 1px iron hairline on the `rgba(30,27,22,.04)` wash (never white). Selected = oxblood fill + parchment text. Only the CTA is a 999px pill. Fix `.diag-opt`'s `border-radius:8px` → `0`.
- **Q1 tool (8):** fixed `grid-template-columns:repeat(4,1fr)` radiogroup, 4×2, no flex-wrap.
- **Q3 peek / Q4 mcc / Q5 review:** single-row segmented controls rendered as true ruled ledger cells (shared hairline between segments, square), not a pill/toggle track. Q5 = 4 segments (Analyst / PM ran it / Both / No review).
- **Q3/Q4/Q5 helper (required):** under each, an always-visible one-line helper caption (Inter, muted) restoring the clarifying clause the score depends on (e.g. Q4: "Correcting when you test many metrics, segments, or variants at once (FDR / Bonferroni). Pick 'Not sure' if this is new."). Reserve its line height permanently — one line, no wrap, no reflow.
- **Q2 volume:** keep native `<input type=range min=0 max=30>` (scoring bands untouched, default 5), slimmed, paired with a large italic-ochre live readout ("8 per month") and real −/+ stepper buttons.

## Progress + submit
- Replace "Question N of 5" with a quiet right-aligned serif-italic ochre tally ("iv of v answered"), `aria-live=polite`, non-focus-stealing. No SaaS step pip.
- Submit = full oxblood pill "Score my pipeline →", real `<button type=submit>`, disabled until tool+peek+mcc+review are set.
- Remove the progressive-reveal machinery: `q2AutoTimer`, reveal/collapse helpers, `.is-visible`/`.is-answered`/`.diag-q-collapsed` max-height rules. Render all `.diag-q` static.

## Result reveal — the load-bearing no-CLS fix
The score stays **earned** behind submit (no live/phantom score — that spoofs completion and spends the reveal that licenses the CTA). To avoid layout shift: on load, lock `.diag-panel` `min-height` to the taller of {form body, result block} (result = count-up score + verdict tag + 3 risk blocks + CTA/guarantee/mailto, ~400–550px), so the form already occupies the result's envelope. On submit add `.is-scored`: cross-fade `.diag-sheet` out, render a one-line answer recap (built from `state` via the short-label map, with an "edit" affordance to restore the form) plus the existing `#diag-result` in the reserved space; move focus to the result heading (`role=status`/`aria-live` so score + verdict announce). If a verdict's result still exceeds the reserve, allow controlled downward growth + scroll-to-result rather than clipping.

## Responsive (mobile is first-class)
At ≤720px: `.diag-sheet` → one column in order 01→05, drop the spine border; Q1 grid → `repeat(2,1fr)`; segmented controls stay full-width; every control band/tap target `min-height:44px`. Below ~400px, Q5's 4 segments become a 2×2 grid so labels never wrap mid-row. Submit + tally at the bottom. Verify total height on a real 380px viewport.

## Brand / theme
Re-skin the diagnostic off the legacy `--burgundy-2`/`--gold`/`--cream` tokens onto natural-dye: card `--panel` + paper-grain + 1.5px square hairline; inputs on the `rgba(30,27,22,.04)` wash; oxblood only for selected fill + CTA; ochre for all numbers (labels, tally, count-up score); pine for the "Solid" verdict. Reuse `.diag-verdict-tag` severity classes verbatim; score in italic PT Serif ochre. No em dashes anywhere (swap the few in existing risk strings to commas/periods).

## Accessibility
Each categorical group = `role=radiogroup` + `aria-labelledby` (its numbered label); cells = `role=radio` + `aria-checked`, roving `tabindex`, ArrowLeft/Right + Home/End, one tab stop per group. Put each Q3/Q4 option's qualifier in a per-radio `aria-describedby` (announced during arrow nav, before selection) — not only a post-selection caption. Volume keeps native range + `aria-valuemin/max/now` + the −/+ buttons; single-fire the live readout (one interaction = at most one polite announcement). Selected state conveyed by fill + a check/weight change, never color alone. On submit, focus → result heading. Honor `prefers-reduced-motion` (snap the count-up + cross-fade).

## Acceptance criteria
1. All five questions visible at once on desktop; submit reachable without scrolling (≈≤640px card).
2. Submitting yields the same score/risks/verdict as today for the same answers (logic untouched) — verified end-to-end.
3. No layout shift on selection or on the form→result swap (reserved height).
4. Keyboard-only completion works (radiogroups + arrow keys + native range + submit); SR announces labels, helper clauses, tally, and the result.
5. Clean single-column mobile at 380px with 44px targets, no horizontal overflow.
6. Fully natural-dye themed; no white inputs, no 8px radii, no em dashes, no SaaS gauge.

## Out of scope
Wizard/mobile-wizard mode, changing question content or scoring weights, the email-capture beyond the existing mailto, any blog/other-section changes.
