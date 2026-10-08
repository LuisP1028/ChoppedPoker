---
ticket_id: 6
title: "Error Governance, CDN Resilience, and Acceptance Criteria Verification Framework"
type: task
status: resolved
claimed_by: "wayfinder-read-and-plan"
blocked_by: ["ticket-001.md", "ticket-002.md", "ticket-003.md", "ticket-004.md", "ticket-005.md"]
governing_specification: "functional_specification_001.md"
---

# Ticket 006: Error Governance, CDN Resilience, and Acceptance Criteria Verification Framework

## Question
How will CDN resources (Google Fonts, KaTeX 0.16.9) be reliably pinned, how will missing DOM elements or calculation faults be guarded against fail-fast, and how are all ten objective acceptance criteria (AC-01 through AC-10) systematically verified?

## Context & Specification Grounding
- **Governing Specification:** [`functional_specification_001.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/functional_specification_001.md) Sections 6.1, 7, and 8 (AC-01 through AC-10).
- **Governing Taxonomy:** [`LANGUAGE.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/LANGUAGE.md) (`{errors}`, `{correctness}`, `{functionality}`, `{correct required outputs}`, `{sufficient}`).
- **Current Defect / Conflict Analysis:**
  - If external CDNs fail to load or are referenced inconsistently across sections, KaTeX equations fail to render and fall back to broken raw text.
  - If DOM element bindings fail silently or query nonexistent IDs, event listeners fail without clear diagnostic output.
  - Verification must confirm that mathematical equivalence matches the acceptance criteria precisely down to floating point rounding.

## Architectural Decisions to Lock
1. **Pinned CDN Dependency Architecture:**
   - KaTeX 0.16.9 pinned identically to source `math.html`:
     ```html
     <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css">
     <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js"></script>
     <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js"></script>
     ```
   - Google Fonts preconnected for low-latency asset delivery:
     ```html
     <link rel="preconnect" href="https://fonts.googleapis.com">
     <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
     <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;900&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
     ```
2. **Fail-Fast DOM Initialization Invariant:**
   - In accordance with `RULE[user_global]` and `INV-FAILFAST-01`, every module must validate all required DOM nodes upon bootstrap. If an expected node is absent, throw a fatal error immediately surfacing the exact missing element ID:
     ```javascript
     function getRequiredElement(id) {
         const el = document.getElementById(id);
         if (!el) {
             throw new Error(`[ChoppedPoker Fatal Error] Required DOM element #${id} is missing from the document hierarchy.`);
         }
         return el;
     }
     ```
3. **Acceptance Criteria Verification Protocol:**
   Every verification invariant from Section 8 must be mapped to an objective validation procedure:
   - **AC-01 (Content Preservation):** DOM inspects `#portal-nexus`, `#ev-engine`, `#range-filter`, `#strategy-heatmap`, `#equilibrium-constructor`, `#bluff-efficiency`. Assert all 6 sections exist.
   - **AC-02 (Section 1 EV Math):** Set $P=100, B=75, N=100, E=55\%$. Assert MDF $= 57.1\%$, Range $= 57.1\text{ Hands}$, $EV_{\text{check}} = 55.00$, $EV_{\text{call}} = 62.50$, $\Delta EV = +7.50$, Verdict $=$ `"VALUE BET"`.
   - **AC-03 (Section 2 Range Math):** Set Value $= 70\%$, Bluff $= 40\%$. Assert Value Comp $= 53.8\%$, Air Comp $= 46.2\%$. Set Value $= 0\%$, Bluff $= 0\%$. Assert bar width is `50% / 50%` and readouts display `"-"`.
   - **AC-04 (Section 3 Heatmap):** Set $S=100, P=30, B=20, N=0.25, c_T=0.60, c_R=0.50$. Assert impossible zone is pure white for $X < 0.25$, crosshair updates EV readouts ($EV_{\text{cf}} = 115.00$, Strategy $=$ `"CALL - FOLD"`).
   - **AC-05 (Section 4 GGOP Math):** Set $P_{\text{start}}=10, S_{\text{eff}}=75, V=40$. Assert Growth $R = 16.0$, Sizing $r = 300.0\%$, River Alpha $= 23.1\%$, River Bluffs $= +9.2\%$, Turn Bluffs $= +11.4\%$.
   - **AC-06 (Section 4 Card Popover):** Click `.info-chip[data-key="ggop"]`. Assert `#popover-anchor` gains `.active`. Click card. Assert `#card-inner` gains `.flipped`. Click outside. Assert `#popover-anchor` loses `.active`.
   - **AC-07 (Section 5 Hover Zones):** Hover over `[data-target="cost"]`. Assert title displays `"The Cost of the Bluff"`. Hover over `numerator` and `denominator` sequentially without flicker or console exceptions.
   - **AC-08 (Zero Scroll-Trapping):** Mouse wheel over range sliders and `#hm-canvas`. Assert window scrolls smoothly without altering slider values or trapping pointer events.
   - **AC-09 (Persistent Chapter Rail):** Scroll through document. Assert matching `.rail-item` gains `.active`. Click `.rail-item[data-section="strategy-heatmap"]`. Assert window scrolls smoothly to `#strategy-heatmap`.
   - **AC-10 (Error Governance):** Open browser developer tools console. Assert zero uncaught exceptions, zero script loading warnings, and zero `NaN` values.

## Scope & Invariant Guardrails
- **In Scope:** CDN script/style pinning, element lookup fail-fast assertions, error handling standards, and comprehensive acceptance test catalog.
- **Out of Scope:** Code modification or implementation during the planning stage.

---

## Resolution

### 1. Verification and Fail-Fast Standards Locked
- All modules utilize strict input validation and fail-fast assertions on boot.
- Auto-render KaTeX is bound to `DOMContentLoaded` and exposed to dynamic popover injection handlers.
- Full verification matrix mapped to AC-01 through AC-10.

### 2. Status & Downstream Unblocking
- **Claimed by:** `wayfinder-read-and-plan`
- **Resolution Status:** `resolved`
- **Downstream Unblocking:** Completes the Wayfinder planning phase. Unblocks Stage 6 Documentation Sufficiency Evaluation and Stage 7 Master Matrix Generation.
