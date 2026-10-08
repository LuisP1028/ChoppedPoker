---
ticket_id: 3
title: "Scroll & Gesture Hygiene: Isolation of Sliders, Canvas Drag, and Sticky Sidebars"
type: task
status: resolved
claimed_by: "wayfinder-read-and-plan"
blocked_by: ["ticket-001.md"]
governing_specification: "functional_specification_001.md"
---

# Ticket 003: Scroll & Gesture Hygiene: Isolation of Sliders, Canvas Drag, and Sticky Sidebars

## Question
How must event listeners, pointer captures, and CSS layout rules be configured to eliminate scroll-trapping over range sliders, decouple 2D canvas crosshair dragging from vertical page scrolling, and prevent sticky sidebars from spilling past section boundaries?

## Context & Specification Grounding
- **Governing Specification:** [`functional_specification_001.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/functional_specification_001.md) Sections 5.2, 8 (AC-08).
- **Affected Components:**
  - Range sliders in Section 1 (`ev-pot-input`, `ev-bet-input`, `ev-combo-input`, `ev-equity-input`)
  - Range sliders in Section 2 (`turn-val-slider`, `turn-bluff-slider`)
  - Range sliders in Section 3 (`hm-input-s`, `hm-input-p`, `hm-input-b`, `hm-input-n`, `hm-input-ct`, `hm-input-cr`)
  - Strategy heatmap canvas in Section 3 (`#hm-canvas`)
  - Sticky sidebars in Section 3 (`.sidebar`) and Section 4 (`.input-panel`)
- **Current Defect / Conflict Analysis:**
  - In desktop browsers, scrolling vertically over native `<input type="range">` elements often accidentally modifies input values while freezing page scroll position.
  - In touch devices and mobile browsers, placing a finger on the 2D heatmap canvas in `betMap.html` captures touch gestures, trapping the user inside the canvas and deadlocking page scrolling.
  - In `betMap.html` and `math.html`, the sticky sidebars (`position: sticky; top: 40px`) assume the viewport belongs solely to that tool. In a unified vertical document, unless containment is explicitly bound to the parent `<section>`, sticky panels will overlap and collide with subsequent sections.

## Architectural Decisions to Lock
1. **Slider Wheel Event Suppression & Touch Action:**
   - Sliders must only respond to intentional pointer/mouse dragging or keyboard arrow navigation.
   - Attach a passive wheel event handler to all range sliders that suppresses value modification without blocking document scroll:
     ```javascript
     document.querySelectorAll('input[type="range"]').forEach(slider => {
         slider.addEventListener('wheel', (e) => {
             // Do not stop propagation; allow page scroll, but blur or prevent default slider scrubbing
             slider.blur();
         }, { passive: true });
     });
     ```
   - In CSS, apply `touch-action: pan-y;` across all slider containers so vertical touch gestures pass through to window scrolling unimpeded.
2. **2D Heatmap Canvas Pointer & Touch Hygiene:**
   - On `#hm-canvas`, configure CSS: `touch-action: pan-y; cursor: crosshair;`.
   - Implement explicit pointer capture logic on `pointerdown`:
     ```javascript
     let isDraggingCrosshair = false;

     canvas.addEventListener('pointerdown', (e) => {
         // Only engage crosshair drag if primary button or touch contact
         isDraggingCrosshair = true;
         canvas.setPointerCapture(e.pointerId);
         updateCrosshairFromPointer(e);
     });

     canvas.addEventListener('pointermove', (e) => {
         if (!isDraggingCrosshair) return;
         updateCrosshairFromPointer(e);
     });

     const endDrag = (e) => {
         if (isDraggingCrosshair) {
             isDraggingCrosshair = false;
             try {
                 canvas.releasePointerCapture(e.pointerId);
             } catch (_) {}
         }
     };

     canvas.addEventListener('pointerup', endDrag);
     canvas.addEventListener('pointercancel', endDrag);
     ```
   - Standard vertical touch swipes on the canvas without dragging trigger normal vertical scrolling because `touch-action: pan-y` is preserved.
3. **Sticky Sidebar Section Boundary Containment:**
   - The sticky layout containers in Section 3 (`#strategy-heatmap`) and Section 4 (`#equilibrium-constructor`) must have their parent containers set to `position: relative; display: grid; align-items: start; height: 100%;`.
   - The sticky child panels (`.sidebar` and `.input-panel`) receive:
     ```css
     .section-heatmap .sidebar,
     .section-equilibrium .input-panel {
         position: sticky;
         top: 80px; /* Space below fixed header/rail */
         max-height: calc(100vh - 120px);
         overflow-y: auto;
     }
     ```
   - Because the parent section bounds the stacking context, when the bottom edge of Section 3 or Section 4 scrolls past the viewport, the sticky sidebar naturally releases and moves upward out of view with its parent section.

## Scope & Invariant Guardrails
- **In Scope:** Event listener configurations for sliders and canvas, pointer capture mechanisms, CSS touch actions, and sticky container containment.
- **Out of Scope:** Pixel math in canvas rendering (Ticket 004), modal placement algorithms (Ticket 005), and global chapter rail styling (Ticket 002).

---

## Resolution

### 1. Gesture Hygiene Specification
- **Event Suppression Matrix:**
  | Control | Event | Action Taken | Result |
  | :--- | :--- | :--- | :--- |
  | `input[type="range"]` | `wheel` | Remove focus / ignore wheel scrubbing | Page scrolls naturally; slider value remains untouched. |
  | `input[type="range"]` | `touchstart/move` | `touch-action: pan-y` | Vertical swipe scrolls page; horizontal drag adjusts slider. |
  | `#hm-canvas` | `wheel` | Allow default bubbling | Page scrolls over canvas freely. |
  | `#hm-canvas` | `pointerdown` | `setPointerCapture` | Sets active crosshair coordinates on demand. |
  | `.sidebar` / `.input-panel` | `scroll` | Bounded to parent section height | Sticks while section is visible; releases at section bottom. |

### 2. Status & Downstream Unblocking
- **Claimed by:** `wayfinder-read-and-plan`
- **Resolution Status:** `resolved`
- **Downstream Unblocking:** Unblocks Ticket 006 (Verification Framework).
