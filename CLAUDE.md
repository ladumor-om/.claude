# MOT Firmware Learning — Project Context

This repository is a **learning knowledge base**, not a software project.

**Purpose:** the user learns the MOT embedded firmware by asking questions in chat. Every
answer is captured into `Learning-Docs/` so it can be revised later — the goal is to never
repeat the same research twice.

**The user reads the chat, not the docs.** Docs are written for their future self. This means
the chat explanation must be complete and standalone on its own, and the docs must be complete
and standalone on their own. Neither one is a summary of the other.

---

## Paths

**Working directory:** `C:\Users\oml\Documents\BitBucket\Docs`

| What | Where |
|---|---|
| All learning documentation | `Learning-Docs/` |
| Roadmap, status, session log | `Learning-Docs/README.md` |
| Index of every question asked | `Learning-Docs/QUESTIONS_INDEX.md` |
| The learning workflow | `.claude/skills/learn-firmware/SKILL.md` |
| Firmware source — **READ ONLY** | `../MOT/<REPO>/` e.g. `../MOT/MOT-ELVH-STM8/` |
| Old exported chats (reference) | `Learning-Docs/Chats/` |

> ⚠️ `Learning-Docs/` is a **git repository containing documentation only**
> (`github.com/ladumor-om/Learning-Docs.git`). Never place agent config, skills, or workflow
> files inside it. Those live at this level, outside the docs repo.

> ⚠️ `../MOT/Docs/` is **unrelated** to this knowledge base — Jetson setup notes, PDFs,
> deployment scripts. Never write there, never treat it as documentation.

> ⚠️ **Never read firmware from `C:\Users\oml\Documents\GitHub\MOT\`.** That is a stale, older
> variant with different filenames (e.g. `MOT-ELVH-STM8` there is a flat 4-file version;
> the real one is a 14-file layered refactor). Its separate git history means it is not a copy.
> A large `GUIDENS/` folder sits beside it — obsolete, and not a source of truth for anything.
> The only firmware source is `../MOT/<REPO>/`.

> ⚠️ `Learning-Docs/Chats/` contains ~189,000 lines across 5 files (one is 162k lines alone).
> **Grep these, never read them whole.** They are a searchable archive of prior sessions.

## Two IDEs

The user also works from **Antigravity** at home, using its own generated agent/skill config.
`.claude/skills/learn-firmware/SKILL.md` is the **complete, self-contained source of truth** for
the workflow — it is what gets handed to Antigravity to generate its equivalent. Keep it whole
and standalone; never reduce it to a pointer to another file.

### `.agents/` is a reference, not a rulebook

`.agents/` (`skills/firmware-learner/`, `workflows/`, `brain-archive/`) is Antigravity's own
repo. **The user's framing:** it exists so a new agent does not start from zero — it records
work already done and formats the user found easy to understand. **It is not a specification,
and its rules do not all have to be followed.**

- **Read it for what the user has already found useful.** Where it and this file disagree,
  **this file wins.**
- **Never hand-edit it.** Editing it here creates a second hand-maintained copy that drifts.
  Write only to `.claude/` and `CLAUDE.md`.
- Its `references/path_map.md` describes the **home machine's** layout (`MIPL/BitBucket/MOT/`),
  not this one. Paths in this file are authoritative here.
- Its rule list is **pre-trim**: it still contains "proactive learning", "completeness over
  brevity" and mandatory-everything styling. Those were deliberately dropped or bounded here
  after they caused real problems. Do not reintroduce them from that file.

---

## The MOT system

**MOT = Modular Operation Theatre** — a multi-controller system for hospital operating theatres:
lights, curtain, medical gas, climate, RGB ambient lighting, and a surgeon display.

| Board | MCU | Role | Link to Jetson |
|---|---|---|---|
| Central hub | Jetson Nano (ARM/Linux) | Python/Flask backend + HTML/JS frontend | — |
| Main PCB | STM32F407VGTx | FreeRTOS — lights, curtain, gas, climate | UART |
| Pressure sensor board | STM8S003F3P + ELVH-L01D | Differential pressure (±1 inH₂O) over I2C → PWM out | via STM32 ADC |
| Temp/humidity board | STM8S003F3P + SHT40 | Temp + humidity over I2C → PWM out | via STM32 ADC |
| Surgeon display | STM32 + LCD | Shows OT parameters | USB → RS-485 |
| RGB Modbus slave | ESP32 | RGB LEDs over Modbus | USB → RS-485 |
| RGB TCP slave | ESP32 | RGB LEDs over TCP | WiFi |
| RGB DMX slave | ESP32 | RGB LEDs over DMX | DMX |
| Bootloader | STM32F407VGTx | OTA update — learning/testing only | UART5 |

**The signal path worth remembering** — the STM8 boards have no digital link to the STM32:

```
Sensor ──I2C──► STM8 ──PWM──► RC low-pass ──0-10 V──► divider ──0-3.3 V──► STM32 ADC ──► 0..4095
```

> 📖 Full wiring, converters and data flow: [Architecture/MOT_System_Architecture.md](Learning-Docs/Architecture/MOT_System_Architecture.md)

**Out of scope — the user already knows these, do not document them:**
`MOT-PROCESSOR-CARD` · `MOT-Desktop-App` · `Udaipur Showroom` · `deployment` · `test_scripts`

---

## Hard rules

1. **Never modify firmware source.** Everything under `../MOT/` is read-only. All documentation
   writes go inside `Learning-Docs/`.
2. **Answer in chat. Write files at the boundary.** The chat response must contain the actual
   explanation, and no files get written in an answering turn. End each answer with a one-line
   `📋 Pending:` list of what is owed, then flush it automatically when the user changes topic,
   says something that reads like closing, says "save it", or the list passes ~5 files.
   "I've updated the docs" is never an answer.
3. **Answer only what was asked.** No adjacent topics, other vendors, other chip families, or
   industry comparisons unless requested. Worth-knowing extras get offered in one line, then you
   stop and let the user choose.
4. **Verify against source.** Never answer a firmware question from assumption or from what a
   placeholder doc claims. Open the real `.c`/`.h` files. Existing stub docs were written from
   guesses and contain known errors.
5. **Never link to a placeholder.** If a doc needs to reference `I2C.md` and `I2C.md` is still a
   stub, fill it in first.
6. **Child before parent.** Explain callees before callers, always.
7. **Size budget: `Firmware/<REPO>/` stays under ~4× that firmware's source line count.** The
   shared layers (`Fundamentals/`, `Protocols/`, `Controllers/`) amortise across every firmware
   and are governed instead by one-home-per-concept and the ~700-line split. Splitting is never
   a way around a budget. Full accounting in the skill.

---

## When any firmware or embedded question is asked

Load and follow **`.claude/skills/learn-firmware/SKILL.md`** in full. This applies whether or
not the user typed `/learn-firmware` — a bare question about I2C, a register, a clock tree, or
any MOT firmware triggers the same workflow.

---

## Explanation style

**The reference example is the style guide:**
[`.claude/skills/learn-firmware/reference/good-answer.md`](.claude/skills/learn-firmware/reference/good-answer.md)

It is a real answer from a real session that the user confirmed was right. Match its depth and
its shape. It beats any list of rules, because it shows what "good" looks like instead of
describing it.

The few things worth stating separately, because they are user-specific:

- **Register level, not API level.** What bits change, not which function was called.
- **Derive the maths, never assert it.** Formula → substitute real values → result, on separate
  lines. "This gives 25 kHz" with no working shown is not acceptable.
- **State WHO acts:** "Hardware sets this flag", "Software must clear it by reading DR".
- **Answer the "why", not only the "what".** The user asks conceptual questions
  ("Is NVIC hardware or firmware?"). Address the underlying concept, don't deflect.
- **Flag questionable code.** If the firmware's numbers or comments don't add up, say so plainly
  rather than narrating them as correct. This is the highest-value output of a session.
- **Skip what the user says they already know** in chat — but never in the docs.
- **Memory addresses and hex values** — show where a value physically lives, not just its name.
- **Theory first, then the code that uses it.** Concept, then the actual line from the firmware.
- **Timing in µs/ms** wherever it can be stated.
- **Say what breaks if it's wrong.** Not the same as "what if you change it" — that one explores
  alternatives, this one names the failure. Both are useful; the failure mode teaches more.

Analogies land well. Diagrams land well. Neither is mandatory on every answer.

---

## The three documentation layers

```
Fundamentals/ · Protocols/ · Libraries/        Controllers/STM8/ · STM32/ · ESP32.md
        what a thing is FOR, generically              this chip's register cards
                    \                                       /
                     \                                     /
                      Firmware/<REPO>/  — which value was written, and why THAT value
```

| Layer | Content | "In MOT" notes |
|---|---|---|
| **Fundamentals / Protocols / Libraries** | What the thing is for, generically. No addresses, no bit positions. Someone outside this company could learn from it. | ❌ Never |
| **Controllers** | The register card for this chip — address, reset value, bit layout, encodings — plus project notes and a link down to Fundamentals. | ✅ Yes |
| **Firmware** | Code walkthroughs and execution traces. Links to the card; never re-explains it. | ✅ Yes |

Knowledge flows **both ways**: theory → MCU → code when learning, and firmware → Fundamentals
when the code reveals a concept the generic docs don't cover yet.

Full content-distribution and linking rules live in the skill.

---

## Status legend

Used in `Learning-Docs/README.md` and in every guide's coverage table.

| Mark | Meaning |
|---|---|
| ✅ | Complete — verified against source, cross-referenced |
| 🟡 | Partial — some sections done, coverage table shows what |
| ⬜ | Placeholder / not started — **do not trust its contents** |
