![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Instructor Guide: AI for Software Engineers (COBOL)

**Contains answer-key locations. Do not share this file, the speaker notes, the lab guides' answer-key tabs or any reference solutions with learners.**

---

## The course in one page

| | |
|---|---|
| Audience | HSBC COBOL mainframe engineers, from senior SMEs who analyse and sign off changes to newer engineers. Most have not used AI tools at work before |
| Format | Four 60-minute modules over two days, 2 hours a day, remote with a live instructor. Each module runs I Do (10 min), We Do (20 min) and You Do (30 min) |
| Environment | VS Code with the IBM Z Open Editor COBOL extension, GitHub Copilot Chat in **Ask** mode with an approved **Claude** model, and the course pack (`module-1` to `module-4`). Synthetic data only; no live mainframe |
| The through-line | AI drafts and explains. The engineer verifies against the source, gathers evidence and decides |

**Run of show**

| Day | Module | You Do produces | Lab guide |
|---|---|---|---|
| 1 | 1. Legacy COBOL Docs | `ACCTREC-dictionary.md`, `INTCALC-README.md` with two Mermaid diagrams, a corrections log and a time estimate | `Legacy-COBOL-Docs-Labs.md` |
| 1 | 2. COBOL Testing | Boundary records added to the synthetic file, a harness report read and explained | Not yet supplied (see Decisions still open) |
| 2 | 3. Safe Change Review | `review-checklist.md` applied to `FEEASSES-v2.cbl`, harness evidence and a **Not safe** verdict | `Safe-Change-Review-Labs.md` |
| 2 | 4. Advanced Prompting & Workflow Redesign | `incident-2/SAVOPS-2317-resolution.md`, an updated `incident-playbook.md` and a time estimate | `Advanced-Prompting-Labs.md` |

Every module's You Do ends with the same sentence: *"Without AI, this would take me about \_\_\_. Today it took \_\_\_."* Those estimates are the client's throughput baseline. Collect them.

## Prepare (about 2 hours the day before)

1. **Open each module folder on its own** (**File > Open Folder > module-N**), never the parent folder. Check there's no Restricted Mode banner, and that **Ctrl+click** on `ACCTREC` in a `COPY` statement opens the copybook. The lookup depends on each module's `zapp.yaml`.
2. **Set up Copilot exactly as learners will have it:** signed in, **Ask** mode, the approved Claude model. In Agent mode Copilot can open files on its own, which spoils Module 3.
3. **Run every I Do and We Do prompt once**, in order, and note the shape of the answers. In Module 4, keep your Step 1 vague answer: if a learner's model happens to get it right, you'll want yours on screen.
4. **Run the harness** for Modules 2 and 3 with **Ctrl+Shift+B**. Module 3 must report **1,447 unchanged, 6 changed as expected, 47 unexplained**. Delete the `reports` folder afterwards, so learners start clean.
5. **Check Mermaid renders:** open any `.md` file with a Mermaid block and press **Ctrl+Shift+V**. Module 1's lab depends on it.
6. **Check instruction files are on:** **Settings** (Ctrl+,), search *instruction files*, and confirm `.github/copilot-instructions.md` is used. Module 4's Step 5 depends on it.
7. **Make a spare copy of `module-4/incident-1` and `module-4/incident-2`** outside the course pack. Masking edits the files, so restore them between groups.
8. **Read each module's speaker notes and lab guide**, especially the answer keys, the timing cuts and the "Before class" checklists.
9. **Decide who removes instructor-only material** from the learner copy of the course pack (speaker notes, answer-key tabs, any reference solutions), and do it.

## Where the answer keys are

| Module | Answer key | What it covers |
|---|---|---|
| 1 | Module 1 speaker notes | The 13 `INTCALC` business rules (and the five the model softens), the `ACCTREC` data dictionary with offsets totalling 100 bytes, the knowledge-check answer |
| 2 | Module 2 speaker notes | The four field changes, the expected-change rule, the 14-row scenario table with expected values, the packed hex for S02, S13 and S14, the coverage gaps |
| 3 | Module 3 speaker notes, and the answer-key tab of the Module 3 lab guide | Fee rules, reachability, risks R1 to R8 and security findings, with line numbers; the planted lost `THRU` range; harness results; the expected **Not safe** verdict |
| 4 | Module 4 speaker notes, and the answer-key tab of the Module 4 lab guide | Incident 1 (the `ACCTUPD` group `MOVE`, not the rate change) with its evidence; the impact list including `CUSTLINK`; incident 2 (the double `CRMLOAD` and the missing unique index) |

> [!IMPORTANT]
> The lab guides describe the course pack **as built**, and some of their details differ from the lessons and speaker notes, which were written first. Where they disagree, trust the lab guide and the files in the pack. The differences are listed under **Known inconsistencies** below; fix them before delivery.

## Facilitation principles

- **Predict, then check.** Ask the room what the model will say, or what the code will do, before anyone runs anything.
- **An explanation is a hypothesis until someone has read the line.** Ask "show me the line" until learners ask it themselves. Comments are claims too; the code wins.
- **Same model, same mode, same files.** Room comparisons only work if everyone is on the same Claude model, in Ask mode, with the same files attached.
- **Protect the key moments.** If time is short, cut elsewhere. The moments that carry each module are: Module 1, Step 3 (the rules the model softens); Module 2, the scenario table and the **Should change?** column; Module 3, the trace that breaks "behaviour preserved"; Module 4, the vague and structured answers side by side.
- **Mask before you attach, every time.** The data is synthetic; the habit isn't. Treat any full account number or secret in a chat as a teaching moment about a real incident.
- **The human decides.** Recovery decisions, sign-off verdicts and what counts as "safe" belong to the engineer. Never let a demo imply otherwise.
- **Name the limits of the tools.** `#codebase` sees only the open folder. The harness proves logic on synthetic data, not mainframe behaviour. Header comments are claims. Say these out loud.

## Things that commonly go wrong

| Symptom | Likely cause and fix |
|---|---|
| Restricted Mode banner, or the COBOL extension doesn't highlight | The folder isn't trusted. Click **Manage** on the banner and trust it |
| **Ctrl+click** on `ACCTREC` doesn't open the copybook | The learner opened the parent folder, so `zapp.yaml` isn't found. Reopen the module folder on its own |
| A learner's answers differ wildly from the room's | Wrong model, Agent mode, or a file not attached. Check the model picker, the mode and the attached files |
| A Mermaid diagram shows as code, not a picture | A syntax error, usually unquoted parentheses in a label. Paste the error back to Copilot. If no diagram renders at all, the preview's Mermaid support is missing (see Decisions still open) |
| A learner opened `FEEASSES-v2.cbl` before the Module 3 Lab | Usually Agent mode, or curiosity. Switch to Ask mode, close the file, and keep going. Don't restart |
| The Module 3 harness doesn't show 1,447 / 6 / 47 | A `reports` folder left from a previous run, or an edited source file. Delete `reports`, restore the source, rerun |
| Masking replaced nothing | **Use Regular Expression** (the `.*` button) is off in the **Ctrl+H** box |
| A new group finds account numbers already masked | The previous group's edits. Restore `incident-1` and `incident-2` from your spare copy |
| A model says the Module 4 failure is at source line 142 | It treated the statement number as a line number. Statement 142 is LineID 000164 in the listing excerpt; source line 142 is blank |
| `#codebase` returns `CUSTLINK` in Module 4, Step 4 | Expected: it searches the whole folder, including `incident-2`. It's a genuine finding; use it to make the point |
| `copilot-instructions.md` doesn't change the vague answer | The instruction-files setting is off, or the file isn't saved in `.github`. Check both, then start a new chat |
| A model prints a full secret or account number | Stop and discuss it: on real code, that's a security incident. Then check the prompt's masking instruction |
| Learners finish early | Each lab has stretch tasks at the end of its You Do section |
