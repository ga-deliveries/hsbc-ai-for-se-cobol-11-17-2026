![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Module 4 Lab Guide: Advanced Prompting & Workflow Redesign

## Introduction

It's Sunday 1 November, 04:12. Ticket SAVOPS-2317 says the statement-preparation job, STMTPREP, failed with RC=16 and SQLCODE -811 in program `CUSTLINK`, and November statements are blocked. They're due by Tuesday at noon, a regulatory deadline. You're the SME on call. In this lab you take the incident from ticket to a change that's ready for review, using the playbook and instructions file you built in guided practice.

---

## Learning Objectives

By the end of this lab, you'll be able to:

- Package and mask incident evidence before attaching it to a prompt
- Reuse saved playbook prompts to reach a root cause backed by evidence, then make the AI challenge its own diagnosis
- Trace the impact of a data problem across programs, jobs and processes, and separate confirmed impact from impact only named in a header
- Weigh AI-proposed recovery plans and fixes, and judge what each one would hide or break
- Write a change description (PR) you'd put your name on, and improve your playbook from what worked

---

## Getting Started

### Prerequisites

Before beginning this lab, ensure you have:

- Completed the Module 4 guided practice, with an `incident-playbook.md` containing at least three tested prompts, and a working `.github/copilot-instructions.md`
- VS Code open on the `module-4` folder, with no "Restricted Mode" banner
- Copilot Chat signed in, in **Ask** mode, with a **Claude** model selected
- Basic familiarity with embedded SQL in COBOL and DB2 SQLCODEs

### Setup Instructions

1. In VS Code, choose **File > Open Folder** and open `module-4` if it isn't already open.
2. Check that `.github/copilot-instructions.md` contains your rules from guided practice. Copilot adds it to every chat in this folder automatically.
3. Open `incident-playbook.md` so your prompts are ready to copy.
4. Create an empty file for your results: right-click the `incident-2` folder in the Explorer, choose **New File**, and name it `SAVOPS-2317-resolution.md`.

### Lab Structure

- `incident-2/SAVOPS-2317.md`: the incident ticket
- `incident-2/STMTPREP-joblog.txt`: the failing job's log
- `incident-2/CRMLOAD-joblog.txt`: the log of the job that loads customer data
- `incident-2/dba-query.txt`: the DBA's query results
- `incident-2/ACCT_CUSTOMER.sql`: the DDL for the account-customer table
- `incident-2/CUSTLINK.cbl`: the failing program
- `incident-2/DCLACUST.cpy` and `incident-2/STMTXREC.cpy`: `CUSTLINK`'s copybooks
- `incident-2/SAVOPS-2317-resolution.md`: you'll write your results here
- `incident-playbook.md`: your playbook from guided practice; you'll update it
- `.github/copilot-instructions.md`: your standing instructions for Copilot
- `src/` and `incident-1/`: the programs and evidence from guided practice, for reference

## Exercise Overview

You'll mask and package the evidence, find the root cause with your playbook prompts, trace the impact, and propose a recovery plan, a permanent fix and test scenarios. Your goal is a `SAVOPS-2317-resolution.md` file a reviewer could act on, ending in a change description, plus an `incident-playbook.md` that's better than it was this morning.

### How to Work Through the Lab

1. **Read the ticket, then check the masking and package the evidence (4 minutes).**
    1. Open `incident-2/SAVOPS-2317.md`. Note the deadline and the ops notes.
    2. Check the masking you did at the start of guided practice. Copilot can read any file in the open folder, so nothing in `incident-2` may stay unmasked. Open `STMTPREP-joblog.txt`, choose **Edit > Find** (or Ctrl+F), turn on **Use Regular Expression** (the `.*` button) and search for `7100\d{8}`, then for `C00\d{7}`. Do the same in `dba-query.txt`. All four searches should show **No results**.
    3. If a search finds anything, mask it now with **Edit > Replace** (or Ctrl+H) and **Replace All**, then check the first line of the file: the pattern belongs in the Find box, not the file.
        - Account numbers: find `7100(\d{4})(\d{4})`, replace with `7100****$2`. Unmasked, `STMTPREP-joblog.txt` has 4 and `dba-query.txt` has 4.
        - Customer IDs (`C00...`), which identify customers too: find `(C00\d)\d{6}`, replace with `$1******`. Unmasked, `dba-query.txt` has 3.
    4. Decide your evidence pack: the ticket, both job logs, the DBA query, the table DDL, `CUSTLINK.cbl`, `DCLACUST.cpy` and `STMTXREC.cpy`.

2. **Find the root cause (8 minutes).** Start a new chat with **+** (New Chat) at the top of the chat panel, or Ctrl+N.
    1. **Set the clock first.** Send this as your first message in the new chat: *"Treat the current time as Sun 1 Nov 2026 04:12, the time on the ticket."* Copilot otherwise uses today's real date, and will tell you the Tuesday deadline has already passed. Keep working in this same chat through Task 4; if you start a new chat, send the clock line again.
    2. Copy your playbook's **structured diagnosis** prompt and fill in its placeholders for this incident: job `STMTPREP`, step `STEP010`, `SQLCODE -811`, and the evidence files from step 1. **Replace every placeholder** before you send: Copilot can't use `<job>` or `<the first theory you heard>`, and a good model will tell you so.
    3. Then run your **self-challenge** prompt in the same chat. For the other engineer's diagnosis, use this one, which the night shift suggested:

        > "Joint accounts have two holders, so the SELECT returns two rows. Add FETCH FIRST 1 ROW ONLY to CUSTLINK's SELECT and rerun STMTPREP."

    If your diagnosis prompt asks for things this incident doesn't have (for example, the fields a dump shows as invalid, when there's no dump), the answer will say "not established". That's fine; note it for Task 5.

    Before you write anything down, verify it yourself:
    - [ ] Find the singleton `SELECT` in `CUSTLINK.cbl`, and what its `WHEN OTHER` does with a -811
    - [ ] Read `dba-query.txt`: what's identical, and what differs, between the rows the query returns?
    - [ ] Compare the `LOAD_TS` times in the DBA query with the start and end times of each LOAD step in `CRMLOAD-joblog.txt`. Does each row fall inside a run? Decide your own confidence, whatever the AI says
    - [ ] Find what *should* have stopped the duplicates in `ACCT_CUSTOMER.sql`

    Write the root cause and its evidence in your resolution file.

3. **Trace the impact (5 minutes).** In the same chat, run:

    ```text
    #codebase #file:DCLACUST.cpy #file:STMTXREC.cpy #file:SAVOPS-2317.md

    The ACCT_CUSTOMER table now has duplicate primary-holder rows for some
    accounts. List every program, job and process this affects, using the
    code and the copybook headers. For each: how it's affected, when it next
    runs or is used, and whether it's confirmed in this workspace or only
    named in a header. Also estimate how many accounts are affected, and
    say what evidence the estimate rests on. Say when customers would first
    notice, and which deadline in the ticket is at risk.
    ```

    Add the impact list to your resolution file. Mark anything only named in a header as "not confirmed".

4. **Propose recovery, a fix and tests (7 minutes).** In the same chat, run:

    ```text
    Propose (1) a recovery plan that gets statements out by the deadline,
    (2) a permanent fix, separating data, database, process and code
    changes, and (3) test scenarios for the CUSTLINK code change as a table:
    Scenario | Data set-up | Expected result. Include joint accounts, no
    link, duplicate primary rows, two different customers both flagged
    primary, a DB2 outage, an account with one holder, a closed account,
    and rerunning the same CRM delta after the fix. For each proposal, say
    what it would hide or break if it were the only change made.
    ```

    **Judge it yourself.** If the AI proposes `FETCH FIRST 1 ROW ONLY` as the fix, decide whether you'd accept it for statements that go to customers, and write down why. If it wants to check every duplicate against CRM data before fixing anything, look at the duplicate rows again: is there anything to choose between? The recovery decision is yours, not the AI's.

5. **Write the PR description, update your playbook and record your time (6 minutes).**
    1. Ask Copilot for a change description (PR) for the `CUSTLINK` change: problem, root cause, change, related changes on separate tickets, testing and risk. Edit it until you'd put your name on it, then paste it into your resolution file. Before you sign it, ask: *if this happens again next month, do the other customers' statements still go out?*
    2. Add any prompt that worked better than your playbook's version to `incident-playbook.md`. Make your diagnosis prompt fit any incident, not just an abend with a dump: for example, *"From the job log and any dump or error output, state the failing statement, the error, and the program's counters."* Add a line to set the clock: *"Treat the current time as `<incident time>`."*
    3. Record your estimate: *"Without AI, this investigation would take me about ___. Today it took ___."*

6. **Finished early? Try a stretch task.**
    - Ask Copilot to draft the DBA's data-fix request. Then list what you'd check before approving it.
    - Add a rule about singleton `SELECT`s to `.github/copilot-instructions.md`, and test it with a new chat.

### Tips for Success

- **Attach files by picking them.** Type `#file:` and choose the file from the list that pops up, or drag it from the Explorer into the chat. A file chip should appear. A pasted or mistyped file name can attach nothing, and Copilot answers without that file. (In some VS Code versions the paperclip or **+** in the chat box only offers folders; use `#file:` instead.)
- **Stay in Ask mode, and copy only the answer.** Ask mode can't edit files or run commands, but it can still open and read any file in the folder you have open, even ones you didn't attach. Its answer may include a working log ("Read… Completed with input…"). To copy only the answer, end your prompt with *"Put your whole answer in a single Markdown code block"*, then use the **Copy** icon on the code block. Check the mode dropdown in the chat box says **Ask**.
- **Mask before you attach.** Account numbers and customer IDs come out of every file before it goes into a prompt, even on synthetic data.
- **The first plausible answer is often wrong.** Use your self-challenge prompt before you act on any diagnosis.
- **Make it cite the evidence.** Every claim should point to a line in a log, the code or the DDL. Where the evidence doesn't establish something, the answer should say so.
- **Separate confirmed from named.** A program listed in a copybook header isn't confirmed impact until you've found it in the code.
- **Your playbook is the deliverable you keep.** Every prompt that works better today should go back into it.

## Additional Resources

- [Use custom instructions in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-instructions): how `.github/copilot-instructions.md` works
- [Adding repository custom instructions for GitHub Copilot in your IDE](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions-in-your-ide/add-repository-instructions-in-your-ide)
- [Manage context for AI in VS Code](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context): `#codebase` and `#file`
- [Sections of the Language Environment dump, IBM z/OS](https://www.ibm.com/docs/SSLTBW_2.4.0/com.ibm.zos.v2r4.ceea100/dumphd.htm): for reading a CEEDUMP like the one in incident 1
