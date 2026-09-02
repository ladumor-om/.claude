---
name: learn-firmware
description: Learn MOT embedded firmware — answer the user's questions in chat with full register-level explanation, then capture everything into Learning-Docs/. Use when the user runs /learn-firmware <REPO>, asks any question about a MOT firmware repository, or asks about an embedded concept (I2C, UART, PWM, ADC, clocks, crystals, DMA, interrupts, registers, FreeRTOS) in this project's context.
---

# Learn Firmware

The user learns by **conversation**. They read the chat, not the docs. Docs are for their future
self, so the same research never has to happen twice.

**The single most important thing in this file is the reference example.**
Read [`reference/good-answer.md`](reference/good-answer.md) before answering. Match its depth and
its shape. Everything below is context for *that*, not a checklist to satisfy.

> **Portability.** This file is the complete source of truth for the workflow and is handed to
> other agents (Antigravity at home) to generate their equivalent config. Keep it whole and
> self-contained. It pairs with `CLAUDE.md`, which holds project paths and hard rules.

---

## Answer now, write at the boundary

**Explain in chat. Write no files. Carry a visible list of what is owed. Flush it at the next
natural boundary.**

```
   ┌─ answering turns ──────────────────────────────────────────┐
   │  question → full answer in chat → write NOTHING            │
   │  end the answer with one line:                             │
   │      📋 Pending: Protocols/I2C.md · Controllers/STM8/07    │
   │  (list grows as the topic continues)                       │
   └──────────────────────────┬─────────────────────────────────┘
                              │
                   FLUSH at the first of:
                     · the user changes topic
                     · the user says anything that reads like closing
                       ("ok", "next", "thanks", "let's do X")
                     · the user says "save it" / "update docs"
                     · pending grows past ~5 files
                              │
                              ▼
                   do the documentation sweep
```

**Why the separation, precisely.** The old rule wrote docs in the answering turn, and the write
ran *before* the explanation existed — so the doc write was the thinking and the chat answer came
out as a summary of files the user could not see. They noticed immediately.

**Why a flush is nonetheless cheap.** By the time it runs, the content already exists in the
chat. Writing it down is **transcription, not composition** — which is why it does not degrade
the answer beside it the way the old same-turn write did.

**Why batching produces better docs, not just cheaper ones.** Five questions about I2C answered
one-by-one into `I2C.md` gives five successive appends — a changelog. One flush after the topic
gives one coherent section.

**The pending line is not optional.** It is what makes the automation safe: the user reads chat,
not docs, so an un-flushed sweep is invisible to them unless it is stated. If boundary detection
misfires, the visible list lets them say the word.

The exception: if the user's message itself asks for a doc change ("fix that file", "add this to
Timers"), do it immediately — that *is* the request.

---

## Scope — answer what was asked

**Do not add adjacent topics.** Not other vendors, not other chip families, not portability
advice, not "how industry does it", not interview questions — unless the user asks.

If you notice something genuinely worth knowing, **offer it in one line and stop**:

> Worth knowing separately: TIM1's prescaler uses a different encoding to TIM4's. Say the word.

That is the whole mechanism. One line, then wait. The user decides what they learn next.

**Skip what they already know.** If they say "I understand the I2C init", don't re-explain it.
Say one line confirming what you're skipping, so a wrong assumption gets caught early.

---

## What a good answer contains

Read the reference example. In short:

**Open with the shape.** Group the thing before detailing it — "two jobs, three registers each".
The user needs a frame before they get facts.

**Line by line for anything touching hardware.** Register writes, bit manipulation, init
sequences. Boilerplate may be grouped, never a register write.

**Name the actor.** "Hardware sets this on the next edge." "Software must clear it by reading DR."

**Derive the maths, never assert it.** Formula, substitution, result — on separate lines:

```
FREQR = 2        → f_MASTER = 2 MHz  → t_MASTER = 0.5 µs
CCR   = 40

T_SCL = 2 × CCR × t_MASTER = 2 × 40 × 0.5 µs = 40 µs
f_SCL = 1 / T_SCL          = 1 / 40 µs       = 25 kHz
```

**"What if you change it."** Show two or three alternative values and what they'd produce. This
is where the understanding actually lands.

**Flag questionable code.** If the firmware's numbers don't match its comments, say so plainly:

> `CCR = 40` produces 25 kHz on a 2 MHz peripheral clock — not the 100 kHz the comment claims.
> That value was calculated for an 8 MHz clock. It still works because the sensor tolerates slow
> SCL, but the comment is wrong.

This is the highest-value thing these sessions produce. Never smooth it over.

**Close with the summary table and the traps** — the two or three mistakes that actually catch
people, stated as facts.

**Offer the next step as a question.** "Shall I do the same for TIM1's PWM registers?"

### What a bad answer looks like

```
❌ "The I2C is configured for standard mode with a 2 MHz clock."
❌ "FREQR is set to 2 for a 2 MHz peripheral clock."     ← restates code, explains nothing
❌ "This gives 25 kHz."                                  ← no derivation
❌ "I've updated I2C.md with the full explanation."      ← never acceptable as the answer
```

Each is *true* and *useless*. The test: could the user have got this by reading the code? If yes,
it isn't an explanation.

---

## Before answering

**Required, non-negotiable:**

1. **The actual source files** in `../MOT/<REPO>/`. Placeholder guides contain wrong filenames
   and invented function names — never answer from a stub or from memory.
2. **The datasheet / reference manual** for any register or silicon behaviour you are not certain
   of. Search the web if needed. A confidently wrong register explanation is worse than no answer.

**Load only if the question needs it:** `Learning-Docs/README.md` (status) · relevant ✅ docs ·
`Architecture/MOT_System_Architecture.md` when the question crosses board boundaries.

Don't make the user wait while ten files are opened.

---

## Two entry modes

**Mode A — question-driven (the default).** A `/learn-firmware <REPO>` with questions attached,
or a bare question with no command. Answer every question fully, in order, in chat. The firmware
gets learned in patches — that's expected, the coverage table tracks it.

**Mode B — full walkthrough.** `/learn-firmware <REPO>` with no questions, or "walk me through
X". Work from the entry point onward — `main()` (STM32/STM8) or `setup()`→`loop()` (ESP32).
Explain in chat in chunks the user can interrupt. See phases at the bottom of this file.

**One thing comes first in both modes:** identify the MCU, and if `Controllers/<MCU>/` is ⬜, say
so up front — it's a prerequisite and the user may want it built before going further.

---

## The documentation sweep

**Runs at a boundary, never in an answering turn** — see *Answer now, write at the boundary*.
Triggers: topic change · a closing-sounding message · "save it" · pending past ~5 files.

Open a task list before writing, so nothing is silently skipped. Sweep everything accumulated
since the last flush, not just the most recent answer:

| Trigger | Action |
|---|---|
| A concept was explained | `Fundamentals/<TOPIC>.md` — create if absent, fill if ⬜, extend if ✅ |
| A protocol was involved | `Protocols/<PROTOCOL>.md` |
| MCU-specific behaviour was learned | `Controllers/<MCU>/` |
| A HAL / library call was explained | `Libraries/HAL.md` |
| Firmware code was walked through | `Firmware/<REPO>/FIRMWARE_GUIDE.md` + coverage table |
| A full execution path was traced | `Firmware/<REPO>/REAL_TIME_EXAMPLES.md` |
| The user asked a good question | Inline Q&A block + `QUESTIONS_INDEX.md` row |
| Any file's status changed | `Learning-Docs/README.md` (⬜ → 🟡 → ✅) |
| A session happened | `README.md` session log — one row |

Close with a compact footer, nothing verbose:

```
📝 Docs: Protocols/I2C.md (ACK/NACK, +Q&A) · Controllers/STM8/ (I2C registers) · QUESTIONS_INDEX.md
```

**Commit `Learning-Docs/` at the end of every session.** Two different agents write to that repo
and it has no intermediate history. A commit is the only restore point.

---

## Size budget

**The budget applies to `Firmware/<REPO>/` — the project-specific layer — at ~4× that
firmware's source line count.** That is the only layer written *for one firmware*, so it is the
only one a per-firmware budget can honestly govern.

```
MOT-ELVH-STM8:   545 source lines  →  ceiling ~2,200 lines in Firmware/MOT-ELVH-STM8/
                                      currently 1,701  ✓ inside
```

**The shared layers are not budgeted per firmware, because they amortise.**
`Fundamentals/Timers/` is written once and serves ELVH, SHT40, CONTROLLER-CARD, SURGEON-DISP and
XRAY. `Controllers/STM8/` serves the three STM8 firmwares. Charging all of it to whichever
firmware happened to trigger it produces a fake number — it read as 9,653 lines for a 545-line
firmware (17.7×), when the honest attribution is:

```
   Firmware/MOT-ELVH-STM8/   1,701  ÷ 1  (only ELVH uses it)      = 1,701
   Controllers/STM8/         2,691  ÷ 3  (ELVH, SHT40, XRAY)      =   897
   Fundamentals (shared)     5,261  ÷ 9  (every firmware)         =   585
                                                          total     3,183   ≈ 5.8× source
```

**What governs the shared layers instead** — these are live limits, not accounting:

- one concept has exactly one home, linked to from everywhere else
- a register is explained once, at the layer where the fact is true
- split a file at ~700 lines, and **splitting is a last resort, never a way to keep writing**

> ⚠️ A budget that is already violated gets ignored, and then so does the next rule. If a number
> here ever reads as unreachable, say so and re-derive it — do not quietly write past it.

Why any of this exists: a previous tool produced 7,260 lines for this one firmware and the user
abandoned it unread. Volume is the failure mode here, not the goal.

---

## Writing rules for docs

1. **Bottom line first.** Every section opens with one bold line stating the conclusion.
2. **If a table can hold it, never write it as prose.** Register fields, options, comparisons,
   timings, failure modes, pin maps.
3. **Diagrams replace paragraphs — they don't accompany them.** If the diagram carries the point,
   delete the sentence that repeats it.
4. **`<details>` for depth.** Long derivations, full listings, edge cases.
5. **Recap box at the top of every file** — the whole file in ~8 lines. This is what gets pasted
   to an LLM for context.
6. **No filler.** No "as we can see", no "it is important to note", no sentence restating the
   heading. No paragraph over 3–4 lines.
7. **Headings stop at H3.** No H4.
8. **Split at ~700 lines** — but see the size budget first. Splitting is a last resort, not a
   way to keep writing.
9. **Never create a supplement file.** No `ADDITIONAL_*`, `DETAILED_*`, `PART_2`. If a doc needs
   a supplement, the doc itself is wrong.

**Tabular data is a table, never an ASCII box.** Reserve ASCII drawing for what is genuinely
spatial or temporal: register bit layouts, waveforms, signal paths, state machines, memory maps.
If you're drawing a box to hold rows of `name → meaning`, that's a table.

**Each register is explained once, at the layer where the fact is true.** Firmware guides link to
the register card and add only what is specific to the call site.

---

## The register card

The standard shape for a register in a `Controllers/` doc. A lookup card, not prose:

````markdown
#### I2C_CR2 — Control Register 2 · `0x5211` · reset `0x00`

┌─────┬───┬───┬───┬────┬─────┬───┬───┐
│  7  │ 6 │ 5 │ 4 │ 3  │  2  │ 1 │ 0 │
├─────┼───┼───┼───┼────┼─────┼───┼───┤
│SWRST│ - │POS│ACK│STOP│START│ - │ - │
└─────┴───┴───┴───┴────┴─────┴───┴───┘

| Bit | Name | 0 | 1 | Cleared by |
|---|---|---|---|---|
| 0 | START | no start | generate START | hardware, after START is sent |
| 2 | ACK | NACK returned | ACK returned after each byte | software |
| 7 | SWRST | normal | peripheral held in reset | software |

⚠ **Constraint:** `PE` in CR1 must be 0 while configuring FREQR/CCR.

**Used in this firmware:**
`i2c_drv.c:41` → `I2C->CR2 |= I2C_CR2_START;`  (start a read transaction)
`i2c_drv.c:58` → `I2C->CR2 &= ~I2C_CR2_ACK;`   (NACK the final byte)
````

Three parts that are easy to skip and shouldn't be:

- **Reset value in the heading** — the baseline for every BEFORE/AFTER diagram.
- **"Cleared by" column** — hardware-cleared vs software-cleared is the most common source of
  real bugs.
- **"Used in this firmware" with `file:line`** — this is what makes the reference doc and the
  firmware guide one system instead of two disconnected documents.

---

## Content distribution — what goes where

| Content | Fundamentals | Controllers | Firmware |
|---|---|---|---|
| "What is a prescaler for?" | ✅ full, generic, no addresses | brief | — |
| "What are TIM4's registers?" | — | ✅ the card: address, reset, bits | — |
| "Why `PSCR = 4` here?" | — | — | ✅ the value and the reason |
| "How does a UART frame work?" | ✅ `Protocols/UART.md` | brief + this chip's regs | reference |
| "What baud rate does MOT use?" | — | ✅ "In MOT: 9600" | ✅ in code |

**Rules:**
1. **Fundamentals** = what it is FOR, generically. No addresses, no bit positions, **no "In MOT"
   notes, ever.**
2. **Controllers** = the register card for this chip + project notes.
3. **Firmware** = which value was written and why THAT value.
4. No duplicated deep content — link between layers instead of repeating.

**Section-level links are mandatory** where a Controllers section touches a Fundamentals concept:

```markdown
> 📖 **Deep dive:** [Prescaler & period](../../Fundamentals/Timers/02-prescaler-and-period.md#21-prescaler-and-auto-reload)
```

---

## Capturing questions

**The user's own questions — inline, where the theory is.** Placed after the section that answers
it, so context is right there on re-read.

```markdown
> ❓ **Asked 2026-08-07:** Why does the master send NACK on the final byte?
>
> **A:** ACK means "send me another". The master must NACK the last byte to tell the slave to
> release SDA, otherwise the slave keeps driving the bus and the STOP condition can't be
> generated.
```

Capture when the question is conceptual, non-obvious, or reveals a gap. Skip pure navigation
("which file is that in?"). Append one row per captured question to `QUESTIONS_INDEX.md`.

**"Why this way?" callouts.** The goal is to never redo R&D. What costs research time is not what
the code does — it's why it was built that way. Capture design decisions where you find them.

**Interview questions — 1–2 per completed topic, at the end of the file.** The user asked for
these. Not per *section* — per topic; per-section is what produced 47 of them in one folder.
Real questions asked for embedded roles, not filler.

Always collapsed, so the file works as a self-test rather than as more reading:

```markdown
<details>
<summary><b>Q:</b> Why does I2C need external pull-up resistors? <code>[Basic]</code></summary>

**A:** I2C pins are open-drain — a device can only pull the line low, never drive it high…
</details>
```

Tag `[Basic]` / `[Deep]`. Collapsing is the point: re-reading a doc feels like learning but
barely works, whereas having to answer before seeing the answer is what reveals whether you
actually know it. Left open, they are just more text on the page.

---

## Coverage tables

Every `FIRMWARE_GUIDE.md` carries one near the top. Question-driven learning fills a firmware in
patches; this is how the user sees what is genuinely covered.

```markdown
## 📊 Coverage

| Area | Status |
|---|---|
| Hardware, pinout, wiring | ✅ |
| `main()` + init sequence | ✅ |
| I2C driver (`i2c_drv.c`) | 🟡 read path only, no error handling |
| PWM output (`pwm_out.c`) | ⬜ |
```

README status: ⬜ → 🟡 once any real content lands → ✅ only when the coverage table is all green.

---

## Folder structure

```
Learning-Docs/
├── README.md              ← roadmap, status tracker, session log
├── QUESTIONS_INDEX.md     ← every question asked, with links
├── Architecture/          ← system-level documentation
├── Firmware/<REPO>/       ← FIRMWARE_GUIDE.md + REAL_TIME_EXAMPLES.md
├── Controllers/           ← MCU references (STM8, STM32, ESP32)
├── Protocols/             ← one file per protocol; complex ones get a subfolder
├── Fundamentals/          ← embedded theory and concepts
├── Libraries/             ← HAL and other library references
└── Chats/                 ← exported chat archive (grep only, never read whole)
```

**Structural changes require approval.** To move, rename, split, or reorganise: write the plan,
show it, wait. The user has links and habits built on the current layout.

---

## Full walkthrough phases (Mode B only)

**Phase 0 — Context.** README roadmap · architecture doc · the firmware's current stub (treat as
untrusted) · relevant ✅ docs · any files the user referenced.

**Phase 1 — Controller doc.** If this is the first firmware for this MCU and `Controllers/<MCU>/`
is ⬜, build it first: architecture, memory map, clock system, peripheral registers, boot, tools.

**Phase 2 — Firmware analysis.** Read every source file. Map the call hierarchy. Identify entry
point, init order, main loop, peripherals, protocols.

**Phase 3 — `FIRMWARE_GUIDE.md`.** Written in execution order from the entry point. Overview and
hardware · file map · **State Flow Diagram** · **Program Flow Diagram** · init sequence function
by function · main loop, **children before parents** · data flow and timing · error handling.

Both flow diagrams are mandatory — they are the fastest way back into a firmware months later.

**Phase 4 — `REAL_TIME_EXAMPLES.md`.** 2–5 complete traces. Simple firmware: power-on + init, one
full loop cycle. FreeRTOS: one command per task, packet arrival → response. Each trace shows
initial register state (address + hex) · step-by-step with BEFORE/AFTER bit diagrams · memory
snapshots at checkpoints · bus waveforms · timing per step · final state.

**Phase 5 — Upward sweep.** The sweep table above, within the size budget.

**Phase 6 — Status.** Coverage table · README status and session log · commit · summary.

---

## Plan before large changes

A full walkthrough, a new controller doc, a file split, a folder restructure — **write the plan
and show it before executing.** A visible task list counts as the plan.

Routine work needs no approval: answering a question, filling a placeholder, extending a section.

---

## End of session

1. `README.md` — status and session log row.
2. `QUESTIONS_INDEX.md` — confirm every captured question has a row.
3. **Commit `Learning-Docs/`.**
4. If the user gave feedback about *how you should work* — a format that landed, one that didn't
   — update this file. But **prefer editing the reference example over adding a rule.** This file
   got to 555 lines once by accretion, and that was the problem it was trying to solve.
5. Present a summary of files created and updated.
