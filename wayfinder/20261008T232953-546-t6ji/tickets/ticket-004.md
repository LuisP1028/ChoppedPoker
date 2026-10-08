---
ticket_id: 4
title: "Mathematical Models, Baseline Values, and Precision Calibration"
type: task
status: resolved
claimed_by: "wayfinder-read-and-plan"
blocked_by: ["ticket-001.md"]
governing_specification: "functional_specification_001.md"
---

# Ticket 004: Mathematical Models, Baseline Values, and Precision Calibration

## Question
How must each mathematical calculation engine across Sections 1 through 5 be calibrated, what are the exact updated default baseline parameters, and how are edge cases (such as zero combos and impossible strategy regions) strictly governed?

## Context & Specification Grounding
- **Governing Specification:** [`functional_specification_001.md`](file:///Users/diesel/Desktop/ChoppedPoker/.sandcastle/worktrees/agent-plan-20261008T232953-546-t6ji/functional_specification_001.md) Sections 4.2, 4.3, 4.4, 4.5, 4.6, 7.1, 7.2, and 8 (AC-02, AC-03, AC-04, AC-05, AC-07).
- **Target Mathematical Engines:**
  - Section 1: Check vs. Bet EV Differential & Indifference ($\alpha_{\text{defense}}$, Combos, $EV_{\text{check}}$, $EV_{\text{call}}$, $\Delta EV$)
  - Section 2: Turn Range Composition Filtering ($T_{\text{value}}$, $T_{\text{air}}$, $T_{\text{total}}$, Value %, Air %)
  - Section 3: PvBC 2D Strategy Heatmap ($EV_{\text{fold}}$, $EV_{\text{cf}}$, $EV_{\text{cc}}$, Strategy Classification, Coordinate Projection)
  - Section 4: Equilibrium Multi-Street GGOP & Recursive Bluff Allocation ($R$, $r$, $P_{\text{turn}}$, $P_{\text{river}}$, $\alpha_{\text{river}}$, $B_R$, $\alpha_{\text{turn}}$, $B_T$)
  - Section 5: Bluffing Efficiency Math Decomposition (Equation 11.7 Cost vs. Efficiency)
- **Current Defect / Conflict Analysis:**
  - Source files contained mismatched or placeholder default values (e.g. Section 1 used $B=50, N=60$; Section 2 used $100\%$ and $50\%$; Section 3 used $S=21, P=8, B=6$; Section 4 used $P=5, S=27.5, V=25$). The specification establishes authoritative defaults and acceptance test cases.
  - Division by zero in Section 2 when both sliders are 0% must not emit `NaN%`.
  - In Section 3, evaluating coordinates where $c_T < N$ without strict short-circuiting can produce invalid or misleading EV calculations.

## Architectural Decisions to Lock
1. **Section 1 (EV Decision Engine) Calibration:**
   - **Default Inputs:** Pot $P = 100$, Bet $B = 75$, Combos $N = 100$, Equity $E = 55\%$.
   - **Formulas & Output Precision:**
     $$\alpha = \frac{P}{P + B} = \frac{100}{100 + 75} = 57.1428\dots\% \implies \mathbf{57.1\%}$$
     $$\text{Combos} = N \times \alpha = 100 \times \frac{100}{175} = 57.1428\dots \implies \mathbf{57.1\text{ Hands}}$$
     $$EV_{\text{check}} = E \times P = 0.55 \times 100 = \mathbf{55.00}$$
     $$EV_{\text{call}} = (E \times (P + 2B)) - B = (0.55 \times 250) - 75 = 137.50 - 75 = \mathbf{62.50}$$
     $$\Delta EV = EV_{\text{call}} - EV_{\text{check}} = 62.50 - 55.00 = \mathbf{+7.50}$$
   - **Verdict Badge:**
     - $E > 0.50 \implies$ `"VALUE BET"` (Orange background `#ff4d00`, dark text)
     - $E = 0.50 \implies$ `"INDIFFERENT"` (White background `#ffffff`, black text)
     - $E < 0.50 \implies$ `"CHECK"` (Dark grey background `#333333`, white text)

2. **Section 2 (Turn Range Composition) Calibration & Zero-Division Guard:**
   - **Default Inputs:** Flop Value C-Bet $= 70\%$, Flop Bluff C-Bet $= 40\%$.
   - **Base Combos:** Pre-flop Value Combos $C_{\text{value}} = 40$, Pre-flop Air Combos $C_{\text{air}} = 60$.
   - **Formulas:**
     $$T_{\text{value}} = 40 \times 0.70 = 28.0, \quad T_{\text{air}} = 60 \times 0.40 = 24.0$$
     $$T_{\text{total}} = 28.0 + 24.0 = 52.0$$
     $$\text{Value \%} = \frac{28.0}{52.0} \times 100 = 53.846\dots\% \implies \mathbf{53.8\%}$$
     $$\text{Air \%} = \frac{24.0}{52.0} \times 100 = 46.153\dots\% \implies \mathbf{46.2\%}$$
   - **Zero Combos Guard:** If $T_{\text{total}} === 0$ (both sliders at 0%), set `barValue.style.width = '50%'`, `barAir.style.width = '50%'`, and output `"-"` for percentage displays.

3. **Section 3 (PvBC 2D Strategy Heatmap) Calibration & Impossible Region:**
   - **Default Inputs:** Stack Depth $S = 100$, Pot Size $P = 30$, Turn Bet $B = 20$, Nut Advantage $N = 0.25$, Turn C-Bet $c_T = 0.60$, River C-Bet $c_R = 0.50$.
   - **Pixel Classification Contract:**
     - If $c_T < N$: Short-circuit immediately. Return `{ type: 'IMPOSSIBLE', maxEV: 0 }`. Color pixel pure white (`#FFFFFF`).
     - Strategy EV Equations:
       $$EV_{\text{fold}} = S$$
       $$EV_{\text{cf}} = (1 - c_R) \times (S + P + B) + c_R \times (S - B)$$
       $$EV_{\text{cc}} = \left(\frac{c_T - c_R c_T}{c_T}\right) \times (S + P + B) + \left(\frac{c_R c_T - N}{c_T}\right) \times (2S + P)$$
     - **Strategy Assignment:**
       - If $\max(EV) = EV_{\text{fold}} \implies$ `"FOLD"` (`#333333`)
       - If $\max(EV) = EV_{\text{cf}} \implies$ `"CALL - FOLD"` (`#FF3030`)
       - If $\max(EV) = EV_{\text{cc}} \implies$ `"CALL - CALL"` (`#0047BB`)
     - Under baseline defaults: $EV_{\text{fold}} = 100$, $EV_{\text{cf}} = 115.00$, $EV_{\text{cc}} = 94.17$. Max strategy is `"CALL - FOLD"`.

4. **Section 4 (Equilibrium Constructor & GGOP) Calibration:**
   - **Default Inputs:** $P_{\text{start}} = 10$, $S_{\text{eff}} = 75$, $V = 40$.
   - **Calculations:**
     - Growth Multiplier: $R = \frac{P_{\text{start}} + 2 S_{\text{eff}}}{P_{\text{start}}} = \frac{10 + 150}{10} = \mathbf{16.0}$
     - Multi-Street Sizing Factor: $r = \sqrt{R} - 1 = \sqrt{16} - 1 = 3.0 \implies \mathbf{300.0\%}$
     - Street Pots:
       $$P_{\text{turn}} = P_{\text{start}} + 2(r \times P_{\text{start}}) = 10 + 2(30) = 70.0$$
       $$P_{\text{river}} = P_{\text{turn}} + 2(r \times P_{\text{turn}}) = 70 + 2(210) = 490.0$$
     - River Bluff Allocation:
       $$\alpha_{\text{river}} = \frac{\text{Bet}_{\text{river}}}{P_{\text{river}} + 2 \text{Bet}_{\text{river}}} = \frac{210}{490 + 420} = \frac{210}{910} = 23.076\dots\% \implies \mathbf{23.1\%}$$
       $$B_R = V \times \alpha_{\text{river}} = 40 \times \frac{210}{910} = \mathbf{9.2\text{ combos}}$$
     - Turn Bluff Allocation:
       $$V_{\text{turn}} = V + B_R = 40 + 9.23 = 49.23$$
       $$\alpha_{\text{turn}} = \frac{\text{Bet}_{\text{turn}}}{P_{\text{turn}} + 2 \text{Bet}_{\text{turn}}} = \frac{30}{70 + 60} = \frac{30}{130} = 23.076\dots\% \implies \mathbf{23.1\%}$$
       $$B_T = V_{\text{turn}} \times \alpha_{\text{turn}} = 49.23 \times \frac{30}{130} = \mathbf{11.4\text{ combos}}$$
       $$\text{Total Starting Range} = V + B_R + B_T = 40 + 9.23 + 11.36 = \mathbf{60.6\text{ combos}}$$

5. **Section 5 (Bluffing Efficiency Equation 11.7) Presentation:**
   - Equation markup with interactive hover spans:
     $$\frac{B}{B + P} < \frac{F_T - F_T E_f}{1 - E_s}$$
   - Synchronized hover state updates the explanation panel with title and formatted text.

## Scope & Invariant Guardrails
- **In Scope:** Mathematical formulas, exact rounding and floating point operations, edge case short circuits, and baseline default value settings.
- **Out of Scope:** Canvas mouse drag handling (Ticket 003), popover card flip animations (Ticket 005), and global navigation (Ticket 002).

---

## Resolution

### 1. Calculation Engine Implementation Specification
- **Section 1 Script Contract:**
  ```javascript
  function calculateEV() {
      const P = parseFloat(dom.evPot.value);
      const B = parseFloat(dom.evBet.value);
      const N = parseFloat(dom.evCombos.value);
      const E = parseFloat(dom.evEquity.value) / 100;

      dom.evPotVal.textContent = P;
      dom.evBetVal.textContent = B;
      dom.evCombosVal.textContent = N;
      dom.evEquityVal.textContent = Math.round(E * 100);

      const alpha = P / (P + B);
      dom.evMdf.textContent = (alpha * 100).toFixed(1) + "%";

      const rangeCombos = N * alpha;
      dom.evRange.textContent = rangeCombos.toFixed(1) + " Hands";

      const evCheck = E * P;
      const evCall = (E * (P + 2 * B)) - B;
      const diff = evCall - evCheck;

      dom.evCheck.textContent = evCheck.toFixed(2);
      dom.evCall.textContent = evCall.toFixed(2);
      dom.evDiff.textContent = (diff > 0 ? "+" : "") + diff.toFixed(2);
      dom.evDiff.style.color = diff > 0 ? "var(--ev-accent)" : "#888";

      if (E > 0.5) {
          dom.evVerdict.textContent = "VALUE BET";
          dom.evVerdict.style.backgroundColor = "var(--ev-accent)";
          dom.evVerdict.style.color = "#111";
      } else if (E === 0.5) {
          dom.evVerdict.textContent = "INDIFFERENT";
          dom.evVerdict.style.backgroundColor = "#fff";
          dom.evVerdict.style.color = "#000";
      } else {
          dom.evVerdict.textContent = "CHECK";
          dom.evVerdict.style.backgroundColor = "#333";
          dom.evVerdict.style.color = "#fff";
      }
  }
  ```
- **Section 2 Script Contract:**
  ```javascript
  function updateTurnRange() {
      const valFreq = parseFloat(dom.turnValSlider.value) / 100;
      const bluffFreq = parseFloat(dom.turnBluffSlider.value) / 100;

      dom.turnValDisp.textContent = Math.round(valFreq * 100) + "%";
      dom.turnBluffDisp.textContent = Math.round(bluffFreq * 100) + "%";

      const RAW_VALUE = 40;
      const RAW_AIR = 60;

      const turnValue = RAW_VALUE * valFreq;
      const turnAir = RAW_AIR * bluffFreq;
      const total = turnValue + turnAir;

      if (total === 0) {
          dom.turnBarVal.style.width = "50%";
          dom.turnBarAir.style.width = "50%";
          dom.turnStatValPct.textContent = "-";
          dom.turnStatAirPct.textContent = "-";
          return;
      }

      const valPct = (turnValue / total) * 100;
      const airPct = (turnAir / total) * 100;

      dom.turnBarVal.style.width = valPct + "%";
      dom.turnBarAir.style.width = airPct + "%";
      dom.turnStatValPct.textContent = valPct.toFixed(1) + "%";
      dom.turnStatAirPct.textContent = airPct.toFixed(1) + "%";
  }
  ```

### 2. Status & Downstream Unblocking
- **Claimed by:** `wayfinder-read-and-plan`
- **Resolution Status:** `resolved`
- **Downstream Unblocking:** Unblocks Ticket 006 (Verification Framework).
