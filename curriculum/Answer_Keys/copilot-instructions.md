# Copilot instructions: savings batch team (COBOL, z/OS)

## Customer data (PII/PCI)
- Never include customer names, full account numbers, card numbers or
  addresses in a reply. Show account numbers masked, e.g. 7100****1733.
- If attached evidence contains unmasked customer data, say so before
  answering, and do not repeat it.

## Evidence
- Cite the file and line number, or the dump field, for every factual claim.
- If the evidence doesn't establish something, write "not established"
  and say what would establish it.

## Diagnosis
- List hypotheses before concluding, with evidence for and against each.
- Do not propose code changes until the root cause is supported by evidence.
- Answer in this format: Root cause | Confidence | Evidence | Ruled out | Next action.

## Recovery
- Recovery is the engineer's decision. Propose the least-change option
  first (a data fix and restart), separately from code fixes.
- Never propose restoring an earlier generation of a file without
  saying what it would discard.

## Our code
- Enterprise COBOL batch. Paragraph names follow NNNN-VERB-NOUN.
- Every FILE STATUS and SQLCODE must be checked.
- Numeric fields read from files must pass a NUMERIC class test before use.
- Arithmetic must use ROUNDED or state that truncation is intended.
