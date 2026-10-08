# Wayfinder Roadmap: Unified ChoppedPoker Static Engine Consolidation

## Destination
Consolidate all six standalone ChoppedPoker HTML tools (`index.html`, `CHECK_BET_EVs.html`, `TURNRANGECOMPOSITION.html`, `betMap.html`, `math.html`, `BluffingMath.html`) into a single, unified, standalone static HTML application with seamless scrolly-telling transitions, persistent global chapter navigation, flawless scroll and gesture hygiene, 100% mathematical fidelity, and zero browser console errors.

## Notes
- **Governing Specification:** `functional_specification_001.md`
- **Governing Taxonomy:** `LANGUAGE.md`
- **Run ID:** `20261008T232953-546-t6ji`
- **Architectural Scope:** Single static HTML document delivery (`index.html`) retaining all interactive models, visual themes, and calculations.
- **Invariants:** `INV-BOUNDARY-01`, `INV-ALIGN-01`, `INV-MAP-01`, `INV-ATOMIC-01`, `INV-FAILFAST-01`, `INV-HANDOFF-01`.

## Decisions so far
- [Ticket 001: Global Document Shell, Semantic Layout, and Namespace Isolation Architecture](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/wayfinder/20261008T232953-546-t6ji/tickets/ticket-001.md) — Unified DOM hierarchy under `#portal-nexus`, `#ev-engine`, `#range-filter`, `#strategy-heatmap`, `#equilibrium-constructor`, `#bluff-efficiency`, with modular IIFE JavaScript namespacing (`window.ChoppedPoker.*`) and scoped CSS variable domains to prevent collision.
- [Ticket 002: Scrolly-Telling Orchestration, Global Chapter Rail, and Backdrop Transitions](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/wayfinder/20261008T232953-546-t6ji/tickets/ticket-002.md) — Persistent fixed chapter navigation rail with IntersectionObserver active state tracking, smooth scroll anchors, and progressive backdrop gradient transitions with particle canvas fade-out.
- [Ticket 003: Scroll & Gesture Hygiene: Isolation of Sliders, Canvas Drag, and Sticky Sidebars](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/wayfinder/20261008T232953-546-t6ji/tickets/ticket-003.md) — Elimination of wheel event hijacking on range sliders via event suppression, isolation of 2D heatmap canvas drags with pointer capture allowing unimpeded vertical page scrolls, and bounded sticky sidebar containment.
- [Ticket 004: Mathematical Models, Baseline Values, and Precision Calibration](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/wayfinder/20261008T232953-546-t6ji/tickets/ticket-004.md) — Exact mathematical execution across all six sections, calibration of updated default baseline parameters, zero-combo edge case handling in range composition, and pure white coloring in strategy map impossible region ($c_T < N$).
- [Ticket 005: Interactive Modals, Canvas High-DPI Scaling, and Boundary Collision Safeguards](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/wayfinder/20261008T232953-546-t6ji/tickets/ticket-005.md) — Anchored 3D flip card popovers with viewport collision detection, internal card math re-rendering, card tilt effects, and Retina HiDPI canvas backing store scaling.
- [Ticket 006: Error Governance, CDN Resilience, and Acceptance Criteria Verification Framework](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/wayfinder/20261008T232953-546-t6ji/tickets/ticket-006.md) — Pinned CDN resource loading (KaTeX 0.16.9 and Google Fonts), fail-fast DOM element verification, zero console exception enforcement, and objective verification checklist for AC-01 through AC-10.

## Not yet specified
- None. All architectural facets, component schemas, data contracts, and integration requirements are fully deconstructed and resolved.

## Out of scope
- Server-side rendering, Node.js backend dependencies, or npm build pipelines (the application must remain a pure, standalone static HTML/CSS/JS file).
- Modifying underlying poker game theory models or replacing KaTeX with rasterized formula images.
- Unmanaged directory scanning or editing other run directory artifacts.
