![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Module 3: Safe Change Review
 
 
| Topic                                                              | Minutes |
|--------------------------------------------------------------------|---------|
| What a reviewer is actually checking                               | 5       |
| Prompting AI to review, and to compare two versions                | 5       |
| Guided practice: audit the original FEEASSES                       | 20      |
 
**Total lesson run time:** 30 min
 
## Table of Contents
 
- [Learning Objectives](#learning-objectives)
- [What You'll Need](#what-youll-need)
- [Introduction: the release is tomorrow](#introduction-the-release-is-tomorrow)
- [What a reviewer is actually checking](#what-a-reviewer-is-actually-checking)
- [Prompting AI to review, and to compare two versions](#prompting-ai-to-review-and-to-compare-two-versions)
- [Guided practice: audit the original FEEASSES](#guided-practice-audit-the-original-feeasses)
    - [Step 1, map how FEEASSES really runs (5 minutes)](#step-1-map-how-feeasses-really-runs-5-minutes)
    - [Step 2, list and rank the risks (6 minutes)](#step-2-list-and-rank-the-risks-6-minutes)
    - [Step 3, scan for hardcoded credentials (4 minutes)](#step-3-scan-for-hardcoded-credentials-4-minutes)
    - [Step 4, build the checklist for the Lab (5 minutes)](#step-4-build-the-checklist-for-the-lab-5-minutes)
- [Check for Understanding](#check-for-understanding)
- [Summary](#summary)
- [Vocabulary](#vocabulary)
## Learning Objectives
 
- Use a review checklist that covers control flow, error handling, numeric fields, data and copybook impact, and security.
- Prompt Copilot Chat to explain a program's control flow as a reachability table, and identify `GO TO` loops, `PERFORM ... THRU` fall-through and dead paragraphs.
- Flag unchecked `FILE STATUS` and `SQLCODE`, and rank each risk by what it would cost the bank if it went wrong.
- Scan COBOL and JCL for hardcoded credentials without repeating the secret, and know what to do with a finding.
- Make AI compare two versions by tracing their **behaviour** on the same small inputs, rather than summarising the text that changed.
- Recognise an AI review that sounds right and is wrong, and ask for the evidence that settles it.
## What You'll Need
 
Everything runs in VS Code. You'll read code and prompt in this lesson; you'll run the harness in the Lab.
 
- VS Code with the course COBOL extension, and GitHub Copilot Chat signed in
- The `module-3/` folder from the course pack, open as your workspace:
  - `FEEASSES.cbl`, the current production version
  - `FEEMTH.jcl`, the monthly job that runs it
  - `ACCTREC.cpy`, the same shared copybook you met in Module 1
  - `FEE_TABLES.sql`, the DB2 table definitions for `FEE_SCHEDULE` and `FEE_HISTORY`
  - `FEEASSES-v2.cbl`, the developer's delivery. **Don't open it yet.** You'll review it in the Lab, and you want to understand the original first.
  - `CR-48830.md`, the change request. **Don't open it yet.** You'll review it in the Lab, and you want to understand the original first.
  - `harness/`, the regression harness for this program: the same tool as Module 2, with a stand-in for DB2
- A new, empty file called `review-checklist.md`
> [!IMPORTANT]
> The credentials in this course pack are invented. When you scan your own team's code, follow the bank's policy on what may be shared with Copilot, ask the model to mask anything it finds, and report real findings through your security process. An AI scan is a fast first pass. It doesn't replace the bank's approved scanning tools.
 
## Introduction: the release is tomorrow
 
It's 4 p.m. A developer on the offshore team has delivered a new version of `FEEASSES`, the monthly maintenance-fee program. The change request asked for a new fee tier. While they were in there, they "cleaned up the old `GO TO` loop", and the code does look much tidier.
 
The release is tomorrow. The developer has signed off. Their AI assistant summarised the refactor as "structure modernised, behaviour preserved". Now it needs your sign-off, because you're the SME who owns this program.
 
Fees are taken directly from customer balances, and some fee rules exist because a regulator required them. If this change charges the wrong amount, the bank finds out from customer complaints, and then from the regulator.
 
Here's the uncomfortable truth about reviewing someone else's change: **you can't judge a change to code you don't understand.** Most review time goes into reading the new version and feeling reassured by how clean it looks. Clean is not the same as correct. The most dangerous defects in a COBOL refactor are behaviours the original had *without saying so*, the paragraph that only runs because the one above it falls into it.
 
So this session works on the original first. You'll use AI to map how `FEEASSES` really runs, list its risks, and scan it for secrets. You'll build the checklist you'll use on the developer's version in the Lab. You'll also practise the one review skill that matters most with AI: not accepting "behaviour preserved" until something has proved it.
 
## What a reviewer is actually checking
**Time:** 5 minutes
 
A review isn't a read-through. It's a set of specific questions, asked of every change, whether a person or an AI wrote it.
 
**Figure 01,** the change review checklist
 
| Area                          | Question to answer                                                                         | What usually goes wrong                                                   |
|-------------------------------|---------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| **Control flow**              | Does every paragraph that ran before still run, in the same cases, in the same order?       | Fall-through lost, a `GO TO` loop replaced by a loop with a different exit condition |
| **Error handling**            | Is every `FILE STATUS` and `SQLCODE` that matters checked, and what happens when it fails?  | A failed write or insert that nothing notices                             |
| **Numeric fields**            | Do the `PIC` sizes, signs, rounding and truncation still hold for every value?             | A field that silently loses its high-order digits                         |
| **Data and copybook impact**  | Which files, tables and shared copybooks does the change touch, and who else uses them?     | A layout change that breaks another program's next run                    |
| **Security**                  | Are there credentials, keys or override codes in the code or the JCL?                       | A password in an in-stream `SYSIN`, or a "temporary" override that's been there for years |
 
**Control flow is where COBOL refactors break.** Older COBOL often uses `PERFORM A THRU B`, which runs every paragraph from `A` down to `B`, in the order they appear in the source. A paragraph sitting between them runs even though no statement names it. That's called **fall-through**.
 
```cobol
           PERFORM 2000-CALC-FEE THRU 2099-EXIT
      *    ...
       2000-CALC-FEE.
      *        work out the fee, sometimes GO TO 2099-EXIT
       2050-APPLY-FEE-CAP.
      *        nobody PERFORMs this; it runs because 2000 falls into it
       2099-EXIT.
           EXIT.
```
 
To a modern eye, and to most AI models, a paragraph looks like a function. Refactor `PERFORM 2000-CALC-FEE THRU 2099-EXIT` into `PERFORM 2000-CALC-FEE`, and the program still compiles, still runs and still looks right. It just no longer runs `2050-APPLY-FEE-CAP`.
 
**A dead paragraph is the opposite problem:** code that looks like it matters and never runs. It wastes review time and misleads AI explanations, which tend to describe it as if it were live.
 
> **Discuss:**
>
> When you review a change today, which of the five areas in Figure 01 do you check every time, and which do you check only when something feels wrong?
 
## Prompting AI to review, and to compare two versions
**Time:** 5 minutes
 
Your instructor will run these live. The first is how you ask AI to map control flow; the second is how you ask it to compare two versions without being fooled.
 
**Ask for reachability, not a summary.** "Explain the control flow" gets you a story. A reachability table gets you something you can check, paragraph by paragraph.
 
```text
#file:FEEASSES.cbl
 
For every paragraph in the PROCEDURE DIVISION, return a table:
Paragraph | Reached from | How (PERFORM, PERFORM THRU range,
fall-through, GO TO, or never) | Conditions under which it runs
Treat every PERFORM ... THRU as running all paragraphs between the two
names, in source order. Cite line numbers.
```
 
The instruction about `PERFORM ... THRU` matters. Without it, models often list only the paragraphs a statement names, and the fall-through paragraph shows up as "never reached".
 
**Compare behaviour, not text.** The dangerous AI review is the confident one. Here is a small refactor of a `GO TO` loop that adds up a table of amounts:
 
```cobol
      * BEFORE                                * AFTER
           MOVE 1 TO WS-IDX                         PERFORM WITH TEST AFTER
       4000-SUM-LOOP.                                   VARYING WS-IDX FROM 1 BY 1
           IF WS-IDX > WS-COUNT                         UNTIL WS-IDX >= WS-COUNT
               GO TO 4000-EXIT                          ADD WS-AMT (WS-IDX)
           END-IF                                         TO WS-TOTAL
           ADD WS-AMT (WS-IDX) TO WS-TOTAL          END-PERFORM
           ADD 1 TO WS-IDX
           GO TO 4000-SUM-LOOP.
       4000-EXIT.
           EXIT.
```
 
Ask a model to "summarise the differences" and you'll usually get: *"Replaced the GO TO loop with a structured PERFORM VARYING. Behaviour is preserved."* That's wrong, and it's an easy mistake to make, because the two versions agree for every count of 1 or more. They differ only when the table is empty. `WITH TEST AFTER` runs the loop body *before* it tests the condition, so the new version always runs at least once: with `WS-COUNT` = 0, the original adds nothing, and the new version adds `WS-AMT (1)`. On real data, that's an item that isn't there: leftover data from the previous record, or a subscript out of range.
 
The fix is to make the model **trace both versions with the same small inputs**:
 
```text
Trace both versions with WS-COUNT = 0, 1 and 3. For each, show WS-IDX at
every iteration and the final WS-TOTAL. Then list every input for which
the two versions produce different results.
```
 
Tracing makes the model work through the logic step by step instead of pattern-matching it, and it hands you concrete cases you can check. But a trace is still the model's claim. Models often trace this loop wrongly: they assume the index goes up *before* the test, as in many other languages, and conclude that the new version "stops one item early" for larger counts. In COBOL, `TEST AFTER ... VARYING` tests the condition first and only then adds 1 to the index. When it matters, **run both versions**. That's what the harness does in the Lab.
 
**Figure 02,** what an AI review says, and what you need instead
 
| An AI review says                     | What you need before you believe it                                                       |
|---------------------------------------|-------------------------------------------------------------------------------------------|
| "Behaviour preserved"                 | A trace on the same small inputs, then a run or the harness report                        |
| "Refactored into structured COBOL"    | A reachability table for both versions, compared paragraph by paragraph                   |
| "Error handling improved"             | Every `FILE STATUS` and `SQLCODE` check listed, before and after                          |
| "No functional changes"               | A list of behaviour differences, including the ones the change request asked for          |
 
The rule from Module 1 still applies, with higher stakes. An AI's review is a hypothesis, and so is its trace. The evidence is a run, a harness report or a line of code you've read yourself.
 
**Knowledge check, why the trace?** In the loop example, which value of `WS-COUNT` would you choose if you were only allowed one trace, and why?
 
## Guided practice: audit the original FEEASSES
**Time:** 20 minutes
 
Work through four steps with your instructor. Run each prompt in your own Copilot Chat, compare your answer with the room's, then check it against the source. By the end you'll have a map of how `FEEASSES` runs, a ranked risk list, a security finding, and `review-checklist.md`, ready for the Lab. To paste an answer into your file without Copilot's working log, end each prompt with *"Put your whole answer in a single Markdown code block"* and use the code block's **Copy** icon.
 
### Step 1, map how FEEASSES really runs (5 minutes)
 
```text
#file:FEEASSES.cbl #file:ACCTREC.cpy #file:FEE_TABLES.sql
 
1. In plain language a business reader would follow, explain what
   FEEASSES does to one account, step by step, and every rule that
   decides whether a fee is charged and how much.
2. For every paragraph in the PROCEDURE DIVISION, return a table:
   Paragraph | Reached from | How (PERFORM, PERFORM THRU range,
   fall-through, GO TO, or never) | Conditions under which it runs
Treat every PERFORM ... THRU as running all paragraphs between the two
names, in source order. Cite line numbers. If something is not in the
code, write "not in code".
```
 
**Expected result, the fee rules**
 
1. Closed and frozen accounts (status `C`, `F` or `L`) are not charged. Their record is written to the new master unchanged.
2. The base fee and the waiver balance come from DB2 table `FEE_SCHEDULE`, for the account's type, on the run date.
3. If there is no schedule row for that type and date (`SQLCODE` 100), no fee is charged.
4. Any other SQL error stops the program with return code 16.
5. If the balance is at or above the waiver balance, no fee is charged.
6. An activity fee of 2.00 is added for every withdrawal over 3 in the month.
7. The total fee is capped at 15.00 a month. The header comment ties this to regulatory notice 17-114.
8. The fee is subtracted from the balance, even if that takes the account below zero.
9. Every fee is written to the fee report. It is also recorded in `FEE_HISTORY`, **unless** the run's `PARM` carries the operations override code.
**Expected result, reachability**
 
| Paragraph                    | Reached from               | How                                   | Runs when                                           |
|------------------------------|----------------------------|---------------------------------------|-----------------------------------------------------|
| `0000-MAIN`                  | Program entry              | n/a                                   | Always                                              |
| `0100-INITIALISE`            | `0000-MAIN`                | `PERFORM`                             | Once                                                |
| `1000-READ-LOOP`             | `0000-MAIN`, `1000-WRITE`  | `PERFORM THRU 1000-EXIT`, then `GO TO` | Once per record, a `GO TO` loop                    |
| `1000-WRITE`                 | `1000-READ-LOOP`           | Fall-through, or `GO TO`              | Every record except at end of file                  |
| `1000-EXIT`                  | `1000-READ-LOOP`           | `GO TO`                               | End of file                                         |
| `2000-CALC-FEE`              | `1000-READ-LOOP`           | `PERFORM THRU 2099-EXIT`              | Accounts that aren't closed or frozen               |
| **`2050-APPLY-FEE-CAP`**     | **`2000-CALC-FEE`**        | **Fall-through only**                 | **When `2000` reaches its last line without `GO TO 2099-EXIT`, that is, whenever a fee is calculated** |
| `2099-EXIT`                  | `2000-CALC-FEE`            | Fall-through, or `GO TO`              | End of every fee calculation                        |
| `2500-APPLY-STUDENT-WAIVER`  | Nothing                    | **Never**                             | Dead code                                           |
| `3000-APPLY-FEE`             | `1000-READ-LOOP`           | `PERFORM`                             | When the total fee is above zero                    |
| `9000-TERMINATE`             | `0000-MAIN`                | `PERFORM`                             | Once                                                |
 
**Check it against the source.** Two rows deserve your own eyes:
 
- **`2050-APPLY-FEE-CAP`.** Search for `2050` in VS Code (`Ctrl+Shift+F`). It appears exactly once, as its own paragraph name. Nothing performs it. The regulatory fee cap is applied purely because of where the paragraph sits in the source. If your model marked it "never reached", it ignored the `THRU` range, so correct the table.
- **`2500-APPLY-STUDENT-WAIVER`.** The same search finds no `PERFORM` and no `GO TO`. The change log says the student product closed in 2011, and `ACCTREC` has no `'ST'` account type. That's two independent pieces of evidence that it's dead.
Paste the fee rules and the reachability table into the top of `review-checklist.md`, under **Baseline: how the original behaves**.
 
### Step 2, list and rank the risks (6 minutes)
 
```text
#file:FEEASSES.cbl #file:ACCTREC.cpy #file:FEE_TABLES.sql
 
Act as a senior reviewer auditing this program before anyone changes it.
List every risk under these headings: control flow, error handling,
numeric fields, data and table impact. For each, give the paragraph and
line numbers, a severity (High, Medium or Low) based on the impact on
customers' money or the audit trail, and one sentence on why a reviewer
should care. Flag any behaviour that depends on paragraph order.
Do not suggest fixes yet.
```
 
**Expected result.** Your model may group or word these differently. Use this as the answer key.
 
**Figure 03,** risk register for the original `FEEASSES`
 
| #  | Area             | Risk                                                                                          | Where                     | Severity | Why a reviewer should care |
|----|------------------|-----------------------------------------------------------------------------------------------|---------------------------|----------|----------------------------|
| R1 | Control flow     | The 15.00 fee cap is reached only by fall-through inside `PERFORM 2000 THRU 2099`             | `2000`, `2050`            | **High** | Any restructuring of this range can silently remove a regulatory cap |
| R2 | Error handling   | The `INSERT` into `FEE_HISTORY` has no `SQLCODE` check                                        | `3000-APPLY-FEE`          | **High** | A customer can be charged with no history record, a gap in the audit trail |
| R3 | Error handling   | `WS-ACCTOUT-STATUS` and `WS-FEERPT-STATUS` are declared but never tested                     | `1000-WRITE`, `3000`      | Medium   | A failed write to the new master or the fee report goes unnoticed |
| R4 | Numeric fields   | A fee can take the balance below zero; there is no check                                      | `3000-APPLY-FEE`          | Medium   | A business question: is charging an account into overdraft intended? |
| R5 | Numeric fields   | Fee rules are hardcoded: the 15.00 cap, 2.00 per withdrawal, the threshold of 3               | `2000`, `2050`            | Medium   | A regulatory value can only change through a code release |
| R6 | Data impact      | No `COMMIT` in the program, and it can stop part-way through on an SQL error                  | `2000`, `3000`            | Medium   | What happens to the inserts already made depends on how the job ends. Confirm with your DBA, not the model |
| R7 | Control flow     | `2500-APPLY-STUDENT-WAIVER` is dead code                                                     | `2500`                    | Low      | It misleads readers and AI explanations. Remove it only with the owner's agreement |
| R8 | Numeric fields   | `WS-ACTIVITY-FEE` overflows silently at 503 or more withdrawals, and the cap currently hides it | `2000`                  | Low      | Mostly hidden by R1's cap. At 503 to 507 withdrawals it undercharges, and the cap can't hide that |
 
**Check it against the source.** Two things come up in almost every room:
 
- **R8 is easy to miss and worth understanding.** `COMPUTE` without `ON SIZE ERROR` drops the high-order digits when the result doesn't fit, so a 1,992.00 activity fee becomes 992.00. The cap then reduces it to 15.00, so nobody has ever noticed. Notice how R1 and R8 interact. A change that loses the cap doesn't just remove a limit; it exposes a second, hidden defect. Not always, though. At 503 withdrawals the activity fee should be 1,000.00 and truncates to 0.00, so the account pays only its base fee.
- **Watch for confident answers on R6.** Models will often state flatly that DB2 "rolls back" or "commits" on `STOP RUN`. The real answer depends on how the program ends and how the job is run, which is why the risk is phrased as a question. A reviewer who copies the model's confident answer into a sign-off is signing something they haven't checked.
Add the risk register to `review-checklist.md`, under **Known risks in the original**.
 
### Step 3, scan for hardcoded credentials (4 minutes)
 
```text
#file:FEEASSES.cbl #file:FEEMTH.jcl
 
Scan the COBOL and the JCL for hardcoded credentials, passwords, keys,
override codes or any other secret. For each finding, give the file,
line number, what the secret controls, and the risk.
Do not repeat any secret value in full: show at most its first four
characters followed by asterisks.
```
 
**Expected result**
 
| File          | Where                          | Finding                                      | What it controls                                    | Risk |
|---------------|--------------------------------|----------------------------------------------|-----------------------------------------------------|------|
| `FEEASSES.cbl` | `WS-OVERRIDE-CODE`            | Override code `OVRD****` as a literal        | Whether fees are recorded in `FEE_HISTORY`          | Anyone with the source, or a copy of the load module, can run the program so that fees are charged without a history record |
| `FEEMTH.jcl`  | `STEP020`, in-stream `INPUT`   | FTP user ID and password `Fee$****` in plain text | Access to the fee reconciliation server         | Readable by anyone who can read the JCL library, and sent over plain FTP, not an encrypted transfer |
 
**Check it against the source.** Look at how the prompt asked for masking. That habit matters more on real code than on the course pack: a model that repeats the secret in its answer has copied it into your chat history.
 
Then decide what a reviewer *does* with these findings. Neither one was introduced by the developer's change, so neither is a reason to reject this release on its own. They are findings to raise, to the program owner and through the security process, with the credential rotated. But if the developer's version touches either line, or adds a new secret, that **is** part of this review.
 
Add both findings to `review-checklist.md`, under **Security findings**, with masked values only.
 
Then search `review-checklist.md` for the override code (**Edit > Find**, or Ctrl+F) and mask every copy. Step 1's plain-language answer may already have printed it in full. Masking in the prompt only covers what the model decides is secret, so mask before you attach.
 
### Step 4, build the checklist for the Lab (5 minutes)
 
Now turn everything you found into the checklist you'll apply to `FEEASSES-v2.cbl`. Ask the model to draft it from your notes, then edit it as a room.
 
```text
#file:review-checklist.md
 
Using the baseline behaviour, risk register and security findings in
this file, write a review checklist for a refactored version of
FEEASSES. Organise it under: control flow, error handling, numeric
fields, data and table impact, security, and evidence. Each item must be
a yes/no question a reviewer can answer by reading code or running the
harness. Include a check for every paragraph the baseline shows is
reached only by fall-through.
```
 
**Expected result.** A good checklist looks something like this. Edit yours until every item is something you could actually answer tomorrow morning.
 
```markdown
## Review checklist: FEEASSES refactor
 
### Control flow
- [ ] Does every paragraph in the baseline reachability table still run, in the same cases?
- [ ] Is every paragraph that was reached only by fall-through still executed, on every path that reached it before?
- [ ] Does every replaced GO TO loop have the same exit condition, tested at the same point? (Trace with 0, 1 and 3 records, then run it.)
- [ ] Has any dead code been removed, and has the owner agreed?
 
### Error handling
- [ ] Is every FILE STATUS and SQLCODE that was checked before still checked?
- [ ] Are any of the unchecked ones (R2, R3) now checked, and is that change intended?
 
### Numeric fields
- [ ] Are fee field PICs, signs and rounding unchanged, except where the change request says otherwise?
- [ ] Is the 15.00 cap still applied to the total fee, including the new fee tier?
 
### Data and table impact
- [ ] Are FEE_SCHEDULE and FEE_HISTORY used in the same way, with the same columns?
- [ ] Does the change touch ACCTREC or any other shared copybook?
 
### Security
- [ ] Are the existing findings unchanged, and has no new secret been added?
 
### Evidence
- [ ] Does the harness report match the expected changes for the new fee tier, and nothing else?
- [ ] Is every unexplained difference explained, with the line that causes it?
```
 
**Check it against the source.** Read every item and ask, "Could I answer this with a yes or a no, and point to the evidence?" Cut or rewrite anything vague, like "Is the code good quality?" A checklist item that can't fail isn't checking anything. If your checklist mentions anything that isn't in `review-checklist.md`, such as the new fee tier, Copilot read another file in the folder. Check where each item came from before you rely on it.
 
> **Discuss:**
>
> The developer's AI said "behaviour preserved". After this audit, which two checklist items would you check first, and why those two?
 
## Check for Understanding
 
Before moving into the Lab, work through these, out loud or on paper.
 
- Explain fall-through to a developer who has only written Java. Why does `2050-APPLY-FEE-CAP` run, when nothing performs it?
- A refactor replaces `PERFORM 2000-CALC-FEE THRU 2099-EXIT` with `PERFORM 2000-CALC-FEE`. It compiles and runs cleanly. What has changed for a customer below the waiver balance with twelve withdrawals this month?
- Copilot reviews a change and says "behaviour preserved". Name two pieces of evidence you'd want before you agreed.
- Why does the credentials scan prompt ask the model to mask what it finds?
- The security findings weren't introduced by the developer's change. When would they still block your sign-off?
## Summary
 
- You can't review a change to code you don't understand. Map the original first.
- A review is a checklist, not a read-through: control flow, error handling, numeric fields, data and copybook impact, and security.
- `PERFORM ... THRU` runs every paragraph in its range. Fall-through behaviour is invisible in the code and easy for a refactor, or an AI, to lose.
- Ask AI for a reachability table, not a summary, and tell it how `THRU` ranges work.
- Make AI compare versions by tracing their behaviour on the same small inputs. "Behaviour preserved" is a claim until a run, or the harness, proves it. A trace tells you where to look.
- Rank risks by impact on customers' money and the audit trail. Where the honest answer is "it depends", write it as a question, not as the model's confident guess.
- Scan for secrets with masked output, and report what you find. A finding you didn't cause still needs an owner.
In the Lab, you open `FEEASSES-v2.cbl`, apply your checklist, generate an AI before-and-after comparison, run the regression harness, and find the behaviour change hiding in the refactor. Then you write a change-safety verdict that a stakeholder can act on.
 
## Vocabulary
 
- **Change review**, checking a change against a defined set of questions before it's approved for release. The reviewer signs off on behaviour, not on how the code looks.
- **Change-safety verdict**, a short written decision on whether a change is safe to release: safe, safe with conditions, or not safe, with the evidence for it.
- **Control flow**, the order in which a program's statements and paragraphs actually run, including loops, jumps and fall-through.
- **`PERFORM ... THRU`**, a COBOL statement that runs every paragraph from the first named paragraph to the second, in source order.
- **Fall-through**, execution continuing from one paragraph into the next because nothing stopped it. Inside a `PERFORM ... THRU` range, this is how unnamed paragraphs run.
- **`GO TO` loop**, a loop built from a paragraph that jumps back to its own start, usually exiting with a `GO TO` to an exit paragraph.
- **Dead code**, a paragraph or statement that can never run. It still costs reading time, and misleads AI explanations.
- **Reachability table**, a list of every paragraph, what reaches it, how, and under what conditions. The baseline you compare a refactor against.
- **`FILE STATUS`**, a two-character field the system sets after every file operation. `'00'` is success. Declaring it does nothing unless the program tests it.
- **`SQLCODE`**, the DB2 return code after every SQL statement. 0 is success, 100 is "no row found", and a negative value is an error.
- **Unit of work**, the set of database changes that are committed or rolled back together. A program with no `COMMIT` makes the whole run one unit of work.
- **Hardcoded credential**, a password, key or access code written directly into source or JCL, where anyone who can read the file can use it.
- **Behavioural comparison**, comparing two versions of a program by what they do with the same inputs, rather than by which lines of text changed.
 
