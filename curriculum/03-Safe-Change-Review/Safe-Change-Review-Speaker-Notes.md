![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)


# **Safe Change Review: Speaker Notes**
 
**Module Time:** 60 minutes
- 10-min content
- 20-min guided practice
- 30-min independent practice
---
 
**What This Module is About:**
 
An offshore developer has delivered a refactored `FEEASSES`, the monthly fee program, with a new fee tier and the old `GO TO` loop "cleaned up". Their AI says "behaviour preserved", and the release is tomorrow. Learners use GitHub Copilot Chat to map how the **original** really runs, rank its risks, scan it for secrets and build a review checklist. In the Lab, they apply that checklist to the developer's version, run the regression harness, find the planted behaviour change, and write a change-safety verdict.
 
**Core Objectives:**  
- Review against a checklist: control flow, error handling, numeric fields, data and copybook impact, security
- Prompt for a reachability table, and find `GO TO` loops, `PERFORM ... THRU` fall-through and dead paragraphs
- Flag unchecked `FILE STATUS` and `SQLCODE`, and rank risks by impact on customers' money and the audit trail
- Scan COBOL and JCL for hardcoded credentials, with masked output
- Compare two versions by tracing behaviour, not by summarising text
- Catch AI output that sounds right and is wrong, and write a verdict backed by evidence

**Must-Hit Topics:**  
- You can't review a change to code you don't understand. Map the original first.
- Fall-through: a paragraph inside a `PERFORM ... THRU` range runs because of where it sits. Refactors, and AI, lose it.
- "Behaviour preserved" is a claim until a trace or the harness proves it.
- The 15.00 fee cap is regulatory, and it's reached only by fall-through.
- A security finding you didn't cause still needs an owner.
> [!IMPORTANT]
> Discovery told us the US team mainly acts as SMEs who analyse and sign off on changes while others write the code. This module is built for that role. Keep framing every activity as "what you'd need before you sign", not "how to write better code". The learners' authority in this module is the reviewer's, and the content should respect that.
 
---
 
## 0. **Before the Session Starts**
**(Time: before Day 2 begins, about 10 minutes)**
 
-   **Purpose:** Day 2 starts here. Learners' VMs may have been reset overnight, and the Lab runs this program in its own harness, with a stand-in for DB2.
-   **Checklist, for every learner:**
    -   The `module-3/` folder is open as the workspace (**File > Open Folder**), with no "Restricted Mode" banner. It holds `FEEASSES.cbl`, `FEEASSES-v2.cbl`, `CR-48830.md`, `FEEMTH.jcl`, `ACCTREC.cpy`, `FEE_TABLES.sql`, `data/` and `harness/`.
    -   **Terminal > Run Task…** lists "Run regression: FEEASSES vs FEEASSES-v2 (2026-10-31)". Learners don't need anything from Module 2.
    -   `review-checklist.md` is empty (it comes in the course pack).
    -   Copilot Chat is signed in, in **Ask** mode, with a **Claude** model selected.
> [!WARNING]
> **Tell learners not to open `FEEASSES-v2.cbl` until the Lab.** It sits in the workspace, and curious learners will open it first. If they read the clean new version before they understand the original, they lose the point of the guided practice. Using `#file` only isn't enough: Ask mode can still open any file in the folder. In testing, the Step 4 answer used the new fee rule from `CR-48830.md`, which nobody attached. That's not a disaster here, but it is a good "who chose the evidence?" moment.
 
> [!NOTE]
> **How the harness runs `FEEASSES`.** DB2 isn't available in the VM, so for the run only the harness swaps each `EXEC SQL` for a call to a stand-in (`harness/SQLSTUB.cbl`). `FEE_SCHEDULE` is read from `harness/fee-schedule.csv`, with the same rows as `FEE_TABLES.sql`, and `FEE_HISTORY` rows are written to a file and counted. The `.cbl` files learners review are not changed.
 
-   **Teaching Tips:**
    -   Run every prompt in the module the day before, including the loop-comparison demo. Check whether your model marks `2050-APPLY-FEE-CAP` as "never reached" in Step 1. Many do on the first try, which makes a strong teaching moment.
    -   The day before, run the loop demo program: `bash ~/coursepack/facilitator/module-3/loop-demo/run-loop-demo.sh`. It compiles and runs both loop versions on the same table for `WS-COUNT` 0 to 3. Only 0 should show DIFFERENT. You'll run it live in Section 3.
---
 
## 1. **Welcome Back and Provocation: the Release Is Tomorrow**
**(Time: ~2 minutes, from the Content block)**
 
-   **Purpose:** Re-open the course for Day 2, and put learners in the reviewer's chair.

-   **Talking Points:**
    -   Quick recap of Day 1 in one sentence each: Module 1, understand code you didn't write; Module 2, prove a change didn't break anything.
    -   Set the scene: 4 p.m., a refactored `FEEASSES` with a new fee tier and a cleaned-up `GO TO` loop. The developer has signed off. Their AI says "structure modernised, behaviour preserved". The release is tomorrow, and you're the SME.
    -   Fees come straight out of customer balances, and some fee rules exist because a regulator required them.
    -   The uncomfortable truth: most review time goes into reading the new version and feeling reassured by how clean it looks. Clean is not the same as correct.

-   **Teaching Tips:**
    -   Ask: "When you review a change today, do you start with the old code or the new code?" Most will say the new. That's the habit this module changes.
---
 
## 2. **What a Reviewer Is Actually Checking**
**(Time: ~4 minutes)**
 
-   **Purpose:** Give the room a shared review checklist, and make fall-through concrete.

-   **Talking Points:**
    -   Walk Figure 01, the five areas. Spend the time on control flow: that's where COBOL refactors break.
    -   Explain fall-through with the code excerpt in the lesson. `PERFORM 2000 THRU 2099` runs every paragraph between them in source order, so `2050` runs even though nothing names it.
    -   The key sentence: **to a modern eye, and to most AI models, a paragraph looks like a function.** Change `PERFORM ... THRU` to a plain `PERFORM`, and the program still compiles, runs and looks right. It just doesn't run `2050` any more.
    -   Dead code is the opposite problem: it looks important and never runs, and AI explanations describe it as if it were live.

-   **Teaching Tips:**
    -   The senior SMEs in the room will know fall-through cold. Ask one of them to explain why it was written that way in the first place. Their answer usually makes the risk vivid for the newer engineers.
---
 
## 3. **Prompting AI to Review, and to Compare Two Versions**
**(Time: ~4 minutes)**
 
-   **Purpose:** Show two prompt patterns: reachability for one program, and behavioural tracing for two versions.

-   **Talking Points:**
    -   Reachability, not summary. Show the prompt, and point out the instruction about `THRU` ranges. Without it, models often list the fall-through paragraph as "never reached".
    -   **Chat A:** run "summarise the differences" on the loop refactor. Read out the answer. In testing it mistraced it, saying the last element is dropped.
    -   **Chat B:** run the tracing prompt with `WS-COUNT` = 0, 1 and 3. In testing the trace caught its own mistake: "My earlier claim … was wrong".
    -   **Demo C:** run the loop demo program. Only `WS-COUNT` 0 is DIFFERENT. The run, not either answer, settles it.
    -   Walk Figure 02: what an AI review says, and what you need before you believe it.

-   **Teaching Tips:**
    -   Give the room 20 seconds to find the bug in the loop before you say anything. Someone usually spots that `TEST AFTER` runs the body before the test. Whoever finds it, credit them by name.
    -   If your model gets the comparison right the first time, don't hide it. Say so, then point out that it was right *this time*, and that the trace is what lets you know that.
> [!NOTE]
> **Knowledge check answer.** If you could run only one trace, choose `WS-COUNT = 0`. For 1 or more, both versions add the same items. With zero items, the original tests first and adds nothing, while `WITH TEST AFTER` runs the body once and adds `WS-AMT (1)`. The empty case is the one most likely to be missed, and the one most likely to differ.
 
---
 
## 4. **Guided Practice: Audit the Original FEEASSES**
**(Time: ~20 minutes: 5-min flow, 6-min risks, 4-min credentials, 5-min checklist)**
 
-   **Purpose:** Build the baseline the Lab depends on: how the original runs, what its risks are, and what a reviewer must check.

-   **Talking Points:**
    -   Same rhythm as Modules 1 and 2: everyone runs the prompt, two or three read out their results, the room checks against the source.
    -   End each prompt with *"Put your whole answer in a single Markdown code block"* and copy with the code block's **Copy** icon, so Copilot's working log stays out of `review-checklist.md`.
    -   Remind them why this matters: in 25 minutes they'll compare a refactor against exactly what they're building now.
-   **Step-by-step notes:**
    -   **Step 1, map how it runs (5 min).** Go straight to the reachability table. Ask: "Whose model marked `2050-APPLY-FEE-CAP` as never reached?" Then have everyone search for `2050` with `Ctrl+Shift+F`: it appears once, as its own name. Let that land: a regulatory cap applied purely by position in the source. Confirm `2500` as dead with two pieces of evidence, the change log and the missing `'ST'` type in `ACCTREC`. Watch the plain-language answer: in testing it printed the override code in full, and the learner pasted it into `review-checklist.md`. Have everyone search their checklist for it after Step 3 and mask every copy.
    -   **Step 2, rank the risks (6 min).** Ask the room to agree on severities, not just the list. Spend your time on R1 and R8 together. The cap hides an overflow, so a change that loses the cap exposes a second defect. Then R6: if a model stated confidently that DB2 commits or rolls back on `STOP RUN`, use it as the example of a confident answer the reviewer can't sign.
    -   **Step 3, scan for secrets (4 min).** Check that everyone's output is masked. If anyone's model printed a value in full, use it: that value is now in their chat history. Then discuss what a reviewer *does* with findings the developer didn't cause. Two things seen in testing. A model may decide the override code is "not a credential or secret" and print it in full, even though the prompt names override codes: that's the moment to say it works like a password for bypassing the audit trail. And check the cited lines: the FTP password is on `FEEMTH.jcl` line 40, and one run cited line 37.
    -   **Step 4, build the checklist (5 min).** Edit as a room. Apply the test to every item: "Could I answer yes or no, and point to evidence?" Cut anything vague.
> [!IMPORTANT]
> Make sure every learner's checklist includes the fall-through item: **"Is every paragraph that was reached only by fall-through still executed, on every path that reached it before?"** That's how they'll find the planted defect in the Lab. Don't tell them what the defect is. The checklist item is generic; connecting it to `2050` is their job.
 
> [!CAUTION]
> In Step 3, stress that an AI scan is a fast first pass, not the bank's security tooling. On real code, learners must follow HSBC's policy on what may be shared with Copilot, ask for masked output, and report real findings through the security process. If anyone mentions finding real credentials in their own team's code, don't discuss specifics in the room. Ask them to raise it through the proper channel.
 
-   **Teaching Tips:**
    -   If a learner asks whether the security findings should block tomorrow's release, turn it back to the room. The answer: not on their own, because the developer didn't introduce them, but they **do** block it if the new version touches either line or adds a new secret.
    -   If you're behind, shorten Step 3, never Step 1 or Step 4. The Lab depends on both.
**Answer key: fee rules in the original**
 
1. Closed and frozen (`C`, `F`, `L`) accounts: no fee; record written unchanged.
2. Base fee and waiver balance from `FEE_SCHEDULE`, by account type, on the run date.
3. No schedule row (`SQLCODE` 100): no fee.
4. Any other SQL error: stop with return code 16.
5. Balance at or above the waiver balance: no fee.
6. Activity fee of 2.00 per withdrawal over 3 in the month.
7. Total fee capped at 15.00, per regulatory notice 17-114. **Reached only by fall-through.**
8. Fee subtracted from the balance, even into overdraft.
9. Every fee on the report; recorded in `FEE_HISTORY` unless the `PARM` carries the override code.
**Answer key: reachability**
 
| Paragraph | How it's reached | Notes |
|---|---|---|
| `0000-MAIN` | Program entry | |
| `0100-INITIALISE` | `PERFORM` | |
| `1000-READ-LOOP` | `PERFORM THRU 1000-EXIT`, then `GO TO` | The `GO TO` loop |
| `1000-WRITE` | Fall-through, or `GO TO` | Closed and frozen accounts jump here |
| `1000-EXIT` | `GO TO` at end of file | |
| `2000-CALC-FEE` | `PERFORM THRU 2099-EXIT` | |
| **`2050-APPLY-FEE-CAP`** | **Fall-through only** | Runs whenever a fee is calculated |
| `2099-EXIT` | Fall-through, or `GO TO` | |
| `2500-APPLY-STUDENT-WAIVER` | **Never** | Dead code |
| `3000-APPLY-FEE` | `PERFORM` | When the total fee is above zero |
| `9000-TERMINATE` | `PERFORM` | |
 
**Answer key: risk register**
 
| # | Risk | Severity |
|---|---|---|
| R1 | Fee cap reached only by fall-through | High |
| R2 | `FEE_HISTORY` insert has no `SQLCODE` check | High |
| R3 | `ACCTOUT` and `FEERPT` file statuses never tested | Medium |
| R4 | A fee can take the balance below zero | Medium |
| R5 | Cap, per-withdrawal charge and threshold are hardcoded | Medium |
| R6 | No `COMMIT`; partial-run behaviour on SQL error depends on how the job ends | Medium, as a question for the DBA |
| R7 | `2500` is dead code | Low |
| R8 | `WS-ACTIVITY-FEE` overflows silently at 503+ withdrawals. Mostly hidden by the cap, but 503–507 withdrawals undercharge | Low |
 
**Answer key: security findings**
 
| File | Finding | Masked value |
|---|---|---|
| `FEEASSES.cbl` | Hardcoded override code that suppresses `FEE_HISTORY` inserts | `OVRD****` |
| `FEEMTH.jcl`, `STEP020` | FTP user ID and password in in-stream `INPUT`, sent over plain FTP | `Fee$****` |
 
---
 
## 5. **Independent Practice: Review the Refactor and Write the Verdict**
**(Time: ~30 minutes: 5-min read the change, 10-min AI review, 7-min harness, 8-min checklist and verdict)**
 
-   **Purpose:** Learners apply their checklist to the developer's version, gather evidence from AI and the harness, find the planted behaviour change, and decide whether it's safe to release.
-   **Talking Points:**
    -   First they open `CR-48830.md`: what was asked for, and what the developer says they did. Then **Compare Selected** on the two `.cbl` files, noting every change the change request didn't ask for.
    -   They prompt for a reachability table for v2 and compare it with their baseline, then trace three accounts (A, B and C) in both versions, and check one trace by hand.
    -   They run the harness (**Ctrl+Shift+B**) and read the report: unchanged, changed as expected, unexplained.
    -   They answer every checklist item yes or no, with evidence, and write a verdict: **safe**, **safe with conditions** or **not safe**.

-   **Teaching Tips:**
    -   Circulate during the AI comparison and ask: "Did your model say behaviour was preserved? What's your evidence that it is?" Many models will miss the planted change on the first prompt. That's the lesson working.
    -   If a learner is stuck after the harness, point them back to their checklist, not to the code: "Which checklist item haven't you answered yet?"
    -   Most common problem in verdicts: "not safe because the code is different". Push for specificity: which behaviour changed, for which customers, by how much, and which line causes it.
    -   The reachability prompt found the defect cleanly in testing (`2050` "Never"). Two inaccuracies to look for: a muddled explanation of when `2050` *could* be reached, and "`2100` runs under the same conditions as the original activity-fee code". It runs for every account below the waiver, and sets 0 for 3 or fewer withdrawals.
    -   The trace was right for A, B and C, but one sentence contradicted itself before reaching the right answer. A sign-off can't carry a sentence like that.
    -   Fast finishers: the stretch task. They back up `FEEASSES-v2.cbl`, make the fix their verdict calls for, write down the result they expect, and rerun the harness. Expected: 6 changed, 0 unexplained. Or they ask Copilot to rewrite the developer's "behaviour preserved" note so it's accurate, or to write the message to the developer.
> [!NOTE]
> **The Lab as built.** CR-48830 adds a 3.00 fee for the 7th withdrawal and every one after (4th to 6th stay at 2.00). The developer also replaced `PERFORM 2000-CALC-FEE THRU 2099-EXIT` with `PERFORM 2000-CALC-FEE`, removed `2099-EXIT`, and rewrote the `GO TO 2099-EXIT` jumps as an `EVALUATE`. `2050-APPLY-FEE-CAP` is still in the source, but nothing reaches it. The harness report is **1,447 unchanged, 6 changed as expected, 47 unexplained**. The 6 expected are the SV accounts below the waiver with exactly 7 withdrawals: 13.00 becomes 14.00. All 47 unexplained are cap breaches, up to 31.50. Over the run, v2 charges 4,312.00 against 3,914.00, which is 398.00 more, across the same 525 accounts. `FEE_HISTORY` has 525 rows in both, so the overcharges are fully recorded. The full key is `facilitator/module-3/review-lab-key.md`.
 
**Answer key: the verdict**
 
-   **Verdict:** Not safe.
-   **Behaviour change:** the 15.00 regulatory fee cap is no longer applied.
-   **Who is affected:** SV accounts below the waiver balance with 8 or more withdrawals, and BS accounts below the waiver with 7 or more: 47 accounts in the test file, charged up to 31.50.
-   **Cause:** `PERFORM 2000-CALC-FEE THRU 2099-EXIT` was replaced by `PERFORM 2000-CALC-FEE`; `2050-APPLY-FEE-CAP` is now unreachable.
-   **Evidence:** reachability table for v2 compared with the baseline; harness report showing unexplained fee increases for these accounts; and a hand-checked trace, e.g. SV, balance 500.00, 9 withdrawals: original 5.00 + 12.00 = 17.00, capped to 15.00; v2 5.00 + 6.00 + 9.00 = 20.00, no cap.
-   **Conditions to release:** apply the cap to the total fee after `2100-CALC-ACTIVITY-FEE` (e.g. `PERFORM 2050-APPLY-FEE-CAP` at the end of the `WHEN OTHER` branch), then rerun the harness and expect 6 changed and 0 unexplained.
-   **Separately raised:** the two pre-existing security findings, to the program owner and security.
> [!TIP]
> When learners share verdicts at the end, ask: "If the developer's AI had been right and behaviour *was* preserved, what evidence would you have needed to say 'safe'?" It reinforces that a "safe" verdict needs as much evidence as a "not safe" one.
 
---
 
## **Wrap-Up Reflection**
**(Time: ~2 minutes, from the end of the Lab)**
 
-   **Key Takeaways:**
    -   Map the original before you read the change
    -   Review with a checklist: control flow, error handling, numeric fields, data impact, security
    -   Fall-through is invisible in the code and easy to lose. Ask for reachability, and say how `THRU` works.
    -   "Behaviour preserved" needs a run or a harness report behind it. A trace tells you where to look.
    -   A verdict names the behaviour, the customers, the line and the evidence
-   **Closing question for the room:** "How long does a review like this take you today, and how confident are you at the end of it?" Keep the answers for the client's throughput measure.
-   **Next Steps:** Take the break, then move to Module 4: the same evidence discipline, applied to a production incident at 2 a.m.
---
 
## **What Was Tested and What Was Not**
 
-   **Checked while authoring:** the fee rules, reachability table and risk register, against the `FEEASSES` source. The R8 thresholds and field sizes, against the `PIC` clauses.
-   **Run end to end:** `FEEASSES` and `FEEASSES-v2` compiled with GnuCOBOL 3.1 (`-std=ibm`) and run through the harness with the DB2 stand-in: 1,447 / 6 / 47. With the fix in the verdict key applied, it gives 1,494 unchanged, 6 changed as expected, 0 unexplained (rerun 8 November). The loop demo program runs both loop versions; only `WS-COUNT` 0 differs.
-   **Tested in the Skillable VM** on 7 November 2026 with Copilot and Claude Sonnet 5.5: every guided-practice prompt, the I Do demo and the Lab. What your assistant says will still vary from run to run, so run every prompt once before class.
-   **Still to confirm:** the R6 commit behaviour, with an HSBC SME or DBA. It has deliberately not been stated as fact. And the DB2 behaviour of the real program, which the stand-in doesn't reproduce.
