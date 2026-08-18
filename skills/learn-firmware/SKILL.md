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

---

## Before answering anything

1. Read `Learning-Docs/README.md` — status, roadmap, session log.
2. Identify the MCU. If `Learning-Docs/Controllers/<MCU>.md` is ⬜, that is a **prerequisite**:
   say so, and offer to build it first (see Phase 1).
3. **Read the actual source** in `../MOT/<REPO>/`. List the real files — placeholder guides
   contain wrong filenames and invented function names. Never answer from a stub.
4. Read the relevant ✅ docs to match established style and avoid contradicting them.
5. For deep questions, verify against the datasheet / reference manual. Search the web if the
   register behaviour is not certain. Never guess at silicon behaviour.

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
`setup()`→`loop()` (ESP32). Overview and hardware · architecture and file map · init sequence,
function by function · main loop, **children before parents** · data flow and timing · error
handling. Mermaid diagrams, ASCII register bit layouts, annotated code, timing tables.

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

### Fixed section skeletons

Predictable positions mean the user scans instead of searches.

| Doc type | Skeleton |
|---|---|
| **Protocols/** | Recap → Why it exists → Frame structure → Electrical & signals → Transaction sequence → Errors & failure modes → vs. sibling protocols → Real-world use → Q&A → Interview questions |
| **Fundamentals/** | Recap → Concept → How it works (hardware level) → Configuration & usage → Trade-offs → Common mistakes → Q&A → Interview questions |
| **Controllers/** | Recap → Architecture → Memory map → Clock → Peripherals (one section each, each linking down to Fundamentals) → Boot & toolchain → In MOT → Q&A → Interview questions |
| **Firmware/** | Recap → Coverage table → Hardware & wiring → File map → Init sequence → Main loop (children before parents) → Data flow & timing → Error handling → Q&A → Interview questions |

### Formatting

Mermaid for flow and architecture · ASCII box-drawing for register bit layouts · tables for
comparisons, pin maps, timing · code blocks with inline `// ←` annotations · children before
parents · BEFORE/AFTER for every register write · always name the actor ("hardware sets",
"software must clear") · timing in µs/ms · concepts live in Fundamentals/ and get referenced,
never duplicated.
