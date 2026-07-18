# Reveal Balance Report Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a three-number starting/ending balance report to the top of the reveal (truth) section of the Odd-Even rigged-dice demo.

**Architecture:** A new static markup block sits at the top of `#truthStage`, styled with the existing truth-mode CSS tokens. A single JS function (`buildBalanceReport()`) reads `START_BALANCE` and the final `state.balance` at reveal time and fills in the three values, adapting the third block's label, value, and color to whether the player lost, came out ahead, or broke even. It is called from `revealTruth()` alongside the existing `buildLedger()`.

**Tech Stack:** Plain HTML/CSS/vanilla JS in a single self-contained file (`index.html`). No build step, no dependencies, no test framework.

## Global Constraints

- All work is in one file: `index.html`. No new files, no dependencies.
- All peso values MUST be formatted with the existing `formatPeso` helper (`index.html:710`): `const formatPeso = (n) => '₱' + Math.round(n).toLocaleString('en-PH');`.
- Use `$(id)` for element lookup — defined at `index.html:618` as `document.getElementById`.
- Reuse existing CSS custom properties only (defined in `:root`, `index.html:11-33`): `--loss:#ff5a6e`, `--win:#5ff0a0`, `--gold`, `--gold-bright`, `--paper`, `--paper-dim`, `--ink-bg-2`, `--mono-text`. Do NOT introduce new color literals.
- Keep the truth-mode aesthetic sober (it is deliberately restrained — see the `TRUTH MODE (kept sober on purpose)` CSS comment at `index.html:354`). No glow/animation on this block.
- No new game state — `START_BALANCE` and `state.balance` already hold both numbers at reveal time.
- Verification is manual, in-browser (open `index.html` directly). There is no automated test runner in this repo.

---

### Task 1: Static markup + styling for the balance report

**Files:**
- Modify: `index.html` — insert markup between `.truth-header` (closes at line 553) and `.receipt` (opens at line 555)
- Modify: `index.html` — insert CSS after the `.reset-btn:hover` rule (line 382), before the `@media (prefers-reduced-motion: reduce)` block (line 384)

**Interfaces:**
- Produces (consumed by Task 2): DOM element IDs `brsStart`, `brsEnd`, `brsOutcomeStat`, `brsOutcomeLabel`, `brsOutcomeValue`; outcome CSS classes `outcome-loss`, `outcome-win`, `outcome-even` applied to `#brsOutcomeStat`.

- [ ] **Step 1: Add the CSS block**

Insert immediately after the `.reset-btn:hover{ background:var(--gold); color:var(--ink-bg); }` line (line 382):

```css
  .balance-report{ display:grid; grid-template-columns:repeat(3,1fr); gap:14px; margin:0 0 26px; }
  .balance-report-stat{ background:var(--ink-bg-2); border:1px solid rgba(255,255,255,.08); border-radius:10px; padding:18px 16px; text-align:center; }
  .balance-report-stat .brs-label{ font-family:'IBM Plex Mono',monospace; font-size:11px; letter-spacing:.08em; text-transform:uppercase; color:var(--paper-dim); margin:0 0 8px; }
  .balance-report-stat .brs-value{ font-family:'Fraunces',serif; font-weight:700; font-size:clamp(22px,5vw,30px); line-height:1; color:var(--paper); margin:0; }
  .balance-report-stat.outcome-loss{ border-left:3px solid var(--loss); }
  .balance-report-stat.outcome-loss .brs-value{ color:var(--loss); }
  .balance-report-stat.outcome-win{ border-left:3px solid var(--win); }
  .balance-report-stat.outcome-win .brs-value{ color:var(--win); }
  .balance-report-stat.outcome-even{ border-left:3px solid var(--gold); }
  @media (max-width:560px){ .balance-report{ grid-template-columns:1fr; } }
```

- [ ] **Step 2: Add the HTML markup**

Insert between the closing `</div>` of `.truth-header` (line 553) and the `<div class="receipt">` line (line 555):

```html
    <div class="balance-report" id="balanceReport">
      <div class="balance-report-stat">
        <p class="brs-label">You walked in with</p>
        <p class="brs-value" id="brsStart">—</p>
      </div>
      <div class="balance-report-stat">
        <p class="brs-label">You walked out with</p>
        <p class="brs-value" id="brsEnd">—</p>
      </div>
      <div class="balance-report-stat" id="brsOutcomeStat">
        <p class="brs-label" id="brsOutcomeLabel">The house took</p>
        <p class="brs-value" id="brsOutcomeValue">—</p>
      </div>
    </div>
```

- [ ] **Step 3: Verify the shell renders**

Open `index.html` in a browser. Play one round (any bet + guess + roll), then click "Cash out & leave the table →" to reach the reveal. 

Expected: A row of three cards appears directly under the RIGGED header and above the Session Ledger. Labels read "You walked in with", "You walked out with", "The house took". Values show the em-dash placeholder `—` (Task 2 fills them in). Cards use the dark sober card style matching the explainer cards below. Narrow the window below 560px: the three cards stack into a single column.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add balance report shell to reveal section"
```

---

### Task 2: Populate the report and drive the outcome states

**Files:**
- Modify: `index.html` — add the `buildBalanceReport()` function next to `buildLedger()` (which ends at line 1004)
- Modify: `index.html` — call `buildBalanceReport()` inside `revealTruth()` (after `buildLedger();`, line 1023)

**Interfaces:**
- Consumes (from Task 1): element IDs `brsStart`, `brsEnd`, `brsOutcomeStat`, `brsOutcomeLabel`, `brsOutcomeValue`; classes `outcome-loss`, `outcome-win`, `outcome-even`.
- Consumes (existing): global `START_BALANCE` (number), `state.balance` (number), `formatPeso(n)`, `$(id)`.

- [ ] **Step 1: Add the `buildBalanceReport` function**

Insert immediately after the closing brace of `buildLedger()` (line 1004), before `function drawProbChart(){` (line 1005):

```javascript
  function buildBalanceReport(){
    const start = Math.round(START_BALANCE), end = Math.round(state.balance);
    $('brsStart').textContent = formatPeso(start);
    $('brsEnd').textContent = formatPeso(end);
    const stat = $('brsOutcomeStat'), label = $('brsOutcomeLabel'), value = $('brsOutcomeValue');
    stat.classList.remove('outcome-loss','outcome-win','outcome-even');
    if (end < start){
      stat.classList.add('outcome-loss');
      label.textContent = 'The house took';
      value.textContent = formatPeso(start - end);
    } else if (end > start){
      stat.classList.add('outcome-win');
      label.textContent = 'You came out ahead by';
      value.textContent = formatPeso(end - start);
    } else {
      stat.classList.add('outcome-even');
      label.textContent = 'You broke even';
      value.textContent = formatPeso(0);
    }
  }
```

- [ ] **Step 2: Call it from `revealTruth()`**

In `revealTruth()`, change the opening lines from:

```javascript
  function revealTruth(){
    buildLedger();
```

to:

```javascript
  function revealTruth(){
    buildLedger();
    buildBalanceReport();
```

- [ ] **Step 3: Verify the loss case**

Open `index.html`. Accept the default ₱5,000 start. Play rounds and keep betting until you lose money (play well past the early hook rounds so the odds grind you down), then cash out.

Expected: "You walked in with ₱5,000", "You walked out with ₱[less]", third card reads "The house took ₱[5,000 − end]" with a red left border and red value. Confirm the arithmetic: start − end equals the third number.

- [ ] **Step 4: Verify the came-out-ahead case**

Reset ("Reset and play again"). Start a fresh game. Win an early round (the rig hands out early wins) so your balance exceeds the start, then immediately cash out while ahead.

Expected: third card reads "You came out ahead by ₱[end − start]" with a green left border and green value.

- [ ] **Step 5: Verify the break-even and custom-balance cases**

Reset. On the start prompt, enter a custom balance (e.g. `3000` instead of the default). Play and manage bets so you cash out at exactly your starting balance (e.g. win one round then lose the same amount back, or cash out before betting).

Expected: "You walked in with ₱3,000" (custom value flows through), and when end equals start the third card reads "You broke even ₱0" with a gold left border and neutral (paper-colored) value.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: populate reveal balance report with outcome-aware bottom line"
```

---

## Notes

- Update `CHANGELOG.md` under `## [Unreleased]` → `### Added` when opening the PR, per project convention (`CLAUDE.md`).
- The feature branch `feature/reveal-balance-report` already exists and holds the design spec commit.
