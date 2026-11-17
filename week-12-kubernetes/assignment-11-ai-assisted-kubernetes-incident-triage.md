# Assignment 11 — AI-Assisted Kubernetes Incident Triage

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Configure a read-only Bash triage script for your Book Review App, connect it to Claude Code through `/k8s-triage`, simulate one controlled failure, recover the application manually, and verify recovery.

**Workflow: Gather → Analyze → Human Act → Verify**

---

## Submission Details

**Full Name:** Add your full name here.

**Kubernetes Platform:** Add your platform here.

**Selected Workload:** Add the Deployment name here.

**Namespace:** Add the namespace here.

**GitHub Repository URL:** Add your project repository URL here.

**Simulated Incident:** Invalid image or broken readiness probe.

**Baseline Status:** HEALTHY or explained WARN.

**Recovery Status:** HEALTHY or explained WARN, with no FAIL.

---

# Task 1 — Discover the Application and Capture a Healthy Baseline

## Goal

Identify the actual application configuration and prove that the selected workload is healthy before incident simulation.

### Evidence

#### Screenshot 1 — Healthy Pods and ready Service backends

![PALLOTI](./screenshots/12wk111.png)

---

#### Screenshot 2 — Workspace directory structure

![PALLOTI](./screenshots/12wk112.png)

<<<<<<< HEAD
---

# Task 2 — Add Safety Rules to CLAUDE.md

## Goal

Add a `CLAUDE.md` telling Claude this is a read-only Kubernetes triage workflow: it must never run `kubectl apply`, `delete`, `rollout restart`, `edit`, or `scale` on its own, and every recovery action must be reviewed and run by you.

### Evidence

#### Screenshot 3 — `CLAUDE.md` showing the project overview, incident workflow, and safety rules

![PALLOTI](./screenshots/12wk113.png)

---

# Task 3 — Use Agentic AI to Plan the Triage Checks

## Goal

Ask Claude to propose a read-only, five-check `kubectl` triage plan — pod status, pod readiness, restart counts, recent namespace events, and Service endpoints — before you write any script.

### Evidence

#### Screenshot 4 — Claude's proposed five-check plan, with no file created or cluster state changed

![PALLOTI](./screenshots/12wk114.png)

---

# Task 4 — Build the Kubernetes Triage Script

## Goal

Write a Bash script that runs the five read-only `kubectl` checks and writes a PASS/WARN/FAIL report with an overall status and exit code.

### Evidence

#### Screenshot 5 — Script showing the five checks and how each one gathers its evidence

![PALLOTI](./screenshots/12wk1105.png)

---

#### Screenshot 6 — `bash -n` and `ls -l` confirming the script is valid and executable

![PALLOTI](./screenshots/12wk116.png)

---

# Task 5 — Run the Script Against the Healthy Cluster

## Goal

Run the script against your current, unmodified deployment and confirm it reports a clean, healthy baseline before you simulate anything.

### Evidence

#### Screenshot 7 — Script output showing all five checks passing

![PALLOTI](./screenshots/12wk117.png)

---

# Task 6 — Build and Run the /k8s-triage Skill

## Goal

Wrap the script in a Claude Code skill restricted to read-only tools (no `Write`), and confirm `/k8s-triage` reports the healthy baseline using the script's evidence without touching the cluster.

### Evidence

#### Screenshot 8 — `SKILL.md` frontmatter showing the tool restrictions and safety rules

![PALLOTI](./screenshots/12wk118.png)

---

#### Screenshot 9 — `/k8s-triage` output for the healthy cluster

![PALLOTI](./screenshots/12wk119.png)

---

# Task 7 — Simulate an Incident and Let the Skill Diagnose It

## Goal

Deliberately break your own deployment — an image tag that does not exist, or a misconfigured readiness probe — then run `/k8s-triage` and confirm it correctly identifies the failure, quotes the evidence, and recommends a fix without applying it.

### Evidence

#### Screenshot 10 — The pod in a failed state and the Service showing no Endpoints

![PALLOTI](./screenshots/12wk1110.png)

---

#### Screenshot 11 — `/k8s-triage` output showing the diagnosis and the recommended `kubectl` command, not executed by Claude

![PALLOTI](./screenshots/12wk1111.png)

---

# Task 8 — Recover, Verify, and Summarize

## Goal

Review the recommendation, apply the fix yourself, confirm the pod returns to `Running`/`Ready` with active Endpoints, run `/k8s-triage` again to verify, and write a short incident summary.

### Evidence

#### Screenshot 12 — Pod back to `Running`/`Ready` and the Service showing Endpoints again

![PALLOTI](./screenshots/12wk1112.png)

---

#### Screenshot 13 — Second `/k8s-triage` run confirming recovery

![PALLOTI](./screenshots/12wk1113.png)

---

=======
>>>>>>> upstream
### Notes

**1. What proves the selected Pods are ready, not merely running?**

<<<<<<< HEAD
omparing the Kubernetes incident-triage workflow from Week 12 to the Linux Nginx incident from Week 3 highlights how core operational philosophies translate across layers of the infrastructure stack.

What Stayed the Same (The Process)
The Agentic Loop (Gather → Analyze → Human Act → Verify): Both workflows strictly decouple data collection from remediation. In both cases, a deterministic script (Bash) collects the raw facts, an AI assistant analyzes the findings without executing mutations, and you (the human engineer) review the recommendation, perform the manual fix, and re-run the script to verify recovery.

Strict Safety Guardrails: Both scenarios enforce a human-in-the-loop operational model. The AI is explicitly prohibited from autonomously executing mutating commands (such as restarting services or modifying cluster resources), keeping operational control firmly in your hands.

Preservation of Evidence: Both workflows emphasize capturing the failure state in a persistent report (reports/ directory) before attempting remediation, creating a transparent audit trail of the incident.

What Changed (The Evidence Gathered)
Scope of Abstraction (Host vs. Cluster): In the Week 3 Linux health check, evidence gathering focused on single-node operating system primitives—such as checking systemctl status nginx, querying local socket listeners on Port 80 via ss or netstat, and inspecting system journal logs. In contrast, Week 12 required distributed, cloud-native orchestration primitives.

Health Metrics and Observability Indicators:

Instead of single-process uptime, evidence collection expanded to Pod phase and waiting reasons (e.g., ImagePullBackOff), container restart counts, and Pod readiness conditions.

Namespace-Wide Events and Routing State:

Evidence gathering shifted from local syslog files to namespace Warning events and Service endpoint health. Rather than checking if a local port is bound, you had to inspect EndpointSlices or Endpoints to verify whether traffic could successfully route to active backends across worker nodes in an AWS EKS cluster.
=======
Add your answer here.

**2. What proves the Service has ready backends?**

Add your answer here.

**3. Why must the original image be recorded before incident simulation?**

Add your answer here.

---

# Task 2 — Configure CLAUDE.md

## Goal

Define the project configuration, incident workflow, and safety boundary for Claude Code.

### Evidence

#### Screenshot 3 — CLAUDE.md showing application configuration, workflow, safety rules, and output rules

Add your screenshot here.

### Notes

**1. Why does Claude need project-specific cluster rules?**

Add your answer here.

**2. Why must a human execute recovery commands?**

Add your answer here.

**3. Which rule prevents an unsupported diagnosis?**

Add your answer here.

---

# Task 3 — Plan the Read-Only Triage Workflow

## Goal

Ask Claude Code to explain the five health checks before running the supplied script.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check read-only plan

Add your screenshot here.

### Notes

**1. Which part represents the Gather phase?**

Add your answer here.

**2. How did you verify that Claude did not change anything?**

Add your answer here.

**3. Why is planning before coding useful for incident triage?**

Add your answer here.

---

# Task 4 — Configure and Validate the Bash Script

## Goal

Configure the supplied triage script for the selected workload and validate its syntax.

### Evidence

#### Screenshot 5 — Script configuration variables and checks array

Add your screenshot here.

---

#### Screenshot 6 — Functions for status, readiness, restarts, events, and Service backends

Add your screenshot here.

---

#### Screenshot 7 — Preflight, summary, and exit-code logic

Add your screenshot here.

---

#### Screenshot 8 — Successful syntax validation and executable permission

Add your screenshot here.

### Notes

**1. Why must the script fail when no Pods match the configured selector?**

Add your answer here.

**2. How does the readiness check detect a running but unready Pod?**

Add your answer here.

**3. Why are Service backends checked separately from Pod health?**

Add your answer here.

**4. What do exit codes 0, 1, and 2 represent?**

Add your answer here.

---

# Task 5 — Run the Healthy Baseline Report

## Goal

Run the script on the healthy application and confirm that it contains no failed checks.

### Evidence

#### Screenshot 9 — Baseline report showing your full name and all checks

Add your screenshot here.

---

#### Screenshot 10 — Captured exit code and final summary

Add your screenshot here.

### Notes

**1. What is the baseline status?**

Add your answer here.

**2. Which evidence proves the Service has a ready backend?**

Add your answer here.

**3. What is the difference between a warning and a failure?**

Add your answer here.

---

# Task 6 — Configure and Run the Claude Code Skill

## Goal

Connect the Bash script to a manually invoked Claude Code evidence-analysis workflow.

### Evidence

#### Screenshot 11 — SKILL.md showing frontmatter and safety rules

Add your screenshot here.

---

#### Screenshot 12 — Baseline `/k8s-triage` analysis

Add your screenshot here.

### Notes

**1. What does Bash do in this workflow?**

Add your answer here.

**2. What does Claude do in this workflow?**

Add your answer here.

**3. Why are instructions alone insufficient to guarantee read-only access?**

Add your answer here.

---

# Task 7 — Simulate One Controlled Incident

## Goal

Create one controlled failure, save its evidence, and let Claude analyze it without executing recovery.

### Evidence

#### Screenshot 13 — Failed or unready Pod evidence and current Service backends

Add your screenshot here.

---

#### Screenshot 14 — Failed `/k8s-triage` analysis with evidence and a suggested manual recovery command

Add your screenshot here.

---

#### Screenshot 15 — Saved `incident-failure-report.txt`

Add your screenshot here.

### Notes

**1. Which failure did you choose?**

Add your answer here.

**2. Which checks failed, and what evidence proved it?**

Add your answer here.

**3. Did Claude execute recovery? Why is that important?**

Add your answer here.

**4. Why might the Service retain ready backends during the incident?**

Add your answer here.

---

# Task 8 — Recover Manually and Verify

## Goal

Restore the healthy configuration yourself and verify recovery using a second triage report.

### Evidence

#### Screenshot 16 — Recovered Pods and ready Service backends

Add your screenshot here.

---

#### Screenshot 17 — Recovered `/k8s-triage` analysis

Add your screenshot here.

---

#### Screenshot 18 — Reports directory showing failure and recovery reports

Add your screenshot here.

---

#### Screenshot 19 — `incident-summary.md` with all required sections

Add your screenshot here.

### Notes

**1. What manual recovery action did you execute?**

Add your answer here.

**2. What proves the workload recovered?**

Add your answer here.

**3. Why is the second triage run necessary?**

Add your answer here.

**4. Why should AI not automatically restart every failing production workload?**

Add your answer here.

### Remaining WARN Results

Explain any WARN results remaining after recovery, or write “None.”

Add your explanation here.

---

# Task 9 — Publish Your Kubernetes Incident-Triage Workflow on LinkedIn

## Goal

Share the incident-triage workflow you completed and explain the role of human control.

### Evidence

#### Screenshot 20 — Published LinkedIn post about your Kubernetes incident-triage workflow

Add your screenshot here.

### LinkedIn Post URL

Add your published LinkedIn post URL here.

---

# Required Project Files

Include these files in your project repository:

- `CLAUDE.md`
- `lab-config.md`
- `scripts/k8s-triage.sh`
- `.claude/skills/k8s-triage/SKILL.md`
- `reports/incident-failure-report.txt`
- `reports/recovery-report.txt`
- `incident-summary.md`

Your `incident-summary.md` must contain:

1. Reported Symptom
2. Evidence Collected
3. Most Likely Cause
4. Human-Approved Recovery Action
5. Verification
6. Safety Decision
7. Agentic Loop Mapping

---

# Final Summary

## Evidence and Cause

Summarize the evidence and the supported cause of the incident.

Add your summary here.

## Manual Recovery

Describe the command or configuration change you performed.

Add your summary here.

## Verification

Explain how the recovery report and Kubernetes outputs confirmed recovery.

Add your summary here.

## Safety Decision

Explain how you kept Claude’s role limited to evidence gathering and analysis.

Add your summary here.
>>>>>>> upstream

---

# Submission Instructions

- Include all 20 required screenshots under their corresponding Evidence sections.
- Complete all Notes questions in your own words.
- Include the required project files in your linked repository.
- Include your published LinkedIn post URL.
- Use clear, readable screenshots showing relevant commands, configuration, and outputs.
- Display your full name before taking required terminal screenshots.
- Preserve the failure report before recovery.
- Ensure the baseline and recovery reports contain no FAIL results.
- Explain any WARN results remaining after recovery.
- Review reports, logs, configuration records, and screenshots before publishing.
- Do not expose credentials, kubeconfig contents, tokens, certificates, JWT secrets, or confidential account information.

---

# Completion Checklist

- [ ] Confirmed the personal lab context
- [ ] Discovered and recorded application configuration
- [ ] Recorded the original working image
- [ ] Captured healthy Pods and ready Service backends (Screenshot 1)
- [ ] Created the workspace (Screenshot 2)
- [ ] Configured CLAUDE.md (Screenshot 3)
- [ ] Reviewed the five-check triage plan (Screenshot 4)
- [ ] Configured and reviewed the supplied script (Screenshots 5–7)
- [ ] Passed Bash syntax validation and set executable permission (Screenshot 8)
- [ ] Captured a baseline report with no FAIL (Screenshot 9)
- [ ] Captured the correct exit code (Screenshot 10)
- [ ] Configured and invoked `/k8s-triage` (Screenshots 11–12)
- [ ] Simulated one controlled incident manually (Screenshot 13)
- [ ] Reviewed Claude’s failed-state analysis (Screenshot 14)
- [ ] Saved failure evidence before recovery (Screenshot 15)
- [ ] Executed recovery manually and waited for rollout completion
- [ ] Confirmed recovered Pods and ready Service backends (Screenshot 16)
- [ ] Captured recovery analysis with no FAIL (Screenshot 17)
- [ ] Explained any remaining warnings
- [ ] Saved failure and recovery reports (Screenshot 18)
- [ ] Completed the seven-section incident summary (Screenshot 19)
- [ ] Answered all Notes questions
- [ ] Published the LinkedIn post (Screenshot 20)
- [ ] Included the LinkedIn post URL
- [ ] Included all required project files
- [ ] Checked that no sensitive information is exposed

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
