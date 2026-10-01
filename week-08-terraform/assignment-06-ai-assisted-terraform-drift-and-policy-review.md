# Assignment 6 — AI-Assisted Terraform Drift and Policy Review

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Student Details

**Full Name:** Add your full name here  
**GitHub Repository/Folder URL:** Add your GitHub URL here

---

## Purpose

Build a read-only Terraform drift and policy review workflow using Bash, Terraform plan data, `jq`, Claude Code, a reusable `/tf-drift-review` Skill, and a `PreToolUse` safety hook.

The workflow must follow this pattern:

```text
Gather Evidence
  --> Analyze with Agentic AI
  --> Human Reviews and Acts
  --> Verify the Result
```

The `/tf-drift-review` Skill and `tf-drift-check.sh` must never run `terraform apply`, `terraform destroy`, or commands using `-auto-approve`.

---

# Task 1 — Confirm the Clean Baseline and Create the Workspace

## Goal

Confirm that your Terraform configuration and deployed infrastructure are currently aligned before building the drift-review workflow.

## Evidence

### Screenshot 1 — Clean Terraform Plan

Add a screenshot of `terraform plan` showing no pending changes.

![PALLOTI](./screenshots/wk861.png)

---

### Screenshot 2 — Assignment Workspace

Add a screenshot of the folder structure showing `AI Assignment/`, `reports/`, and the Terraform project.

![PALLOTI](./screenshots/wk862.png)

![PALLOTI](./screenshots/wk862x.png)
---

# Task 2 — Create Project Context and Safety Rules in `CLAUDE.md`

## Goal

Add a `CLAUDE.md` describing the read-only drift-review workflow and the safety rules Claude must follow — never run `apply` or `destroy`, never use `-auto-approve`, only recommend a next step.

### Evidence

#### Screenshot 3 — `CLAUDE.md` open showing the project overview, review workflow, and safety rules

![PALLOTI](./screenshots/wk863.png)

---

# Task 3 — Build the Terraform Drift and Policy Check Script

## Goal

Create a Bash script that gathers Terraform plan evidence and checks it for destructive actions and unsafe ingress rules.

## Evidence

### Screenshot 4 — Script Variables and Checks Array

Add a screenshot of the top section of `tf-drift-check.sh` showing the variables and `checks` array.

![PALLOTI](./screenshots/wk864.png)

---

#### Screenshot 5 — Terminal showing the script passes a syntax check and is executable
### add

![PALLOTI](./screenshots/wk861.png) 

---

# Task 4 — Run the Script Against the Clean Baseline

## Goal

Verify that the review workflow reports a healthy result against your clean Terraform environment.

## Evidence

### Screenshot 7 — Healthy Baseline Report

#### Screenshot 6 — Script output showing a healthy result against the clean baseline
#### add

![PALLOTI](./screenshots/wk861.png)

## Questions

### 1. What is the Overall Status of your baseline?

Write your answer here.

### 2. Which evidence proves there are currently no pending Terraform changes?

Write your answer here.

### 3. Was `reports/tfplan.json` created? Explain why or why not.

Write your answer here.

---

# Task 5 — Create and Run the /tf-drift-review Skill

## Goal

Turn the script into a `/tf-drift-review` skill that reads the drift report, explains any risk in plain language, and states whether `apply` looks safe — restricted to read-only tools so it can never modify a file or run `apply`/`destroy` itself.

### Evidence

#### Screenshot 7 — Skill file showing the tool restrictions and safety rules

![PALLOTI](./screenshots/wk867.png)

---

#### Screenshot 8 — `/tf-drift-review` output against the healthy baseline

![PALLOTI](./screenshots/wk868.png)

---

# Task 6 — Simulate Drift and Let the Skill Catch It

## Goal

Deliberately introduce a change Terraform did not make — a destructive change or a rule opening access to the whole internet — and confirm the skill flags it and does not run `apply` on its own.

### Evidence

#### Screenshot 9 — The drift you introduced, visible in your Terraform config or the cloud console

![PALLOTI](./screenshots/wk869.png)

---

#### Screenshot 10 — `/tf-drift-review` output flagging the drift and explaining the risk

![PALLOTI](./screenshots/wk8610.png)

## Questions

### 1. What change did you introduce?

the change introduced in Task 6 was an intentional, manual modification made directly in the AWS Management Console outside of Terraform:

The Change: A custom tag with the Key: TestDrift and Value: manual-change was added directly to the EC2 security group named book-review-dev-web-sg.

### 2. Was it true infrastructure drift or a Terraform configuration change?

It was **true infrastructure drift**.

The change was made directly in the live environment (the AWS Management Console) *outside* of Terraform, while the local Terraform configuration files (`.tf`) remained completely untouched.

This created a direct divergence between the actual state of the AWS resource and the desired state declared in your code. When `terraform plan` ran, Terraform detected the extra tag present in AWS that wasn't in the configuration, correctly treating it as drift and proposing an in-place change to remove it.

### 3. What Terraform plan evidence proves that a change is pending?

Based on the workflow guide and Terraform's mechanics, there are three primary pieces of evidence that prove a change is pending:

* **Terraform Exit Code (`2`):** The script executes `terraform plan -detailed-exitcode`. An exit code of **`2`** explicitly signals that Terraform has detected differences between the desired configuration and the actual infrastructure (compared to `0` for no changes or `1` for an error).
* **Resource Summary Counts:** The plan summary indicates non-zero change operations—such as **`0 to add, 1 to change, 0 to destroy`** (as seen when the `TestDrift` tag was detected on the security group).
* **Generated JSON Plan (`tfplan.json`):** The drift script is programmed to automatically generate a JSON version of the plan *only* when changes are found. The presence of this file allows `jq` to parse specific resource actions (like modifications or deletions) for policy and safety checks.

### 4. Was the action an update, deletion, replacement, or security-rule change?

The action was an **update** (specifically, an **in-place modification**).

Here is why:

* **The Plan Metrics:** The Terraform plan summary showed **`0 to add, 1 to change, 0 to destroy`**.
* **The Nature of the Change:** Terraform modified the existing security group (`book-review-dev-web-sg`) in place to remove the unmanaged `TestDrift` tag, bringing the live resource back into alignment with the configuration file. It did not destroy or replace the resource.

*Note on security rules:* While the automated script report generated an overall **FAIL** status, that was due to pre-existing open ingress rules (`0.0.0.0/0` for SSH/HTTP) flagged by the `check_open_ingress` policy check—the actual *Terraform plan action itself* did not modify any security rules, only the tags.

### 5. What did Claude recommend?

Based on the workflow guide, when Claude Code ran the `/tf-drift-review` Skill after the drift was introduced, it recommended the following:

* **Human Review for the Security Findings:** It recommended that an engineer carefully review the pre-existing security issues—specifically the open ingress rules (such as port 22/SSH exposed to `0.0.0.0/0`) that caused the script to report an overall **FAIL**.
* **Human Discretion on the Tag Change:** It identified the pending Terraform change as a low-risk, in-place tag reconciliation (removing the manually added `TestDrift` tag) to bring AWS back in line with the configuration.
* **No Autonomous Action:** Most importantly, **Claude recommended no automated infrastructure changes.** It did not run `terraform apply` or attempt to fix the drift itself, leaving the final decision and execution strictly under human control.

### 6. Why should you review the recommendation before taking action?

Reviewing Claude's recommendation before taking action is critical because an AI model provides analysis, but the human engineer remains solely responsible for production infrastructure.

Here is why that human review step is vital in this specific pipeline:

Separating Drift from Pre-Existing Issues: As seen during the drift review, the automated script reported an overall FAIL due to open ingress rules (0.0.0.0/0), even though the actual Terraform plan was only removing an unmanaged tag. An engineer must review the recommendation to understand that context—ensuring they don't panic over a red "FAIL" status, while also making sure they don't overlook genuine security concerns (like an exposed SSH port) that need separate hardening.

Preventing Unintended Side Effects: Even a seemingly harmless "in-place change" like removing a tag can sometimes disrupt automated monitoring, compliance labeling, or tracking systems if that tag was actually needed. A human review verifies whether the drift represents something that should be reconciled in AWS or if the Terraform code itself needs to be updated to match the new reality.

Enforcing the Safety Boundary: The workflow is specifically designed so that deterministic scripts gather data and AI provides reasoning, but humans make the final operational decisions. Reviewing the recommendation ensures you keep a human-in-the-loop safety net active before any infrastructure-changing commands (terraform apply) are executed.

---

# Task 7 — Add a PreToolUse Hook That Blocks Apply on a Failed Report

## Goal

Extend the Week 2 hooks pattern with a `PreToolUse` hook that blocks any `terraform apply` command while the last drift report's status is failing.

### Evidence

#### Screenshot 11 — `settings.json` showing the new `PreToolUse` hook

![PALLOTI](./screenshots/wk8611.png)

---

#### Screenshot 12 — Claude's blocked response when attempting `terraform apply` while the report is failing

![PALLOTI](./screenshots/wk8612.png)

![PALLOTI](./screenshots/wk8612x.png)
---

# Task 8 — Resolve the Drift, Verify, and Write the Review Summary

## Goal

Review the recommendation, resolve the drift yourself with a human-reviewed `terraform apply`, and confirm the hook no longer blocks it once the report is clean again.

### Evidence

#### Screenshot 13 — `terraform apply` completing successfully after your review

![PALLOTI](./screenshots/wk8613.png)

---

#### Screenshot 14 — Second `/tf-drift-review` run showing a healthy result
##### add
![PALLOTI](./screenshots/wk8610.png)

---

### Notes

Explain why this workflow needs both a fixed-rule hook that blocks `apply` outright and an AI skill that explains the risk in plain language — why isn't one of the two enough on its own?

1.Why the fixed-rule hook is necessary alone:Deterministic Enforcement.An AI model can occasionally experience hallucinations, misinterpret context, or bypass instructions if prompted cleverly by a user. A fixed-rule hook (BeforeTool) provides a hard, non-bypassable programmatic wall. It guarantees that if the drift status is FAIL, the command is physically blocked at the system level regardless of what the AI attempts or thinks.

2.Why the AI skill is necessary alone:Contextual Understanding & Usability.A hard block without explanation leaves the user frustrated and blind as to why their action was rejected. The AI skill reads the specific drift report and translates complex infrastructure failures into actionable, plain-language insights so the user understands the exact risk and knows how to remediate the underlying drift before trying again.

---

# Submission Instructions

- Complete Tasks 1–8 in sequence.
- Include Screenshots 1–19 exactly as specified.
- Answer every question under Tasks 1–8 in your own words.
- Complete all seven sections of the Terraform Drift Review Summary.
- Include the GitHub repository/folder URL containing the assignment files.
- Include your full name in the required reports and screenshots.
- Include the LinkedIn post URL and a screenshot of the published LinkedIn post.
- Do not expose access keys, passwords, tokens, account IDs, private keys, Terraform secrets, or other sensitive information.
- Review all screenshots carefully and hide or redact sensitive details where necessary.

---

# Completion Checklist

- [ ] Confirmed a clean Terraform baseline
- [ ] Created the required assignment workspace
- [ ] Created or updated `CLAUDE.md`
- [ ] Added project context and safety rules
- [ ] Created `tf-drift-check.sh`
- [ ] Added my full name to the report
- [ ] Validated the Bash script
- [ ] Made the script executable
- [ ] Used `terraform plan -detailed-exitcode`
- [ ] Used Terraform plan JSON
- [ ] Used `jq` to inspect destructive actions
- [ ] Used `jq` to inspect unsafe ingress
- [ ] Confirmed the baseline returns `HEALTHY`
- [ ] Created `/tf-drift-review`
- [ ] Restricted the Skill to appropriate tools
- [ ] Confirmed the Skill remains read-only
- [ ] Confirmed the Skill never runs `terraform apply`
- [ ] Confirmed the Skill never runs `terraform destroy`
- [ ] Introduced a controlled detectable difference
- [ ] Correctly identified whether it was true drift or a configuration change
- [ ] Saved `drift-detected-report.txt`
- [ ] Added the `PreToolUse` safety hook
- [ ] Verified the hook blocks `terraform apply` when the report is `FAIL`
- [ ] Reviewed the Terraform evidence before resolving the change
- [ ] Performed any infrastructure-changing action manually
- [ ] Ran the drift review again after resolution
- [ ] Confirmed the final status is `HEALTHY`
- [ ] Saved `resolved-report.txt`
- [ ] Completed `drift-review-summary.md`
- [ ] Mapped the workflow to `Gather --> Analyze --> Human Act --> Verify`
- [ ] Included all 19 numbered screenshots
- [ ] Answered all required questions
- [ ] Published the required LinkedIn post
- [ ] Added the LinkedIn post URL and screenshot
- [ ] Included the GitHub repository/folder URL
- [ ] Confirmed that no sensitive information is exposed

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
