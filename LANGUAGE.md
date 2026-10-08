---
name: system-glossary-evaluation-parameters
hostname: local-workspace
description: A comprehensive glossary defining the strict evaluation parameters used for assessing code errors, correctness, functionality, and documentation sufficiency within the ChoppedPoker system.
---

# Evaluation Parameters

## {errors}
- Explicit failure states surfaced during the execution of a component.
- **Qualifiers:** Stack traces, syntax errors, unhandled JavaScript exceptions, compilation failures, non-zero exit codes, console error logs, broken resource links, failed asset loads, or severe warnings that halt execution.

## {correctness}
- The precise, local execution of a specific component, verifying that it performs its designated task without logical flaws, even if no explicit `{errors}` are thrown.
- **Qualifiers:** The code processes data properly, mathematical calculations (e.g., MDF, pot odds, EV differences, geometric bet sizing) are accurate to floating-point precision, event handlers bind without memory leaks, transitions terminate cleanly, and DOM hierarchy remains valid.

## {functionality}
- The broader, global objective or business logic the component is meant to achieve within the larger system.
- **Qualifiers:** Does the unified static interface successfully integrate and display all tool modules? Does the scrolly-telling transition seamlessly between sections without obscuring tools or degrading interactive responsiveness? Does the outcome preserve 100% of the original content?

## {correct required outputs}
- The exact, literal expected state after a component executes successfully. This must be objectively measurable.
- **Qualifiers:** A single standalone HTML file containing all six discrete poker tool engines, responsive scroll-driven transitions, working interactive range sliders, real-time math computation displays matching original baseline values, and absence of broken visual artifacts.

## {sufficient}
- The state of documentation where absolutely no ambiguity remains, and the coding assistant can proceed to implementation without guessing or hallucinating context.
- **Qualifiers:** All source component HTML/CSS/JS behaviors are cataloged, input and output boundaries are defined, mathematical formulas are explicitly stated, scrolly-telling interaction paradigms and transition thresholds are specified, and `{correct required outputs}` are established.

## {insufficient}
- The state of documentation where critical context is missing, ambiguous, or contradictory, requiring the assistant to make assumptions to write code.
- **Qualifiers:** Vague instructions (e.g., "merge the files smoothly"), omitted mathematical formulas, undefined scroll trigger states, lack of viewport boundary specifications, or failure to explicitly define content preservation requirements.

# ChoppedPoker Domain Glossary

## {MDF (Minimum Defense Frequency)}
- The minimum percentage of a defender's range that must continue (call or raise) facing a bet of size $B$ into pot $P$ to prevent the bettor from earning an immediate automatic profit with zero-equity bluffs: $\alpha_{\text{defense}} = \frac{P}{P + B}$.

## {Indifference Principle}
- A foundational Nash Equilibrium property wherein a player creates a mixed strategy (balancing value bets and bluffs) such that the opponent's expected value of calling versus folding becomes exactly equal ($EV_{\text{call}} = EV_{\text{fold}} = 0$ for marginal bluff-catchers).

## {Check vs Bet EV Difference}
- The quantitative differential $\Delta EV = EV_{\text{bet}} - EV_{\text{check}}$ that determines the threshold equity required for a hand to profitably value-bet against an opponent's calling range versus realizing equity via checking.

## {Geometric Bet Sizing}
- A sizing strategy that scales bets across multiple streets such that the bet-to-pot ratio remains constant while committing the exact effective stack depth by the river.

## {Turn Range Composition}
- The conditional probability distribution of player holding types (value, draw, marginal showdown, air) arriving at the turn after applying flop betting and checking frequency filters.

## {PvBC Strategy Map}
- Visual heatmap matrix plotting optimal big-blind defense frequencies and action distributions against varying turn and river continuation-bet frequencies.

## {Scrolly-Telling}
- A narrative presentation paradigm in web design where vertical scroll position orchestrates dynamic visual progression, activating fixed or pinned interface views, transitioning between distinct theoretical concepts, and smoothly morphing or scrolling interactive tool stages while maintaining full user control over tool inputs.

## {Unified Static Artifact}
- A single standalone HTML document embedding all structure, styling, canvas rendering, and calculation engines without external runtime build steps, package managers, or server-side rendering requirements.
