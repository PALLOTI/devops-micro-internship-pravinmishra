# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

Add your screenshot here.
![week 03](./screenshots/3w%202deployed.png)
---

#### Screenshot 2 — Output of `ip a`

Add your screenshot here.
![week 03](./screenshots/3w%202ip-a.png)
---

#### Screenshot 3 — Output of `sudo ss -tulpen`

Add your screenshot here.
![week 03](./screenshots/3w%203sstlup.png)
---

#### Screenshot 4 — Output of `sudo ufw status`

Add your screenshot here.
![week 03](./screenshots/3w%202ufw.png)
---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

Write your answer here.
sudo ss -tlnp | grep :80
which has the below screenshot
![week 03](./screenshots/3w%203sstlup.png)
---

**2. What proves SSH is active on port 22?**

Write your answer here.
sudo ss -tlnp | grep :22
---

**3. Did you find any unexpected open ports? Explain briefly.**

Write your answer here.

YES
---
All ports are specific system services because static web app (Nginx on port 80/http) is running perfectly alongside the essential background system services that Ubuntu requires to stay updated, keep time, resolve domains, and let you log in via SSH.
1> Systemd-Network (DHCP Client) >> This is your server's DHCP client. It listens on the internal AWS network interface
2> Systemd-Resolved (DNS Resolver) >> This is Ubuntu's internal DNS manager. Whenever your server needs to look up a domain name . etc

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

Add your screenshot here.
![week 03](./screenshots/3w%203systemctl.png)
---

#### Screenshot 2 — Output of `sudo nginx -t`

Add your screenshot here.
![week 03](./screenshots/3w%203nginx-t.png)
---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

Add your screenshot here.
![week 03](./screenshots/3w%203sstlup.png)
---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

Write your answer here.
The immediate consequence is usually a complete service outage for your users. Because Nginx typically sits at the very front of your infrastructure, any failure there breaks the entry point to your entire application.
---

**2. What's your basic rollback plan?**

Write your answer here.
The mechanism of your rollback depends entirely on how your application is deployed.

Nginx Reload: Run sudo nginx -t && sudo systemctl reload nginx to apply changes instantly.

Backup Revert: Pull the previous release artifact from your build registry (S3, GitHub Releases, etc.), overwrite the directory, and restart your app services.
---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

Add your screenshot here.
![week 3](./screenshots/3w%203sudotail.png)
---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

Add your screenshot here.
![week 3](./screenshots/3w%203error.png)
---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

Add your screenshot here.
![week 3](./screenshots/3w%203journal.png)
---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

Write your answer here.
NO



---

**2. If there were no errors, what does that indicate about the system?**

Write your answer here.
Inherited sockets from '5;6 which simply means nginx just performed a zero-downtime reload or binary upgrade.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Write your answer here.
The curl request was not visible in the log entries

If a curl request successfully reached your Nginx server, it would generate a log line in /var/log/nginx/access.log containing:
---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

Add your screenshot here.
![week 3](./screenshots/3w%203uptime.png)
---

#### Screenshot 2 — Output of `free -h`

Add your screenshot here.
![week 3](./screenshots/3w%203free.png)
---

#### Screenshot 3 — Output of `df -h`

Add your screenshot here.
![week 3](./screenshots/3w%203df-h.png)
---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

Add your screenshot here.
![week 3](./screenshots/3w%203du-sh.png)
---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Write your answer here.
Memory looks most critical with
---
Available Memory: Only 554mb is truly available.

**2. What happens if disk becomes 100% full in a production server?**

Write your answer here.

it triggers a dangerous chain reaction, because operating systems, databases, and web applications constantly write temporary data, session files, and logs to the disk, a complete lack of storage space causes services to fail instantly.
---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

Add your screenshot here.
![week 03](./screenshots/3w%203head-n.png)
---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

Add your screenshot here.
![week 03](./screenshots/3w%203zz.png)
---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

Add your screenshot here.
![week 03](./screenshots/3w%203tryfiles.png)
---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

Write your answer here.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

Add your screenshot here.
![week 03](./screenshots/3w%203nginx-t.png)
---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

Add your screenshot here.
![week 03](./screenshots/3w%20200k.png)
---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

Write your answer here.

---

**2. How did you fix the issue?**

Write your answer here.

There wasn't actually an active Nginx configuration failure, my web server is running and successfully serving my React app's HTML and JavaScript files (proven by the 200 OK responses in your access logs).
---

**3. How can you avoid this kind of issue in real production systems?**

Write your answer here.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

Add your screenshot here.
I didn't experience any failure
---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

Add your screenshot here.
![week 03](./screenshots/3w%20200k.png)
---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

Write your answer here
The only issue I faced was my web page not been served the browser
---

**2. How did you fix the issue and restore the application?**

Write your answer here.
Choosing the right security group inbound rules
---

**3. What steps would you take to prevent this kind of issue in real production systems?**

Write your answer here.
Choosing the right security group inbound rules
---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

Write your answer here.
When you log into a remote server (like your AWS EC2 instance) using a private key file (.pem or .pub), you aren't just using a "stronger password" you are using an entirely different, mathematically secure authentication model.

With SSH key your private key never leaves your machine during the cryptographic handshake, the fake server cannot steal it. The login attempt will simply fail, and your credentials remain safe.
---

**2. Why should only required ports be open on a production server?**

Write your answer here.

---

**3. Why is it important for Nginx to be enabled on boot?**

Write your answer here.
Restricting your server's access to only the absolute minimum required ports (a practice known as the Principle of Least Privilege) is a fundamental security requirement.
---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Write your answer here.
The Attack: Bots immediately use the leaked credentials to provision high-compute resources (such as massive GPU instances) to mine cryptocurrency or launch distributed denial-of-service (DDoS) attacks.
---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Write your answer here.
The consequence of leaving unnecessary cloud resources running is a ballooning bill. A common misconception is that you only pay for what you "use." In reality, you pay for what you provision.
---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

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