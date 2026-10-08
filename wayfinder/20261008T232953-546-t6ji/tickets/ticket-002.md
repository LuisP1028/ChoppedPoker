---
ticket_id: 2
title: "Scrolly-Telling Orchestration, Global Chapter Rail, and Backdrop Transitions"
type: task
status: resolved
claimed_by: "wayfinder-read-and-plan"
blocked_by: ["ticket-001.md"]
governing_specification: "functional_specification_001.md"
---

# Ticket 002: Scrolly-Telling Orchestration, Global Chapter Rail, and Backdrop Transitions

## Question
How will the global persistent navigation rail track section visibility, how are smooth scrolling jumps handled, and how are visual backdrop transitions and canvas particle fade-outs smoothly coordinated during vertical scrolling?

## Context & Specification Grounding
- **Governing Specification:** [`functional_specification_001.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/functional_specification_001.md) Sections 3, 5.1, 5.3, 8 (AC-08, AC-09).
- **Target Sections:**
  - `#portal-nexus`: Overview / Directory
  - `#ev-engine`: Section 1 (EV Engine)
  - `#range-filter`: Section 2 (Turn Range)
  - `#strategy-heatmap`: Section 3 (PvBC Heatmap)
  - `#equilibrium-constructor`: Section 4 (Equilibrium)
  - `#bluff-efficiency`: Section 5 (Bluff Efficiency)
- **Current Defect / Conflict Analysis:**
  - Individual files had no global navigation. Moving between tools required browser back/forward or external page links.
  - The ambient particle canvas in `index.html` runs continuously in an unbounded `requestAnimationFrame` loop. If left running at full opacity throughout all sections, it causes visual clutter over white/editorial sections (Sections 3 and 4) and induces unnecessary GPU memory overhead during canvas shader re-renders.

## Architectural Decisions to Lock
1. **Persistent Global Chapter Rail Component:**
   - Position: Fixed to the right viewport edge (or top-right header on mobile screens), styled with minimal glassmorphism (`backdrop-filter: blur(8px); background: rgba(0, 0, 0, 0.4); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 24px; padding: 12px 18px; z-index: 1000;`).
   - Chapter items register explicit data targets:
     ```html
     <nav id="global-chapter-rail" class="chapter-rail" aria-label="Tool Navigation">
         <div class="rail-progress-line"></div>
         <a href="#portal-nexus" class="rail-item active" data-section="portal-nexus">
             <span class="rail-dot"></span>
             <span class="rail-label">Overview</span>
         </a>
         <a href="#ev-engine" class="rail-item" data-section="ev-engine">
             <span class="rail-dot"></span>
             <span class="rail-label">EV Engine</span>
         </a>
         <a href="#range-filter" class="rail-item" data-section="range-filter">
             <span class="rail-dot"></span>
             <span class="rail-label">Turn Range</span>
         </a>
         <a href="#strategy-heatmap" class="rail-item" data-section="strategy-heatmap">
             <span class="rail-dot"></span>
             <span class="rail-label">PvBC Heatmap</span>
         </a>
         <a href="#equilibrium-constructor" class="rail-item" data-section="equilibrium-constructor">
             <span class="rail-dot"></span>
             <span class="rail-label">Equilibrium</span>
         </a>
         <a href="#bluff-efficiency" class="rail-item" data-section="bluff-efficiency">
             <span class="rail-dot"></span>
             <span class="rail-label">Bluff Efficiency</span>
         </a>
     </nav>
     ```
2. **IntersectionObserver Viewport Synchronization:**
   - To avoid expensive `scroll` event polling, an `IntersectionObserver` with threshold options `[0.2, 0.5, 0.8]` observes all six sections.
   - When an intersection ratio exceeds 0.5 (or is the dominant section in view), the active state `.active` is added to the matching `.rail-item` and removed from siblings.
   - Rail item click handlers invoke `element.scrollIntoView({ behavior: 'smooth' })` and update history state using `history.pushState(null, null, '#' + sectionId)` without triggering browser page reloads.
3. **Particle Canvas Performance Lifecycle & Fade-Out:**
   - The particle canvas (`#particle-canvas`) is set to `position: fixed; pointer-events: none; z-index: 1;`.
   - As the user scrolls from `#portal-nexus` into `#ev-engine`, the particle canvas opacity smoothly interpolates from `0.4` down to `0.0`.
   - When `#portal-nexus` is completely out of view (`intersectionRatio === 0`), the animation loop is paused (`cancelAnimationFrame`) to conserve GPU and battery life. When `#portal-nexus` re-enters view, the animation loop automatically resumes.
4. **Harmonious Visual Transition Dividers:**
   - Each section transition uses subtle gradient blend masks (`background: linear-gradient(180deg, var(--section-prev-bg) 0%, var(--section-curr-bg) 100%)`) or frosted glass divider bars at section borders to transition between contrasting aesthetics:
     - Section 0 (#050505) -> Section 1 (#111111 dark brutalism)
     - Section 1 (#111111) -> Section 2 (#121212 blueprint)
     - Section 2 (#121212) -> Section 3 (#ffffff Swiss light)
     - Section 3 (#ffffff) -> Section 4 (#eaeaea editorial paper)
     - Section 4 (#eaeaea) -> Section 5 (#0f0c29 deep indigo)

## Scope & Invariant Guardrails
- **In Scope:** DOM structure and styling of the global navigation rail, IntersectionObserver synchronization logic, smooth scrolling behaviors, particle canvas lifecycle management, and section backdrop transition styling.
- **Out of Scope:** Slider event suppression (handled in Ticket 003), calculation updates (handled in Ticket 004), and popover modal z-index management (handled in Ticket 005).

---

## Resolution

### 1. Scrolly-Telling & Rail Implementation Contract
- **IntersectionObserver Setup:**
  ```javascript
  const sections = document.querySelectorAll('.tool-section');
  const railItems = document.querySelectorAll('.rail-item');
  const particleCanvas = document.getElementById('particle-canvas');

  const observerOptions = {
      root: null,
      rootMargin: '-20% 0px -20% 0px',
      threshold: [0.1, 0.5]
  };

  const sectionObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
          if (entry.isIntersecting && entry.intersectionRatio >= 0.5) {
              const currentId = entry.target.id;
              railItems.forEach(item => {
                  if (item.dataset.section === currentId) {
                      item.classList.add('active');
                  } else {
                      item.classList.remove('active');
                  }
              });
              
              // Manage Particle Canvas Visibility & Lifecycle
              if (currentId === 'portal-nexus') {
                  particleCanvas.style.opacity = '0.4';
                  ChoppedPoker.portal.startParticles();
              } else {
                  particleCanvas.style.opacity = '0';
                  ChoppedPoker.portal.stopParticles();
              }
          }
      });
  }, observerOptions);

  sections.forEach(sec => sectionObserver.observe(sec));
  ```
- **Z-Index Layering Order:**
  - Ambient background noise and particles: `z-index: 0` to `1`.
  - Section layout and content cards: `z-index: 10` to `20`.
  - Section 1 verdict box: `z-index: 50`.
  - Sticky control panels (Sections 3 & 4): `z-index: 100`.
  - Global chapter rail (`#global-chapter-rail`): `z-index: 1000`.
  - Section 4 anchored card popover (`#popover-anchor`): `z-index: 9999`.

### 2. Status & Downstream Unblocking
- **Claimed by:** `wayfinder-read-and-plan`
- **Resolution Status:** `resolved`
- **Downstream Unblocking:** Unblocks Ticket 003 (Scroll & Gesture Hygiene) and Ticket 006 (Verification Framework).
