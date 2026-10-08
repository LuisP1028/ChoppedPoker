---
ticket_id: 5
title: "Interactive Modals, Canvas High-DPI Scaling, and Boundary Collision Safeguards"
type: task
status: resolved
claimed_by: "wayfinder-read-and-plan"
blocked_by: ["ticket-001.md"]
governing_specification: "functional_specification_001.md"
---

# Ticket 005: Interactive Modals, Canvas High-DPI Scaling, and Boundary Collision Safeguards

## Question
How must the anchored 3D flip-card popover in Section 4 handle viewport boundary collision and dynamic math re-rendering, how will the 3D feature cards in Section 0 handle perspective tilt, and how will canvases support Retina/HiDPI displays without blurring?

## Context & Specification Grounding
- **Governing Specification:** [`functional_specification_001.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/functional_specification_001.md) Sections 4.1, 4.5, 7.3, 7.4, and 8 (AC-06).
- **Target Interactive Elements:**
  - Section 0 Feature Cards: 3D perspective tilt on hover, smooth anchor navigation (`#ev-engine`, etc.).
  - Section 4 Anchored Popover: `#popover-anchor` with 3D flip `.card-inner`, dynamic card suit/rank randomization, KaTeX re-rendering, and viewport boundary checks.
  - Canvases: `#particle-canvas` (Section 0) and `#hm-canvas` (Section 3).
- **Current Defect / Conflict Analysis:**
  - In `math.html`, the card popover computes position based on raw `scrollX` and `scrollY`. If an info-chip near the right edge of the screen is clicked on a narrow viewport or mobile screen, the popover overflows `window.innerWidth`, producing horizontal window blowout and horizontal scrollbars.
  - When KaTeX renders math inside the dynamic popover content, if the auto-render extension is not explicitly invoked on the updated DOM container, formula markup remains raw LaTeX strings.
  - On Retina screens (`window.devicePixelRatio >= 2`), rendering a canvas at standard 1x CSS resolution produces fuzzy suit glyphs and blurred heatmap pixel borders.

## Architectural Decisions to Lock
1. **Section 0 Feature Cards 3D Tilt & Anchor Redirection:**
   - Instead of navigating to external pages (`CHECK_BET_EVs.html`, etc.), card anchors point to `#ev-engine`, `#range-filter`, `#strategy-heatmap`, and `#equilibrium-constructor`.
   - The 3D tilt handler calculates pointer coordinates relative to the card's bounding rectangle:
     ```javascript
     card.addEventListener('mousemove', (e) => {
         const rect = card.getBoundingClientRect();
         const xPct = ((e.clientX - rect.left) / rect.width - 0.5) * 2;
         const yPct = ((e.clientY - rect.top) / rect.height - 0.5) * 2;
         card.style.transform = `perspective(1000px) rotateY(${xPct * 6}deg) rotateX(${yPct * -6}deg) translateY(-5px) scale(1.02)`;
     });
     card.addEventListener('mouseleave', () => {
         card.style.transform = 'perspective(1000px) rotateY(0) rotateX(0) translateY(0) scale(1)';
     });
     ```
2. **Section 4 Anchored Card Popover Collision & Flip Architecture:**
   - **Popover Positioning Algorithm:**
     ```javascript
     function openPopover(chipBtn, key) {
         const rect = chipBtn.getBoundingClientRect();
         const scrollTop = window.scrollY || document.documentElement.scrollTop;
         const scrollLeft = window.scrollX || document.documentElement.scrollLeft;

         const cardWidth = 300;
         const cardHeight = 420;
         const margin = 20;

         // Determine top position: place below button, or above if near viewport bottom
         let topPos = rect.bottom + scrollTop + 15;
         if (rect.bottom + cardHeight + margin > window.innerHeight && rect.top - cardHeight > margin) {
             topPos = rect.top + scrollTop - cardHeight - 15;
         }

         // Determine left position with horizontal boundary collision avoidance
         let leftPos = rect.left + scrollLeft;
         if (leftPos + cardWidth + margin > window.innerWidth) {
             leftPos = window.innerWidth - cardWidth - margin;
         }
         if (leftPos < margin) {
             leftPos = margin;
         }

         popoverAnchor.style.top = `${topPos}px`;
         popoverAnchor.style.left = `${leftPos}px`;
         popoverAnchor.classList.add('active');
         cardInner.classList.remove('flipped');

         // Populate Content and re-render KaTeX
         randomizeCard();
         infoTitleEl.textContent = dataMap[key].title;
         infoBodyEl.innerHTML = dataMap[key].html;
         if (window.renderMathInElement) {
             window.renderMathInElement(infoBodyEl, {
                 delimiters: [{ left: "$$", right: "$$", display: true }]
             });
         }
     }
     ```
   - **Clean Dismissal:**
     Clicking outside the popover (`!popoverAnchor.contains(e.target) && !e.target.closest('.info-chip')`) triggers `closePopover()`.
3. **Canvas HiDPI Retina Display Scaling:**
   - **Particle Canvas:**
     ```javascript
     function resizeParticleCanvas() {
         const dpr = window.devicePixelRatio || 1;
         particleCanvas.width = window.innerWidth * dpr;
         particleCanvas.height = window.innerHeight * dpr;
         particleCanvas.style.width = window.innerWidth + 'px';
         particleCanvas.style.height = window.innerHeight + 'px';
         particleCtx.scale(dpr, dpr);
     }
     ```
   - **Heatmap Canvas:**
     The heatmap pixel matrix is calculated at a fixed high-resolution internal grid (600x600 pixels) and displayed via CSS aspect-ratio containment (`width: 100%; height: 100%; max-width: 600px; max-height: 600px; image-rendering: pixelated;`), ensuring pixel-crisp edge rendering across all display densities.

## Scope & Invariant Guardrails
- **In Scope:** Viewport collision detection, 3D card flip transitions, popover DOM attachment, dynamic KaTeX re-rendering inside popover, and HiDPI canvas backing store scaling.
- **Out of Scope:** Overall document layout (Ticket 001) and GGOP math algorithms (Ticket 004).

---

## Resolution

### 1. Interactive Modals Implementation Contract
- Popover DOM element `#popover-anchor` is attached directly as a child of `<body>` to prevent parent CSS transform clippings or stacking context traps.
- `z-index: 9999` guarantees that the popover renders above the sticky sidebars, chapter rail, and ambient overlays.
- Card face flip uses CSS 3D transforms (`transform-style: preserve-3d; backface-visibility: hidden; transition: transform 0.8s cubic-bezier(0.23, 1, 0.32, 1);`).

### 2. Status & Downstream Unblocking
- **Claimed by:** `wayfinder-read-and-plan`
- **Resolution Status:** `resolved`
- **Downstream Unblocking:** Unblocks Ticket 006 (Verification Framework).
