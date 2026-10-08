---
ticket_id: 1
title: "Global Document Shell, Semantic Layout, and Namespace Isolation Architecture"
type: task
status: resolved
claimed_by: "wayfinder-read-and-plan"
blocked_by: []
governing_specification: "functional_specification_001.md"
---

# Ticket 001: Global Document Shell, Semantic Layout, and Namespace Isolation Architecture

## Question
How must the single static HTML document shell (`index.html`) be structured to host all six poker tool engines simultaneously without DOM ID collisions, global JavaScript variable pollution, or conflicting CSS stylesheet overrides?

## Context & Specification Grounding
- **Governing Specification:** [`functional_specification_001.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/functional_specification_001.md) Sections 1, 3, 4, 6.1, 6.2, and 7.5.
- **Affected Source Files:**
  - [`index.html`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/index.html) (Portal Nexus & Particle Engine)
  - [`CHECK_BET_EVs.html`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/CHECK_BET_EVs.html) (Check vs Bet EV Engine)
  - [`TURNRANGECOMPOSITION.html`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/TURNRANGECOMPOSITION.html) (Turn Range Composition Visualizer)
  - [`betMap.html`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/betMap.html) (PvBC 2D Strategy Heatmap)
  - [`math.html`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/math.html) (Equilibrium Constructor & GGOP Engine)
  - [`BluffingMath.html`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/BluffingMath.html) (Bluffing Efficiency Math Breakdown)
- **Current Defect / Conflict Analysis:**
  - Multiple standalone source files declare identical global variable names: `canvas`, `ctx`, `update`, `calculate`, `suits`, `cards`, `inputs`, `displays`, `state`, `data`, and `popover`. Unifying them verbatim into a single document causes fatal identifier collisions and overwritten function bindings.
  - Four source files (`CHECK_BET_EVs.html`, `TURNRANGECOMPOSITION.html`, `betMap.html`, `BluffingMath.html`) specify `body { height: 100vh; overflow: hidden; }`. Merging these directly prevents vertical scrolling across the consolidated application.
  - Global `:root` CSS custom properties conflict: `--bg`, `--accent`, `--border`, and typography rules collide across the brutalist, blueprint, Swiss, and glassmorphism styles.

## Architectural Decisions to Lock
1. **Semantic HTML Section Anchor Contract:**
   The consolidated document must wrap each tool within a top-level semantic `<section>` element using deterministic IDs mandated by Section 6.2:
   - `<section id="portal-nexus" class="tool-section section-portal">`: Central Portal & Navigation Nexus.
   - `<section id="ev-engine" class="tool-section section-ev">`: Check vs Bet EV Decision Engine.
   - `<section id="range-filter" class="tool-section section-turn">`: Turn Range Composition Visualizer.
   - `<section id="strategy-heatmap" class="tool-section section-heatmap">`: PvBC 2D Strategy Heatmap.
   - `<section id="equilibrium-constructor" class="tool-section section-equilibrium">`: Equilibrium Constructor & GGOP Engine.
   - `<section id="bluff-efficiency" class="tool-section section-bluff">`: Bluffing Efficiency Math Breakdown.
2. **Global Document Flow and Scoped CSS Architecture:**
   - The global `body` rule must enforce continuous vertical progression:
     ```css
     html { scroll-behavior: smooth; }
     body {
         margin: 0;
         padding: 0;
         min-height: 100vh;
         overflow-x: hidden;
         overflow-y: auto;
         background-color: #050505;
         color: #ffffff;
         font-family: 'Inter', sans-serif;
     }
     ```
   - Each section defines its own scoped class namespace (e.g. `.section-ev`, `.section-turn`, `.section-heatmap`, `.section-equilibrium`, `.section-bluff`) to isolate local CSS custom properties, fonts, borders, and margins without polluting `:root`.
3. **Deterministic DOM Identifier Namespacing:**
   To guarantee unambiguous element lookups, all interactive inputs and dynamic display spans receive section-scoped unique IDs:
   - Section 1: `ev-pot-input`, `ev-bet-input`, `ev-combo-input`, `ev-equity-input`, `ev-pot-val`, `ev-bet-val`, `ev-combo-val`, `ev-equity-val`, `ev-mdf-result`, `ev-range-result`, `ev-check-result`, `ev-call-result`, `ev-diff-result`, `ev-verdict`.
   - Section 2: `turn-val-slider`, `turn-bluff-slider`, `turn-val-disp`, `turn-bluff-disp`, `turn-bar-val`, `turn-bar-air`, `turn-stat-val-pct`, `turn-stat-air-pct`.
   - Section 3: `hm-input-s`, `hm-num-s`, `hm-input-p`, `hm-num-p`, `hm-input-b`, `hm-num-b`, `hm-input-n`, `hm-val-n`, `hm-input-ct`, `hm-val-ct`, `hm-input-cr`, `hm-val-cr`, `hm-canvas`, `hm-strategy-name`, `hm-strategy-ev`.
   - Section 4: `eq-in-pot`, `eq-in-stack`, `eq-in-freq`, `eq-out-bet`, `eq-out-alpha`, `eq-out-rbluff`, `eq-out-tbluff`, `eq-popover-anchor`, `eq-card-inner`, `eq-info-title`, `eq-info-body`.
   - Section 5: `bluff-exp-title`, `bluff-exp-body`, `bluff-zone-cost`, `bluff-zone-num`, `bluff-zone-den`.
4. **JavaScript Modular Namespace Pattern:**
   All execution logic is housed under a unified namespace `window.ChoppedPoker` and structured as self-executing modular closures:
   ```javascript
   window.ChoppedPoker = {
       portal: (function() { ... })(),
       evEngine: (function() { ... })(),
       rangeFilter: (function() { ... })(),
       strategyHeatmap: (function() { ... })(),
       equilibrium: (function() { ... })(),
       bluffEfficiency: (function() { ... })()
   };
   ```

## Scope & Invariant Guardrails
- **In Scope:** Defining the global HTML hierarchy, section boundaries, unique element ID catalog, CSS encapsulation strategy, and JavaScript namespace architecture.
- **Out of Scope:** Implementation of specific mathematical formulas (handled in Ticket 004), popover flip logic (handled in Ticket 005), and scroll event listeners (handled in Tickets 002 and 003).

---

## Resolution

### 1. Unified Static Architecture Specification
- **Single File Contract:** The primary deliverable is `index.html`. It incorporates all required `<head>` metadata, preconnected fonts (`Inter`, `JetBrains Mono`), KaTeX stylesheets, inline CSS blocks partitioned by section class, semantic `<section>` containers for Sections 0 through 5, and modular `<script>` blocks executed on `DOMContentLoaded`.
- **Global Reset & Section Dimensions:**
  - Top-level sections `.tool-section` have `min-height: 100vh; position: relative; display: flex; align-items: center; justify-content: center; padding: 80px 20px; box-sizing: border-box;`.
  - Sections maintain localized background colors and aesthetics while outer wrappers prevent layout shifts or horizontal scrollbars (`overflow-x: clip;`).
- **Namespace Contracts:**
  - Every module exposes an `init()` method and an explicit `update()` / `calculate()` method.
  - On page load, `window.addEventListener('DOMContentLoaded', () => { ... })` invokes each module's `init()` cleanly and deterministically.

### 2. Status & Downstream Unblocking
- **Claimed by:** `wayfinder-read-and-plan`
- **Resolution Status:** `resolved`
- **Downstream Unblocking:** Unblocks Ticket 002 (Scrolly-Telling Orchestration), Ticket 003 (Scroll Hygiene), Ticket 004 (Mathematical Models), Ticket 005 (Interactive Modals), and Ticket 006 (Verification Framework).
