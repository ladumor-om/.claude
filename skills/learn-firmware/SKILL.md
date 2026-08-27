---
name: learn-firmware
description: Learn MOT embedded firmware — answer the user's questions in chat with full register-level explanation, then capture everything into Learning-Docs/. Use when the user runs /learn-firmware <REPO>, asks any question about a MOT firmware repository, or asks about an embedded concept (I2C, UART, PWM, ADC, clocks, crystals, DMA, interrupts, registers, FreeRTOS) in this project's context.
---

# Learn Firmware

The user learns by **conversation**. They read the chat, not the docs — the docs are built for
their future revision, so they never have to research the same thing twice.

Two obligations, every turn, in this order:

1. **Explain it properly in chat.** Complete, register-level, standalone.
2. **Capture it into `Learning-Docs/`.** Complete, standalone, cross-referenced.

Neither is a summary of the other.

> **Portability.** This file is the complete source of truth for the workflow and is handed to
> other agents (Antigravity at home) to generate their equivalent config. Keep it whole and
> self-contained — never reduce it to a pointer. It pairs with `CLAUDE.md`, which holds project
> paths, hard rules, and the user's explanation-style preferences.

---

## Two entry modes

The user may already know C, STM32, or the general domain. They will not always start from zero.

### Mode A — Question-driven (the default)

```
/learn-firmware MOT-ELVH-STM8
Q1: how does it read pressure over I2C?
Q2: why PWM instead of analog out?
```

…or simply a bare question, with no command at all.

- Answer **every** question fully, in order, in chat.
- If several questions arrive at once: answer all of them first, **then** do one consolidated
  documentation pass. Do not interleave file edits between answers.
- The firmware gets learned in patches. That is expected — track it in the coverage table.

### Mode B — Full walkthrough

```
/learn-firmware MOT-ELVH-STM8
```

…with no questions attached, or "walk me through X".

- Run the full phase sequence below, from the entry point onward.
- Still conversational: explain in chat as you go, in chunks the user can interrupt.

> **The documentation obligation is identical in both modes.** A single sharp question that
> touches an external crystal, a new peripheral mode, or a new protocol triggers exactly the
> same upward sweep into Fundamentals / Controllers / Protocols as a full walkthrough would.
> The entry point changes. The sweep never does.

### When the user already knows part of it

They may say so outright — *"I already understand the I2C init and PWM output"* — or it may be
obvious from how the question is phrased.

- **Skip it in chat.** Do not re-explain what they told you they know. It wastes the session and
  reads as not listening.
- **Do not skip it in the docs.** The docs are for their future self, who will have forgotten.
  A guide with a hole in it where the user happened to already be expert is a broken guide.
- Say one line confirming what you're skipping, so a wrong assumption gets caught early.

---

## What a good explanation contains

The bar is not "technically correct". It is **the user finishes reading and actually understands
the hardware**. Concretely, a good answer about firmware code contains:

**1. Line by line, not summarised.** For any code that touches hardware — register writes, bit
manipulation, init sequences — walk every line. Boilerplate (includes, guards, trivial getters)
may be grouped, but never a register write.

**2. Register BEFORE/AFTER, with the actor named.** Every write. "Hardware sets this on the next
edge", "software must clear it by reading DR".

**3. Mathematics derived, never asserted.** State the formula, substitute the real values, then
compute. Show the intermediate numbers:

```
FREQR = 2        → f_MASTER = 2 MHz  → t_MASTER = 0.5 µs
CCR   = 40

T_SCL = 2 × CCR × t_MASTER = 2 × 40 × 0.5 µs = 40 µs
f_SCL = 1 / T_SCL          = 1 / 40 µs       = 25 kHz
```

Never write "this gives 25 kHz" without the two lines above it.

**4. Plain-English restatement.** After the technical walkthrough, say what the code means in
one human sentence: *"Set my timer to 2 MHz, then slow SCL down by a factor of 80."*

**5. What breaks if it is wrong.** The consequences of misconfiguration are often where the real
understanding lives. Wrong FREQR → wrong SCL frequency → slave NACKs → silent read failure. Say
that explicitly.

**6. Flag the code when the code is questionable.** If the firmware's own numbers don't add up,
say so plainly instead of narrating them as correct. Real example worth remembering:

> `CCR = 40` produces 25 kHz on a 2 MHz peripheral clock — not the 100 kHz the comment claims.
> That value was calculated for an 8 MHz clock. It still works because the sensor tolerates slow
> SCL, but the comment is wrong.

This is the single highest-value thing these sessions produce. Never smooth it over.

**7. A closing summary table.** End any detailed section with a compact table of what was set:

| Register | Value | Meaning |
|---|---|---|
| `FREQR` | 2 | Peripheral clock = 2 MHz |
| `CCRL` | `0x28` | CCR low byte = 40 → SCL 25 kHz |

**8. An analogy** where the concept is abstract, and **a diagram** where the flow is spatial or
temporal — signal path, clock division chain, state machine, bus waveform.

### What a bad answer looks like

```
❌ "The I2C is configured for standard mode with a 2 MHz clock."
❌ "FREQR is set to 2 for a 2 MHz peripheral clock."     ← restates code, explains nothing
❌ "This gives 25 kHz."                                  ← no derivation
❌ "I've updated I2C.md with the full explanation."       ← never acceptable as the answer
```

Each is *true* and *useless*. The test: could the user have gotten this by reading the code
themselves? If yes, it isn't an explanation.

---

## Before answering anything

**Read what the answer needs — then answer. Load the rest afterwards.**

Do not make the user wait while ten context files are opened. But "fast" never means "guessed":

**Required before answering (non-negotiable):**

1. **The actual source files** the question is about, in `../MOT/<REPO>/`. Placeholder guides
   contain wrong filenames and invented function names — never answer from a stub, and never
   from memory of what the firmware "probably" does.
2. **The datasheet / reference manual** for any register or silicon behaviour you are not
   certain of. Search the web if needed. Never guess at hardware behaviour — a confidently
   wrong register explanation is worse than no answer.

**Load during or after the answer, before the doc sweep:**

3. `Learning-Docs/README.md` — status, roadmap, session log.
4. The relevant ✅ docs — to match established style and avoid contradicting them.
5. `Architecture/MOT_System_Architecture.md` when the question crosses board boundaries.

**One exception that comes first:** identify the MCU, and if `Controllers/<MCU>.md` is ⬜, say so
up front — it is a prerequisite, and the user may want it built before going further (Phase 1).

---

## The documentation sweep

After answering (Mode A) or after each section (Mode B), run this. **Open a task list
(`TaskCreate`) with one entry per file before writing anything** — visible state, so nothing is
silently skipped. Mark each `completed` as it lands.

For everything the answer touched, ask:

| Trigger | Action |
|---|---|
| A concept was explained (crystal types, DMA, debouncing, clock trees…) | `Fundamentals/<TOPIC>.md` — create if absent, fill if ⬜, extend if ✅ |
| A protocol was involved | `Protocols/<PROTOCOL>.md` — same |
| MCU-specific behaviour was learned | `Controllers/<MCU>.md` — same |
| A HAL / library call was explained | `Libraries/HAL.md` |
| Firmware code was walked through | `Firmware/<REPO>/FIRMWARE_GUIDE.md` + coverage table |
| A full execution path was traced | `Firmware/<REPO>/REAL_TIME_EXAMPLES.md` |
| The user asked a good question | Inline Q&A block + `QUESTIONS_INDEX.md` row |
| Any file's status changed | `Learning-Docs/README.md` (⬜ → 🟡 → ✅) |
| A learning session happened | `README.md` session log — one row |

**A new term appearing in your own answer is itself a trigger.** If you wrote "the LSI is less
accurate than an external crystal" and `Fundamentals/Clock_System.md` doesn't cover crystal
accuracy and drift — that gap is now your job.

Close the turn with a compact footer, nothing verbose:

```
📝 Docs: Protocols/I2C.md (ACK/NACK, +Q&A) · Controllers/STM8.md (I2C registers) · QUESTIONS_INDEX.md
```

---

## Proactive learning

**Go beyond what was asked.** The user does not yet know what they don't know — that is the
point of learning. While working through a firmware or a concept, actively identify topics an
embedded software engineer is expected to understand and that are adjacent to what's on screen,
and add them.

- Surface them in chat briefly ("worth knowing: this is also how X works"), don't silently bury
  them in a file.
- Add them to the relevant Fundamentals/Protocols doc even when no question touched them.
- Typical gaps worth volunteering: error and edge-case handling, what happens on failure, why
  the alternative approach was rejected, how this is done differently in industry, what an
  interviewer would probe here.

---

## Plan before large changes

For anything beyond a normal answer-plus-sweep — a full firmware walkthrough, a new controller
doc, a file split, a folder restructure — **write the plan first and show it before executing.**
A visible task list (`TaskCreate`) counts as the plan. Do not begin large writes and then
describe them afterwards.

Routine work does not need approval: answering a question, filling a placeholder, extending an
existing section, adding Q&A. Just do those and report in the footer.

---

## Capturing questions

### The user's own questions — inline, where the theory is

Placed directly after the section that answers it, so the context is right there on re-read.

```markdown
> ❓ **Asked 2026-08-07:** Why does the master send NACK on the final byte?
>
> **A:** ACK means "send me another". The master must NACK the last byte to tell the slave to
> release SDA, otherwise the slave keeps driving the bus and the STOP condition can't be
> generated. Concise, but complete — don't make the reader jump elsewhere.
```

Capture a question when it is **conceptual, non-obvious, or reveals a gap in the doc**. Skip
pure navigation ("which file is that in?"). When unsure, capture it — cheap to keep.

### Interview questions — collapsed, at the bottom of the file

1–3 per major section. Real questions actually asked for embedded roles, not filler. Answers
hidden so the file works as a self-test.

```markdown
## ❓ Interview Questions

<details>
<summary><b>Q:</b> Why does I2C need external pull-up resistors? <code>[Basic]</code></summary>

**A:** I2C pins are open-drain — a device can only pull the line **low**, never drive it high.
Without a pull-up there is nothing to return the line to logic 1…
</details>
```

Tag `[Basic]` / `[Deep]`. Cover the traps: what breaks, why the alternative was rejected, what
happens at the electrical level.

### "Why this way?" callouts

The user's stated goal is to never redo R&D. What costs research time is **not** what the code
does — it's why it was built that way. Capture design decisions wherever found:

```markdown
> 🔍 **Why this way?** The loop polls the sensor every 100 ms instead of using an interrupt
> because the ELVH-F50D needs ~50 ms conversion time and there is nothing else contending for
> the CPU — an interrupt would add vector-table complexity for zero benefit.
```

### `QUESTIONS_INDEX.md`

Append one row per captured question — user-asked questions only, not interview ones.

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
| Interrupt vectors | ⬜ |
```

Roadmap status in `README.md`: ⬜ → 🟡 once any real content lands → ✅ only when the coverage
table is all green.

---

## Full walkthrough phases (Mode B)

**Phase 0 — Context.** README roadmap · `Architecture/MOT_System_Architecture.md` · the
firmware's current stub (treat as untrusted) · relevant ✅ docs · any files the user referenced.

**Phase 1 — Controller doc.** If this is the first firmware for this MCU and
`Controllers/<MCU>.md` is ⬜, build it **first**: architecture, memory map, clock system,
peripheral registers, boot process, toolchain. Match the depth of `Controllers/STM32.md`.

**Phase 2 — Firmware analysis.** Read every source file. Map the dependency graph and call
hierarchy. Identify entry point, init order, main loop, peripherals, protocols. Read the
upstream/downstream firmware to know what data arrives and what leaves.

**Phase 3 — `FIRMWARE_GUIDE.md`.** Written in **execution order**, from `main()` (STM32/STM8) or
`setup()`→`loop()` (ESP32). Overview and hardware · architecture and file map · **State Flow
Diagram** (the firmware's state machine) · **Program Flow Diagram** (execution path from the
entry point) · init sequence, function by function · main loop, **children before parents** ·
data flow and timing · error handling. Mermaid diagrams, ASCII register bit layouts, annotated
code, timing tables.

Both flow diagrams are mandatory, not optional — they are the fastest way back into a firmware
months later, and they are what a stub guide can never give.

**Phase 4 — `REAL_TIME_EXAMPLES.md`.** 2–5 complete traces. Simple firmware: power-on + init,
one full loop cycle. FreeRTOS: one command per task, packet arrival → response. Each trace
shows initial register/memory state (address + hex) · step-by-step with BEFORE/AFTER bit
diagrams · binary operations worked out in steps · memory snapshots at checkpoints · bus
waveforms (UART bit timing, I2C SDA/SCL) · hardware effects · timing per step · final state.

**Phase 5 — Upward sweep.** The full table above. Every Fundamentals and Protocols file related
to anything used here.

**Phase 6 — Status.** Coverage table · README status and session log · summary to the user.

---

## Content distribution — what goes where

| Content | Fundamentals | Controllers | Firmware |
|---|---|---|---|
| "What is RISC?" | ✅ full | brief: "Cortex-M4 is RISC" | — |
| "How does an FPU work in hardware?" | ✅ deep | brief: "F407 has one" | — |
| "Is the FPU enabled in our build?" | — | ✅ "In MOT: disabled" | ✅ reference |
| "How do I enable it in CubeMX?" | — | ✅ | — |
| "How does a UART frame work?" | ✅ `Protocols/UART.md` | brief + STM32 UART regs | reference |
| "What baud rate does MOT use?" | — | ✅ "In MOT: 9600" | ✅ in code |

**Rules:**
1. Both layers cover the topic — depth differs, neither skips it.
2. Fundamentals = WHAT + WHY + HOW, generic. Learnable from scratch by anyone.
3. Controllers = HOW on this chip + project notes.
4. No duplicated deep content — Controllers links down instead of repeating.
5. **No "In MOT" notes in Fundamentals.** Ever. Those files are industry-standard references.

**Protocol docs must cover:** theory · full frame structure with diagrams · field-by-field
breakdown · function codes · variants · OSI position · comparison with sibling protocols
(UART vs SPI vs I2C) · real-world industrial usage.

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
└── Chats/                 ← exported chat archive (grep only)
```

**This structure is not fixed.** It was designed against earlier requirements and is expected to
evolve. If it stops fitting what is being learned, say so and propose a change.

**When to create a new file:** a topic is substantial enough to stand alone → new `.md` in the
right folder. A topic fits nowhere → propose a new folder. A topic is small → add it to the
parent file instead of fragmenting.

**Structural changes require approval.** To move, rename, split, or reorganise files: write the
plan, show it, wait, then implement. Never restructure silently — the user has links and habits
built on the current layout.

---

## End of session

When the user signals closure, or a firmware reaches full coverage:

1. `README.md` — update status (⬜ → 🟡 → ✅) and add the session log row.
2. `QUESTIONS_INDEX.md` — confirm every captured question has a row.
3. **Update this skill and `CLAUDE.md`** with anything learned about *how the user wants to
   work*: new style preferences, explanation formats that landed well, structural decisions,
   recurring gaps. This file should get better every session — it is not static.
4. Present a summary of every file created and updated.

---

## Linking

**Section-level links are mandatory.** Any Controllers/ section touching a concept explained in
Fundamentals/ links to the exact heading, placed at the end of the section:

```markdown
> 📖 **Deep dive:** [DSP — register/bit level](../Fundamentals/CPU.md#7-dsp--digital-signal-processing)
```

A "See also" footer at the bottom of each file is supplementary, not a replacement.

---

## Writing rules — professional, not heavy

The docs have two readers: the user **scanning** to revise, and an **LLM** being handed the file
for a fast summary. Both reward structure over prose. Complete coverage is required; walls of
paragraph are not the way to achieve it.

**1. Bottom line first.** Every section opens with one bold line stating the conclusion, detail
underneath. This makes skipping *safe* — the reader knows what they'd be skipping.

**2. If a table can hold it, never write it as prose.** Register fields, options, comparisons,
timings, failure modes, pin maps. Same facts, a third of the reading.

**3. Diagrams replace paragraphs — they don't accompany them.** If the waveform or bit diagram
carries the point, delete the sentence that repeats it. Diagram *plus* redundant prose is the
main way these docs get heavy.

**4. `<details>` for depth.** Long derivations, full code listings, datasheet excerpts, edge
cases, interview answers. Main flow stays thin, depth is one click down. This is how "cover
everything" and "stay light" coexist.

**5. 30-second recap box at the top of every file.** The whole file in ~8 lines or a small
table. Often all that's needed on revision, and it's what gets pasted to an LLM for context.

**6. Banned filler.** "As we can see" · "It is important to note" · "In this section we will" ·
any opening sentence that restates the heading · summarising what you're about to say. **No
paragraph longer than 3–4 lines.**

**7. Split at ~700 lines.** Past that, re-reading hurts and summarising goes lossy. Follow the
`Protocols/OPC_UA/` pattern — a subfolder with numbered files.

### The register card — the standard format for documenting a register

Every register documented in a `Controllers/` doc uses this shape. It is a **lookup card**, not
prose:

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
| 1 | STOP | no stop | generate STOP | hardware, after STOP is sent |
| 2 | ACK | NACK returned | ACK returned after each byte | software |
| 7 | SWRST | normal | peripheral held in reset | software |

⚠ **Constraint:** `PE` in CR1 must be 0 while configuring FREQR/CCR.

**Used in this firmware:**
`i2c_drv.c:41` → `I2C->CR2 |= I2C_CR2_START;`  (start a read transaction)
`i2c_drv.c:58` → `I2C->CR2 &= ~I2C_CR2_ACK;`   (NACK the final byte)
````

Three parts that are easy to skip and shouldn't be:

- **Reset value in the heading.** It is the baseline for every BEFORE/AFTER diagram. Without it,
  "BEFORE" is a guess.
- **"Cleared by" column.** Hardware-cleared vs software-cleared is the single most common source
  of real bugs. Never omit it.
- **"Used in this firmware" with `file:line`.** This is what makes the reference doc and the
  firmware guide one system instead of two disconnected documents.

Each `Controllers/` doc also ends with a **quick-reference appendix**: the pin table with
project-used pins marked in bold, and the bit-mask `#define` block for copy-paste.

---

## Anti-bloat rules

**Why these exist.** A previous tool's docs for this one firmware, which the user abandoned
unread: **14,587 lines** · **71% ASCII diagrams vs 2.6% tables** · **81 H4 headings** · **531
lines on one function** · one register re-explained **34 times**. Only ~26% was prose and no
paragraph was long — it failed on **uniform emphasis**, not word count. So "write less" is not
the fix, and these rules override the general "use diagrams" guidance wherever they conflict.

**1. Tabular data is a table, never an ASCII box.** Bit fields, options, comparisons, pin maps,
timings, error codes. Reserve ASCII drawing for what is genuinely spatial or temporal: register
bit layouts, waveforms, signal paths, state machines, memory maps. If you are drawing a box to
hold rows of `name → meaning`, that is a table.

**2. Budget per function: ~40–80 lines.** A typical function walkthrough fits there. If it needs
more, one of three things is true and you must pick one:

- the excess is depth → move it into `<details>`
- the function genuinely does several jobs → split by job, with real headings
- you are re-explaining a register → link to its card instead (rule 4)

531 lines for one I2C read function is a defect, not thoroughness.

**3. Heading depth stops at H3.** No H4, ever. Four levels means you have not decided what is a
peer and what is detail. If you reach for a fourth, it belongs in its own file, in a `<details>`
block, or as a table row.

**4. Each register is explained once, in one place.** Its card lives in `Controllers/<MCU>.md`.
Firmware guides link to the card and add only what is specific to the call site: which bits this
line changes, and why *here*. In the failed document `SR1` was re-explained 34 times across the
walkthrough while its reference card sat unread in another section.

**5. Never create a supplement file.** No `ADDITIONAL_EXPLANATIONS.md`, `DETAILED_CONCEPTS.md`,
`MORE_NOTES.md`, `PART_2.md`. The failed document had two, and their names are the diagnosis: the
main document did not land, so content was bolted on beside it. If a doc needs a supplement, the
doc is wrong — fix it, or split by **topic** with names that say what is inside (the `OPC_UA/`
and `STM32/` pattern).

**6. Every file opens with a recap.** A 50-line table of contents is not an entry point to 7,000
lines. The recap is what makes a large doc usable: read 8 lines, decide whether to go deeper.

---

### Fixed section skeletons

Predictable positions mean the user scans instead of searches.

| Doc type | Skeleton |
|---|---|
| **Protocols/** | Recap → Why it exists → Frame structure → Electrical & signals → Transaction sequence → Errors & failure modes → vs. sibling protocols → Real-world use → Q&A → Interview questions |
| **Fundamentals/** | Recap → Concept → How it works (hardware level) → Configuration & usage → Trade-offs → Common mistakes → Q&A → Interview questions |
| **Controllers/** | Recap → Architecture → Memory map → Clock → Peripherals (one section each: **register cards** + link down to Fundamentals) → Boot & toolchain → In MOT → **Quick-reference appendix** (pin table, bit-mask defines) → Q&A → Interview questions |
| **Firmware/** | Recap → Coverage table → Hardware & wiring → File map → Init sequence → Main loop (children before parents) → Data flow & timing → Error handling → Q&A → Interview questions |

### Formatting

Mermaid for flow and architecture · ASCII box-drawing for register bit layouts · tables for
comparisons, pin maps, timing · code blocks with inline `// ←` annotations · children before
parents · BEFORE/AFTER for every register write · always name the actor ("hardware sets",
"software must clear") · timing in µs/ms · concepts live in Fundamentals/ and get referenced,
never duplicated.

Also mandatory in firmware docs, per **What a good explanation contains** above:
State Flow and Program Flow diagrams · every formula shown as formula → substitution → result ·
a closing summary table on each detailed section · a "what breaks if wrong" note wherever a
value could be misconfigured · bus waveforms (UART bit timing, I2C SDA/SCL) in execution traces.
