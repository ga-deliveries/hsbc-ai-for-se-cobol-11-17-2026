![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)


# Legacy COBOL Docs: Speaker Notes**
 
**Module Time:** 60 minutes
- 10-min content
- 20-min guided practice
- 30-min independent practice
---
 
**What This Module is About:**
 
Learners use GitHub Copilot Chat in VS Code to get oriented in a 30-year-old COBOL batch program they didn't write. They package the program with its copybooks and JCL, prompt for explanations they can check, extract the business rules into a spec, verify every rule against the source, and turn the result into Markdown documentation, Mermaid diagrams and a copybook data dictionary.
 
**Core Objectives:**  
- Assemble the right context for an assistant: program, copybooks and JCL
- Correlate JCL DD statements with `SELECT ... ASSIGN` clauses
- Prompt for paragraph maps, decoded 88-levels and business rules, with line citations
- Verify AI explanations against the source, and correct what's wrong
- Identify which programs a shared-copybook change could affect, and where the AI's view stops
- Produce documentation and diagrams a business stakeholder could read

**Must-Hit Topics:**  
- A COBOL program on its own is half a program. Without the copybooks and JCL, the model guesses.
- Orient, extract, cite, and always give the model the escape hatch: "If the code does not say it, write 'not in code'."
- An AI explanation is a hypothesis until you've read the line that proves it.
- The rules the model softens: closed accounts are **dropped**, frozen includes **legal hold**, daily interest is **truncated**, frozen accounts are still **capitalised**, and a balance overflow **loses** interest.
- `#codebase` sees only the workspace, and a header comment is a claim, not evidence.
> [!IMPORTANT]
> This is the learners' **first** hands-on experience with AI on COBOL. Discovery told us they use no AI tools today and trace logic entirely by manual search. The goal of this hour is not to impress them with what Copilot can do. It's to leave them with a method they trust, because they checked it themselves.
 
---
 
## 0. **Before the Session Starts**
**(Time: before the clock, about 10 minutes)**
 
-   **Purpose:** There is no spare time in this module. A setup problem found during the guided practice costs the whole room.
-   **Checklist, for every learner:**
    -   VS Code is open on `module-1` (`code ~/coursepack/module-1`): the Explorer's top line says **MODULE-1**, not COURSEPACK, and there's no Restricted Mode banner.
    -   The four files are visible in the Explorer: `INTCALC.cbl`, `ACCTREC.cpy`, `RATETIER.cpy` and `EODINT.jcl`.
    -   The COBOL extension is highlighting `INTCALC.cbl`.
    -   Copilot Chat is signed in, with a Claude model selected (Sonnet 5.5 in testing), target **Local** and mode **Ask**, and it answers a test question with `#file:ACCTREC.cpy` attached. On VS Code 1.141, untick **Default to Copilot Harness** and **Prefer Copilot Harness** in Settings, or every new chat reverts to Agent mode.
    -   An empty `INTCALC-spec.md` exists in the folder.
    -   Markdown preview draws Mermaid: right-click any `.md` file with a Mermaid block and choose **Open Preview**.

-   **Teaching Tips:**
    -   Run all of the guided-practice prompts yourself the day before, on the same Copilot setup the learners will use. Your answers will differ from the expected results in the lesson, and you need to know how.
    -   These engineers work in PCOM and IDZ, not VS Code. Expect a few to be slow with the Explorer, `Ctrl+Shift+F` and the chat panel. Pair anyone struggling with a neighbour now.
    -   Keyboard shortcuts may not reach the lab VM (Ctrl+Shift+F, Ctrl+Shift+V and Ctrl+H did nothing in the Skillable console). Teach the menu routes: Edit > Find in Files, right-click > Open Preview, Edit > Replace. And tell the room: leave the lab running during breaks; logging out ends it and loses your work.
---
 
## 1. **Welcome and Provocation: the Friday Audit Handoff**
**(Time: ~2 minutes, from the Content block)**
 
-   **Purpose:** Anchor the module in a pain the room already feels, then reframe the goal from "Can AI read COBOL?" to "Would I put my name on this explanation?"
-   **Talking Points:**
    -   Set the scene: it's Tuesday, Internal Audit wants a handoff pack for `INTCALC` by Friday, and the author retired years ago.
    -   Describe how this goes today: search one field at a time, flip to the copybook, sketch the flow on paper. Discovery heard exactly this, including "very terrible" hand-drawn diagrams.
    -   State the real goal: an explanation you'd be willing to sign, in front of an auditor, for code that moves customer money.
    -   Everything in the course pack is synthetic. Nothing is an HSBC system, program or customer.
-   **Teaching Tips:**
    -   Ask: "Who has had to explain a program they didn't write, against a deadline, in the last six months?" Let one person tell the story briefly. It sets the tone better than any slide.
> [!CAUTION]
> Say the data rule out loud now, and repeat it whenever a learner mentions their own code: **never paste production data, real account numbers or customer details into a prompt.** Mask first, every time. You'll come back to this in Module 4.
 
---
 
## 2. **Give the Model the Whole Picture**
**(Time: ~4 minutes)**
 
-   **Purpose:** Establish the context-packing habit before anyone writes a prompt.
-   **Talking Points:**
    -   Walk Figure 01 in the lesson: what each artifact gives the model, and what it guesses wrong without it.
    -   The key line: **`COPY ACCTREC.` is a pointer, not the content.** Copilot doesn't expand copybooks for you.
    -   Put the `FILE-CONTROL` excerpt and the `STEP030` JCL side by side. Trace one link aloud: `ACCT-IN-FILE` → `ASSIGN TO ACCTIN` → `//ACCTIN DD` → `PROD.SAV.ACCTMAST.SORTED`.
    -   Show Figure 02, the Mermaid data-flow diagram. Point out that it's text, and that they'll generate diagrams like it in the Lab.
    -   Demo how to attach files: type `#file:` and pick from the list, or drag a file from the Explorer. Show the file chip. (The chat's + button may only offer folders.)

-   **Teaching Tips:**
    -   Quick live contrast, if time allows: ask Copilot "What does `ACCT-FROZEN` mean?" with only `INTCALC.cbl` attached, then again with `ACCTREC.cpy`. The first answer may still find `'F'` and `'L'`, because Copilot opens the copybook on its own. Use that: ask the room who chose the evidence, and whether they'd know it had looked.
    -   Run the Discuss prompt only if the room is talkative. Otherwise, ask one person and move on. The guided practice is where the time goes.
---
 
## 3. **Prompts That Explain, Extract and Cite**
**(Time: ~4 minutes)**
 
-   **Purpose:** Show the difference between a fluent answer and a checkable one.
-   **Talking Points:**
    -   Live demo: run `Explain this program.` with only `INTCALC.cbl` attached. Read the answer aloud; with a current model it's often complete. Then ask: "Which of these claims could you verify in under a minute? Where's the line number? What did it assume?" It cites nothing, marks nothing "not in code", and may state "tier 1 is the lowest rate" as fact.
    -   Introduce **orient, extract, cite** (Figure 03), and the parts of a strong prompt: context, role, task, format, and rules.
    -   Highlight the escape hatch: "If the code does not say it, write 'not in code'." Without it, the model invents a reason rather than admit there isn't one.
    -   Walk Figure 04, the patterns AI gets wrong in COBOL. Don't explain each one in depth. Tell them they're about to find several of these themselves.
    -   Close with the rule: **an AI explanation is a hypothesis until you've read the line that proves it.**

-   **Teaching Tips:**
    -   Be explicit that you're performing, not presenting a model answer. Your assistant's output will differ from the learners', and from last time you ran it. That's part of the lesson.
    -   Leave the knowledge check as a question to carry into the guided practice. Its answer appears in Step 3.
> [!NOTE]
> **Knowledge check answer.** "Excluded" could mean *no interest is accrued* or *the record is removed from the output*. For zero and negative balances, it's only the first: they're still written to the new master, and still capitalised at month-end. The paragraph to read is `1000-PROCESS-ACCOUNT`.
 
---
 
## 4. **Guided Practice: Explain INTCALC and Extract its Business Rules**
**(Time: ~20 minutes: 4-min files, 5-min flow, 7-min rules, 4-min verify and impact)**
 
-   **Purpose:** Learners run each prompt themselves, compare answers, and check every claim against the source. The output is the first draft of `INTCALC-spec.md`.
-   **Talking Points:**
    -   Run each step the same way: everyone runs the prompt, two or three people read out what they got, then the room checks it against the code together.
    -   Remind them that their wording won't match the expected result. What matters is whether each claim matches the code.
    -   Tell the room to end each prompt with "Put your whole answer in a single Markdown code block" and paste with the block's Copy icon, so Copilot's working log stays out of the spec.
-   **Step-by-step notes:**
    -   **Step 1, map the files (4 min).** Draw out the three things the model rarely volunteers: `ACCTOUT` is tomorrow's master file; `EXCEPTION-FILE` has no `FILE STATUS`; `WS-ACCTOUT-STATUS` is declared but never tested. Have everyone confirm the last one with `Ctrl+Shift+F`. It's the first time they verify a claim themselves, so make a moment of it.
    -   **Step 2, map the flow (5 min).** If someone's model claims a paragraph is never performed, ask them to make it show the line. Watching it correct itself is one of the most useful moments in the hour. Send everyone to read `1000-PROCESS-ACCOUNT` slowly: the month-end check sits **outside** the `EVALUATE`.
    -   **Step 3, extract the rules (7 min).** This is the heart of the module. Don't show the answer key first. Ask the room to compare their rules with each other, then go through the five commonly missed rules one at a time, and ask: "Did your model get this? Does the line say what it said?" A good self-check finds a precondition: a dormant account with a zero or negative balance never reaches the tier rule (rule 4). Strong models also report extra findings (rate carry-over, unknown statuses accrue, ACCTOUT status and PARM never checked). They're correct: rules 14 to 16 in the facilitator key.
    -   **Step 4, verify and check impact (4 min).** Run the "quote the line" prompt and ask who got an UNSUPPORTED rule back. Then run the `#codebase` impact prompt. Land the point: the workspace only contains `INTCALC`, the header names four programs, and both of those facts are limits, not answers. Check the Explorer says MODULE-1 before the #codebase prompt; with the course pack open, the impact list spans every module. In testing the self-check missed a line slip (it cited the legal-hold comment as ACCTREC line 11; it's 12), and marked language behaviour ("balance unchanged on SIZE ERROR", "ROUNDED default mode") UNSUPPORTED. Both make good room moments.
> [!TIP]
> Pair a senior SME with a newer engineer for Step 3. The SME usually spots the business meaning, such as what legal hold implies for a customer, and the newer engineer is usually quicker with the prompting. Both learn something, and the discussion of rules 10 and 12 gets better.
 
> [!IMPORTANT]
> Rules 10 and 12 are the most valuable findings of the hour. Frozen and legal-hold accounts still have interest capitalised at month-end, and a balance overflow at capitalisation silently loses the customer's accrued interest. Don't call either one a bug. The code says *what* happens, not whether anyone intended it. They go in the spec under **Open questions for the business**. That framing is what an auditor wants to see.
 
-   **Teaching Tips:**
    -   The most common problem is the model saying "rounds" when the code truncates (rule 7). Ask the learner to find `ROUNDED` in `2000-ACCRUE-INTEREST`. It isn't there.
    -   If a learner's model refuses or gives a very short answer, check they attached all four files (a file chip for each) and that the Explorer says MODULE-1. Missing copybooks are the usual cause.
    -   If you're running behind, shorten Step 2, never Step 3 or 4. Verification is the point of the module.
**Answer key: INTCALC business rules (13)**
 
| #  | Rule | Where | Commonly missed? |
|----|------|-------|------------------|
| 1  | Closed accounts get no interest **and are removed from the new master** | `1000-PROCESS-ACCOUNT` | **Yes**: "skipped" |
| 2  | Frozen (`F`) **or legal hold (`L`)** (legal hold per the change-log comment): no interest, exception written | `ACCTREC`, `1000` | **Yes**: `L` dropped |
| 3  | Zero or negative balance: no interest, no exception | `1000` | |
| 4  | Dormant accounts with a positive balance get the tier 1 rate | `2100-DETERMINE-TIER` | The zero-balance case |
| 5  | Highest tier whose floor ≤ balance; applies to the **whole** balance; on the floor gets the higher tier | `2100` | Sometimes: "banded" |
| 6  | Bonus Saver + flag `Y` + zero withdrawals: +0.50 points, including dormant accounts; rate lives in `RATETIER.cpy` | `2000-ACCRUE-INTEREST` | The dormant case |
| 7  | Daily interest = balance × rate ÷ 100 ÷ days, **truncated** to 4 places | `2000` | **Yes**: "rounds" |
| 8  | 366 days if the run year divides by 4, else 365 (wrong for 2100) | `0100-INITIALISE` | |
| 9  | Accrued overflow: day's interest not added, exception written, still counted | `2000` | |
| 10 | Month-end (from `PARM`, not the calendar): accrued added to balance, rounded to 2 places, for every non-closed account **including frozen** | `1000`, `3000-CAPITALISE-INTEREST` | **Yes** |
| 11 | After capitalisation: accrued reset to zero, last-interest date set, withdrawals reset | `3000` | |
| 12 | Balance overflow at capitalisation: balance unchanged **but accrued still reset**, so interest is lost | `3000` | **Yes** |
| 13 | Only the first four rate records are loaded; extras ignored | `0200-LOAD-RATE-TABLE` | |
 
---
 
## 5. **Independent Practice: Documentation Lab**
**(Time: ~30 minutes: 8-min data dictionary, 10-min README, 8-min two Mermaid diagrams, 4-min corrections and time estimate)**
 
-   **Purpose:** Learners turn their verified spec into documentation they'd be comfortable showing a business analyst, and check every artifact against the source the same way.
-   **Talking Points:**
    -   They produce three files in `module-1`: `ACCTREC-dictionary.md`, `INTCALC-README.md` with a Mermaid flowchart and sequence diagram, and a Corrections section with a time estimate at the end of the README. The handout is in the course pack's `labs/` folder, not the module folder, so Copilot can't read its hints.
    -   The README is built **from the spec**, not from a fresh "explain this program" prompt. Anything in the README that isn't in the verified spec has to be checked before it stays.
    -   Diagrams are claims too. Every box and arrow should match a paragraph, a `PERFORM` or a DD statement.

-   **Teaching Tips:**
    -   Circulate during the flowchart step and ask each learner to point to the box for `3000-CAPITALISE-INTEREST` and show you the condition on the arrow into it. It should say month-end runs only.
    -   Most common Mermaid problems: labels with parentheses or quotes that aren't wrapped in double quotes, which breaks the render; paragraphs drawn as a straight line with no loops; and the closed-account path shown as "skip" instead of "not written".
    -   Most common data dictionary problems: `COMP-3` fields given as 9 or 10 bytes instead of 5; offsets that don't total 100.
    -   Typos seen in testing: a dictionary row calling tier 1 "the lowest" rate (not in code), a README naming `ACCOUT-REC` (no such field), and "Updated todo list" (Copilot's working log) pasted above each diagram.
    -   Fast finishers: ask Copilot to critique their README as an auditor would. Which sentence would an auditor challenge first?
> [!WARNING]
> Watch for learners who regenerate the business rules from scratch for the README rather than using their verified spec. The new version will read more smoothly and may quietly reintroduce "rounds" or "skipped". If you see it, ask them to diff the README's rules against the spec.
 
**Answer key: `ACCTREC` data dictionary**
 
| Field | `PIC` | Format | Offset | Length | Meaning |
|---|---|---|---|---|---|
| `ACCT-NUMBER` | `X(12)` | Text | 1 | 12 | Account number, the sort key |
| `ACCT-TYPE` | `X(02)` | Text | 13 | 2 | `SV` savings, `BS` Bonus Saver (`ST` Student Saver withdrawn in 2011, per comment) |
| `ACCT-STATUS` | `X(01)` | Text | 15 | 1 | `A` active, `D` dormant, `F` frozen, `L` legal hold (per comment), also frozen, `C` closed |
| `ACCT-BALANCE` | `S9(7)V99` | `COMP-3` | 16 | 5 | Ledger balance; maximum 9,999,999.99 |
| `ACCT-ACCRUED-INT` | `S9(5)V9(4)` | `COMP-3` | 21 | 5 | Interest accrued since last capitalisation, 4 decimal places |
| `ACCT-LAST-INT-DATE` | `9(08)` | Zoned | 26 | 8 | Date of last capitalisation, `CCYYMMDD` |
| `ACCT-BONUS-FLAG` | `X(01)` | Text | 34 | 1 | `Y` bonus eligible |
| `ACCT-WITHDRAWALS-MTD` | `9(03)` | Zoned | 35 | 3 | Withdrawals this month; reset at capitalisation |
| `FILLER` | `X(63)` | Text | 38 | 63 | Unused |
| | | | **Total** | **100** | |
 
> [!NOTE]
> `ACCT-LAST-INT-DATE` is a good verification question. Its name suggests it changes every day, but `INTCALC` only sets it at month-end capitalisation. If a learner's dictionary says "date interest was last calculated", ask them to find where it's set.
 
---
 
## **Wrap-Up Reflection**
**(Time: ~2 minutes, from the end of the Lab)**
 
-   **Key Takeaways:**
    -   Give the model the whole picture: program, copybooks and JCL
    -   Ask for answers you can check: orient, extract, cite, with "not in code" as the escape hatch
    -   An explanation is a hypothesis until you've read the line
    -   Documentation produced during analysis stays behind for the next person
-   **Closing question for the room:** "How long would the Friday handoff have taken you without this? And which of today's findings would you have missed?" Note the answers. They're useful evidence for the client's throughput measure.
-   **Next Steps:** Take the break, then move to Module 2. The `INTCALC-spec.md` they wrote today is reused as context in Module 2, so make sure everyone saves it.
> [!IMPORTANT]
> Remind learners to **save `INTCALC-spec.md`**. Module 2 attaches it to several prompts. Anyone who loses it can get the reference copy from you (`facilitator/module-1/INTCALC-spec-key.md`, which isn't on learner VMs), but they lose the benefit of their own verified work.
 
---
 
## **What Was Tested and What Was Not**
 
-   **Tested:** every prompt in the lesson and lab, run in the Skillable VM on 7 November 2026 with Claude Sonnet 5.5; the lab is built and timed (about 25 minutes for a prepared tester); the built-in Markdown preview draws the lab's Mermaid diagrams. Answers vary from run to run, so run them once yourself before class.
-   **Still to confirm with HSBC:** IBM Z Open Editor with Java 21, the Claude model available in their Copilot, and that `#file` and `#codebase` work in their environment.
 
