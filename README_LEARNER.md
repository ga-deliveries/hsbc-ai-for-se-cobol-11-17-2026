![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Start Here: AI for Software Engineers (COBOL)

- **Format:** Instructor-Led, remote, with a live instructor
- **Duration:** 1 Days • 5 Hours 
- **Instructor:** Michelle Fields

---

## Overview

This course teaches COBOL mainframe engineers to use GitHub Copilot Chat, with a Claude model, as a disciplined assistant across the work they do every week: understanding code they didn't write, proving a change is safe, reviewing someone else's change, and resolving production incidents. Each module opens with a realistic banking problem, such as an undocumented 30-year-old interest program due for audit, or a 2 a.m. S0C7 with the batch window closing, and ends with something learners have built and checked themselves.

The through-line is simple: AI drafts and explains; the engineer verifies against the source, gathers evidence and decides. All code and data are synthetic, and nothing touches a live system.

---

## Outcomes

- Prompt an AI assistant to explain unfamiliar COBOL programs, copybooks and JCL, and extract their business rules into documentation an auditor can trace to the source
- Design boundary test scenarios for a COBOL change, generate copybook-compliant synthetic test data, and prove with a regression harness that only the intended records changed
- Review a refactored program against a checklist covering control flow, error handling, numeric fields, data impact and security, and write a change-safety verdict backed by evidence
- Diagnose a production incident with structured, evidence-forcing prompts, trace its impact across shared copybooks, and take it from root cause to a release-ready change
- Save what works as a reusable prompt playbook and a `copilot-instructions.md` file with team conventions and a PII/PCI rule
- Recognise when an AI answer sounds right and is wrong, and know what evidence settles it

---

## Prerequisites

- Working knowledge of COBOL, including copybooks, `PIC` clauses, `COMP-3` fields and `PERFORM` structures
- Working knowledge of JCL: steps, DD statements and `PARM`
- Familiarity with batch processing and, for Module 3, embedded DB2 SQL
- No previous experience with AI tools is needed
- A GitHub account with a Copilot licence, with the approved Claude model enabled

---

## System Requirements

- A Windows machine that learners use on both days, with the course pack copied locally
- **VS Code**, current release
- **IBM Z Open Editor** extension, for COBOL and JCL support and copybook lookup, plus the **Java** runtime it requires
- **GitHub Copilot** and **GitHub Copilot Chat** extensions, signed in, used in **Ask** mode with the approved **Claude** model
- A **Mermaid** preview extension, such as Markdown Preview Mermaid Support, for the Module 1 lab
- Copilot instruction files turned on in VS Code settings, for Module 4
- The regression harness's runtime, for Modules 2 and 3 (see the tech setup guide)
- Network access through HSBC's proxy to GitHub Copilot and to the VS Code Marketplace, or approved offline copies of the extensions

Full setup steps, checks and troubleshooting are in the [Tech Setup Guide](tech-setup-guide.md).

---

## 📂 Module Links

Each module is placed in one folder within this repository with all necessary files and information needed to deliver it. Open each module folder on its own in VS Code.

| Module name | Associated Lab | Tool(s) Used |
|---|---|---|
| [Module 1: Legacy COBOL Docs](./curriculum/01-Legacy-COBOL-Docs/legacy-cobol-docs-lesson.md)| [Lab: Document INTCALC for an Audit](./curriculum/01-Legacy-COBOL-Docs/Legacy-COBOL-Docs-Labs.md) | VS Code, IBM Z Open Editor, GitHub Copilot Chat (Claude), Mermaid preview |
| [Module 2: COBOL Testing](./curriculum/02-COBOL-Testing/COBOL-Testing-Lesson.md) | [Lab: Document Prove the Rate Change is Safe](./curriculum/02-COBOL-Testing/COBOL-Testing-Labs.md) ) | VS Code, IBM Z Open Editor, GitHub Copilot Chat (Claude), regression harness |
| [Module 3: Safe Change Review](./curriculum/03-Safe-Change-Review/Safe-Change-Review-Lesson.md) | [Lab: Review the Refactor and Write the Verdict](./curriculum/03-Safe-Change-Review/Safe-Change-Review-Labs.md) | VS Code, IBM Z Open Editor, GitHub Copilot Chat (Claude), VS Code compare, regression harness |
| [Module 4: Advanced Prompting & Workflow Redesign](curriculum/04-Advanced-Prompting-&-Workflow-Redesign/Advanced-Prompting-&-Workflow-Redesign-Lesson.md) | [Lab: SAVOPS-2317, SQLCODE -811](curriculum/04-Advanced-Prompting-&-Workflow-Redesign/Advanced-Prompting-Labs.md) | VS Code, IBM Z Open Editor, GitHub Copilot Chat (Claude), Copilot instruction files |

## Instructional Resources


