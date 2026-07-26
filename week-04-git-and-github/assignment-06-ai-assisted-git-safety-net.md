# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch


![week 04](./screenshots/wk4%2061.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

1  Protects the Production Line (main)
main is usually tied directly to deployment environments. If you work on main directly and commit half-finished code, broken dependencies, or unexpected bugs, you risk breaking the build for everyone else on your team—or worse, pushing broken code directly to live users. A feature branch acts as a safe sandbox.

2 Enables Unhindered Collaboration
When multiple people are working on the same project:
No code stomping: Everyone can commit freely on their own branch without causing continuous merge conflicts on every git pull.

3 Parallel development: Team members can work on completely different features or bug fixes simultaneously without interfering with one another.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![week 04](./screenshots/wk4%2061.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

sing an obviously fake key (or mock credential) in assignments, tutorials, and documentation is standard practice to protect security and avoid costly accidents.

Here are the main reasons why:

1. Preventing Secret Leakage
Public GitHub repositories, public forums, and automated web scrapers are constantly scanned by automated bots looking for AWS access keys (AKIA...).

The Bot Threat: If a real, active AWS key is accidentally committed to Git and pushed to a public repository, credential-harvesting bots will usually detect and misuse it within seconds.

Hardcoding Risk: Teaching students or developers to put real keys directly inside script files or assignment code builds dangerous habits. Using fake placeholders reinforces the habit of separating code from sensitive credentials.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![week 04](./screenshots/wk4%2062.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![week 04](./screenshots/wk4%2063.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

This is a classic Git design quirk that trips up almost everyone setting up team automation for the first time.

By default, Git explicitly ignores the .git/ folder when tracking files, which means the .git/hooks/ directory is local to your machine and is never pushed to remote repositories (like GitHub or GitLab).

Here is why projects keep a hooks/pre-commit (or a root .githooks/ directory) tracked in source control instead:

1. Enforcing Team-Wide Consistency
If hooks only lived inside .git/hooks/, every single developer on the team would have to write or copy the pre-commit script manually onto their own machine.

By tracking the script in the main repo tree (e.g., in hooks/pre-commit or .githooks/pre-commit), the exact same hook logic is shared with everyone who clones or pulls the repository.

Everyone runs the same code linters, security scanners, or secret-detection checks before making a commit.

2. Version Control for Hook Logic
When pre-commit checks change—say, you add a tool to prevent hardcoded AWS keys or enforce a new Bash formatting rule—tracking the hook file in Git ensures:

Changes to the hook logic are reviewed via Pull Requests just like regular code.

Updates automatically propagate to the whole team when they pull main.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**


Git pre-commit Hook
Intercepts: The local Git commit action triggered by a developer running git commit.

When it runs: After changes have been staged (git add), but before Git creates the new commit object in the repository.

Scope of inspection: Inspects staged files (git diff --cached) for policy violations—in your provided script, it checks for hardcoded secret keys (AWS, SSH keys) and files exceeding the 1MB size limit.

Action taken: Exits with status code 1 to abort the commit process if a violation is detected.

Claude Code PreToolUse HookIntercepts: 
An AI agent's tool execution attempt (specifically the Bash tool in this example).  When it runs: After the AI model decides to execute a system/terminal command, but before that command is actually executed in the shell.  
Scope of inspection: Inspects JSON delivered via standard input (stdin) to analyze the command intended by the AI model (.tool_input.command).  Action taken: Emits a JSON payload back (e.g., {"decision": "block", "reason": "..."}) or exits with a non-zero exit status to prevent the AI agent from executing destructive commands.  

2. What They Have in Common
Despite operating in completely different contexts (version control vs. AI agent execution), both scripts share fundamental architectural patterns:

Gatekeeper / Interceptor Pattern: Both sit directly between a user (or agent) intent and actual system execution. They act as automated guardrails.

Fail-Closed Gate Logic: Both evaluate inputs against a blacklist (secrets/file size vs. dangerous CLI commands). If a rule is matched, they halt execution immediately before damage occurs.

Non-Zero Exit / Rejection Output: Both block execution by signaling an explicit failure and returning an informative error message so the user or agent knows why the action was rejected.

Stateless Scripting: Both run as ephemeral Bash scripts receiving system state (either reading from stdin or querying git CLI) and outputting stdout/stderr messages.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![week 04](./screenshots/wk4%2064.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

The line in your hooks/pre-commit script that matched the fake AWS key is this one:

Bash
if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then

Why it matched
This line uses grep with an Extended Regular Expression (-E) to scan the staged changes (git diff --cached) for two specific patterns separated by an "OR" pipe (|).

Your fake AWS key triggered the first half of that regex: AKIA[0-9A-Z]{16}.

Here is exactly how that pattern breaks down:

AKIA: Matches the exact four-letter string "AKIA". AWS uses this specific prefix to denote a standard IAM user access key. Fake testing keys (like AKIAIOSFODNN7EXAMPLE) are intentionally generated using this real-world prefix.

[0-9A-Z]: This is a character class that matches any single digit from 0 to 9, or any single uppercase letter from A to Z.

{16}: This is a quantifier. It dictates that the previous character class ([0-9A-Z]) must appear exactly 16 times in a row.

Because an AWS Access Key ID is exactly 20 characters long (the 4-character AKIA prefix + 16 alphanumeric characters), any fake key designed to look like a valid AWS key will be caught perfectly by this logic. The script doesn't know the key is fake; it just knows the string perfectly matches the structural blueprint of an AWS secret.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No, this hook would completely miss it.

If you named a variable AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY", GCP_SERVICE_KEY, or stored an access key that didn't start with AKIA, this specific regex pattern would let it pass straight through without raising any flags.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![week 04](./screenshots/wk4%2065.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement


![week 04](./screenshots/wk4%2066.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

In agentic workflows (such as custom slash commands, Claude Code routines, or automated helper scripts), the /pr-ready command is specifically designed to evaluate and stage a pull request, not to generate or modify the code directly.

The tool permission choices—enabling Bash and Read while disabling Write—come down to three practical reasons:

1. Read-Only Validation (The "Auditor" Role)
The sole responsibility of /pr-ready is to inspect the state of your repository and tell you if it's fit to be opened as a PR.

Read allows the agent to inspect the code diff, scan modified files, check configuration files, and read git logs.

If it had Write permissions, an unexpected auto-fix or accidental file edit could alter your working directory, changing the exact diff you intended to submit.

2. Bash is Needed for Sanity Checks
Bash permission is included because verifying that a project is "PR-ready" requires running shell-based verification commands:

Running your local test runner (pytest, npm test, go test).

Executing linters and type-checkers (eslint, ruff, tsc).

Querying Git status via CLI (git status, git diff --stat, git log).

Because these CLI commands only report status rather than altering source files, they fit cleanly into a evaluation loop.

3. Safety Against Unwanted Mutations
In automated agent workflows, Write access is intentionally guarded.

Broad write permissions allow an agent to modify files on disk or create unvetted commits.

Enforcing a no-Write policy on /pr-ready ensures strict separation of duties: coding commands write the code, while pre-flight checks only read and verify it.

Summary: /pr-ready acts like an automated peer reviewer. It uses Read to inspect the diff, runs Bash to verify tests and linters pass, and omits Write so it cannot accidentally corrupt or change your changes right before you submit them.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

While both tools inspect your staged changes before code hits main, they evaluate the diff with completely different goals in mind—meaning they did not flag the same things.

Here is a breakdown of what each tool caught, and what one saw that the other missed:

What the pre-commit hook caught (and /pr-ready missed)
The pre-commit hook acts as a hard security and repository guardrail. It focuses strictly on preventing forbidden artifacts from being committed locally.

Hardcoded AWS Key Pattern (AKIA...): The hook directly ran grep against the diff for AKIA[0-9A-Z]{16} and explicitly blocked the commit.

Oversized Files (>1MB): The hook checked git cat-file -s to ensure no massive binary files or uncompressed assets were being staged into Git tracking.

Why /pr-ready missed this: Unless /pr-ready specifically executes the exact same custom pre-commit script, a generic LLM/agent check reading a diff might summarize or review code quality without strictly applying your local 1MB file size limit or running a hard regex filter unless instructed to run a secret-scanner script.

What /pr-ready caught (and the pre-commit hook missed)
The /pr-ready command acts as an automated peer reviewer and sanity checker. It evaluates contextual quality, test health, and readiness for a pull request.

Broken Tests & Linter Errors: Because /pr-ready has Bash access, it runs your test suite (pytest, npm test, go test, etc.) and code linters. The pre-commit script above only checked file sizes and regex patterns—it never ran your test suite.

Contextual Code Quality & Missing PR Requirements: /pr-ready uses its Read permissions to review the diff for missing documentation, untracked logic flaws, poor variable naming, or unhandled edge cases that a simple regex can't spot.

Secret Variables Without AKIA: As discussed, the pre-commit hook only looks for AKIA.... /pr-ready reads the actual code semantics and can flag suspicious variable names like AWS_SECRET_ACCESS_KEY="..." even if there is no AKIA prefix.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![week 04](./screenshots/wk4%2067.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description


![week 04](./screenshots/wk4%2068.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed two lines from the original script and replaced them with a comment:

Removed:

bash
AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP
echo "DEBUG: token is $AWS_ACCESS_KEY_ID"


Replaced with:

bash
# AWS_ACCESS_KEY_ID must be set in the environment before running this script

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)


![week 04](./screenshots/wk4%2069.png)




![week 04](./screenshots/wk4%2069x.png)

---

#### PR Link

https://github.com/PALLOTI/devops-micro-internship-interviews/pulls

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

Here's the PR 

PALLOTI/devops-micro-internship-pravinmishra

I edit the yourusername inputing myGithhub username PALLOTI
Your forked repository

WHY
To enable precised github linking and direction


---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The pull request will be declined
Because of zero pull permission
---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

Full pull request permission

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Step 1 — Run git diff --cached and git status to see exactly what is staged — this is Gather. It's the data-collection step: pulling the actual staged diff and status before any judgment or drafting happens.

---

**2. Which step(s) represent Analyze?**

Step 2 represents Analyze:

"Report any of the following if present: secrets or credential-shaped strings, debug print/echo statements, TODO/FIXME left in code, a diff that mixes unrelated concerns, or a change with no corresponding notes."

This is the step where the raw material gathered in Step 1 (git diff --cached, git status) gets evaluated against specific risk criteria — it's judgment applied to the gathered data, rather than collection (Step 1) or output drafting (Step 3).

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

**Step 4** is Human Act:

> *"Never run `git commit`, `git push`, or `gh pr create`. Never edit files. Your output is a draft for a human to review and use."*

Why this has to stay with a human, not Claude:

Irreversible/external consequences. Committing rewrites repo history; pushing and opening a PR make changes visible to collaborators, trigger CI runs, and notify reviewers. These are actions with real-world side effects outside the sandbox, not just draft text that can be silently discarded if wrong.

Accountability. A commit and PR are attributed to a person (their GitHub identity, their signature on the change). The human needs to be the one who actually decided "yes, this is ready" — not an agent that could be wrong about whether the diff is safe, complete, or actually staged correctly.

Final review checkpoint. The whole point of `pr-ready` is to produce a *draft* for human review specifically because Steps 1–3 (gather/analyze/generate) can miss things — a secret-shaped string that's actually a real key, a TODO that matters, unrelated changes bundled together. Human Act is the checkpoint where a person applies judgment the tool can't fully replicate before anything becomes permanent or public.

Least privilege / blast radius control. The tool's `allowed-tools` are explicitly limited to `Bash, Read, Grep` — no write/commit/push capability at all. That's a deliberate boundary: even if the model were somehow instructed otherwise, it structurally can't take the irreversible step.

---

**4. Which step is Verify?**

STEP 4

The "review" here is where Verify would live — a human checking the drafted title/description/risk report against the actual diff before trusting it — while "and use" (i.e., actually running git commit/push/gh pr create) is the Human Act part. This spec bundles both into one step rather than separating them, likely because the whole point of the tool is that everything downstream of Generate requires human judgment and human hands.

So: no clean single "Verify" step exists here on its own — it's folded into Step 4 alongside Human Act. Worth naming that overlap explicitly in your answer rather than forcing a step number that isn't really there.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**


Both are needed because they cover each other's fatal blind spots: the pre-commit hook acts as a deterministic, instant hard guardrail that deterministically blocks known secrets and heavy files before they reach Git history, while the AI skill acts as an intelligent contextual auditor that catches semantic risks, broken test logic, and unstructured secrets that fixed regex rules will always miss. Neither tool alone provides complete protection.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/ezeobi-palloti-5b231a1b9_devops-agenticai-git-share-7486473114105753601-OrvQ/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADLFS9YBFQ6i_O56Veo32xN5JbLJZhDGNnE

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- The importance of Hooks acting as an nstant hard guardrail that deterministically blocks known secrets and heavy files before they reach Git history

-AI skill acts as an intelligent contextual auditor that catches semantic risks, broken test logic, and unstructured secrets that fixed regex rules will always miss.

-The importance of the principle of least privilege while working with Agentic AI can be ocer emphasised


---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL



https://github.com/PALLOTI/devops-micro-internship-interviews.git

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
