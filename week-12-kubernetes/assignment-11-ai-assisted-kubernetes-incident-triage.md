# Assignment 11 — AI-Assisted Kubernetes Incident Triage

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash script that checks the health of the Book Review App running on your Kubernetes cluster, connect it to Claude Code as a reusable `/k8s-triage` skill, then deliberately break your own deployment, use the skill to diagnose the failure from evidence alone, recover it yourself, and verify the fix. This closes out the AI Assignment series with the same discipline applied at every layer so far — a single Linux host, two clouds, Terraform, Ansible, a CI/CD pipeline, a Docker image, and now an orchestrated Kubernetes cluster: gather evidence with a read-only tool, let AI analyze it and recommend a fix, keep a human in charge of every action that changes the cluster, and verify again.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm every Book Review App pod is `Running` and `Ready` and that its Service has active Endpoints, then set up a workspace folder for this assignment before writing any automation.

### Evidence

#### Screenshot 1 — `kubectl get pods` and `kubectl get endpoints` showing the Book Review App healthy

![PALLOTI](./screenshots/12wk111.png)

---

#### Screenshot 2 — Workspace folder structure for this assignment

![PALLOTI](./screenshots/12wk112.png)

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

### Notes

Compare this incident to the Nginx incident from Week 3's AI-Assisted Linux Health Check. What stayed the same about the process, and what changed about the evidence you had to gather?

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

---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:
- All 13 required screenshots
- Do not expose kubeconfig contents, cluster certificates, or tokens

---

# Completion Checklist

- [ ] Task 1: Healthy baseline confirmed and workspace created (Screenshots 1–2)
- [ ] Task 2: `CLAUDE.md` safety rules added (Screenshot 3)
- [ ] Task 3: Read-only five-check plan produced before scripting (Screenshot 4)
- [ ] Task 4: Triage script built and validated (Screenshots 5–6)
- [ ] Task 5: Script run against the healthy cluster (Screenshot 7)
- [ ] Task 6: `/k8s-triage` skill built and run against the healthy baseline (Screenshots 8–9)
- [ ] Task 7: Incident simulated and correctly diagnosed without being applied (Screenshots 10–11)
- [ ] Task 8: Recovery applied by the human, verified, and summarized (Screenshots 12–13, Notes)
- [ ] No kubeconfig, tokens, or cluster credentials exposed

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
