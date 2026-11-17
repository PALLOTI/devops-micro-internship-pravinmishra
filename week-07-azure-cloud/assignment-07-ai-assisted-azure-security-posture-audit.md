# Assignment 7 — AI-Assisted Azure Security Posture Audit

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Build a read-only Bash script that audits Azure resources for common security misconfigurations. Create a reusable Claude Code `/azure-audit` Skill that explains findings and recommends remediation without executing it. Fix one finding yourself and rerun the audit to verify the result.

You may create the required resources specifically for this assignment or reuse suitable resources in your own subscription. A Load Balancer and a deployed application are not required.

---

# Task 1 — Confirm Your Azure Resources and Create the Workspace

## Goal

Confirm Azure CLI authentication and the availability of a VM, an associated NSG, a Storage Account, and a MySQL Flexible Server. Create the workspace with `scripts`, `reports`, and `.claude/skills/azure-audit` directories.

## Evidence

#### Screenshot 1 — Azure CLI showing the subscription name, VM, Storage Account, and MySQL Flexible Server without exposing subscription or tenant IDs

Add your screenshot here.

---

#### Screenshot 2 — Workspace location and directory structure shown using `pwd` and `find . -maxdepth 4 -type d | sort`

![PALLOTI](./screenshots/7wk71.png)

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Create `CLAUDE.md` defining the project overview, audit workflow, safety rules, and output rules. Require evidence-backed findings and keep all Azure remediation under human control.

## Evidence

#### Screenshot 3 — `CLAUDE.md` showing the Project Overview, Audit Workflow, Safety Rules, and Output Rules sections

![PALLOTI](./screenshots/7wk72.png)

---

# Task 3 — Use Agentic AI to Plan the Audit Before Writing the Script

## Goal

Ask Claude Code to read `CLAUDE.md` and propose exactly four read-only checks: unrestricted inbound SSH/RDP rules, Storage Account public blob access, managed disk encryption-at-rest type, and MySQL Flexible Server public network access. Do not allow file creation or modification during planning.

## Evidence

#### Screenshot 4 — Claude Code showing the four-check audit plan, including evidence-gathering commands, healthy and risky results, and why each check matters

![PALLOTI](./screenshots/7wk73.png)

---

# Task 4 — Build the Azure Audit Bash Script

## Goal

Create `scripts/azure-audit.sh` using read-only Azure CLI commands. Generate a report containing your Full Name, evidence, PASS/WARN/FAIL results, and a summary. Use exit codes **0 for HEALTHY**, **1 for WARN**, and **2 for FAIL**. Validate the script and make it executable.

## Evidence

#### Screenshot 4 — Your script open in your editor, showing the check functions and the `az` commands they call

![PALLOTI](./screenshots/7wk74.png)

---

#### Screenshot 5 — Output of `bash -n` (no syntax errors) and `ls -l` showing the script is executable

![PALLOTI](./screenshots/7wk75.png)

---

# Task 5 — Run the Script and Review the Baseline Report

## Goal

Run the script, capture its exit code immediately, review all four results, and save `reports/baseline-report.txt` before making any remediation.

## Evidence

#### Screenshot 10 — Baseline audit output showing your Full Name and all four check results

Add your screenshot here.

---

#### Screenshot 11 — Captured exit code and final report summary


![PALLOTI](./screenshots/7wk76.png)

---

# Task 6 — Create and Run the /azure-audit Skill

## Goal

Create `.claude/skills/azure-audit/SKILL.md` with Bash, Read, and Grep permissions, without Write permission, and with manual invocation enabled. The Skill must run the audit, read the report, explain findings and their impact, and recommend remediation and verification commands without changing Azure resources.

## Evidence

#### Screenshot 7 — Your skill file's frontmatter showing `allowed-tools` without `Write`

![PALLOTI](./screenshots/7wk77.png)

---

#### Screenshot 8 — `/azure-audit` output showing the baseline findings and Claude's explanation

![PALLOTI](./screenshots/7wk78.png)

---

# Task 7 — Fix a Real Finding and Re-Verify

## Goal

Save the before-fix report, review a recommended remediation, execute the fix yourself, and rerun `/azure-audit` to confirm that the finding is resolved. Save the after-fix report.

If the baseline is clean, use only the controlled temporary SSH-rule exercise permitted in the assignment guidelines. Do not deliberately expose the database or Storage Account.

## Evidence

#### Screenshot 9 — Saved report showing the original finding before the fix

![PALLOTI](./screenshots/7wk79.png)

---

#### Screenshot 10 — Terminal output of the remediation command you ran yourself

![PALLOTI](./screenshots/7wk710.png)

---

#### Screenshot 11 — Second `/azure-audit` run (or report) showing the finding resolved

![PALLOTI](./screenshots/7wk711.png)

---

### Notes

Compare this assignment to the AWS audit you built in Week 6: which finding categories map to each other across the two clouds, and what stayed exactly the same about the workflow even though the `az`/`aws` commands are completely different?

Transitioning from your AWS security posture audit to your Azure audit highlights a fundamental truth in cloud engineering: **cloud providers change their syntax and control planes, but core security principles and automation patterns remain identical.**

---

### 1. Finding Category Mapping (AWS vs. Azure)

The security domains you audited in Week 6 map directly across both cloud environments:

| Security Domain | AWS (Week 6 Equivalent) | Azure (Week 7 Implementation) | Core Security Objective |
| --- | --- | --- | --- |
| **Network Firewall Rules** | **Security Groups (SG)**<br>

<br>Checking inbound rules for unrestricted SSH (22) / RDP (3389) access from `0.0.0.0/0`. | **Network Security Groups (NSGs)**<br>

<br>Checking inbound security rules for unrestricted access on ports 22, 3389, or `*`. | Prevent unauthorized administrative remote access from the public internet. |
| **Object Storage Exposure** | **S3 Public Access Block**<br>

<br>Inspecting bucket policies and public access block configurations. | **Storage Account Public Access**<br>

<br>Querying `allowBlobPublicAccess` on storage accounts. | Prevent accidental data leaks or public exposure of sensitive container objects/blobs. |
| **Compute Disk Encryption** | **EBS Volume Encryption**<br>

<br>Validating KMS encryption keys on EC2 root/data volumes. | **Managed Disk Encryption at Rest**<br>

<br>Querying Azure disk `encryption.type` configurations. | Protect sensitive data stored on disk against physical or unauthorized volume extraction. |
| **Database Network Isolation** | **RDS Public Accessibility**<br>

<br>Checking if database instances have public IP endpoints enabled. | **Azure Database for MySQL**<br>

<br>Querying `network.publicNetworkAccess` settings. | Isolate backend data stores within private network boundaries away from public exposure. |

---

### 2. What Stayed Exactly the Same About the Workflow

Even though you swapped out AWS CLI (`aws ec2 ...`, `aws s3api ...`) for Azure CLI (`az network ...`, `az storage ...`), the **automation framework, architecture, and safety model** remained completely consistent:

* **Read-Only Evidence Collection:** Both scripts are strictly scoped to gather configuration data using inspection commands only (`show`, `list`), ensuring zero infrastructure drift during the audit phase.
* **Standardized Reporting Pipeline:** Both workflows utilize local directories (`reports/`), structured timestamp logging, and consistent status tags (`[PASS]`, `[WARN]`, `[FAIL]`) to produce clean, machine-readable and human-readable text reports.
* **Deterministic Exit Codes:** Both scripts conclude with summary functions that evaluate failure and warning counts, returning standardized exit codes (`0` for healthy, `1` for warnings, `2` for critical failures) suitable for CI/CD pipelines.
* **Human-in-the-Loop AI Safety Model:** The core operating procedure for your AI assistant (`GEMINI.md` / `SKILL.md`) stayed identical:
1. Read the generated report.
2. Explain the security impact of any findings.
3. Propose a single, precise remediation command.
4. **Never execute changes automatically**—leaving final authorization and execution strictly in the hands of the human operator.

---

# Submission Instructions

- Include your Full Name and Screenshots **1–17** under the corresponding evidence headings.
- Answer every **Notes You Must Write** question.
- Include the required source files and saved reports through your GitHub repository/folder link.
- Include your published LinkedIn post URL.
- Ensure required outputs show your Full Name.
- Use multiple captures under an evidence heading where needed for readability.
- Do not expose subscription IDs, tenant IDs, client secrets, passwords, tokens, connection strings, private keys, or other credentials.
- After saving all evidence, remove temporary resources created specifically for this assignment as directed in the guidelines.
- Submit your work using the method specified in the assignment guidelines.

---

# Completion Checklist

- [ ] Task 1: Required resources confirmed and workspace created (Screenshots 1–2)
- [ ] Task 2: `CLAUDE.md` contains project context, workflow, safety rules, and output rules (Screenshot 3)
- [ ] Task 3: Four-check read-only plan completed before scripting (Screenshot 4)
- [ ] Task 4: Script created, syntax-checked, and executable (Screenshots 5–9)
- [ ] Task 5: Baseline report saved and exit code verified (Screenshots 10–11)
- [ ] Task 6: `/azure-audit` Skill created and run without executing remediation (Screenshots 12–13)
- [ ] Task 7: Finding documented, fixed manually, and verified as resolved (Screenshots 14–16)
- [ ] Baseline, before-fix, and after-fix reports saved
- [ ] All required written answers completed in my own words
- [ ] Required files and GitHub repository/folder URL included
- [ ] LinkedIn post published with the applicable credit note and tags (Screenshot 17)
- [ ] LinkedIn post URL included
- [ ] Full Name visible in required outputs
- [ ] No sensitive information exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
