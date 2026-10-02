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

![screenshot1](screenshots/browser-output-app.png)

---

#### Screenshot 2 — Output of `ip a`

![screenshot2](screenshots/output-ip-a.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![screenshot3](screenshots/output-sudo-ss.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

![screenshot4](screenshots/output-sudo-ufw.png)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

The output of this command: sudo ss -lptn 'sport = :80' shows nginx as user and the address port as
0.0.0.0:80 which indicates that nginx is bound to all IPv4 interfaces.

---

**2. What proves SSH is active on port 22?**

The same sudo ss -lptn 'sport = :80' command shows systemd user listening on port 22.

---

**3. Did you find any unexpected open ports? Explain briefly.**

I noticed that port 53 was also open. This is necessary for loopback purposes reachanble from inside
the instance. 

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![screenshot1](screenshots/output-systemctl-status-nginx.png)

---

#### Screenshot 2 — Output of `sudo nginx -t`

![screenshot2](screenshots/output-nginx-t.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![screenshot3](screenshots/output-sudo-ss-lptn.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

The whole site goes down and clients get connection refused error.

---

**2. What's your basic rollback plan?**

I shall undertake a config rollback. This means that the nginx directory should be version-controlled. 

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![screenshot1](screenshots/output-sudo-tail-access.log.png)

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![screenshot2](screenshots/output-sudo-tail-error.log.png)

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![screenshot3](screenshots/output-sudo-journalctl.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

There were no errors. It means nginx did not log any errors.

---

**2. If there were no errors, what does that indicate about the system?**

This means nginx hasn't hit anything at or above its configured log level since the file was last rotated

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

This means that the traffic flow is completely through. Every client in the internet can reach it and the app renders correctly.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![screenshot1](screenshots/output-uptime.png)

---

#### Screenshot 2 — Output of `free -h`

![screenshot2](screenshots/output-free-h.png)

---

#### Screenshot 3 — Output of `df -h`

![screenshot3](screenshots/output-df-h.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![screenshot4](screenshots/output-sudo-du-sh.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Disk

---

**2. What happens if disk becomes 100% full in a production server?**

The server will stay up but anything that writes to disk starts failing silently.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![screenshot1](screenshots/output-ls-lah.png)


---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![screenshot2](screenshots/output-grep-R.png)


---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![screenshot3](screenshots/output-grep-n.png)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

This can be confirmed by running the command: ls lah /var/www/html. This shows which release the current directory points to.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![screenshot1](screenshots/output-sudo-nginx-2.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![screenshot2](screenshots/output-sudo-nginx-4.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![screenshot3](screenshots/output-curl-l-http.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

An error in the configuration of nginx

---

**2. How did you fix the issue?**

Corrected the configuration error

---

**3. How can you avoid this kind of issue in real production systems?**

To avoid failure due to configuration error, treat configuration as code, validate every change before release, and prevent drift across environments.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![screenshot1](screenshots/output-curl-l-3.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![screenshot2](screenshots/output-curl-l-6.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

There was a missing deployment content.

---

**2. How did you fix the issue and restore the application?**

Restored the deployment content from backup.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

The following steps can help prevent missing deployment content in real production systems:
- Automate Packaging
- Validate artifacts before deployment
- Enforce environment parity
- Use checklists that catch gaps early

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH keys provide asymmetric cryptographic authentication, making unauthorized access more difficult than with passwords. 

---

**2. Why should only required ports be open on a production server?**

Keeping only required ports open reduces the attack surface, prevents unauthorized access,and strengthens overall system security.

---

**3. Why is it important for Nginx to be enabled on boot?**

Enabling nginx on boot ensures automatic recovery, high availability, and predictable server behaviour after restart.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Publicly exposed secrets, keys or credentials lead to immediate unauthorized access, data breaches, financial loss, and full system compromise.Attackers actively scan the internet for leaked secrets.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Unused cloud resources should be shut down because they waste money, increase security risks, and create operational clutter that harms reliablity.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://lnkd.in/p/eM6c-5fV`

---

#### Screenshot — Published LinkedIn post

![screenshot](screenshots/maintenance-drill-post.png)

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

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*