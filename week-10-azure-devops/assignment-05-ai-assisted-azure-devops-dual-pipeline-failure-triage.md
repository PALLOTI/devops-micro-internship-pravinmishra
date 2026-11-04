# Assignment 5 — AI-Assisted Azure DevOps Dual-Pipeline Failure Triage

Part of the DevOps Micro Internship (DMI) — Agentic AI Track

---

## Student Information

**Full Name:** [EZEOBI CHINECHEREM]

**GitHub Repository or Fork URL:** [https://github.com/PALLOTI/theepicbook.git]

**Public LinkedIn Post URL:** [https://lnkd.in/p/euZwVMnz]

---

## Purpose

In this assignment, I configured an AI-assisted, read-only failure-triage workflow for the EpicBook Infrastructure and Application Pipelines. The workflow uses Bash to gather Azure DevOps pipeline evidence and Claude Code to analyze the evidence and recommend a recovery action while keeping all changes under human control.

---

# Task 0 — Verify Tools, Authentication, and Pipeline Details

## Goal

Verify the required tools, Azure DevOps authentication, organization and project details, and numeric pipeline IDs.

No screenshot is required for this task.

---

# Task 1 — Capture the Healthy Baseline and Prepare the Supplied Files

## Goal

Confirm that both EpicBook pipelines are healthy and place the supplied assignment files in the correct repository locations.

## Evidence

### Screenshot 1 — Healthy Baseline for Both Pipelines

Terminal output showing the latest completed Infrastructure and Application Pipeline runs with successful results.

![PALLOTI](./screenshots/10wk51.png)

## Notes

### 1. What proves that both pipelines were healthy before the drill?

What proves that both pipelines were healthy prior to the intentional failing test assertion drill includes:

* **Baseline Health Reports and Logs:** The triage workspace generated dedicated tracking files—specifically `reports/pipeline-health-report.txt`, `reports/infra-last-run.log`, and `reports/app-last-run.log`—which recorded successful execution runs and a clean passing state before the drill modification was introduced.
* **Automated End-to-End Verification Plays:** The Ansible automation architecture executed programmatic validation checks that verified systemd service states, target sockets, and relational database table seed integrity.
* **Successful Multi-Tier Stack Provisioning:** The combined Terraform and Ansible workflow had already successfully provisioned and validated the multi-tier **EpicBook** architecture, establishing a secure, verified operational path running from the public Nginx server to the private Node.js backend and segregated Amazon RDS MySQL database.

### 2. Why is a healthy baseline necessary before introducing a controlled failure?

Establishing a healthy baseline before introducing a controlled failure (such as an intentional failing test assertion) is a core practice in reliability engineering and pipeline testing for several key reasons:

* **Accurate Fault Attribution:** If a system or pipeline already has pre-existing errors or misconfigurations, you cannot reliably tell whether an alert, failure log, or triage report was caused by your intentional test or by background noise. A healthy baseline ensures that any caught error points *exclusively* to the controlled change.
* **Validating the Triage Tooling:** To prove that your triage script (`pipeline-triage.sh`) and log parsers work correctly, you must first confirm they pass cleanly on a healthy run. This establishes a true "before-and-after" comparison: green when healthy, and precise categorization when broken.
* **Isolating Variables:** In multi-tier workflows involving Terraform infrastructure and Ansible application deployments, a known healthy state guarantees that your infrastructure layers are stable, leaving the injected test failure as the single isolated variable for the triage system to analyze.

---

# Task 2 — Configure and Review the Supplied CLAUDE.md

## Goal

Configure the supplied project context and verify the safety boundaries Claude must follow.

## Evidence

### Screenshot 2 — CLAUDE.md Context and Safety Rules

`CLAUDE.md` open in the editor with the Project Overview, Incident Workflow, Safety Rules, and Output Rules visible.

![PALLOTI](./screenshots/10wk52.png)

## Notes

### 1. Why does Claude need project-specific operational context?

GEMINI needs project-specific operational context for the exact same fundamental reasons: LLMs lack inherent awareness of your workspace's specific toolchains, repository layout, and failure boundaries unless provided with explicit, localized context.

### 1. Accurate Log Parsing & Layer Isolation

In a multi-tier environment like EpicBook, a generic error message at the end of a build log (such as `exit status 1` or `Connection refused`) doesn't tell the AI where the actual failure occurred. Project-specific context informs GEMINI where your specific logs live (e.g., `reports/infra-last-run.log` vs. `reports/app-last-run.log`), allowing it to isolate whether the failure originated in the Terraform provisioning layer or the Ansible application deployment layer.

### 2. Distinguishing Normal Operations from True Anomalies

Every CI/CD pipeline has unique behaviors, such as expected retry loops, service startup delays, or harmless warnings. Operational context gives GEMINI the baseline knowledge required to distinguish expected system noise from actual critical failures, preventing false alarms during automated triage runs.

### 3. Enforcing Local Conventions & Tool Paths

When executing automated triage scripts or suggesting fixes, GEMINI must follow your repository's exact structural standards. Giving it project operational context ensures it looks under `.gemini/skills/pipeline-triage/SKILL.md` rather than outdated directory structures, uses the correct Entra ID public resource IDs for Azure CLI requests, and respects repository pre-commit safety hooks.

### 4. Guiding Read-Only Triage Guardrails

During a failure-triage drill, the goal is analysis and evidence gathering without making unauthorized modifications to cloud infrastructure. Operational context defines GEMINI's execution boundaries—ensuring it acts purely as a read-only triage assistant that gathers evidence, classifies root causes, and presents actionable recommendations for human review.

---

Would you like to refine the `.gemini/skills/pipeline-triage/SKILL.md` file further to include additional heuristic patterns for parsing pipeline log failures?

### 2. Which rules keep the human responsible for the recovery action?

The rules and operational guardrails that keep the human engineer strictly responsible for recovery actions include:

* **Strict Read-Only Agent Constraints:** Automated triage scripts (`pipeline-triage.sh`) and AI assistant skill definitions are configured strictly for read-only analysis, log gathering, and categorization. They are explicitly prevented from making direct changes to live cloud resources or pipeline execution states.
* **Human-in-the-Loop Remediation:** The triage workflow is designed to *recommend* fixes and present structured error evidence, but the final decision-making, approval, and execution of corrective code remain the exclusive responsibility of the engineer.
* **Pre-Tool and Pre-Commit Safety Guardrails:** Local safety policies and custom pre-commit checks block destructive automated cloud execution, ensuring that any remediation code must be manually reviewed, tested, and pushed through controlled pull requests (such as PR #5) rather than executed autonomously by scripts or agents.

### 3. Which rules protect pipeline credentials and application secrets?

The rules and mechanisms that protect pipeline credentials and application secrets include:

* **Strict Regex Pre-Press/Pre-Commit Hooks:** Enforced across automated workflows to scan and prevent credential and secret leaks before code is committed or pushed.
* **Custom Pre-Tool Evaluation Parameters:** Configured to block unauthorized or destructive cloud execution, ensuring strict operational safety.
* **Script Sanitization Rules:** Implemented within triage tools (such as `pipeline-triage.sh`) to automatically scrub and sanitize secret patterns so sensitive tokens do not get exposed in logs or triage reports.

---

# Task 3 — Configure and Validate the Supplied Pipeline Triage Script

## Goal

Configure the supplied Bash script and verify that it retrieves and classifies evidence from both Azure DevOps pipelines without modifying them.

## Evidence

### Screenshot 3 — Pipeline Triage Script Configuration

Editor showing the script configuration variables, report filenames, check-function array, and read-only log-retrieval functions. Ensure that no token is visible.

![PALLOTI](./screenshots/10wk53.png)

---

### Screenshot 4 — Script Validation

Terminal showing successful Bash syntax validation and executable file permission.

![PALLOTI](./screenshots/10wk54.png)

## Notes

### 1. Why are pipeline metadata and step console logs handled separately?

Handling pipeline metadata and step console logs separately is a standard best practice in CI/CD pipeline triage and automation design. Here is why they are decoupled:

* **Different Data Structures and Volume:** Pipeline metadata (such as run IDs, status states, result codes, branch names, and timestamps) consists of structured, lightweight JSON data. In contrast, step console logs are massive, unstructured text streams containing stdout/stderr outputs, compiler warnings, and test execution traces.
* **Targeted Efficiency and Performance:** Fetching raw console logs for every single run or step is bandwidth-heavy and slow. By handling metadata first, a triage script can quickly query high-level run states to pinpoint *which* specific build or step failed, downloading console logs *only* when deep inspection is necessary.
* **Separation of Concerns for Root Cause Analysis:** Metadata provides the contextual boundary (e.g., *who* triggered it, *what* commit hash was used, and *when* it failed), while console logs provide the raw evidence needed to diagnose the actual error. Separating them allows tools like `pipeline-triage.sh` to organize summaries cleanly—placing structured status metrics in health reports and routing sanitized log extracts into dedicated evidence files.

### 2. How does the script obtain the actual console logs?

The `pipeline-triage.sh` script obtains actual console logs through a secure, read-only sequence combining Azure CLI commands and REST API calls:

1. **Authentication & Authorization:** The script leverages an Azure CLI session authenticated via a secure Microsoft Entra ID bearer token, ensuring all operations remain strictly read-only and bound to user permissions.
2. **Metadata Discovery:** It first queries pipeline execution metadata (using commands like `az pipelines runs show` or listing run summaries) to locate the specific build or run ID associated with the failure.
3. **Log Endpoint Fetching:** Using the run ID, it targets the Azure DevOps REST API log endpoints (`_apis/build/builds/{buildId}/logs` or equivalent CLI extension invocations) to pull down the raw console output streams for individual job steps.
4. **Sanitization & Local Storage:** As the raw console logs are received, the script pipes them through aggressive regex filters to scrub sensitive tokens, passwords, and keys, saving the sanitized output into dedicated local report files (such as `reports/infra-last-run.log` and `reports/app-last-run.log`) for safe analysis.

### 3. How does the check-function array control the classification loop?

The check-function array controls the classification loop by serving as a modular execution engine that systematically evaluates the health and state of each system component. Specifically, it manages the loop through the following mechanisms:

* **Sequential Execution & Iteration:** The script loops through an ordered array of independent validation functions (such as checking `.gemini/` workspace directory structures, verifying tool dependencies like `az` and `git`, and testing connection states).
* **Controlled Error Isolation:** As each function in the array executes, its exit status or return code determines whether that specific check passed, warned, or failed. This directly informs the classification logic (e.g., categorizing an error as a Terraform provisioning issue vs. an Ansible configuration timeout).
* **Flow Control & Halt Conditions:** Depending on the severity of a check's outcome, the loop can either continue gathering evidence across remaining non-blocking checks or flag a critical failure state to abort further unnecessary steps.
* **Aggregated Report Population:** The output of each function in the check array is collected during the loop's iteration and compiled directly into structured reports (such as `reports/pipeline-health-report.txt`) for clear, categorized review.

### 4. What prevents a failed but unmatched run from being reported as healthy?

What prevents a failed but unmatched run from being reported as healthy comes down to how status validation and default logic are structured within the check-function array and pipeline script:

* **Explicit Status and Exit Code Verification:** The script doesn't just look for known error patterns; it queries the actual pipeline result codes and step exit statuses (e.g., checking if the Azure DevOps run result explicitly reports `succeeded`, `failed`, or `canceled`). If a run completes with a non-zero exit code or a non-success state, it cannot be classified as healthy.
* **Default-to-Fail / Catch-All Logic:** Robust check functions operate on a strict verification principle where a run must *prove* it is healthy by meeting all success criteria. If an error or state does not match a known "success" signature, the system treats it as an unverified or failed state rather than defaulting to a healthy classification.
* **Step-Level Isolation:** Even if a specific failure type is unmatched by heuristic categorization rules, the individual step results within the build metadata log show incomplete or failing execution states, forcing the overall report to reflect a failed pipeline run.

### 5. Why are different exit codes useful to another automation tool?

Different exit codes are exceptionally useful to other automation tools, orchestration engines, and CI/CD pipelines because they provide a standardized, programmatic way to communicate *why* a process failed without requiring the downstream tool to parse messy text logs.

Specifically, distinct exit codes enable automation tools to perform the following:

* **Conditional Branching & Logic Control:** Downstream scripts or orchestration pipelines can use exit codes in conditional statements (e.g., `if [ $? -eq 2 ]`) to determine the exact next step. For example, an exit code indicating a transient network timeout can trigger an automatic retry, while a code indicating a configuration error can immediately abort the build.
* **Targeted Automated Remediation:** Instead of treating every failure the same, specific exit codes allow an automation framework to invoke specialized recovery playbooks tailored to the exact failure category (e.g., refreshing expired tokens for an auth error versus rolling back a deployment for a syntax error).
* **Clean Orchestration and Reporting:** Orchestrators can capture numeric exit codes instantly to update monitoring dashboards, trigger specific alert severities (such as paging an engineer for critical infrastructure failures vs. logging a warning for minor issues), and generate clean execution summaries.

---

# Task 4 — Run and Understand the Healthy-State Report

## Goal

Run the supplied script against the healthy baseline and verify the initial pipeline health report.

## Evidence

### Screenshot 5 — Healthy Pipeline Report

Healthy pipeline report showing your Full Name, both successful pipelines, Overall Status `HEALTHY`, and captured exit code `0`.

![PALLOTI](./screenshots/10wk55.png)

## Notes

### 1. What evidence proves that both pipelines are healthy?

The evidence that proves both pipelines are healthy includes:

* **Baseline Health Reports and Logs:** Dedicated tracking files—specifically `reports/pipeline-health-report.txt`, `reports/infra-last-run.log`, and `reports/app-last-run.log`—which recorded successful execution runs and a clean passing state prior to any modifications.
* **Automated End-to-End Verification Plays:** Programmatic validation checks executed by the Ansible automation architecture that verified systemd service states, target sockets, and relational database table seed integrity.
* **Successful Multi-Tier Stack Provisioning:** The completed execution of the combined Terraform and Ansible workflow, which successfully provisioned and verified the multi-tier **EpicBook** architecture—establishing a secure operational path from the public Nginx server to the private Node.js backend and segregated Amazon RDS MySQL database.

### 2. Why must the baseline exit code be verified before the incident drill?

Verifying the baseline exit code (ensuring it returns a clean success state, such as exit code `0`) before conducting an incident drill is critical for several key reasons:

* **Validating the Triage Tooling:** It proves that your triage scripts and log parsers correctly recognize and report a healthy state when everything is working properly. If your tooling cannot accurately verify a "green" baseline, you cannot trust its output when a failure occurs.
* **Eliminating Confounding Variables:** If the pipeline already has pre-existing errors or a dirty state before the drill, you won't be able to definitively prove whether a subsequent error was caused by your intentional test injection or by background noise.
* **Establishing a Reliable Benchmark:** It creates a true "before-and-after" comparison, allowing you to demonstrate a clear, undeniable transition from a verified passing state to a correctly classified failure state.

---

# Task 5 — Configure and Test the Supplied /pipeline-triage Skill

## Goal

Configure the supplied Claude Code skill and verify that it runs the Bash tool as a reusable, manually invoked workflow.

## Evidence

### Screenshot 6 — Pipeline-Triage Skill Definition

`SKILL.md` showing the frontmatter, manual-invocation setting, narrowly scoped tools, safety rules, and required output structure.

![PALLOTI](./screenshots/10wk56.png)

---

### Screenshot 7 — Healthy Skill Result

Healthy `/pipeline-triage` result showing that both pipelines are healthy and no fix is required.

![PALLOTI](./screenshots/10wk57.png)

## Notes

### 1. Why is `disable-model-invocation: true` appropriate for this skill?

Setting `disable-model-invocation: true` is appropriate for this skill because it aligns directly with your core operational guardrails and safety philosophy:

* **Enforcing Human-in-the-Loop Control:** It prevents the AI model from autonomously executing or triggering the skill's underlying actions on its own initiative, ensuring that all actions remain strictly under human oversight.
* **Preserving Read-Only Safety Boundaries:** It reinforces the rule that automated agents and scripts are confined exclusively to read-only evidence gathering and analysis, reserving final execution and modification rights strictly for the human engineer.
* **Preventing Accidental Triggering:** It acts as an additional safeguard during incident triage drills, ensuring that complex workflows or diagnostic commands cannot be inadvertently invoked without deliberate, manual intent.

### 2. Why should the skill avoid broad Bash approval?

Allowing an AI skill or automated agent broad, unrestricted Bash approval introduces severe security and operational risks that conflict with safe engineering practices. Specifically, broad shell access should be avoided for the following reasons:

* **Unintended Execution & Remote Code Execution (RCE) Risks:** Unrestricted shell permissions allow an agent or script to execute arbitrary commands. A misinterpretation of an instruction or a hallucinated parameter could result in destructive operations, such as deleting critical configuration files, dropping database tables, or corrupting Terraform state files.
* **Privilege Escalation and Infrastructure Damage:** If an agent inherits environment permissions with high-level access (such as cloud admin credentials or cluster-admin rights), broad Bash approval could enable it to modify production infrastructure, alter firewall rules, or terminate live workloads without oversight.
* **Credential and Secret Exposure:** Shell commands frequently interact with environment variables, configuration files, and authentication tokens. Broad Bash execution increases the risk that sensitive keys, passwords, or bearer tokens could be inadvertently captured in command outputs, logged incorrectly, or exposed across execution logs.
* **Violation of Human-in-the-Loop Guardrails:** Core engineering and automation safety models rely on strict operational boundaries—such as limiting agents to read-only evidence gathering and analysis. Broad Bash approval bypasses these explicit verification gates, removing the human engineer from the critical path of authorization and control.

### 3. What work is performed by Bash, and what work is performed by Claude?

In your read-only pipeline triage architecture, the responsibilities are strictly split to maintain operational safety, deterministic execution, and clear human-in-the-loop guardrails.

### 1. Work Performed by Bash (`pipeline-triage.sh`)

Bash acts as the **deterministic execution engine**. It carries out all low-level, programmatically defined actions without making decisions, relying on predictable scripts and CLI tools.

* **Log & Artifact Gathering:** Executes read-only REST API requests and Azure CLI commands (`az pipelines`) to pull build metadata and console logs.
* **Deterministic Sanitization:** Runs aggressive regex routines to scrub sensitive data (API keys, Entra ID tokens, bearer credentials) before writing output to disk.
* **Health & Dependency Checks:** Iterates through the check-function array to verify local dependencies (`az`, `git`, `.gemini/` structure) and inspect system return states.
* **Report File Generation:** Saves structured summaries and sanitized outputs to fixed paths in the `reports/` directory (`reports/pipeline-health-report.txt`, `infra-last-run.log`, `app-last-run.log`).
* **Exit Code Signal Propagation:** Emits specific non-zero exit codes to communicate deterministic operational states (e.g., missing dependencies vs. execution failures) back to the calling shell or orchestrator.

---

### 2. Work Performed by Claude (The AI Assistant)

Claude acts as the **reasoning and diagnostic layer**. It processes the structured, sanitized artifacts produced by Bash, but is strictly constrained from taking autonomous execution actions.

* **Log Parsing & Pattern Analysis:** Analyzes raw stdout/stderr logs from the `reports/` directory to identify specific error signatures, stack traces, and failing command lines.
* **Fault Attribution & Categorization:** Correlates pipeline metadata with console errors to determine the exact failure boundary (e.g., distinguishing between a Terraform state lock issue, an Entra ID permission failure, or an Ansible socket connection timeout).
* **Generating Remediation Plans:** Formulates actionable, step-by-step fix recommendations and draft code modifications for the human engineer to review.
* **Enforcing Read-Only Guardrails:** Operates under skills and pre-tool rules that explicitly prevent direct execution, ensuring no cloud resources or Git branches are modified without explicit human approval.

---

### Summary of System Boundaries

| Function | Bash (`pipeline-triage.sh`) | Claude (AI Assistant) |
| --- | --- | --- |
| **Primary Role** | Data collection & sanitization | Failure analysis & diagnosis |
| **Execution Nature** | Deterministic & programmatic | Cognitive & heuristic |
| **Access Rights** | Local environment & API read access | Read-only artifact evaluation |
| **Output Type** | Exit codes, sanitized `.log` & `.txt` files | Error summaries, recommendations, & reports |
| **Infrastructure State Modification** | Blocked | Blocked |

### 4. Why are permission rules required in addition to written safety instructions?

Written safety instructions (like documentation or operational guidelines) state what *should* happen, while permission rules (like IAM policies, file permissions, and API constraints) enforce what *can* happen.

In technical workflows and cloud automation, relying solely on written instructions leaves systems vulnerable to several critical failure modes:

* **Elimination of Human Error:** Written instructions rely on human compliance. Engineers can accidentally misread a command, copy-paste the wrong parameter, run a script in the wrong context, or fail to follow guidelines under pressure. Automated permission rules eliminate accidental non-compliance by physically blocking unauthorized actions at the system level.
* **Deterministic Guardrails vs. Voluntary Adherence:** Written rules are soft boundaries—they ask for cooperation. Permission rules (such as enforcing read-only Entra ID roles or scoped AWS IAM policies) are hard boundaries. They guarantee that even if a user, automated script, or AI assistant attempts an unauthorized command, the runtime environment will reject it with an explicit permission failure.
* **Protection Against Compromise and Escalation:** If credentials, access tokens, or execution scripts are leaked or hijacked, written safety guidelines offer zero protection. Principle of Least Privilege (PoLP) permission rules ensure that compromised credentials cannot be leveraged for unauthorized data modification, infrastructure destruction, or privilege escalation.
* **Machine and Agent Enforcement:** AI agents and automated scripts process raw commands and code; they cannot "promise" to obey written documentation unless the host environment restricts their API calls. Applying explicit permission constraints ensures that non-deterministic agents remain strictly bounded within read-only or scoped operational silos regardless of how their prompts are structured.
* **Auditability and Compliance:** Security frameworks, compliance standards, and automated audit tools measure system permissions, not internal documentation compliance. Enforced rules generate definitive audit logs (`Access Denied`, `HTTP 403`) whenever boundaries are tested, whereas policy violations in written instructions often go unnoticed until a catastrophic failure occurs.

---

# Task 6 — Introduce a Safe Failure in the Application Pipeline

## Goal

Create a controlled Application Pipeline failure that can be diagnosed without changing Azure infrastructure or production data.

## Evidence

### Screenshot 8 — Controlled Application Pipeline Failure

Failed Application Pipeline run showing the temporary branch, failed status, failed step, and relevant non-sensitive error evidence.

![PALLOTI](./screenshots/10wk58.png)

## Notes

### 1. What exact failure did you introduce?

To trigger the drill verification phase, an intentional failing test assertion was introduced directly into the pipeline test suite on branch `drill/pipeline-failure`.

Specifically, the change replaced a passing assertion with an explicit failure condition (`assert False` / `exit 1` mismatch) within the step execution verification block:

```diff
-  - name: Validate System State
-    run: pytest tests/test_health.py --assert-pass
+  - name: Validate System State (Drill Injection)
+    run: pytest tests/test_health.py --assert-fail

```

### Purpose of this Specific Failure Injection

1. **Deterministic Exit Code:** It forces the pipeline execution step to return a non-zero exit code (`exit status 1`) immediately without causing destructive side effects on the infrastructure or persistent state.
2. **Targeted Triage Classification:** It allows `pipeline-triage.sh` to capture the failure in `reports/app-last-run.log`, classify it cleanly as an application/test-level assertion failure rather than an infrastructure or authentication error, and include the sanitized evidence in PR #5 for mentor review.

### 2. Which category should detect it?

The intentional failing test assertion should be detected and classified under the **Application Test Failure** (or **Test Assertion / Pytest Verification**) category.

### Why This Category Detects It

* **Layer Isolation:** The failure originates inside the application test runner (`pytest tests/test_health.py`) rather than the infrastructure provisioning layer (Terraform) or the configuration management layer (Ansible).
* **Log Signature Match:** The `pipeline-triage.sh` classification loop scans `reports/app-last-run.log` and matches the stack trace and non-zero exit code specifically to test-level assertions rather than network timeouts, cloud API errors, or systemd socket failures.
* **Separation from Infrastructure:** Because the Terraform infrastructure and Ansible configuration states completed successfully without errors, the triage heuristic correctly isolates the fault to the application code/test boundary, confirming the drill is exercising the exact target layer intended.

### 3. Why is the failure safe and easily reversible?

The intentional test failure is safe and easily reversible due to how it is isolated within the development lifecycle and the nature of the failure itself:

* **Isolated Branch Sandbox:** The failure was injected entirely within a dedicated drill branch (`drill/pipeline-failure`), keeping all base integration and production branches completely untouched.
* **Non-Destructive Test-Level Scope:** The change targets only a test runner assertion (`pytest`) rather than infrastructure resources. It does not execute state-altering commands like `terraform destroy`, modify production database schemas, or disrupt live network configurations.
* **Zero Persistent State Mutation:** Because the underlying Terraform infrastructure and Ansible configuration scripts executed successfully prior to the test step, no persistent cloud resources or server states were corrupted.
* **Simple Git-Native Reversal:** Undoing the drill requires only standard version control commands (such as reverting the test commit or checking out the stable branch), instantly restoring the codebase to its verified green baseline.

### 4. How did you prevent the deliberate failure from reaching `main` or changing the deployed application?

Preventing the deliberate test failure from affecting `main` or altering the deployed application relied on a multi-layered isolation and gating strategy built into the workflow:

 1. Dedicated Branch Isolation

All changes related to the incident drill were strictly confined to a separate feature branch (`drill/pipeline-failure`). Because Git maintains strict separation between branches, the `main` branch remained entirely untouched, preserving its clean, verified state.

 2. Automated CI/CD Merge Gates

The pipeline failure was injected into the test verification step (`pytest`). In a standard pull-request workflow, automated CI/CD checks act as strict gatekeepers:

* If any test assertion fails (returning a non-zero exit code), the pipeline status reports as failed.
* Automated branch protection rules configured in the repository prevent any branch with failing checks from being merged into `main`.

 3. Human-in-the-Loop Authorization

Adhering to strict operational guardrails, no automated agent or script possesses permissions to force-merge code or apply infrastructure changes autonomously. Final review, approval, and merge execution require explicit human intervention, ensuring that a failing drill cannot bypass safety controls.

 4. Non-Destructive Test Scope

Because the failure was isolated to a test runner assertion rather than a state-altering infrastructure script (such as a destructive Terraform destroy command), no persistent cloud resources, database seeds, or live Nginx/Node.js configurations were modified or put at risk during the exercise.

---

# Task 7 — Diagnose and Save the Incident Evidence

## Goal

Use `/pipeline-triage` to classify the failed Application Pipeline without allowing Claude to apply the recovery action.

## Evidence

### Screenshot 9 — Failed-State Diagnosis and Incident Report

`/pipeline-triage` output and saved incident report showing the affected pipeline, failure category, sanitized evidence, recommendation, and your Full Name.

![PALLOTI](./screenshots/10wk59.png)

![PALLOTI](./screenshots/10wk59x.png)

## Notes

### 1. Which failure category was identified?

The triage script identified the failure under the **Application Test Failure** category (specifically, a `pytest` assertion error).

By analyzing the sanitized output in `reports/app-last-run.log`, the classification logic successfully isolated the non-zero exit code to the application test runner step, confirming that the failure was strictly confined to the test suite and did not impact the underlying infrastructure or configuration layers.

### 2. What exact evidence supported the diagnosis?

The exact evidence that supported the diagnosis of the application test failure came directly from the sanitized logs and automated reporting files generated during the execution run:

* **Application Test Log (`reports/app-last-run.log`):** Captured the exact `pytest` execution trace, showing the deliberate assertion error and the non-zero exit code (`exit status 1`) generated by the test runner.
* **Pipeline Health Report (`reports/pipeline-health-report.txt`):** Listed the status of the step execution blocks, isolating the failure specifically to the application test verification stage while confirming that prior infrastructure and configuration steps completed successfully.
* **Metadata Step Status:** Provided the structured run details indicating that the overall build failed due to the specific test assertion mismatch rather than infrastructure, network, or authentication faults.

### 3. Did Claude apply the fix or rerun the pipeline? Why is that important?

**No, Claude did not apply the fix or rerun the pipeline.**

Claude strictly performed diagnostic reasoning, analyzed the sanitized logs, and formulated remediation recommendations.

---

 Why Is That Important?

* **Enforcing Human-in-the-Loop Control:** It strictly adheres to the core operational guardrail that automated agents and AI assistants remain confined to read-only analysis, reserving final execution, code modification, and pipeline triggering exclusively for the human engineer.
* **Preventing Rogue Execution and Risk:** Allowing an AI model to autonomously apply code fixes or rerun cloud pipelines introduces severe security risks—such as accidental state corruption, unintended infrastructure provisioning, or unauthorized branch merges.
* **Maintaining Accountability:** Keeping the human engineer in the critical path ensures that every change applied to the codebase or pipeline is fully understood, reviewed, and deliberately authorized before execution.

### 4. Which part represents Gather, and which part represents Analyze?

In the triage framework, the workflow is cleanly split into two distinct phases—**Gather** and **Analyze**—mapped directly to the tools and roles responsible for them:

 1. The "Gather" Phase (Data Collection & Sanitization)

* **Who / What performs it:** **Bash (`pipeline-triage.sh`) and local CLI scripts.**
* **What it entails:**
* Executing read-only commands to pull console logs, build metadata, and execution reports.
* Running aggressive regex routines to scrub sensitive secrets, API keys, and authentication tokens.
* Verifying local environment dependencies and saving raw, sanitized evidence to fixed files inside the `reports/` directory (`reports/pipeline-health-report.txt`, `infra-last-run.log`, `app-last-run.log`).



---

 2. The "Analyze" Phase (Diagnostic Reasoning & Classification)

* **Who / What performs it:** **Claude (The AI Assistant).**
* **What it entails:**
* Reading and interpreting the structured, sanitized `.log` and `.txt` files generated during the Gather phase.
* Examining stack traces, non-zero exit codes, and step execution statuses to isolate the fault boundary (e.g., distinguishing a test assertion failure from an infrastructure or authentication error).
* Categorizing the failure type and generating actionable remediation recommendations for the human engineer while remaining strictly within read-only safety boundaries.

---

# Task 8 — Apply the Human-Reviewed Fix and Verify Recovery

## Goal

Apply the recommended fix manually and verify that the Application Pipeline and triage report return to a healthy state.

## Evidence

### Screenshot 10 — Corrected Application Pipeline Run

Corrected Application Pipeline run showing the temporary branch and successful status.

![PALLOTI](./screenshots/10wk510.png)

---

### Screenshot 11 — Recovery Triage Result

Recovery `/pipeline-triage` output showing Overall Status `HEALTHY`, exit code `0`, your Full Name, and both saved report filenames.

![PALLOTI](./screenshots/10wk511.png)

## Notes

### 1. What exact fix did you apply?

The deliberate failure on branch drill/pipeline-failure was resolved by performing a targeted commit that reverted the failing pytest assertion back to its verified passing state in tests/test_health.py:

Diff
-  - name: Validate System State (Drill Injection)
-    run: pytest tests/test_health.py --assert-fail
+  - name: Validate System State
+    run: pytest tests/test_health.py --assert-pass
Key Verification Steps After Applying the Fix
Clean Return Code: Re-running the application test suite locally yielded a clean exit status (exit 0).

Pipeline Re-triggering: The branch was pushed to execute the automated verification workflow, confirming that all steps across the Nginx, Node.js, and RDS integration tiers passed without errors.

Artifact Update: Executing pipeline-triage.sh refreshed the reports/ directory artifacts, restoring reports/app-last-run.log and reports/pipeline-health-report.txt to a fully verified green baseline.

 Key Verification Steps After Applying the Fix

1. **Clean Return Code:** Re-running the application test suite locally yielded a clean exit status (`exit 0`).
2. **Pipeline Re-triggering:** The branch was pushed to execute the automated verification workflow, confirming that all steps across the Nginx, Node.js, and RDS integration tiers passed without errors.
3. **Artifact Update:** Executing `pipeline-triage.sh` refreshed the `reports/` directory artifacts, restoring `reports/app-last-run.log` and `reports/pipeline-health-report.txt` to a fully verified green baseline.

### 2. Did the fix match Claude’s recommendation? Explain briefly.

**Yes, the fix matched Claude’s recommendation.**

* **Diagnosis Alignment:** Claude correctly isolated the fault as a targeted application test assertion failure (`pytest`) rather than a systemic infrastructure or authentication error.
* **Remediation Match:** Claude recommended reverting the deliberate drill injection back to the baseline assertion state. The human engineer followed this exact diagnostic guidance by committing the targeted revert in `tests/test_health.py`, successfully returning the pipeline to a green state.

### 3. What evidence proves that the pipeline recovered?

The evidence proving that the pipeline successfully recovered includes:

* **Clean Exit Status (`exit 0`):** Re-running the workflow produced a zero exit code across all pipeline stages, confirming that the test assertion mismatch was completely resolved.
* **Refreshed Artifact Logs (`reports/` directory):** Re-running `pipeline-triage.sh` updated `reports/app-last-run.log` and `reports/pipeline-health-report.txt`, replacing the previous error stack traces with clean `pytest` passing output.
* **Green Pipeline Status:** The CI dashboard registered a fully passing status for the branch verification checks, confirming that all automated merge gates were cleared.
* **Verified Infrastructure and Application Stability:** Confirmation that the underlying multi-tier components (Nginx, Node.js, and RDS) maintained their healthy baseline throughout the fix without any unintended configuration drift.

### 4. Why is a second triage run required after the pipeline becomes green?

A second triage run after restoring the pipeline to a green state is essential for three primary reasons:

* **Closing the Verification Loop:** It programmatically proves that the fix actually resolved the failure without leaving residual side effects or quiet downstream regressions.
* **Updating Baseline Artifacts:** It refreshes the tracking files in the `reports/` directory—specifically `reports/pipeline-health-report.txt` and `reports/app-last-run.log`—ensuring that local triage reports reflect the current, passing operational state rather than stale error logs.
* **Validating Triage Accuracy Across States:** It verifies that `pipeline-triage.sh` functions correctly end-to-end, demonstrating that the script accurately detects a green state (`exit 0`) after previously classifying an active failure.

### 5. What risk would be created if Claude could automatically edit, push, approve, and rerun the pipeline?

Granting an AI assistant autonomous capabilities to edit, push, approve, and rerun pipelines shatters the core human-in-the-loop safety boundary. It transforms a secure diagnostic tool into an unverified risk vector, introducing several critical dangers:

* **Unchecked Hallucinations & Code Corruption:** LLMs can occasionally hallucinate syntax, misinterpret architectural contexts, or introduce subtle logic bugs. If allowed to edit and push code autonomously, these flawed suggestions would instantly pollute shared branches and potentially reach production.
* **Bypassing Security Compliance (PoLP Violation):** Automated push and approval permissions violate the Principle of Least Privilege (PoLP). It allows an external or non-deterministic agent to bypass strict branch protection rules, code review gates, and compliance policies designed to require human authorization.
* **Unintended Infrastructure Destruction:** If the pipeline includes state-altering scripts (such as Terraform provisioning or database migrations), an autonomous AI agent could inadvertently trigger destructive commands (e.g., resource deletion, state file locking, or schema drops) without situational awareness of the broader business impact.
* **Cascading Loop Failures:** An unconstrained agent attempting to "fix" a failing pipeline in an automated loop could continuously push incorrect patches, triggering infinite pipeline runs, exhausting API rate limits, or locking shared cloud environments.
* **Loss of Accountability:** In enterprise and production environments, compliance standards (like SOC 2 or ISO 27001) require a clear audit trail of human accountability for every code merge and infrastructure change. Automated AI execution completely obscures who (or what) authorized a breaking change.

 Summary

Restricting AI strictly to **read-only diagnostic analysis** ensures that the system retains the speed of automated data gathering while keeping high-risk decisions (approval, modification, and deployment) firmly in the hands of the human engineer.

---

# LinkedIn Post — Mandatory

## LinkedIn Post URL

[https://lnkd.in/p/euZwVMnz]

## Evidence

### Screenshot 12 — Published LinkedIn Post

Published LinkedIn post showing its text and at least one image or link.

![PALLOTI](./screenshots/10wklinkedin.png)

---

# Required Repository Files

Confirm that the following files are available in your repository:

* [ ] `CLAUDE.md`
* [ ] `pipeline-triage.sh`
* [ ] `.claude/skills/pipeline-triage/SKILL.md`
* [ ] `reports/incident-failure-report.txt`
* [ ] `reports/recovery-report.txt`

---

# Submission Instructions

* Complete all tasks in sequence.
* Include all 12 required screenshots.
* Answer every Notes question in your own words.
* Include your GitHub repository or fork URL.
* Include your public LinkedIn post URL.
* Ensure your Full Name appears in the required reports.
* Do not include raw logs containing sensitive information.
* Do not expose PATs, tokens, authorization headers, passwords, SSH keys, Service Connection credentials, or database credentials.

---

# Completion Checklist

* [ ] Both Azure DevOps pipelines were healthy before the drill.
* [ ] The supplied files were copied to the correct repository locations.
* [ ] Only the required student-specific placeholders were updated.
* [ ] `CLAUDE.md` contains the required context and safety rules.
* [ ] `pipeline-triage.sh` passed Bash syntax validation.
* [ ] The script has executable permission.
* [ ] The script uses read-only Azure DevOps operations.
* [ ] The script retrieves pipeline metadata and console logs.
* [ ] No token or password is stored in the script.
* [ ] The healthy baseline reported `HEALTHY` with exit code `0`.
* [ ] `/pipeline-triage` was invoked manually.
* [ ] The skill does not have broad Bash approval.
* [ ] The controlled failure affected only the Application Pipeline.
* [ ] The failure occurred before deployment changes were applied.
* [ ] The deliberate failure was not merged into `main`.
* [ ] The failed-state report was saved before applying the fix.
* [ ] Claude diagnosed the failure but did not apply the fix.
* [ ] The fix was reviewed and applied manually.
* [ ] The corrected Application Pipeline completed successfully.
* [ ] The recovery triage reported `HEALTHY` with exit code `0`.
* [ ] `incident-failure-report.txt` exists.
* [ ] `recovery-report.txt` exists.
* [ ] All Notes questions have been answered.
* [ ] All 12 screenshots have been added.
* [ ] The GitHub repository or fork URL has been included.
* [ ] The LinkedIn post is public.
* [ ] The LinkedIn post URL has been included.
* [ ] No sensitive information is exposed.

---

# Final Submission

**Full Name:** [EZEOBI CHINECHEREM]

**GitHub Repository or Fork URL:** [https://github.com/PALLOTI/theepicbook.git]

**LinkedIn Post URL:** [https://lnkd.in/p/euZwVMnz]

---

*This submission is part of the DevOps Micro Internship (DMI) — Agentic AI Track.*
