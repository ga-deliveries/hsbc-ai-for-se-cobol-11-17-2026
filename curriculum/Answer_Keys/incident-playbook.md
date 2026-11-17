# Incident playbook (reference copy)

Reference copy for learners who didn't finish the guided practice. Save it as `incident-playbook.md` in `module-4`. Replace every `<placeholder>` before you send a prompt.

**Mask account numbers and customer data before attaching anything.**

## 1. Structured diagnosis (new chat)

```text
<evidence files, each attached with #file:>

Treat the current time as <incident time>.

You are a mainframe production-support engineer. <job> failed in
<step> with <abend code or SQLCODE>. Find the root cause.

1. From the job log and the abend output, state the failing statement,
   the source line it maps to, every field or value the evidence shows
   as invalid, and the program's record counters at the time of the
   failure.
2. List at least three hypotheses for the cause, including each recent
   change. For each, give the evidence for and against.
3. Reason step by step to the most likely root cause.

Cite the file and line, or the dump field, for every claim. If the
evidence does not establish something, write "not established".

Return: Root cause | Confidence | Evidence | Ruled out (and why) |
Next action
```

## 2. Self-challenge (same chat)

```text
Argue against your own diagnosis. What is the strongest alternative
explanation? What evidence in the pack would prove it, and is that
evidence present? Then compare your diagnosis with this one from
another engineer, and say which is better supported and why:

"<the other diagnosis, or your own first answer>"

Finally, what one thing should I check before acting?
```

## 3. Impact trace (same chat)

```text
#codebase #file:<shared copybook or table definition> #file:<change calendar or schedule>

<what is bad, and where: for example, which records have bad data in
which fields>. Which other programs use this <copybook or table>? For
each program in this workspace that uses it, say whether it would read
the bad data, in which paragraph, and when it next runs. List only what
you find in the code and the schedule. Separately, list programs that
headers or comments name but that are not in this workspace.
```
