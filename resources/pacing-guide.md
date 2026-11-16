![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Pacing Guide: AI for Software Engineers (COBOL)

**Total time:** 240 minutes over two sessions: Session 1 (120 minutes, Modules 1 and 2) and Session 2 (120 minutes, Modules 3 and 4). Every module is 60 minutes: I Do (10), We Do (20) and You Do (30). No breaks are scheduled inside a session.

---

## Agenda

### Session 1 · Understand and Test Legacy COBOL · Day 1 · 120 minutes

| Time | Segment | Format |
|---|---|---|
| 0:00 - 0:10 | **Module 1 I Do:** Legacy COBOL Docs (includes 2 minutes of welcome and the Friday audit provocation) | 4 min context packing, 4 min live prompt demo: vague versus orient, extract, cite |
| 0:10 - 0:30 | **Module 1 We Do:** Explain INTCALC and extract its business rules | Step 1 map the files (4), Step 2 map the flow (5), Step 3 extract the rules (7), Step 4 verify and check impact (4) |
| 0:30 - 1:00 | **Module 1 You Do:** Lab: Document INTCALC for an Audit (includes a 2-minute wrap-up) | Data dictionary (8), README (10), two Mermaid diagrams (8), corrections and time estimate (4) |
| 1:00 - 1:10 | **Module 2 I Do:** COBOL Testing (includes the rate-precision provocation) | 2 min provocation, 4 min scenarios before data, 4 min records and packed decimal |
| 1:10 - 1:30 | **Module 2 We Do:** Scenario table and boundary records for the rate change | Step 1 the change (4), Step 2 scenario table (8), Step 3 boundary records (6), Step 4 coverage (2) |
| 1:30 - 2:00 | **Module 2 You Do:** Lab: Regression harness (includes a 2-minute recap and Day 1 close) | Add records (5), run the harness (5), read the report (8), explain the differences (10), close (2) |

**Output:** each learner's `INTCALC-spec.md`, `ACCTREC-dictionary.md`, `INTCALC-README.md`, `scenarios.md` and `boundary-records.csv`. Learners keep the Module 2 harness for Session 2.

### Session 2 · Review Changes and Resolve Incidents · Day 2 · 120 minutes

| Time | Segment | Format |
|---|---|---|
| 0:00 - 0:10 | **Module 3 I Do:** Safe Change Review (includes a one-minute Day 1 recap in the opening) | Scene and developer's note (2), what a reviewer checks (2), Demo A summary (3), Demo B trace (2), name the pattern (1) |
| 0:10 - 0:30 | **Module 3 We Do:** Audit the original FEEASSES | Step 1 map how it runs (5), Step 2 rank the risks (6), Step 3 scan for credentials (4), Step 4 build the checklist (5) |
| 0:30 - 1:00 | **Module 3 You Do:** Lab: Review the refactor and write the verdict | Read the change (5), AI reviews behaviour (10), run the harness (7), checklist and verdict (8) |
| 1:00 - 1:10 | **Module 4 I Do:** Advanced Prompting & Workflow Redesign | Scene and change calendar (2), package and mask the evidence (3), a prompt that forces evidence (3), self-challenge and reuse (2) |
| 1:10 - 1:30 | **Module 4 We Do:** Diagnose the S0C7 | Step 1 vague prompt (3), Step 2 structured prompt (6), Step 3 self-challenge (4), Step 4 impact (4), Step 5 save what worked (3) |
| 1:30 - 2:00 | **Module 4 You Do:** Lab: SAVOPS-2317, SQLCODE -811 (includes a 2-minute recap and course close) | Ticket and masking (4), root cause (8), impact (5), recovery, fix and tests (7), PR, playbook and time (6) |

**Output:** each learner's `review-checklist.md` with a verdict, `incident-2/SAVOPS-2317-resolution.md`, `incident-playbook.md`, `.github/copilot-instructions.md`, and a time estimate from every lab.

## Hands-on share

Independent lab time is 30 + 30 + 30 + 30 = 120 minutes of 240 (50%). In every We Do, learners run each prompt themselves and check the source, another 80 minutes, which puts the course at about 83% hands-on. The 40 minutes of I Do are instructor-led demos, with questions to the room throughout. Sessions 1 and 2 are equally hands-on, with 100 of their 120 minutes spent by learners at the keyboard.

## If you are running late

Use these cuts, in this order within each session:

**Session 1**
1. Module 1 I Do: skip the live `ACCT-FROZEN` contrast demo and use the lesson's Figure 01.
2. Module 1 We Do: shorten Step 2 (map the flow) to 3 minutes.
3. Module 2 I Do: explain the sign half-byte yourself rather than inviting an SME to.
4. Module 2 We Do: in Step 4 (coverage), choose the two gaps to add for the room instead of discussing them.

**Session 2**
1. Module 3 I Do: drop the knowledge-check question at 9:00 and go straight to the transition.
2. Module 3 We Do: shorten Step 3 (credentials) to 3 minutes.
3. Module 4 I Do: show the evidence-forcing prompt on screen without running it.
4. Module 4 We Do: learners finish Step 5's files in the first minutes of the Lab.

Never cut: Module 1 Steps 3 and 4 (extract and verify the rules), the Module 2 scenario table and its live calculator check, the Module 3 Demo B trace and We Do Steps 1 and 4, the Module 4 masking and the Step 1 and Step 2 answers side by side, the time estimate at the end of every lab, or the recap at the end of each session.

## Two-hour sessions

Both sessions are two hours with no scheduled break, and each changes module at the hour. Use that change as the natural pause: stand up, stretch, open the next folder. Watch the room around the 40-minute mark of each module, during the lab. If attention is flagging, a 2-minute stretch costs less than a distracted lab.
