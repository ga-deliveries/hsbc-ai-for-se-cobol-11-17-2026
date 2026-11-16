![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Module 3 Lab Guide: Safe Change Review

## Introduction

It's 4 p.m. and the release is tomorrow. The developer's version of `FEEASSES`, the monthly fee program, is in, along with a note saying Copilot reviewed it: "structure modernised, behaviour preserved." In this lab you apply the checklist you built in guided practice, gather evidence with traces and the regression harness, and write the sign-off decision a stakeholder can act on.

---

## Learning Objectives

By the end of this lab, you'll be able to:

- Compare two versions of a COBOL program and spot changes the change request didn't ask for
- Prompt Copilot Chat for a paragraph reachability table that accounts for `PERFORM ... THRU` ranges and fall-through
- Make the AI trace both versions on the same small inputs, and verify one trace by hand
- Use regression-harness evidence to confirm or reject a "behaviour preserved" claim
- Write a change-safety verdict with evidence, the fix required and findings to raise separately

---

## Getting Started

### Prerequisites

Before beginning this lab, ensure you have:

- Completed the Module 3 guided practice, with a `review-checklist.md` in the `module-3` folder containing your baseline, risk register, security findings and checklist
- VS Code open on the `module-3` folder, with no "Restricted Mode" banner
- Copilot Chat signed in, in **Ask** mode, with a **Claude** model selected
- Basic familiarity with `PERFORM`, `PERFORM ... THRU` and embedded SQL in COBOL

### Setup Instructions

1. In VS Code, choose **File > Open Folder** and open `module-3` if it isn't already open.
2. Open `review-checklist.md` and check that your baseline reachability table and checklist are there. You'll compare against them.
3. Check the harness task is available: **Terminal > Run Task...** should list "Run regression: FEEASSES vs FEEASSES-v2".
4. Start a new chat in Copilot Chat with **+** (New Chat) at the top of the chat panel, or Ctrl+N.

### Lab Structure

- `FEEASSES.cbl`: the production version
- `FEEASSES-v2.cbl`: the developer's refactored version, which you're reviewing
- `CR-48830.md`: the change request and the developer's note
- `FEEMTH.jcl`: the monthly job that runs `FEEASSES`
- `ACCTREC.cpy`: the account record layout
- `FEE_TABLES.sql`: the DB2 table definitions and fee schedule rows
- `data/ACCTMAST-20261031.dat`: 1,500 synthetic accounts for the month-end run
- `harness/`: the pre-built regression tools, including a stand-in for DB2
- `review-checklist.md`: your work from guided practice; you'll complete it in the lab
- `reports/`: created by the harness when you run it

## Exercise Overview

You'll read the change, make the AI review the code's behaviour rather than its text, run the regression harness, and answer your checklist with evidence. Your goal is a completed checklist and a **Verdict** section in `review-checklist.md` that tells a stakeholder whether this change can go in tomorrow, and why.

### How to Work Through the Lab

1. **Read the change, then the code (5 minutes).**
    1. Open `CR-48830.md`. Note what was asked for and what the developer says they did.
    2. Select `FEEASSES.cbl` and `FEEASSES-v2.cbl` in the Explorer (Ctrl+click the second one), then right-click and choose **Compare Selected**.
    3. Scroll the differences. Note every change that **wasn't** in the change request.

2. **Make the AI review behaviour, not text (10 minutes).** In a new chat, run:

    ```text
    #file:FEEASSES-v2.cbl #file:review-checklist.md

    For every paragraph in the PROCEDURE DIVISION of FEEASSES-v2, return a
    reachability table: Paragraph | Reached from | How (PERFORM, PERFORM THRU
    range, fall-through, GO TO, or never) | Conditions under which it runs.
    Treat every PERFORM ... THRU as running all paragraphs between the two
    names, in source order. Cite line numbers. Then compare it with the
    baseline table in review-checklist.md and list every paragraph whose
    reachability changed.
    ```

    Then, in the same chat:

    ```text
    #file:FEEASSES.cbl #file:FEE_TABLES.sql

    Trace the original and v2 for these accounts on run date 2026-10-31,
    using the FEE_SCHEDULE rows in FEE_TABLES.sql. Show the base fee,
    activity fee, any cap applied, and the total fee for each version:
      A: type SV, status A, balance 500.00, 7 withdrawals
      B: type SV, status A, balance 500.00, 9 withdrawals
      C: type BS, status A, balance 2,000.00, 7 withdrawals
    Which results differ, and is each difference asked for by CR-48830?
    ```

    **Verify one trace by hand** before you trust it. Work out the base fee, the activity fee for each withdrawal tier and any cap, using the rules in your baseline and the change request.

3. **Run the harness (7 minutes).** Press **Ctrl+Shift+B**, or choose **Terminal > Run Task > Run regression: FEEASSES vs FEEASSES-v2 (2026-10-31)**. The report is saved in `reports/regression-FEEASSES-2026-10-31.txt`.
    - Read the first line: unchanged, changed as expected, unexplained.
    - Read the reason code next to the unexplained accounts.
    - Compare **FEES CHARGED** for the two versions: totals and the largest single fee.
    - Pick two unexplained accounts and work out their fee by hand. Does it match your trace?

4. **Apply the checklist and write the verdict (8 minutes).** In `review-checklist.md`:
    1. Answer every checklist item **yes or no, with its evidence**: a line number, a trace or the harness report.
    2. Under a **Verdict** heading, write *Safe*, *Safe with conditions* or *Not safe*; the evidence in two or three sentences; the fix required; and anything to raise separately.
    3. Add your time estimate: *"Without AI, this review would take me about ___. Today it took ___."*

5. **Finished early? Try a stretch task.**
    - **Fix it and prove it.** The harness always tests the file named `FEEASSES-v2.cbl`, so back it up first: run `cp FEEASSES-v2.cbl FEEASSES-v2-original.cbl` in the terminal. Then make the fix your verdict calls for in `FEEASSES-v2.cbl`, save, and rerun the harness. Write down the result you expect *before* you run it. When you're done, put the original back with `cp FEEASSES-v2-original.cbl FEEASSES-v2.cbl`.
    - Ask Copilot to rewrite the developer's "behaviour preserved" note so it's accurate.

### Tips for Success

- **Attach files by picking them.** Type `#file:` and choose the file from the list that pops up, or drag it from the Explorer into the chat. A file chip should appear. A pasted or mistyped file name can attach nothing, and Copilot answers without that file. (In some VS Code versions the paperclip or **+** in the chat box only offers folders; use `#file:` instead.)
- **Stay in Ask mode, and copy only the answer.** Ask mode can't edit files or run commands, but it can still open and read any file in the folder you have open, even ones you didn't attach. Its answer may include a working log ("Read… Completed with input…"). To copy only the answer, end your prompt with *"Put your whole answer in a single Markdown code block"*, then use the **Copy** icon on the code block. Check the mode dropdown in the chat box says **Ask**.
- **"Behaviour preserved" is a claim.** A trace tells you where to look; a run, or the harness, settles it.
- **A trace beats a summary, but it's still the model's claim.** When the AI says two versions are equivalent, ask it to trace both, check one trace by hand, and let the harness have the final word.
- **Watch for fall-through.** In code that uses `PERFORM ... THRU`, a paragraph can run without anything performing it by name. Check how each paragraph is reached, not just whether it's still in the source.
- **Small inputs find big differences.** Choose trace values on either side of each boundary in the fee rules.
- **Mask secrets before you attach, not just in the prompt.** A model that decides a value isn't secret may repeat it in full, whatever the prompt says. Mask any credential in your notes too.

## Additional Resources

- [PERFORM statement, IBM Enterprise COBOL for z/OS Language Reference](https://www.ibm.com/docs/en/SS6SG3_6.4.0/lr/ref/rlpsperf.html): including how `THRU` ranges run
- [Manage context for AI in VS Code](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context)
- `harness/README.txt` in this folder: what the harness does, and how it stands in for DB2
