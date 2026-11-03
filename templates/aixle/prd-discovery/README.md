# PRD Discovery

Turn whatever material a team has about a product into a PRD you can build from.
Two interactive workflows, one per depth:

- **PRD Fast (grill-me)**: about 30-60 minutes. The agent reads your material,
  grills you on every open decision, then writes a PRD made of user stories and
  breaks it into vertical-slice issues.
- **PRD Full Discovery**: about 1-3 hours, depending on the optional steps. A full
  discovery built on the BMad Method: optional brainstorming, pressure-test and
  research (market, competitive, domain, technical, user voice, academic), then a
  product brief, PRFAQ, PRD, PRD validation, UX, architecture, epics and stories,
  and a readiness check.

Both are project-agnostic: everything about your product comes from the files you
select and from your answers.

## What you get

**PRD Fast (grill-me)**

| File | What it is |
|---|---|
| `source-digest.md` | Every source, the facts it states, contradictions, what could not be read |
| `track.md` | Run passport: project type, document language, Figma on or off, product folder, decision owner |
| `design-tree.md` | Every decision, settled (with its source) or open |
| `grill-log.md` | Each question, the answer, and who decided it (you, or your acceptance of the agent's recommendation) |
| `open-questions.md` | Open decisions and evidence gaps, each with who can close it |
| `prd.md` | Problem, solution, numbered user stories (each citing the decision it comes from), implementation and testing decisions, out of scope, open items |
| `issues.md` | Vertical-slice issues in dependency order, with acceptance criteria and a story coverage table |

**PRD Full Discovery**

`source-digest.md`, `track.md`, the research reports you chose, `product-brief.md`,
`brief-decisions.md`, `prfaq.md`, `prd.md` with globally numbered FRs (each with a
source and a confirmation status), `prd-addendum.md`, `prd-decisions.md`,
`prd-validation.md`, `questions-register.md`, `DESIGN.md` / `EXPERIENCE.md`,
`architecture.md`, `epics.md`, `readiness-report.md`. You only get the optional
documents for the steps you keep.

## How to run it

1. **Upload your material first.** Put documents, PDFs, spreadsheets and code archives
   into the project's **Assets**. They cannot be added once a run has started.
   During a run you can only paste text and paste or drop images into the terminal.
2. **Start the run from the Workflows page.** Open the workflow, press **Run workflow**,
   and choose **Interactive** as the execution mode. The other modes cannot run these
   workflows, because every step talks to you. Select your files under
   **Input assets**.
3. **Talk to the agent.** Each step opens silently, so type anything (for example
   "go") to start it. When the agent waits for you, it ends its message with
   **👉 Your turn**.
4. **End each step with Finish session.** When a step is done, the agent says
   "✅ Step done". Press **Finish session** in the bar at the top of the run page.
   The next step then starts in a new session with every file from the previous
   steps. There is no Approve button.
5. **Pick a product folder.** The first step asks for a short folder name, for
   example `calm-day`. Results are saved under that folder in the project's
   Assets, so files of different products do not mix.

### Decisions: what "yes" means

The agent recommends an answer to every question, so you can move fast by
replying "yes". An accepted recommendation is logged as exactly that: an accepted
recommendation.

High-impact items are re-asked as a choice: **A** (I decide this now) or **B**
(open for the named owner). High-impact means pricing, legal, safety or
compliance, scope changes, reversing an earlier decision, and anything the decision
owner or another named person owns. Reply with letters, for example "Q2 A, Q7 B".
Items answered B stay open, with their owner, in the open questions or the question
register.

Step 00 lists the **sensitive topics** in your material (health, distress or crisis,
safety, legal, privacy, compliance) and every item of any open-items list, even when
the material decides nothing about them. Each of them is asked as an A/B choice
later, so none can drop out of the PRD silently. In PRD Full Discovery, what you
decided or left open in the brief is carried into the PRD without asking again.

The **decision owner** is the person whose written confirmation makes a decision
final: a client, a founder or a product owner. Step 00 asks who it is. Only that
person's written confirmation marks anything as Confirmed.

### Choosing the optional steps (PRD Full Discovery)

Step 00 lists the optional steps with a run or skip recommendation for your
product. Reply with the ones to drop (for example "drop 3, 8, 10"), "keep all", or
"as recommended". Dropped steps are skipped automatically, with no session. Each
kept step asks "run or skip?" once more when it comes up. Steps B09 (brief), B11
(PRD), B12 (validation), B15 (epics and stories) and B17 (save) always run.

The workflow skips steps with small marker files (`step-<ID>.done`) in the run.
Each optional step writes its marker first thing, so pressing Finish session early
does not fail the step. A marker that still says `started` means the step was not
finished.

## Saving the results to the project

The last step of each workflow, **Save to project** (`S` in PRD Fast, `B17` in PRD
Full Discovery), saves every document of the run into your product folder in the
project's Assets. It asks nothing: type "go", let it report what it saved, and
press **Finish session**. In PRD Full Discovery it saves the validated `prd.md`
from B12 and skips the `step-*.done` marker files.

You do not need **Promote all** on the run page. If you use it anyway, enter the
same product folder name, and expect it to also copy the marker files and both
copies of `prd.md` (the draft from B11 and the validated one from B12).

Every file also stays on the run page (**Assets** tab), where you can download it.

## Good to know

- **Name clashes.** If you select an input asset with the same name as a file the
  workflow writes (for example an old `prd.md`), it would replace the fresh file in
  later steps. Step 00 warns you. Rename the old file first (for example
  `prd-previous.md`).
- **Two `prd.md` files in a Full Discovery run** are expected. The one from B12
  (`status: reviewed`) is current; the one from B11 is the draft.
- **Figma is optional and off by default.** See SETUP.md to connect it.
- **Language.** The agent talks to you in the language you write in. Documents are
  written in the language you choose in step 00.
- **Idle sessions** are closed after about a day. Finish each step within a day.

## Attribution

**PRD Fast (grill-me).** The grill, PRD and issues steps are adapted from Matt
Pocock's skills (https://github.com/mattpocock/skills): grilling, to-spec (formerly
to-prd) and to-tickets (formerly to-issues), under the MIT License below.

**PRD Full Discovery.** This workflow is built on the BMad Method
(https://github.com/bmad-code-org/BMAD-METHOD) and uses its skills: brainstorming,
deep recon, product brief, PRFAQ, PRD, UX, architecture, epics and stories, and
sprint planning. The BMad Method is open source under the MIT License below.
BMad™, BMad Method™ and BMad Core™ are trademarks of BMad Code, LLC. This template
is an independent workflow. It is not affiliated with, endorsed, approved or
certified by BMad Code, LLC.

### MIT License: mattpocock/skills

```
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### MIT License: BMad Method

```
MIT License

Copyright (c) 2025 BMad Code, LLC

This project incorporates contributions from the open source community.
See CONTRIBUTORS.md (https://github.com/bmad-code-org/BMAD-METHOD/blob/main/CONTRIBUTORS.md)
for contributor attribution.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

TRADEMARK NOTICE:
BMad™, BMad Method™, and BMad Core™ are trademarks of BMad Code, LLC, covering all
casings and variations (including BMAD, bmad, BMadMethod, BMAD-METHOD, etc.). The use of
these trademarks in this software does not grant any rights to use the trademarks
for any other purpose. See TRADEMARK.md (https://github.com/bmad-code-org/BMAD-METHOD/blob/main/TRADEMARK.md)
for detailed guidelines.
```
