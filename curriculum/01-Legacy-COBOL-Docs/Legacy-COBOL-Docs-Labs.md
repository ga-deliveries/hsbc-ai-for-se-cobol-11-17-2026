![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Module 1 Lab Guide: Legacy COBOL Docs

## Introduction

Internal Audit wants a handoff pack for `INTCALC`, the nightly savings-interest program, by Friday. It was written in 1994, it has no documentation, and its author has retired. In this lab you turn the verified spec you built in guided practice into documentation a new team member and an auditor can both use, and you check every item against the source before you'd sign it.

---

## Learning Objectives

By the end of this lab, you'll be able to:

- Prompt Copilot Chat to derive a data dictionary from a copybook, including offsets and `COMP-3` lengths
- Turn a verified business-rule spec into a technical README that cites paragraphs and line numbers
- Generate Mermaid flowcharts and sequence diagrams from COBOL and JCL, and check them against the code
- Catch and record the places where the AI's answer didn't match the source

---

## Getting Started

### Prerequisites

Before beginning this lab, ensure you have:

- Completed the Module 1 guided practice, with a verified `INTCALC-spec.md` in the `module-1` folder
- VS Code open on the `module-1` folder, with no "Restricted Mode" banner
- Copilot Chat signed in, in **Ask** mode, with a **Claude** model selected
- Basic familiarity with COBOL copybooks, `PIC` clauses and JCL DD statements

### Setup Instructions

1. In VS Code, choose **File > Open Folder** and open `module-1` if it isn't already open.
2. Open `INTCALC.cbl` and check it's colour-coded. Ctrl+click on `ACCTREC` in the `COPY` statement; the copybook should open.
3. Open Copilot Chat (the chat icon in the side bar) and start a new chat with **+** (New Chat) at the top of the chat panel, or Ctrl+N.
4. Check that `INTCALC-spec.md` is in the Explorer and contains your inputs and outputs, business rules, open questions and impact note from guided practice.

### Lab Structure

- `INTCALC.cbl`: the program you're documenting
- `ACCTREC.cpy`: the account record layout
- `RATETIER.cpy`: the rate table and bonus rate
- `EODINT.jcl`: the nightly job that runs `INTCALC`
- `INTCALC-spec.md`: your verified spec from guided practice
- `zapp.yaml`: copybook lookup settings for the COBOL extension (leave it as it is)
- `ACCTREC-dictionary.md`: you'll create this in the lab
- `INTCALC-README.md`: you'll create this in the lab

## Exercise Overview

You'll produce three things in the `module-1` folder: a data dictionary for `ACCTREC`, a technical README for `INTCALC` with two Mermaid diagrams, and a short correction log with a time estimate. Your goal is documentation where every claim can be traced to a line of code, and where you've checked the AI's work rather than trusted it.

To create a file, right-click in the Explorer and choose **New File**, then paste Copilot's answer in. In **Agent** mode you can ask Copilot to create the file for you; if you do, read every line before you keep it.

### How to Work Through the Lab

1. **Build the `ACCTREC` data dictionary (8 minutes).** In a new chat, run:

    ```text
    #file:ACCTREC.cpy #file:INTCALC.cbl

    Create a data dictionary for the ACCTREC copybook as a Markdown table:
    Field | Level | PIC | Usage (DISPLAY or COMP-3) | Offset | Length (bytes) |
    88-level values and meanings | Business meaning |
    Paragraphs in INTCALC that read or change it

    Start offsets at 1 and give the total record length. A COMP-3 field's
    length is (number of digits + 1) / 2, rounded up. Cite line numbers.
    If a meaning is not in the code, write "not in code".
    ```

    Before you move on, check:
    - [ ] The offsets add up to **100** bytes
    - [ ] `ACCT-BALANCE` and `ACCT-ACCRUED-INT` are **5 bytes** each
    - [ ] Every 88-level condition lists **all** of its values. Check them against the copybook
    - [ ] Every "paragraphs" entry is real: search for the field name with **Edit > Find in Files** (Ctrl+Shift+F)

    Save the table as `ACCTREC-dictionary.md`.

2. **Write the technical README (10 minutes).** In the same chat, run:

    ```text
    #file:INTCALC-spec.md #file:INTCALC.cbl #file:EODINT.jcl

    Using the verified spec as the source of truth, write a README for
    INTCALC for a new team member and an auditor, in Markdown, with these
    sections: Purpose | How it runs (job, step, PARM) | Inputs and outputs |
    Processing for one account | Business rules (summarise, and refer to the
    spec) | Calculations (precision, truncation, rounding) | Exceptions and
    error handling, including what is NOT checked | Known risks and open
    questions | Shared copybooks and impact.
    Cite paragraph names and line numbers. Do not add anything that is not
    in the spec or the code; write "not in code" instead.
    ```

    Read the **Calculations** section against the `COMPUTE` statements in the code, not against the program's comments. Save the result as `INTCALC-README.md`.

3. **Add two Mermaid diagrams (8 minutes).** Ask for each diagram separately, in the same chat. Paste each one into the README under its own heading: **Processing flow** and **Job flow**.

    ```text
    Write a Mermaid flowchart (flowchart TD) of how INTCALC processes one
    account record, from reading it to writing it. Use paragraph names in the
    boxes and show every decision in 1000-PROCESS-ACCOUNT as a diamond,
    including the month-end check. Include only paths that exist in the code.
    ```

    ```text
    Write a Mermaid sequence diagram of job EODINT: the scheduler, STEP010,
    STEP020 (SORT) and STEP030 (INTCALC), and the datasets each step reads
    and writes. Use the DD names and dataset names from the JCL.
    ```

    To see the pictures, right-click the README in the Explorer and choose **Open Preview** (or press Ctrl+Shift+V). If a diagram shows as code instead of a picture, it has a syntax error: paste the error back into Copilot and ask it to fix it.

    Check the flowchart against `1000-PROCESS-ACCOUNT`:
    - [ ] What happens to a closed account matches the code, including whether it's written to the new master
    - [ ] What happens to a frozen account matches the code
    - [ ] The month-end check sits where it sits in the code, and applies to the accounts the code applies it to
    - [ ] There's no box or path that isn't in the code

4. **Check, correct and record (4 minutes).** Add two short sections at the end of the README:
    1. **Corrections:** every place you changed what the AI wrote, and why. Expect at least one. If you found none, look again at rounding, the 88-level values and closed accounts.
    2. **Time estimate:** *"Without AI, producing this pack would take me about ___. Today it took ___."* This feeds the team's sprint baseline.

5. **Finished early? Try a stretch task.**
    - Build the same data dictionary for `RATETIER.cpy`. Where does the bonus rate live?
    - Ask Copilot to review your README and list every sentence that has no line citation.
    - Draw the job's data flow yourself, then ask Copilot to check your drawing against `EODINT.jcl`.

### Tips for Success

- **Attach files by picking them.** Type `#file:` and choose the file from the list that pops up, or drag it from the Explorer into the chat. A file chip should appear. A pasted or mistyped file name can attach nothing, and Copilot answers without that file. (In some VS Code versions the paperclip or **+** in the chat box only offers folders; use `#file:` instead.)
- **Stay in Ask mode, and copy only the answer.** Ask mode can't edit files or run commands, but it can still open and read any file in the folder you have open, even ones you didn't attach. Its answer may include a working log ("Read… Completed with input…"). To copy only the answer, end your prompt with *"Put your whole answer in a single Markdown code block"*, then use the **Copy** icon on the code block. Check the mode dropdown in the chat box says **Ask**.
- **Comments are claims, not evidence.** If the AI's answer matches a header comment but not the code, the code wins.
- **A `COPY` statement is a pointer, not the content.** Attach the copybook whenever a prompt asks about its fields.
- **Click the line numbers.** An explanation is a hypothesis until you've read the line it cites.
- **Keep one chat for related steps** so later prompts can build on earlier answers. Start a new chat (**+** (New Chat) at the top of the chat panel, or Ctrl+N) when you change subject.
- **Read before you keep.** If Copilot creates or edits a file for you, review every line before you save it.

## Additional Resources

- [Manage context for AI in VS Code](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context): how `#file` and `#codebase` work
- [IBM Z Open Editor documentation](https://ibm.github.io/zopeneditor-about/Docs/introduction.html): the COBOL extension used in this course
- [Mermaid flowchart syntax](https://mermaid.js.org/syntax/flowchart.html)
- [Mermaid sequence diagram syntax](https://mermaid.js.org/syntax/sequenceDiagram.html)
- [PACKED-DECIMAL (COMP-3), IBM Enterprise COBOL for z/OS](https://www.ibm.com/docs/en/SS6SG3_6.3.0/perf/packed_decimal.html)
