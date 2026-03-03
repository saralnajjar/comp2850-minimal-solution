# Accessibility Audit — COMP2850 Task Manager Web App

A two-round WCAG 2.1 accessibility evaluation of a task management web application, conducted as part of COMP2850 (Human-Computer Interaction) at the University of Leeds.

This case study covers the full audit lifecycle: study design, real participant testing across three different input methods, findings prioritisation, code fixes, and a re-pilot to verify improvements.

---

## Overview

| | |
|---|---|
| **Application** | Task Manager web app (Kotlin + Ktor + HTMX) |
| **Evaluation standard** | WCAG 2.1 (Levels A and AA) |
| **Participants** | 3 (initial study) + 2 (re-pilot) |
| **Input methods tested** | Touch (iPhone), Keyboard-only (MacBook), Voice Control (iPad) |
| **Fixes implemented** | 3 |
| **WCAG compliance improvement** | 11/20 → 13/20 passing criteria |

---

## Methodology

### Job Stories
Four job stories were written to ground the evaluation in real user goals:
- Quick task addition during a time-sensitive situation
- Finding a specific task among many
- Editing a task in place without re-entering it
- Managing tasks entirely without a mouse

### Tasks
Five tasks were designed to match the job stories, with defined success criteria and target completion times. Task 5 had three variants to cover the three input methods (5a: keyboard, 5b: touch, 5c: Voice Control).

### Participants

| ID | Device | Input Method |
|----|--------|-------------|
| P1 | iPhone | Touch / finger |
| P2 | MacBook | Mouse (Tasks 1–4) + Keyboard-only (Task 5a) |
| P3 | iPad | Voice Control ("Show Numbers" feature) |

All participants gave informed consent. Data was anonymised using session IDs (P1_xxxx format). No personally identifiable information was recorded.

### Data Collection
- Observational notes during each task
- Participant quotes
- Quantitative metrics logged via the app (timestamps, response codes, task types)
- Post-test questionnaires
- WCAG checklist evaluation

---

## Findings

Six issues were identified and prioritised using an **Impact + Inclusion − Effort** scoring framework.

| Finding | WCAG Criterion | Level | Priority Score |
|---------|---------------|-------|---------------|
| Status message not visible | 4.1.3 Status Messages | AA | 7 |
| Keyboard trap — header navigation | 2.1.2 No Keyboard Trap | A | 6 |
| Keyboard trap — Add Task form | 2.1.2 No Keyboard Trap | A | 6 |
| Focus indicator invisible on blue buttons | 2.4.7 Focus Visible / 1.4.11 Non-text Contrast | AA | 6 |
| Voice Control — alphanumeric input difficulty | Usability (not WCAG) | — | 2 |
| No task completion / checkbox feature | Usability (not WCAG) | — | 1 |

### Key observations

**P1 (iPhone / touch):** All three participants asked "Did it work?" after submitting tasks. The status message sat in the top-left corner — none of the participants noticed it. P1 had to scroll down manually to verify every action.

**P2 (MacBook / keyboard-only):** Encountered two critical keyboard traps. The header navigation between "Tasks" and "Health" links required a full page refresh to escape. The Add Task form cycled between the input field and the submit button for 3+ rounds after submission. Task 5a took 75 seconds (target: 45s) with 45+ Tab presses. P2 said: *"Can I use my mouse to just select the task I want?"*

**P3 (iPad / Voice Control):** Task 3 (editing a task with the text "COMP2850") took 131 seconds against a 15-second target — 773% over time. The alphanumeric keyboard switching was unpredictable, and cursor positioning voice commands were typed as text instead of executing. P3: *"This is very frustrating."*

---

## Fixes Implemented

### Fix 1 — Status Message Visibility (WCAG 4.1.3)

**Problem:** The status message `<div>` had correct ARIA attributes (`role="status"`, `aria-live="polite"`) but no visual styling — it was small, plain, and positioned in the top-left corner where no participant looked.

**Fix:** Added inline CSS to the Kotlin `messageStatusFragment()` function to give the message a colour-coded background (green for success, red for error), bold text, padding, and margin. The ARIA attributes were already correct; this was purely a visual fix.

```kotlin
// Before — no visual styling
val cssClass = if (isError) """ class="error"""" else ""

// After — colour-coded, prominent
val backColour = if (isError) "#FFCCCB" else "#90EE90"
val textColour = if (isError) "#8B0000" else "#006400"
val cssClass = """background-color: $backColour; border: 3px solid $backColour;
  color: $textColour; padding: 15px; font-size: 17px;
  margin: 10px 0; font-weight: bold;"""
```

---

### Fix 2 — Keyboard Trap in Add Task Form (WCAG 2.1.2) — *Attempted, unsuccessful*

**Problem:** After submitting a task via keyboard, focus remained trapped cycling between the task title input and the Add Task button. P2 required 3 full cycles to escape.

**Attempted fix:** Adding a JavaScript event listener for HTMX's `task-added` event to programmatically move focus after submission.

```javascript
document.body.addEventListener('task-added', function(evt) {
  const input = document.getElementById('task-title-input');
  if (input) {
    input.value = '';
    input.focus();
  }
});
```

**Outcome:** Multiple implementation attempts caused the application to crash, likely due to timing conflicts between HTMX DOM manipulation and JavaScript trying to access elements mid-swap. The fix was not included in the final submission to preserve application stability.

**How it could be resolved:** Ensure the ARIA live region container persists in the DOM and use HTMX's out-of-band swap to update only the message content — never removing the container itself. Add `tabindex="-1"` to newly created task items and move focus there post-submission.

---

### Fix 3 — Focus Indicator Contrast (WCAG 2.4.7 / 1.4.11)

**Problem:** The default focus outline colour (`#4A90E2`) almost exactly matched the button background blue, giving approximately a **1.5:1 contrast ratio** — well below the required 3:1. P2: *"It was the same colour as the button."*

**Fix:** Overrode the focus indicator in CSS with a 12px yellow (`#FFFF00`) outline at 4px offset. The yellow-on-blue combination gives an **8:1 contrast ratio**. The `!important` declarations and `:where()` overrides were necessary to dominate Pico CSS's deeply nested default styles.

```css
button:focus,
button:focus-visible,
a:focus,
a:focus-visible,
input:focus,
input:focus-visible {
  outline: 12px solid #FFFF00 !important;
  outline-offset: 4px !important;
  box-shadow: none !important;
}

/* Override Pico CSS :where() selectors */
:where(button):focus,
:where(input):focus {
  outline: 12px solid #FFFF00 !important;
  outline-offset: 4px !important;
}
```

---

## Re-Pilot Results

The same participants (P1 and P2) re-tested the fixed application the following day.

### Quantitative improvement

| Task | Target | Study 1 Mean | Study 2 Mean | Improvement |
|------|--------|-------------|-------------|-------------|
| Task 1 | 15s | 31s | 12s | −61% |
| Task 2 | 10s | 19s | 9s | −53% |
| Task 3 | 15s | 58s | 12s | −79% |
| Task 4 | 20s | 42s | 12s | −71% |
| Task 5a (keyboard) | 45s | 75s | 45s | −40% |
| Task 5b (touch) | 10s | 8s | 6s | −25% |

- P1 average task time: 18.6s → 9.2s (**51% improvement**)
- P2 average task time: 33.4s → 18.4s (**45% improvement**)
- Keyboard Tab presses (Task 5a): 45+ → ~25 (**44% reduction**)
- Times P2 lost track of keyboard position: "many" → **0**

### Qualitative feedback

> *"There's a message unlike last time."* — P1, on noticing the status message immediately

> *"That new isn't it. Much better."* — P2, on the yellow focus indicator

> *"The yellow outline and the skip link. I could actually navigate properly this time."* — P2

> *"It's definitely better. Still a bit annoying with the cycling but much more usable than before."* — P2, on the remaining form trap

### WCAG compliance
- Before: 11/20 criteria passing
- After: 13/20 criteria passing (+18%)
- 3 out of 4 main findings fully resolved (75% success rate)
- 1 Level A violation remains: WCAG 2.1.2 (Add Task form keyboard trap)

---

## Remaining Issues

| Issue | WCAG | Status |
|-------|------|--------|
| Keyboard trap — Add Task form | 2.1.2 (A) | Unresolved — fix crashed app |
| Focus management after task submission | 2.4.3 (A) | Unresolved |
| ARIA live region DOM persistence with HTMX | 4.1.3 (AA) | Partial pass |
| Bypass blocks for task list | 2.4.1 (A) | Unresolved |

---

## Reflections

This project highlighted how accessible design failures often compound each other. The keyboard trap issue was made worse by the invisible focus indicator — if P2 could have seen where they were in the tab order, they might have navigated around the trap faster. Fixing the focus indicator partially mitigated a separate, unresolved bug.

The Voice Control findings (P3) point to a broader gap in how web applications handle mixed alphanumeric input — WCAG doesn't fully address this, but it's a significant real-world barrier for users who rely on speech recognition.

The HTMX + server-side rendering architecture created genuine constraints around JavaScript-based focus management that I wasn't able to solve within the project timeline. This is a known tension in HTMX-driven apps and would require a more systematic approach to focus handling at the framework level.

---

## Tech Stack

- **Backend:** Kotlin, Ktor
- **Frontend:** HTML, CSS (Pico CSS), HTMX
- **Build:** Gradle
- **IDE:** IntelliJ IDEA
- **Evaluation standard:** WCAG 2.1 (Levels A and AA)

---

*Submitted December 2025 as part of COMP2850 HCI Portfolio, University of Leeds.*
*All participant data anonymised. Informed consent obtained from all participants.*