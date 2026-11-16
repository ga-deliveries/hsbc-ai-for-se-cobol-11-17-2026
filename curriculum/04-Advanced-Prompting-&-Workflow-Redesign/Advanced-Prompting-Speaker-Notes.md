![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)


# **Advanced Prompting: Speaker Notes**
 
**Module Time:** 60 minutes
- 10-min content
- 20-min guided practice
- 30-min independent practice
---
 
**What This Module is About:**
 
At 2:10 a.m. the end-of-day interest job abends in step 3 with an S0C7. The program that failed, `INTCALC`, was changed yesterday, so the obvious answer is to back the change out. Learners use GitHub Copilot Chat to package and mask the evidence, compare a vague prompt with a structured one, make the model argue against its own diagnosis, and trace the bad data's impact across shared copybooks. They save what worked as a prompt playbook and a `copilot-instructions.md` file. In the Lab, they take a second incident, a DB2 `SQLCODE -811`, from ticket to release-ready change.
 
**Core Objectives:**  
- Package incident evidence, and mask customer data before prompting
- Write structured prompts that force evidence, competing hypotheses and a fixed output format
- Ask the model to challenge its own diagnosis, and rule out the strongest alternative with evidence
- Compare vague and structured prompting on the same incident
- Trace the impact of bad data across shared copybooks and programs
- Save prompts as a playbook and a `copilot-instructions.md` with team conventions and a PII/PCI rule
**Must-Hit Topics:**  
- The first plausible answer is the dangerous one. A vague prompt gives it to you faster, and with more confidence.
- Mask before you attach. Every time.
- One number separates the two diagnoses: `WS-READ-CNT`. Thousands of accounts ran through the new rate code before the failure.
- Bad data in a shared file is tomorrow's incident in a different program.
- AI drafts; the engineer decides. The recovery decision is never the model's.
> [!IMPORTANT]
> This module answers the client manager's strongest request: cutting production-support root-cause time from a day to minutes. Discovery also told us agentic AI is not a fit for this audience. Keep the framing on **prompting and workflow**, with the engineer in control at every step. If learners ask about agents that act on their own, acknowledge the question and bring it back to what they can use on Monday.
 
---
 
## 0. **Before the Session Starts**
**(Time: during the break before Module 4, about 10 minutes)**
 
-   **Purpose:** This module depends on files learners create and on a Copilot setting that may be disabled.
-   **Checklist, for every learner:**
    -   The `module-4/` folder is open as the workspace (**File > Open Folder**), with no "Restricted Mode" banner. The Explorer's top line says **MODULE-4**, not the whole course pack. It holds `incident-1/`, `incident-2/`, `src/`, `incident-playbook.md` and `.github/copilot-instructions.md`.
    -   `incident-playbook.md` and `.github/copilot-instructions.md` are both **empty**. They come in the course pack, so nobody needs to create the `.github` folder. Ask learners not to fill them until Step 5.
    -   Copilot Chat is signed in, in **Ask** mode, with a **Claude** model selected.
    -   No lab handout is inside `module-4`. The handouts live in `labs/`, outside the module folders, because Copilot read the hints in a handout left inside the folder.
> [!WARNING]
> **Confirm that Copilot instruction files are enabled in HSBC's configuration**, and test it in their environment before the course. In the Skillable VM on 7 November 2026, Copilot read `.github/copilot-instructions.md` automatically, and the Step 5 check worked. Step 5 depends on Copilot reading `.github/copilot-instructions.md` automatically. If the setting is disabled or blocked by policy, learners can still write the file and attach it with `#file`, but the "lazy question inherits your standards" demonstration won't work. Know which version you're teaching before you start.
 
-   **Teaching Tips:**
    -   Run all five guided steps the day before, in order, in separate chats as the lesson says. Save your Step 1 vague answer. If a learner's model happens to get the right answer to the vague prompt, you'll want yours to show the room.
    -   Rehearse the arithmetic of the evidence so you can say it without notes: failing record 4,183, so 4,182 processed before it. Those 4,182 are 3,906 accrued + 187 frozen + 31 closed + 58 zero or negative balance. Statement 142 is source line 164. 3 onboarding accounts were added, taking the master from 4,194 to 4,197.
---
 
## 1. **Provocation: 2:10 A.M., Step 3**
**(Time: ~2 minutes, from the Content block)**
 
-   **Purpose:** Make the room feel the pull of the obvious answer before you teach them to resist it.

-   **Talking Points:**
    -   Saturday, 31 October, 2:10 a.m. `EODINT` abends in step 3 with an S0C7. The window closes at 5:00.
    -   Step 3 is `INTCALC`, the program they documented and tested on Day 1.
    -   The change calendar shows three changes went in yesterday evening, and one of them was the rate change to `INTCALC` they tested in Module 2.
    -   Name the instinct out loud: the program that failed changed yesterday, so back it out and go back to bed.

-   **Teaching Tips:**
    -   Before teaching anything, ask: "Hands up if your first move would be to back out the rate change." Most hands go up. Don't comment yet. You'll come back to it after Step 3.
---
 
## 2. **Package the Incident, Then Force Evidence**
**(Time: ~4 minutes)**
 
-   **Purpose:** Establish the incident evidence pack, masking, and the anatomy of a structured prompt.

-   **Talking Points:**
    -   Walk Figure 01, the evidence pack. The row that surprises people: without the change calendar, the model will **assume** the failing program changed.
    -   Masking: show **Edit > Replace** with **Use Regular Expression** on, turning `7100(\d{4})(\d{4})` into `7100****$2`. Learners mask all five evidence files at the start of guided practice, before Step 1, and the lesson has the match counts (2, 8, 4, 4, then 3 customer IDs). Ctrl+H may not reach the Skillable VM, so teach the menu route. After Replace All, check line 1 of each file: one tester typed the pattern into the file instead of the Find box, and saved it.
    -   Walk Figure 02, the six parts of a structured prompt. For each part, say which gap it closes. The two that matter most tonight: **list at least three hypotheses**, and **cite the file and line, or the dump field, for every claim**.

-   **Teaching Tips:**
    -   Connect to the earlier modules: context packing is Module 1's habit, and the evidence rule is Module 1's "cite the line", applied to an incident.
> [!CAUTION]
> Be absolutely clear that real job logs and dumps contain customer data: account numbers, balances, sometimes names. The course pack is synthetic, but learners practise masking anyway, because on a real incident at 2 a.m. it's the step people skip. On their own systems, they must also follow HSBC's policy on what may be shared with Copilot at all.
 
---
 
## 3. **Make It Challenge Itself, Then Make It Reusable**
**(Time: ~4 minutes)**
 
-   **Purpose:** Introduce self-challenge, the playbook, the instructions file, and the workflow view.

-   **Talking Points:**
    -   Self-challenge is the cheapest check in the workflow. If the diagnosis is weak, the model often abandons it. If it's strong, you get the evidence that rules out the alternative, ready for the incident record.
    -   Two places to keep what worked: the **playbook**, for prompts the next on-call engineer copies and fills in; and **`copilot-instructions.md`**, for rules that should apply to every prompt.
    -   Walk Figure 03, the workflow from alert to PR description. Point to each earlier module's place in it: testing is Module 2, review is Module 3.
    -   Land the hexagon: what to do before 5:00 a.m. is the engineer's decision.

-   **Teaching Tips:**
    -   Ask the room where their own team's biggest delay sits in Figure 03. Note the answers; they're useful for the wrap-up and for the client.
> [!NOTE]
> **Knowledge check answer.** The instructions file needs a rule like: "If attached evidence contains unmasked customer data, say so before answering, and do not repeat it." That's the rule in the starter file in Step 5.
 
> [!WARNING]
> Don't oversell the instructions file. It shapes Copilot's behaviour; it is **not a control**. A model can still repeat data it was told not to. If a learner asks "So we don't need to mask any more?", the answer is no: masking is the control, and the instructions file is a safety net.
 
---
 
## 4. **Guided Practice: Diagnose the S0C7**
**(Time: ~20 minutes: 3-min vague prompt, 6-min structured prompt, 4-min self-challenge, 4-min impact, 3-min save)**
 
-   **Purpose:** Learners experience the gap between a vague and a structured prompt on the same incident, then save what worked.

-   **Talking Points:**
    -   **Mask first.** Before Step 1, everyone masks the five files in the lesson's table. Copilot reads files in the open folder on its own, even in Ask mode, so the vague prompt may read the dump. End each prompt with *"Put your whole answer in a single Markdown code block"* and use the **Copy** icon.
    -   Insist on a **new chat** for Step 1 and another for Step 2, so the vague answer can't influence the structured one.
    -   Same rhythm as before: everyone runs the prompt, a few read out results, the room checks against the evidence.
-   **Step-by-step notes:**
    -   **Step 1, the vague prompt (3 min).** Have one or two learners read their answers aloud. Most will blame the rate change, often citing the release risk they found in Module 2. Point out that the model answered the question you *implied*: you said the program changed, and gave it nothing else. Tell everyone to keep the answer for Step 3. In both test runs, the vague prompt **went looking**. It opened the dump, job log and diff and found the ACCTUPD cause. If that happens, have learners open the list of files it read and ask: *"Who chose the evidence? Did it check for other bad records? Did it give a confidence or a next action?"* In the VM it stated "INTCALC is not the faulty program" with no caveat, where Step 2 rated the same link "not established". Same conclusion, very different honesty.
    -   **Step 2, the structured prompt (6 min).** Check the files are masked, and that the prompt starts with the clock line. Walk the expected result, then send everyone to find three things in the evidence themselves: `ACCT-BALANCE` holding `X'4040404040'` in the dump, the `MOVE NEWACCT-REC TO ACCT-REC` in the diff, and `WS-READ-CNT`. Watch for three slips seen in testing. The model may drop the output format and give no Confidence or Next action, so ask for the format again. It may cite the dump's level numbers (31–45) as line numbers: the fields are on lines 48–55. And it may skip the counters, so point learners to CEEDUMP lines 69 and 71. Masking hides the onboarding range, so a careful model says that link is "not established" and makes the case from the 15-byte footprint instead.
    -   **Step 3, self-challenge (4 min).** Ask who got the rate-file mismatch as the strongest alternative. Then return to the hands-up from the provocation: "Who would still back out the rate change?" Walk through what that would have cost: a change, a rerun, and a second S0C7 at about 3:30 a.m., with the real cause still in the file. Accept a different strongest alternative if it's fair: one run chose "the record was already bad", ruled out by `ACCTUPD-sysout.txt` (3 new accounts last night) and the footprint. The "one thing to check" is often wrong: if the model doesn't say "other bad records", ask: *"ACCTUPD added 3 accounts. What happens at about 3:30 if you fix only this one?"* And watch the recovery advice. "Rerun from a known-good master generation" throws away all of last night's account updates, and "don't rerun until ACCTUPD is fixed" misses the 05:00 window. Ask: *"What else was on last night's master?"*
    -   **Step 4, impact (4 min).** Spend your time on the `FEEASSES` row. Month-end fees run later tonight, and they'll abend the same way unless the data is fixed. Then the `ACCTUPD` row: the data fix alone isn't enough, because tomorrow's feed brings new bad records. The next-run times come from the change calendar (FEEMTH Sat 22:00, STMTPREP Sun 03:45), so the prompt attaches it. Expect a fifth row, `CUSTLINK`, from `incident-2`. Check the Explorer still says **MODULE-4**: one run had the whole course pack open and read `module-1/ACCTREC.cpy`.
    -   **Step 5, save (3 min).** Learners paste their three prompts into the playbook with placeholders, and start the instructions file. Then re-run the Step 1 vague question in a new chat, and compare. The playbook keeps the clock line, with an `<incident time>` placeholder. The starter instructions file now has a **Recovery** section. In the VM, the re-run vague question came back in Root cause | Confidence | Evidence | Ruled out | Next action form, listed the files it read, and opened Next action with "Recovery is your decision … least change first".
> [!IMPORTANT]
> **The recovery decision belongs to the engineer.** After Step 3, say it explicitly: the usual safe choice at this hour is a data fix, with approval, then a restart from `STEP020`. Code changes go through a normal change, tested the Module 2 way and reviewed the Module 3 way. AI can draft the data-fix request; whether to apply it is not its decision.
 
> [!TIP]
> The most powerful moment in this module is putting the Step 1 and Step 2 answers side by side. If time is short anywhere, protect that comparison. Ask the room: "Same model, same incident, two answers. What changed?" The answer is only the prompt.
 
-   **Teaching Tips:**
    -   If a learner's vague prompt happens to reach the right answer, don't dismiss it. Ask: "What evidence did it give you? Could you put that in the incident record?" Usually it can't, and that's the point.
    -   If a learner's structured prompt still blames the rate change, check they picked all seven files from the `#file:` list, especially the dump and the diff. A typo such as `ACCTUPD-CR-48700.diff` attaches nothing, and the answer can still look fine because the model found the file by itself.
    -   If you're behind, shorten Step 5. Learners can finish the files at the start of the Lab.
**Answer key: Step 1, typical vague answer**
 
The model attributes the S0C7 to the rate change: a mismatch between the new `RI-TIER-RATE-PCT` layout and the rate file, and recommends backing out CR-48812. Plausible, specific, and wrong.
 
**Answer key: Step 2, structured diagnosis**
 
| Part | Expected content |
|---|---|
| Failing statement | Statement 142, source line 164, `WHEN ACCT-BALANCE NOT > ZERO`, in `1000-PROCESS-ACCOUNT` |
| Invalid fields | `ACCT-BALANCE`, `ACCT-ACCRUED-INT`, `ACCT-LAST-INT-DATE`, `ACCT-WITHDRAWALS-MTD`, all `X'40'` (spaces) |
| Root cause | CR-48790: `ACCTUPD` group-moves the 15-byte `NEWACCT-REC` into the 100-byte `ACCT-REC`, padding numeric fields with spaces; nothing initialises them |
| Confidence | High |
| Evidence | Dump fields; the 15-byte footprint (`'SV'`, `'A'`, then spaces from byte 16); `ACCTUPD` added 3 accounts; `WS-READ-CNT` 4,183, so 4,182 processed first. With the account number masked, its place in the onboarding range is "not established", and that's correct |
| Ruled out | CR-48812: failure would be in the rate fields on the first active account, but 3,906 accounts accrued interest with the new code. CR-48801: `STMTGEN` doesn't run in `EODINT` |
| Next action | Confirm all three onboarding records are bad, then decide recovery |
 
**Answer key: Step 3, self-challenge**
 
-   **Strongest alternative:** a rate-file layout mismatch from CR-48812.
-   **Why it's rejected:** the failure is at statement 142, not in `2000-ACCRUE-INTEREST`; the invalid field is the balance, not a rate field; 4,182 records ran first.
-   **Check before acting:** whether there are more bad records. There are three, so fixing only the one in the dump leads to a second abend.
**Answer key: Step 4, impact**
 
| Program | Risk |
|---|---|
| `INTCALC` | Already failed tonight |
| `FEEASSES` | `FEEMTH`, Sat 22:00 (change calendar): month-end fee run later tonight; compares `ACCT-BALANCE` in `2000-CALC-FEE`, so it will abend the same way |
| `ACCTUPD` | Will create more bad records from tomorrow's feed until the code is fixed: `INITIALIZE ACCT-REC`, then move the three elementary fields. `INITIALIZE` followed by the same group `MOVE` is **not** a fix: the move pads the 85 bytes with spaces again |
| `STMTGEN` | Named in the copybook header, not in the workspace. Unconfirmed |
| `CUSTLINK` | `STMTPREP`, Sun 03:45. Moves `ACCT-BALANCE` into the statement extract in `3000-WRITE-EXTRACT`. Whether it abends or passes the spaces on is not established |
 
---
 
## 5. **Independent Practice: Incident to Release**
**(Time: ~30 minutes: 4-min read the ticket and check the masking, 8-min root cause, 5-min impact, 7-min recovery, fix and tests, 6-min PR description, playbook and time estimate)**
 
-   **Purpose:** Learners take a second, unfamiliar incident through the whole workflow using their own playbook, and measure what it saved them.

-   **Talking Points:**
    -   They open `incident-2/SAVOPS-2317.md`: Sunday 1 November, 04:12. `STMTPREP` failed with RC 16 and `SQLCODE -811` in `CUSTLINK`, and November statements are blocked, with a regulatory deadline of Tuesday 3 November at 12:00. They set the clock as the **first message in the same chat** they use through Task 4.
    -   They use their playbook for diagnosis, self-challenge and impact, then extend it with three new prompts: proposed fix, test scenarios and PR description. For the self-challenge, the handout supplies the night shift's theory: joint accounts, plus `FETCH FIRST 1 ROW ONLY`.
    -   Test scenarios follow Module 2's method: a scenario table with expected results and a **Should change?** column. The PR description should be something a Module 3 reviewer could sign off against.
    -   Finally, each learner records a before-and-after time estimate for each stage: how long it would have taken before this course, and how long it took today.
-   **Teaching Tips:**
    -   Circulate during diagnosis and ask: "Which of your playbook prompts did you use, and what did you have to change?" The changes are where the playbook improves.
    -   Most common problems: skipping the mask step on the new evidence; proposing a fix before the root cause is established; a fix that hides the bad data instead of addressing it; a missing clock line, so the deadline "has already passed"; unfilled placeholders such as `<job>`, which a good model refuses to work with; and a deadline that never appears in the impact list.
    -   Confidence is often marked down to "medium", even though each `LOAD_TS` falls inside its run's LOAD window (21:15:52 in 21:14:07–21:16:39, and 22:04:13 in 22:02:32–22:05:01). Ask: *"What would you need to see to call this high, and is it already here?"*
    -   If the model wants to reconcile every duplicate against CRM first, ask: *"Is there anything to choose between?"* The duplicate rows are identical except `LOAD_TS`.
    -   Fast finishers: ask Copilot to review their PR description as a Module 3 reviewer would, using their review checklist.
> [!NOTE]
> **Incident 2 as built: SAVOPS-2317.** On Saturday 31 October, `CRMLOAD` loaded its 1,388-row delta into `SAV.ACCT_CUSTOMER`. The image-copy step then timed out, and the operator resubmitted the whole job. `LOAD ... RESUME YES` appends, so the same 1,388 rows went in again with a later `LOAD_TS`. The unique index `ACI0` that would have rejected them was dropped in 2019 (CR-31877) and never re-created. `CUSTLINK`'s singleton `SELECT` for the primary holder now finds two rows, gets `-811`, and its `WHEN OTHER` stops `STMTPREP` with RC 16. The evidence pack is the ticket, both job logs, the DBA query, the table DDL, `CUSTLINK.cbl` and its two copybooks. The full key is `facilitator/module-4/incident-2-lab-key.md`.
 
**Answer key: incident 2**
 
-   **Root cause:** the CRMLOAD resubmit loaded the delta twice, and there was no unique index to stop it. It's not a `CUSTLINK` defect: `CUSTLINK` last changed in 2019.
-   **Ruled out:** the STMTPREP rerun (it failed the same way before), joint accounts (`PRIMARY_HOLDER = 'Y'` already excludes holder 2), and a `CUSTLINK` change.
-   **Impact:** STMTPREP and CUSTLINK, up to 1,388 accounts; STMTMTH and STMTGEN, with the **Tue 3 Nov 12:00** deadline; STMTARCH; ACINQ01, the CICS inquiry, already failing online; CRMRECON, not established; and CRMLOAD, which will do it again. STMTGEN, STMTARCH, ACINQ01 and CRMRECON are per header only.
-   **Recovery:** with approval, delete the run-2 copy of the delta (count it first: 1,388 rows, each with a run-1 match), check that no account has two primary rows, then rerun STMTPREP and release STMTMTH. A `CUSTLINK`-only change would leave ACINQ01 failing.
-   **Permanent fix:** re-create `ACI0`; change the CRMLOAD restart procedure; and in `CUSTLINK`, handle `-811` with an exception ("MULTIPLE PRIMARY HOLDERS"), continue, and end RC 4, keeping RC 16 for real DB2 errors.
-   **Test scenarios:** T1–T8 in the key, including two *different* customers both flagged primary, and rerunning the same delta after the fix.
-   **PR description:** problem, root cause, change, related changes on separate tickets, testing and risk. It should say no copybook changed, so nothing needs recompiling.
> [!TIP]
> The `FETCH FIRST 1 ROW ONLY` discussion is the best moment in the Lab. It's the kind of fix an AI suggests confidently, it makes the incident go away, and it hides a data problem. If no one raises it, raise it yourself. So does `ORDER BY LOAD_TS DESC`, and "fail closed" (stop with RC 16 and a clearer message), which still blocks every customer's statement next time.
 
---
 
## **Wrap-Up Reflection: Module and Course**
**(Time: ~2 to 3 minutes, from the end of the Lab)**
 
-   **Key Takeaways for Module 4:**
    -   Package the evidence and mask it before you attach anything
    -   Structured prompts force evidence and competing hypotheses; vague prompts give you the first plausible answer
    -   Ask the model to argue against itself before you act
    -   Trace impact across shared copybooks: bad data travels
    -   Save what worked: a playbook for prompts, an instructions file for standards
-   **Course close:** connect the four modules as one workflow, the same one in Figure 03:
    -   **Module 1:** understand code you didn't write, and verify it
    -   **Module 2:** prove a change is safe, with scenarios and evidence
    -   **Module 3:** review someone else's change, and catch confident but wrong AI output
    -   **Module 4:** take an incident from 2 a.m. to a release-ready change
-   **Closing question for the room:** "Which one thing from these two days will you do differently on Monday?" Ask each learner for one sentence.
> [!IMPORTANT]
> **Collect the before-and-after time estimates** from every learner before they leave. They are the baseline for the sprint and PI throughput measure the client's manager wants to track. Make sure you know how they'll be gathered (a shared form, a file in the workspace, or the facilitator's notes) before the session starts. Each lab already asks for one, so learners have four: Module 1 in its lab write-up, Module 2 in `lab-findings.md`, Module 3 in `review-checklist.md`, and Module 4 in `SAVOPS-2317-resolution.md`. They're in the VM, so collect them before learners log out: a Skillable lab reset loses them.
 
> [!NOTE]
> The team is moving from Endevor to GitHub next year. Mention that `copilot-instructions.md` and the playbook can live in their repositories once they migrate, so the whole team shares the same standards and prompts. It's a concrete next step that keeps today's work alive.
 
---
 
## **What Was Tested and What Was Not**
 
-   **Checked while authoring:** the evidence packs for both incidents are internally consistent. Incident 1: 4,182 = 3,906 accrued + 187 frozen + 31 closed + 58 zero or negative; statement 142 is source line 164; master 4,194 to 4,197. Incident 2: each `LOAD_TS` falls inside its run's LOAD window, and the row counts agree.
-   **Run in GnuCOBOL:** `ACCTUPD` with 3 onboarding records ends RC 0 and writes spaces into the numeric fields. `INTCALC` with `-debug` stops on the bad balance. GnuCOBOL reports it at the `COMPUTE` (line 192) because it doesn't check compares; z/OS raises the S0C7 at the compare (line 164), and the evidence pack shows z/OS behaviour.
-   **Tested in the Skillable VM** on 7 November 2026 with Copilot and Claude Sonnet 5.5: masking with the exact match counts, all five guided steps, the instructions-file check and the incident 2 Lab. What your assistant says will still vary from run to run, so run every prompt once before class.
-   **Still to confirm with HSBC before delivery:** that Copilot instruction files are enabled in their configuration, if the course runs outside the VM. The `CEEDUMP` and job logs are simplified for teaching and are not exact copies of a real system's output.
 

