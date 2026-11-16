![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Module 2 Lab Guide: Prove the Rate Change Is Safe

## Introduction

The developer says the change that moves savings rates from two to three decimal places is ready. Your scenarios and boundary records from guided practice are agreed. In this lab you prove the change on the full 5,000-account test file by running the original and modified `INTCALC` side by side, and you explain every difference nobody predicted.

---

## Learning Objectives

By the end of this lab, you'll be able to:

- Run a regression harness that compares the original and modified program on the same input
- Read a regression report and separate unchanged, expected and unexplained differences
- Check the program's actual output against the expected results in your scenario table
- Use AI to explain an unexplained difference down to the line, the `PIC` clause and the input data, and verify the explanation by hand
- Write a change-safety verdict, and raise defects the change exposes but didn't cause

---

## Getting Started

### Prerequisites

Before beginning this lab, ensure you have:

- Completed the Module 2 guided practice, with a checked `scenarios.md` and a `boundary-records.csv` that packs without errors, both in the `module-2` folder
- `INTCALC-spec.md` from Module 1 copied into `module-2`
- VS Code open on the `module-2` folder, with no "Restricted Mode" banner
- Copilot Chat signed in, in **Ask** mode, with a **Claude** model selected
- Basic familiarity with `COMP-3` fields and `PIC` clause limits

### Setup Instructions

1. In VS Code, choose **File > Open Folder** and open `module-2` if it isn't already open.
2. Check that `boundary-records.csv` is saved in `module-2`. The harness adds it to the 5,000 accounts automatically.
3. Check the harness tasks are available: **Terminal > Run Task...** should list the "Run regression" tasks.
4. Create an empty file for your results: right-click in the Explorer, choose **New File**, and name it `lab-findings.md`.

### Lab Structure

- `INTCALC.cbl` and `RATETIER.cpy`: the current production versions
- `INTCALC-v2.cbl` and `RATETIER-v2.cpy`: the modified versions for the rate change
- `ACCTREC.cpy`: the account record layout (unchanged)
- `rates-current.csv` and `rates-new.csv`: the old and new rate tier files
- `data/ACCTIN-5000.dat`: the 5,000-account synthetic test file
- `harness/`: the pre-built regression tools (`run-regression`, `pack-records`)
- `scenarios.md` and `boundary-records.csv`: your work from guided practice
- `INTCALC-spec.md`: your Module 1 spec
- `lab-findings.md`: you'll write your results here
- `reports/`: created by the harness when you run it

## Exercise Overview

You'll run the regression harness on the daily run, check your own boundary records first, then investigate the differences the harness can't explain. Your goal is a `lab-findings.md` file with your results, an explanation of every unexplained difference that you've checked against the code, and a verdict a release manager could act on.

### How to Work Through the Lab

1. **Run the regression (4 minutes).**
    1. Press **Ctrl+Shift+B**, or choose **Terminal > Run Task > Run regression: daily run D (20261030,N)**.
    2. Read the first result line: **N unchanged, M changed as expected, X unexplained**.
    3. The full report is saved in `reports/regression-20261030-N.txt`, along with both programs' exception reports. Open it in VS Code.

2. **Check your own records first (6 minutes).** Scroll to **YOUR ADDED RECORDS** in the report. For each `BND` record, compare the original and modified output with the Expected columns in `scenarios.md`. The daily report checks your **run D** scenarios. A month-end (M) or leap-year (L) scenario will look different here because it's on the wrong run: it accrues one ordinary day. Check those against the month-end and leap-year tasks instead (stretch task).
    - [ ] Every record marked **Should change? Yes** shows **DIFFERENT**, by the amount you calculated
    - [ ] Every record marked **No** shows **SAME**, or "no output" if the program drops that record
    - [ ] Every mismatch is explained. Is your expected value wrong, or the program? Check it by hand

    Copy a short table of any mismatches into `lab-findings.md`.

3. **Explain the unexplained (14 minutes).** The report lists the **unexplained differences**: accounts nobody's scenario predicted. Start a new chat (**+** (New Chat) at the top of the chat panel, or Ctrl+N) and run:

    ```text
    #file:INTCALC.cbl #file:INTCALC-v2.cbl #file:ACCTREC.cpy
    #file:regression-20261030-N.txt
    #file:exceptions-original-20261030-N.txt #file:exceptions-modified-20261030-N.txt

    For each unexplained difference in the regression report, explain the
    cause. Cite the paragraph and line in INTCALC, the PIC clause involved,
    and the matching line in the exception reports. Show the arithmetic for
    the original and modified daily interest. Say whether the difference is
    caused by the rate change or by existing behaviour that the rate change
    exposes, and what in the input records makes these accounts different
    from the rest.
    ```

    Before you write anything down, verify it yourself:
    - [ ] Find the statement and line the AI cites, and the `PIC` of the field involved in `ACCTREC.cpy`
    - [ ] Hand-check one account's arithmetic against that field's maximum value, for both versions
    - [ ] Find the matching lines in both exception reports
    - [ ] Look at the accounts' input: account range, last-interest date and accrued interest. What do they have in common, and how might they have got that way?

    Write one short paragraph per account in `lab-findings.md`.

4. **Conclude (6 minutes).** In `lab-findings.md`, write:
    1. **Verdict:** *safe*, *safe with conditions* or *not safe*, with the evidence: the counts and your explanations.
    2. **Anything to raise separately:** any defect that isn't caused by this change but that the change exposed.
    3. **Time estimate:** *"Without AI, building these scenarios and test data would take me about ___. Today it took ___."*

5. **Finished early? Try a stretch task.**
    - Run the **month-end** and **leap-year** tasks (**Terminal > Run Task**). The unexplained count changes. Explain why for each run.
    - Run **5,000 accounts only** and explain why its counts differ from your run.
    - Ask Copilot: *"Parity is not correctness. Using my scenarios and the unexplained accounts, explain what that means for this release."*

### Tips for Success

- **Attach files by picking them.** Type `#file:` and choose the file from the list that pops up, or drag it from the Explorer into the chat. A file chip should appear. A pasted or mistyped file name can attach nothing, and Copilot answers without that file. (In some VS Code versions the paperclip or **+** in the chat box only offers folders; use `#file:` instead.)
- **Stay in Ask mode, and copy only the answer.** Ask mode can't edit files or run commands, but it can still open and read any file in the folder you have open, even ones you didn't attach. Its answer may include a working log ("Read… Completed with input…"). To copy only the answer, end your prompt with *"Put your whole answer in a single Markdown code block"*, then use the **Copy** icon on the code block. Check the mode dropdown in the chat box says **Ask**.
- **End your CSV with a line break.** Before you add rows to `boundary-records.csv`, click at the end of the last row and press **Enter**. Otherwise the first new row joins onto the last old one, and `pack-records` rejects the file.
- **Read the first line of the report before anything else.** The counts tell you where to look.
- **The source code is the referee.** If the AI's explanation and the code disagree, the code wins.
- **Models are fluent at COBOL and unreliable at arithmetic.** Work at least one calculation yourself, and remember `INTCALC` truncates daily interest; it doesn't round.
- **Identical output isn't the same as correct output.** An account can match in both versions and still be wrong in both.
- **Never copy production records into a test file or a prompt.** Everything in this lab is synthetic; keep it that way on your own work.

## Additional Resources

- [Manage context for AI in VS Code](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context): attaching files with `#file`
- [PACKED-DECIMAL (COMP-3), IBM Enterprise COBOL for z/OS](https://www.ibm.com/docs/en/SS6SG3_6.3.0/perf/packed_decimal.html)
- [Formats for numeric data, IBM Enterprise COBOL for z/OS](https://www.ibm.com/docs/SS6SG3_6.2.0/pg/concepts/cpari09.html)
- `harness/README.txt` in this folder: every harness option, and how to run it from a terminal
