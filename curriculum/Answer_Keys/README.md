# Catch-up files and masking steps (GitHub)

## How to copy a file from GitHub into VS Code

The reference copies are in the GitHub repository folder **Answer_Keys**. Open the repository in Firefox in the VM, then:

1. In GitHub, open **Answer_Keys** and click the file you need.
2. Click the **Copy raw file** icon (two overlapping squares) at the top right of the file. Don't select the text on the page: the formatted view drops the Markdown, and a CSV loses its layout.
3. In VS Code, open the file named in the table for that step. It's already in your course pack.
4. Choose **Edit > Select All**, then **Edit > Paste**. This replaces what's in the file.
5. Choose **File > Save**.

If you want to keep your own version, first right-click it in the Explorer, choose **Copy**, then **Paste**: VS Code saves a copy named, for example, `scenarios copy.md`.

## Module 1: if you did not finish the guided practice spec

| Copy from Answer_Keys | Paste into this file in VS Code |
|---|---|
| `INTCALC-spec.md` | `module-1/INTCALC-spec.md` |

## Start of Module 2: everyone copies their INTCALC-spec.md into module-2

Run this command in the VS Code terminal:

```
cp ~/coursepack/module-1/INTCALC-spec.md ~/coursepack/module-2/
```

Check that `INTCALC-spec.md` now shows in the Explorer under **MODULE-2**.

## Before the Module 2 Lab: if your scenarios or boundary records are incomplete, copy both files

If you use one, you must use the other: their scenario IDs and account numbers match. These replace your own `scenarios.md` and `boundary-records.csv`.

| Copy from Answer_Keys | Paste into this file in VS Code |
|---|---|
| `scenarios.md` | `module-2/scenarios.md` |
| `boundary-records.csv` | `module-2/boundary-records.csv` |

Then check the records pack: **Terminal > Run Task > pack-records: show packed hex of boundary-records.csv**. Every row should show its packed values with no error.

## Before the Module 3 Lab: if your review-checklist.md does not have all four sections and the checklist

| Copy from Answer_Keys | Paste into this file in VS Code |
|---|---|
| `review-checklist.md` | `module-3/review-checklist.md` |

## Module 3, everyone: mask the override code and FTP password in review-checklist.md

1. Open `review-checklist.md` in the Explorer.
2. Choose **Edit > Find** and type `OVRD`. The search box shows a count, such as "1 of 3". If every match already shows `OVRD****`, go to step 5.
3. Choose **Edit > Replace**, and click the `.*` button (**Use Regular Expression**) in the Find box.
   - Find: `OVRD\d{4}`
   - Replace: `OVRD****`
   - Click **Replace All**. The count should drop to 0.
4. Do the same for the FTP password:
   - Find: `Fee\$[A-Za-z]{5}\d{4}`
   - Replace: `Fee$$****`
   - Click **Replace All**.
5. Search for `OVRD` and `Fee$` again. Every match should end in `****`. Then choose **File > Save**.

## Module 4, before Step 1 of the guided practice: mask the account numbers and customer IDs

1. Check that the Explorer's top line says **MODULE-4**.
2. Open the first file: `incident-1/CEEDUMP-excerpt.txt`.
3. Choose **Edit > Replace**, then click the `.*` button (**Use Regular Expression**) in the Find box. It stays on for the remaining files.
4. Enter the account-number pattern:
   - Find: `7100(\d{4})(\d{4})`
   - Replace: `7100****$2`
5. Check the count before replacing. The Find box shows "1 of N". It should match the table below.
6. Click **Replace All**, then **File > Save**.
7. Open the next file and repeat. The Find and Replace boxes keep their text, so it's: check the count, Replace All, save.

| # | File | Pattern | Count before |
|---|---|---|---|
| 1 | `incident-1/CEEDUMP-excerpt.txt` | account | 2 |
| 2 | `incident-1/EODINT-joblog.txt` | account | 8 |
| 3 | `incident-2/STMTPREP-joblog.txt` | account | 4 |
| 4 | `incident-2/dba-query.txt` | account | 4 |
| 5 | `incident-2/dba-query.txt` (same file) | customer ID | 3 |

8. For row 5, change the pattern while still in `dba-query.txt`:
   - Find: `(C00\d)\d{6}`
   - Replace: `$1******`
   - Check the count is 3, then Replace All and save.
9. Check the result. In each file the count for the pattern should now be 0. Account numbers look like `7100****1733`, and customer IDs like `C001******`.
10. Check line 1 of each file. If the pattern was typed into the file by mistake, delete it and save.

**Don't mask** `incident-1/ACCTUPD-CR-48790.diff` or `src/ACCTUPD.cbl`. Their 12-digit `7100…` numbers are the limits of the onboarding account range, not customer accounts.

## Before the Module 4 Lab: if your incident-playbook.md or copilot-instructions.md is incomplete

Only after Step 5 of the guided practice. The instructions file is in the `.github` folder inside `module-4`.

| Copy from Answer_Keys | Paste into this file in VS Code |
|---|---|
| `incident-playbook.md` | `module-4/incident-playbook.md` |
| `copilot-instructions.md` | `module-4/.github/copilot-instructions.md` |


