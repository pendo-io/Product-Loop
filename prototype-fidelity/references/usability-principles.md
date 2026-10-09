# Usability Principles

## Gestalt Principles
| Principle | Check |
|-----------|-------|
| Proximity | Related items grouped? Unrelated items separated? |
| Similarity | Same appearance = same function? Interactive elements consistent? |
| Closure | Progress/partial shapes reading correctly? |
| Continuity | Visual flows guide to next action? |
| Figure/Ground | Modals, overlays, tooltips clearly elevated? |
| Common Region | Cards, panels, grouped controls clearly bounded? |

## Laws of UX
| Law | Check |
|-----|-------|
| Fitts's Law | Primary actions at least 44px? CTAs close to where users finish reading? |
| Hick's Law | Menus overwhelming? Can choices be chunked? |
| Miller's Law | Nav menus within ~7 items? |
| Jakob's Law | Interactions consistent with platform conventions? |
| Aesthetic-Usability | Does the design look polished and intentional? |
| Von Restorff Effect | Does the primary CTA stand out from secondary actions? |

## WCAG 2.1 AA
| Criterion | Threshold |
|-----------|-----------|
| Text contrast (normal) | ≥ 4.5:1 |
| Text contrast (large 18px+) | ≥ 3:1 |
| UI component contrast | ≥ 3:1 |
| Focus indicator | Visible, ≥ 3:1 contrast |
| Touch target size | ≥ 44×44px |
| Color alone | Never the only way to convey information |
| Form inputs | Labels associated (not just placeholder) |

### Contrast quick checks
Use the quick checks in the active design system's `design-system.md`
(the neutral default's are in `design-systems/neutral/design-system.md`).
For any other color pair, compute the ratio rather than estimating it.

## Nielsen's 10 Heuristics
| # | Heuristic | Check |
|---|-----------|-------|
| 1 | Visibility of system status | Loading states, active items, progress visible? |
| 2 | Match with real world | Language matches how users think? No internal jargon? |
| 3 | User control and freedom | Undo, cancel, back available? |
| 4 | Consistency and standards | Same words/visuals/interactions throughout? |
| 5 | Error prevention | Confirmation dialogs, disabled states for invalid actions? |
| 6 | Recognition over recall | Options visible, not memorized? |
| 7 | Flexibility and efficiency | Power user shortcuts without cluttering novice path? |
| 8 | Aesthetic and minimalist | Every element earns its place? |
| 9 | Error recovery | Errors: plain language, explain problem, suggest fix? |
| 10 | Help and documentation | Complex tasks have contextual help? |
