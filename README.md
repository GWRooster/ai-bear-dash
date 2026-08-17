# Rooster of Wall Street

401K Economic Signal Monitor — AI Bear & Macro Trigger Dashboard System

**Spec baseline: 2026-08-17.** The structure and data flow below is the confirmed, current design — check changes against this file before restructuring tabs or adding signals. Update this file in the same commit as any structural change.

## Tab structure

Six tabs, two parallel tracks feeding a shared decision layer, plus a mobile rollup:

```
dash1_ai_signals.html  ───▶  dash2_ai_heatmap.html   ───┐
  (AI Signals)                (AI Heat Map)              ├──▶ dash3_decision.html
                                                           │      (Decision Engine)
dash5_macro_signals.html ──▶  dash4_macro_heatmap.html ──┘
  (Macro Signals)              (Macro Heat Map)

                                                    dash_summary.html
                                                    (mobile rollup — far right tab)
```

- **dash1_ai_signals.html** — data entry for the 8 AI infrastructure-loop signals (SIG01–SIG08: NVDA guidance, OpenAI revenue, hyperscaler debt, FCF compression, capex/revenue, cloud deceleration, enterprise ROI, infra debt stress). Source of truth for AI-side RAG state.
- **dash2_ai_heatmap.html** — read-only heat map view of the same 8 signals. Renders whatever dash1 last computed; does not recompute thresholds itself.
- **dash5_macro_signals.html** — data entry for the 7 macro signals (M01–M07: labor, S&P technical, bond market, productivity, earnings breadth, chart structure, Treasury/Warsh). Source of truth for macro-side RAG state.
- **dash4_macro_heatmap.html** — read-only heat map view of the same 7 signals, mirrors dash2's pattern.
- **dash3_decision.html** — combines the AI composite (from dash1) and Macro composite (from dash5) into a single master decision score.
- **dash_summary.html** — condensed mobile view, sits on the far right of the nav.

## Data flow / localStorage keys

Each "Signals" page is the only place thresholds are evaluated. It writes three things to `localStorage` every time a field is locked:

| Key | Written by | Contents |
|---|---|---|
| `rooster_dash1_v1` | dash1 | raw AI field values + lock timestamps |
| `rooster_dash1_states` | dash1 | per-signal computed RAG state (SIG01–SIG08), read by dash2 |
| `rooster_dash1_composite` | dash1 | AI composite severity score (0–100), read by dash2 and dash3 |
| `rooster_dash5_v1` | dash5 | raw macro field values + lock timestamps |
| `rooster_dash5_states` | dash5 | per-signal computed RAG state (M01–M07), read by dash4 |
| `rooster_dash5_composite` | dash5 | Macro composite severity score (0–100), read by dash4 and dash3 |

**Rule: heat maps never duplicate threshold logic.** dash2 and dash4 only read `*_states` and `*_v1` — they don't independently decide what counts as red/amber/green. This is deliberate: the earlier version of this system had each page evaluate its own copy of the thresholds, which drifted apart and silently went stale. If a signal's trigger logic changes, it only needs to change in dash1 or dash5.

## Known open items as of 2026-08-17

- S&P absolute-price thresholds in dash5 (6,147 Feb ATH, 5,800 T2, 5,450 T3) are hardcoded and don't move with the market — undecided whether they should stay fixed targets or recalculate off current price/rolling ATH.
- dash3 and dash_summary have not been audited for the ID-mismatch class of bug found and fixed in dash2/dash4/dash5 on this date. dash3's own scoring (`calcAI`/`calcMacro`/`updateScores`) reads both composite keys correctly as of this date.
- dash1's SIG08 `dc-spread` field prompts for a price/percentage figure but the scoring logic reads it as a status keyword ("stress"/"fire"/"watch") — minor mismatch, not yet fixed.
