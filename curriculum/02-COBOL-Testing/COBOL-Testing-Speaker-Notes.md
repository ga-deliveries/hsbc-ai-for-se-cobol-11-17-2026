![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# **COBOL Testing: Speaker Notes**
 
**Module Time:** 60 minutes
- 10-min content
- 20-min guided practice
- 30-min independent practice
---
 
**What This Module is About:**
 
A developer has extended the tier interest rates in `INTCALC` from two to three decimal places. Learners use GitHub Copilot Chat to decide what *should* change before anything runs, build a reviewable scenario table from the program's real boundaries, generate synthetic boundary records that match the copybook, and then run the regression harness to prove the 5,000-account file is identical except where it should differ.
 
**Core Objectives:**  
- Turn a change request into an explicit expected-change rule before testing
- Prompt for a reviewable scenario table: input, expected result, should change, why it matters
- Find boundary values in COBOL: branches, tier floors, `PIC` limits, truncation and rounding, dates, sign
- Generate copybook-compliant synthetic records, including `COMP-3`, and verify offsets and packing
- Check AI-calculated expected values by hand
- Read a regression report, and use AI to explain differences no scenario predicted

**Must-Hit Topics:**  
- Scenarios first, data second. Nobody can review a file of packed decimal.
- The **Should change?** column is what turns "no differences" into a result you can trust.
- A developer who changes the copybook but misses `WS-APPLIED-RATE` produces **identical** output, and without the expected-change rule that looks like a pass.
- Models are fluent at COBOL and unreliable at arithmetic. Ask for the working; check two rows yourself.
- Parity is not correctness. S13 gives identical output in both versions and still loses a customer's interest.
> [!IMPORTANT]
> Test case and test data creation was a specific pain point raised in discovery, and faster, more consistent testing is tied to the throughput measure the client's manager wants to track. Make the time saving visible: ask the room early how long building packed-decimal test records takes them today, and come back to that number at the end.
 
---
 
## 0. **Before the Session Starts**
**(Time: during the break before Module 2, about 10 minutes)**
 
-   **Purpose:** This is the first module where something runs. A harness that fails in the Lab costs half the module.

-   **Checklist, for every learner:**
    -   The `module-2/` folder is open as the workspace (**File > Open Folder**), with no "Restricted Mode" banner. It holds `INTCALC.cbl`, `INTCALC-v2.cbl`, `RATETIER.cpy`, `RATETIER-v2.cpy`, `ACCTREC.cpy`, both rate files, `demo.csv`, `data/` and `harness/`.
    -   Copilot Chat is in **Ask** mode with a **Claude** model selected.
    -   `INTCALC-spec.md` from Module 1 is copied into the folder. If a learner lost theirs, give them the reference copy (`facilitator/module-1/INTCALC-spec-key.md`).
    -   `scenarios.md` is empty and `boundary-records.csv` holds only its header row. Both come in the course pack.
    -   The harness runs. Have every learner choose **Terminal > Run Task > Run regression: 5,000 accounts only** once now. It should report **3,531 unchanged, 1,466 changed as expected, 3 unexplained**.
> [!WARNING]
> **The harness needs GnuCOBOL.** It is installed in the Skillable VM, and the Lab was run end to end there on 7 November 2026. If the course is ever delivered on HSBC machines instead of the VM, confirm GnuCOBOL can be installed there. Without it, learners cannot run their own boundary records.
 
-   **Teaching Tips:**
    -   Run every guided-practice prompt the day before. Note in particular whether your model **rounds** the expected values in Step 2. Most do at least once, and you want to know which rows it gets wrong.
    -   Have a calculator open on your screen. You'll use it live in Step 2.
    -   Don't trust the model's hex, even when it corrects itself. In testing it fixed one wrong packed value and wrote -0.01 as `00 00 00 01 1D` (that's -0.11) in the same answer. `pack-records --hex` is the referee.
---
 
## 1. **Provocation: Prove Nothing Else Moved**
**(Time: ~2 minutes, from the Content block)**
 
-   **Purpose:** Make the change feel small and the risk feel real.

-   **Talking Points:**
    -   The business wants more precise savings rates: a product priced at 2.275% has been running at 2.27%. The developer's change touches a copybook and a handful of fields.
    -   Connect to Module 1: this is the same `INTCALC` they documented an hour ago. `RATETIER` is a shared copybook, and every account passes through this program every night.
    -   The two things to prove: the accounts that should change do, by the right amount, and every other account in the 5,000-account file is identical.
    -   Name how it's done today: work out cases by hand, build records in a hex editor, run, eyeball the output. Slow, so the cases get thin.

-   **Teaching Tips:**
    -   Ask: "How long does building packed-decimal test data take you today, for a change like this?" Write the answers down. You'll use them in the wrap-up.
> [!CAUTION]
> State the test-data rule now: **test data is generated from the copybook, never copied from production.** Copying real account records into a test file or a prompt takes customer data somewhere it should not go, even when the intention is only "realistic values". Expect someone to say that's how their team does it today. Acknowledge it, and hold the line.
 
---
 
## 2. **Scenarios Before Data: the Reviewable Scenario Table**
**(Time: ~4 minutes)**
 
-   **Purpose:** Replace "generate test data" with "agree what to test, then generate".

-   **Talking Points:**
    -   The common mistake: "Generate 20 test records for INTCALC." You get 20 records and no idea what they cover or what the right answers are.
    -   Walk the five columns in Figure 01. Spend your time on **Should change?**: when the Lab report says "M changed", you can only call that good news if you wrote down beforehand which accounts were meant to change.
    -   Walk Figure 02 quickly: boundaries are read out of the code, and most of them are already in their Module 1 spec. Each boundary gets a value on it and a value just beside it.
    -   The prompt pattern: program, copybooks, the change and the spec as context; boundaries by decision point; table format; show the working.

-   **Teaching Tips:**
    -   Make the link to Module 1 explicit: "Your spec is now a test design document. Every rule you verified is a scenario waiting to be written."
---
 
## 3. **From Scenario to Records That Match the Copybook**
**(Time: ~4 minutes)**
 
-   **Purpose:** Demystify packed decimal just enough that learners can check what the model generates.

-   **Talking Points:**
    -   Show Figure 03, the `ACCTREC` layout. The point: offsets must add up to the record length, 100 bytes.
    -   Packed decimal in one minute: two digits per byte, sign in the last half-byte, `C` positive, `D` negative. Length is digits plus one, divided by two, rounded up, so `S9(7)V99` is 5 bytes. The decimal point is implied.
    -   Show the two worked values: -0.01 is `X'000000001D'`, and 1,234.56 is `X'000123456C'`.
    -   **Run it live.** In the `module-2` terminal: `python3 harness/pack-records.py --hex demo.csv`. It packs four demo records and shows 1,234.56 as `X'000123456C'` and -0.01 as `X'000000001D'`. The harness never adds `demo.csv` to a regression run.
    -   Explain the harness: learners write readable decimal in a CSV, and `pack-records` produces the fixed-width file. Their job is to get the values right and check a few packed values themselves.
    -   Walk Figure 04, the layout mistakes AI makes. These are the things to look for in Step 3.

-   **Teaching Tips:**
    -   Senior engineers will know packed decimal well. Invite one of them to explain the sign half-byte instead of you. It respects their expertise and keeps the room engaged.
> [!NOTE]
> Some learners will ask why the harness packs the records instead of the model. The honest answer: generating correct bytes is easy to automate and hard to review, while choosing the right *values* is the judgement worth practising. On their own team's files, they'd ask the model for the hex of a few values and check them, exactly as in Step 3.
 
---
 
## 4. **Guided Practice: Scenario Table and Boundary Records for the Rate Change**
**(Time: ~20 minutes: 4-min change, 8-min scenarios, 6-min records, 2-min coverage)**
 
-   **Purpose:** Learners build `scenarios.md` and `boundary-records.csv` together, checking every number before moving on.

-   **Talking Points:**
    -   Same rhythm as Module 1: everyone runs the prompt, two or three read out their results, the room checks.
    -   Their wording will differ; their **numbers** should not. Where numbers disagree, the source code is the referee.
    -   Before Step 1: end each prompt with *"Put your whole answer in a single Markdown code block"* and copy with the code block's **Copy** icon. That keeps Copilot's working log out of `scenarios.md`.
-   **Step-by-step notes:**
    -   **Step 1, pin down the change (4 min).** The four field changes come back reliably. Land two points: first, if `WS-APPLIED-RATE` had been missed, the output would be identical and look like a pass; second, the rate file has to be released at the same time as `INTCALC`. Plant that second point firmly, because it's the red herring in Module 4.
    -   **Step 2, the scenario table (8 min).** This is where the time goes. Work S05 on your calculator live before showing the answer key. Then ask the room to find any row where their model rounded instead of truncating. Go through the five common errors: rounding, S13 marked "Yes", S10 marked "Yes", S12 missing, and a case quietly dropped from the **Cover** list. Learners' IDs may not match the key, so compare by input. Also watch for stray text in cells: in testing, a leftover "227.00 / 366 = 0.6201" sat beside the correct value, and a wrong hex value sat in a "Why it matters" note.
    -   **Step 3, the records (6 min).** Expect **15** rows. Check the model listed the skipped scenario (S10) and nothing else. Ask everyone to read the last half-byte of each hex value aloud for the negative, maximum and positive balances. Then have them run **Terminal > Run Task > pack-records: show packed hex of boundary-records.csv**: the tool's hex settles any disagreement with the model's. Point out the deliberate gap at `BND000000010`. Two CSV traps were seen in the VM: a file with no final line break (the next pasted row joins the last one), and a row pasted mid-line. Rerun `pack-records --hex` after every edit.
    -   **Step 4, coverage (2 min).** Don't let this become a long debate. Let the room pick two gaps to add, and steer towards the bonus-after-withdrawal and closed-account cases if they're stuck. Credit two file-level risks if they come up: a program compiled against the wrong version of the copybook, and v2 reading a rate file in the old layout.
> [!TIP]
> In Step 2, don't ask "Is the table right?" Ask "Pick any row and prove it to me." Learners who check one row themselves trust the method far more than learners who compare against the answer key.
 
> [!IMPORTANT]
> **S13 is the most important row in the module.** The maximum balance overflows at month-end in both versions, the accrued interest is reset to zero in both, and the output is identical. Parity proves the change didn't introduce a difference; it doesn't prove the original was right. This is Module 1's rule 12 coming back. Ask the Discuss question at the end of the lesson about what a tester should do with it.
 
-   **Teaching Tips:**
    -   The month-end caveat on the expected-change rule is subtle: rounding to the cent can absorb a difference smaller than half a penny. Use the pair: S05 under run M capitalises to 10,000.62 in both versions, while S16 (same balance, accrued 0.0030) ends at 10,000.62 against 10,000.63. If the room is behind, save it for the Lab's month-end stretch task.
    -   If you're behind, shorten Step 4, never Step 2.
**Answer key: Step 1, the change**
 
| File | Field | Old `PIC` | New `PIC` |
|---|---|---|---|
| `RATETIER.cpy` | `RT-TIER-RATE` | `9(02)V99` | `9(02)V999` |
| `RATETIER.cpy` | `RT-BONUS-RATE` | `9(02)V99` | `9(02)V999` |
| `INTCALC.cbl` | `RI-TIER-RATE-PCT` | `9(02)V99` | `9(02)V999`, `FILLER` 71 → 70 |
| `INTCALC.cbl` | `WS-APPLIED-RATE` | `9(02)V99` | `9(02)V999` |
 
**Expected-change rule:** in a daily run, accounts that are not closed, frozen or dormant, with a balance of 10,000.00 or more, should change. Everything else should be identical. At month-end, "should change" means "may change", because rounding to the cent can absorb the difference. S16 shows the difference can also survive: 10,000.62 against 10,000.63.
 
**Answer key: Step 2, scenario table**
 
All accounts are `SV`, status `A`, unless noted. Runs: D = `'20261030,N'`, M = `'20261031,Y'`, L = `'20280229,N'`. Values are daily interest added to accrued, truncated to 4 places.
 
| ID | Input | Original | Modified | Should change? |
|---|---|---|---|---|
| S01 | 0.00, D | No accrual | No accrual | No |
| S02 | -0.01, D | No accrual | No accrual | No |
| S03 | 1,000.00, D | 0.0342 | 0.0342 | No |
| S04 | 999.99, D | 0.0136 | 0.0136 | No |
| S05 | 10,000.00, D | 0.6219 | 0.6232 | **Yes** |
| S06 | 9,999.99, D | 0.3424 | 0.3424 | No |
| S07 | 50,000.00, D | 4.2739 | 4.2808 | **Yes** |
| S08 | `BS`, flag `Y`, 0 withdrawals, 20,000.00, D | 1.5178 | 1.5205 | **Yes** |
| S09 | Status `D`, 20,000.00, D | 0.2739 | 0.2739 | No |
| S10 | S03 record, L | 0.0341 | 0.0341 | No |
| S11 | 100.00, accrued 0.0037, M | Balance 100.01, accrued 0 | Same | No |
| S12 | 100.00, accrued 0.0036, M | Balance 100.00, accrued 0 | Same | No |
| S13 | 9,999,999.99, M | Overflow exception; balance unchanged; accrued 0 | Same | No |
| S14 | 1,234.56, D | 0.0422 | 0.0422 | No |
| S15 | 49,999.99, D | 3.1095 | 3.1164 | **Yes** |
| S16 | 10,000.00, accrued 0.0030, M | Balance 10,000.62, accrued 0 | Balance 10,000.63, accrued 0 | **Yes** |
 
**Answer key: Step 3, packed values**
 
| Scenario | Balance | `COMP-3` hex |
|---|---|---|
| S02 | -0.01 | `X'000000001D'` |
| S13 | 9,999,999.99 | `X'999999999C'` |
| S14 | 1,234.56 | `X'000123456C'` |
 
**Answer key: Step 4, coverage gaps** — closed accounts dropped (rule 1); legal hold `L` (rule 2); bonus lost after a withdrawal (rule 6); accrued-interest overflow (rule 9); a fifth rate record ignored (rule 13); an unexpected status value falling through to `WHEN OTHER` (rule 15). Learners' own specs will number rules differently, so the lesson leaves the numbers out.
 
---
 
## 5. **Independent Practice: Regression Harness Lab**
**(Time: ~30 minutes: 4-min run the regression, 6-min check your own records, 14-min explain the unexplained, 6-min conclude)**
 
-   **Purpose:** Learners run the original and modified `INTCALC` on the same file, read the evidence, and explain every difference they didn't predict.

-   **Talking Points:**
    -   Walk the lab briefly. Learners run **Ctrl+Shift+B** (the daily run D). The harness adds `boundary-records.csv` to the 5,000-account file automatically and writes a report: **N unchanged, M changed as expected, X unexplained**. With the key's 15 records that is **3,540 / 1,472 / 3**. Learners with different records get different first two numbers. The 3 unexplained never change.
    -   In task 2, learners check their **run D** scenarios against the report. Month-end and leap-year scenarios look different on the daily report because they just accrue a day. That's not a mismatch: they check those on the stretch-task runs.
    -   "Changed as expected" is decided by their expected-change rule from Step 1. If their rule was wrong, the report will be too, and that's part of the lesson.
    -   For every unexplained difference, they use Copilot to explain it, then confirm the explanation against the code and their scenario table. An explanation without a line of code behind it isn't an explanation.

-   **Teaching Tips:**
    -   Circulate while reports come back and ask each learner: "What does your X number mean, and what's your first hypothesis?" before they reach for Copilot.
    -   Most common problems: records rejected by `pack-records`, usually a CSV with no final line break or a value that breaks the `PIC`; an expected-change rule that forgot dormant accounts, so they show as "unexplained" when they're actually unchanged as expected; treating month-end rounding absorption as a failure; and M or L scenarios read as mismatches on the daily report.
    -   If a learner's report shows **zero** changes, celebrate the question, not the result: "Is that a pass?" It shouldn't be. The harness always runs `INTCALC-v2` with the new rate file, so zero changes would mean the change did nothing, like a missed `WS-APPLIED-RATE`.
    -   Fast finishers: the lab's stretch tasks. The month-end run gives 1 unexplained (`104`), the leap-year run gives 2 (`109`, `111`), and they explain why. Or they ask Copilot to turn the findings into a note to a release approver.
> [!NOTE]
> **The planted difference.** Accounts `720000000101` to `112` were migrated from an acquired bank that capitalised once a year. They still carry interest since 31 January, so their `ACCT-ACCRUED-INT` (`PIC S9(5)V9(4)`, maximum 99,999.9999) is near its limit. Tonight three overflow (rule 9). `104` changes, but **down**: at the new rate its day overflows, so the customer loses 367.65 *because of* a rate increase. `109` and `111` overflow in both versions, so they are unchanged even though the rule says they should change. Verdict: **safe with conditions**. Release the change and raise the overflow as a separate defect. The full answer key is `facilitator/module-2/unexplained-differences-key.md`.
 
> [!TIP]
> In testing, Copilot explained all three accounts correctly, down to the exception-report lines. It missed the `7200` account range, so it couldn't say where these accounts came from. Ask the room: *"What do these three have in common that the other 4,997 don't?"* Then ask for the cost in money: "lower than the original" isn't a finding, "the customer loses 367.65 tonight" is.
 
> [!TIP]
> Bring S13 back during the report review. Ask: "Your harness says S13 is unchanged. Is S13 fine?" Learners who answer "No, it loses interest in both versions" have understood the module.
 
---
 
## **Wrap-Up Reflection**
**(Time: ~2 minutes, from the end of the Lab)**
 
-   **Key Takeaways:**
    -   Decide what should change before anything runs
    -   Scenarios first, in a table a reviewer can read; data second
    -   Records come from the copybook: check offsets, lengths and sign half-bytes
    -   Check the model's arithmetic, especially where the code truncates
    -   Identical output proves no new difference; it doesn't prove correctness
-   **Closing question for the room:** go back to the time they gave you in the provocation for building test data today. Ask how long today's scenario table and records took. The difference is the number the client's manager cares about.
-   **Next Steps:** Day 1 ends here. In Module 3, they review a change someone else wrote, and the harness from today becomes their evidence. Ask them to keep `scenarios.md`.
---
 
## **What Was Tested and What Was Not**
 
-   **Checked while authoring:** every expected value in the scenario table, including S15 and S16, by exact decimal arithmetic and then by running the harness. The `COMP-3` hex values and the `ACCTREC` offsets (total 100 bytes) were checked against the copybook and with `pack-records --hex`.
-   **Run end to end:** `INTCALC` and `INTCALC-v2` compiled with GnuCOBOL 3.1 (`-std=ibm`). The harness gives 3,540 / 1,472 / 3 with the key's 15 records and 3,531 / 1,466 / 3 with none. The month-end and leap-year runs give 1 and 2 unexplained.
-   **Tested in the Skillable VM** on 7 November 2026 with Copilot and Claude Sonnet 5.5: every guided-practice prompt and the Lab. What your assistant says will still vary from run to run, so run every prompt once before class.
-   **Still to confirm:** GnuCOBOL's arithmetic matches Enterprise COBOL for these values, and the runtime, if the course is ever delivered outside the VM.
 
