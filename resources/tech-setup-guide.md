![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Tech Setup Guide: AI for Software Engineers (COBOL)

Everything a learner's machine needs, and how to check it before delivery.

---

## Machine checklist

| Item | Detail |
|---|---|
| Operating system | Windows, as used on HSBC machines. VS Code and its extensions also run on macOS and Linux. Not yet tested on an HSBC machine |
| Editor | VS Code, current release |
| COBOL support | The **IBM Z Open Editor** extension, for COBOL and JCL highlighting and copybook lookup. It needs a **Java** runtime (Java 11 or later); check the version the installed release asks for |
| AI assistant | The **GitHub Copilot** and **GitHub Copilot Chat** extensions, signed in with an HSBC Copilot licence. Chat is used in **Ask** mode with the approved **Claude** model |
| Diagrams | A Markdown preview that renders Mermaid. VS Code's built-in preview needs an extension for this, such as **Markdown Preview Mermaid Support** |
| Regression harness | Runs from a VS Code task (**Ctrl+Shift+B**) in Modules 2 and 3. Its runtime must be installed as the harness build specifies (see Open items) |
| Course pack | The `module-1` to `module-4` folders, copied to each machine. Remove the speaker notes, lab-guide answer keys and any reference solutions from the learner copy |
| Accounts | A GitHub account with a Copilot licence, and the approved Claude model enabled for it |
| Network | Copilot needs access to GitHub's Copilot service through HSBC's proxy. Installing extensions needs the VS Code Marketplace, or an approved internal copy |
| Persistence | Learners use the same machine and course pack on both days. Module 1's `INTCALC-spec.md` is reused in Module 2, and Module 4's evidence files are edited by masking |

## Nothing is hosted

There is no mainframe, DB2, Endevor or test LPAR to provision. All COBOL source, JCL, copybooks, job logs, dumps and data files in the course pack are synthetic, and learners read them as local files. The harness runs locally on each machine. The only network traffic is GitHub Copilot's calls to the model.

## Learner setup

1. Install **VS Code** from [code.visualstudio.com](https://code.visualstudio.com), and a **Java** runtime if one isn't already installed.
2. In VS Code, open **Extensions** (**Ctrl+Shift+X**) and install:
   - **IBM Z Open Editor**
   - **GitHub Copilot** and **GitHub Copilot Chat**
   - **Markdown Preview Mermaid Support**, or the Mermaid extension HSBC approves
3. Sign in to GitHub when Copilot asks. Open the Chat view, set the mode to **Ask**, and choose the approved **Claude** model in the model picker.
4. Turn on instruction files: **Settings** (**Ctrl+,**), search *instruction files*, and make sure `.github/copilot-instructions.md` is used. Module 4 needs this.
5. Copy the course pack to a local folder. Then open **one module folder at a time** with **File > Open Folder** (for example, `module-1`), never the parent folder. When VS Code asks whether you trust the folder, choose **Yes, I trust the authors**.

## Check the setup

Run these checks on a fresh copy of the course pack, opening each module folder on its own.

| Module | Check | A healthy machine shows |
|---|---|---|
| 1 | Open `INTCALC.cbl` | Colour-coded COBOL, and no Restricted Mode banner |
| 1 | **Ctrl+click** `ACCTREC` in the `COPY` statement | `ACCTREC.cpy` opens. This depends on the folder's `zapp.yaml` |
| 1 | Open a `.md` file containing a Mermaid block and press **Ctrl+Shift+V** | The diagram renders as a picture, not as code |
| 1 | New chat (**Ctrl+N**), attach `ACCTREC.cpy`, ask *"What does ACCT-FROZEN mean?"* | An answer naming both values, `'F'` and `'L'`, from the Claude model in Ask mode |
| 2 | **Ctrl+Shift+B** on the sample data | The harness completes and writes a report. Expected counts to be confirmed from the Module 2 lab guide |
| 3 | **Ctrl+Shift+B** | **1,447 unchanged, 6 changed as expected, 47 unexplained**. Delete the `reports` folder afterwards |
| 4 | Look in the Explorer | `.github/copilot-instructions.md` and `incident-playbook.md` are present and empty; `incident-1`, `incident-2` and `src` are present |
| 4 | Put one line in `copilot-instructions.md` (for example, *"End every answer with the word CHECKED."*), then ask any question in a new chat | The answer follows the rule. Remove the line afterwards |

If these pass, the extensions, the course pack layout, the harness and the Copilot connection are good. They do **not** show how the model behaves on the course prompts: before delivery, run each module's I Do and We Do prompts once with the account type learners will use.

## Fallbacks

- **The harness can't run on a learner's machine.** For Module 3, use pre-computed reports for both versions: learners don't add their own records, so the evidence is the same. Module 2 has no fallback, because learners add records they created; pair the learner with a neighbour whose harness works.
- **Mermaid doesn't render.** Learners can still write and check the Mermaid text against the code in the Module 1 lab. Show the rendered diagrams from the instructor's screen.
- **Instruction files are blocked by policy.** In Module 4, learners attach `copilot-instructions.md` to every prompt with `#file`. Say why, and skip the "re-ask the vague question" check.
- **Copilot can't sign in on a learner's machine.** Pair the learner with a neighbour for the We Do and You Do segments, and have them write the deliverables themselves.

## Troubleshooting

| Problem | Fix |
|---|---|
| Restricted Mode banner, or COBOL isn't colour-coded | The folder isn't trusted. Click **Manage** on the banner and trust it |
| Z Open Editor reports that Java can't be found | Install a supported Java runtime, then restart VS Code. If Java is installed, set its path in the Z Open Editor settings |
| **Ctrl+click** on `ACCTREC` does nothing | The parent folder is open, so `zapp.yaml` isn't found. Open the module folder on its own |
| Copilot Chat isn't signed in, or the Claude model isn't in the picker | Sign in again from the Accounts menu. If the model is still missing, it isn't enabled for that licence: raise it with the customer contact |
| Copilot opened files the learner didn't attach | Chat is in Agent mode. Switch to **Ask** |
| **Ctrl+Shift+B** does nothing, or asks which task to run | The module folder isn't open on its own, or the harness task isn't the default build task. Use **Terminal > Run Task** and choose the regression task |
| The harness counts don't match | A `reports` folder from an earlier run, or an edited source file. Delete `reports`, restore the source from the course pack, and run again |
| A Mermaid diagram shows as code | A syntax error, or the Mermaid preview extension isn't installed. Check the extension first |
| `copilot-instructions.md` has no effect | The instruction-files setting is off, or the file isn't inside `.github` in the open folder. Fix either, then start a new chat |
| Extensions won't install, or Copilot can't connect | HSBC's proxy is blocking the Marketplace or Copilot. Ask IT for access, or for approved offline copies of the extensions |
| Module 4 files are already masked | A previous group's edits. Restore `incident-1` and `incident-2` from the instructor's spare copy |

## Open items for the customer

- GitHub Copilot licences for every learner, the approved Claude model enabled for them, and how they sign in on the course machines
- Approval to install IBM Z Open Editor, a Java runtime and a Mermaid preview extension, or approved internal copies
- Whether HSBC's Copilot policy allows instruction files (`.github/copilot-instructions.md`)
- Proxy access for the VS Code Marketplace and GitHub Copilot
- What the regression harness needs to run, and approval to install it on the course machines
- Whether learners keep the course pack, playbook and instructions file after the course
