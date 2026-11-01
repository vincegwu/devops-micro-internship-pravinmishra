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

![screenshot1](screenshots/A6-output-git-remote-git-branch.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

It is necessary to create a dedicated branch instead of working on main in order to work independently on your own without affecting the work of other team members collaborating together on the same project. Your work can only be added to the main after being reviewed to ensure everything is working well.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![screenshot2](screenshots/A6-output-git-status-staged-file-ai-pr.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

Real keys should not be exposed in the code. Exposing the key increases the attack surface.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![screenshot3](screenshots/A6-hooks-pre-commit-vscode.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![screenshot4](screenshots/A6-output-git-config-core-hooksPath.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

Git never versions, clones or pushes .git/hooks. A hook that only lives there exists on your local machine only. Putting it in hooks/ gives the hook version history and code review like any other file. Everyone also gets the same copy.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

Both are gatekeepers that run before an action and can block it. They guard different actions, though. hooks/pre-commit is a Git hook that checks what you're about to commit. PreToolUse is a Claude Code hook that checks what Claude is about to do.

What they have in common

- Both run before the action, so they prevent it rather than report on it afterward.
- Both are plain scripts that make a pass/fail decision from an exit code.
- Both are local guardrails. Each machine has to set them up: core.hooksPath for Git, settings files for Claude Code.
- Both can be bypassed by whoever controls the machine (git commit --no-verify, or    editing the settings). Neither replaces server-side checks like CI or branch protection.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![screenshot5](screenshots/A6-terminal-rejected-commit.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

Line 7. It matched because it has the prefix "AKIA"

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No, it wouldnot catch a variable without the prefix. The fixed rule is limited. It cannot reason beyond what has already been set.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![screenshot5](screenshots/A6-SKILL-frontmatter.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![screenshot6](screenshots/A6-output-pr-ready.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

This is because it only reads the content and reports risks but cannot change or update any part of the code.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

They both flag secrets but in different ways. The pre-commit hooks also catches files above 1MB and enforcement. The /pre-ready also drafts PR title and description.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![screenshot7](screenshots/A6-git-commit-succeed-fix.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![screenshot8](screenshots/A6-output-pr-ready-fix.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the hardcoded key and the debug echo.

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![screenshot9](screenshots/A6-pull-request-create.png)

---

#### PR Link

https://github.com/pravinmishraaws/devops-micro-internship-interviews/pull/504

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I rephrased the content to suit my need. This is because Claude can make mistake, so I need to review its result before using it. 

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

It might convey a different idea from my intended purpose.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This is because my work has to live in my own repo since I do not have write access to the upstream repository.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

The steps that run the script and the hook represents "gather"

---

**2. Which step(s) represent Analyze?**

The step that runs the /pr-ready skill represents "analyze"

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

This is the part where a human reviews the output of claude code and takes action. A human should run the commands because claude can make mistakes and may go beyond acceptable boundaries.

---

**4. Which step is Verify?**

This is the part where commands are rerun to check the result of claude code analysis.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**
One is not enough, because each covers the other's weak spot. The hook is reliable but narrow. The skill is broad but unreliable, and it can't block anything.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/egwu-oko_dmibypravinmishra-devops-git-ugcPost-7522469873302761472-bhoY/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAEw2bJAB3kAupCs3BrMdP2uO4qDEMg0CtSs

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- Initial Git setup
- Tracking, staging, and committing changes
- Raising a Pull request and collaborating with team members in a project
- Introducing AI into Git workflow

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

Paste your forked repository URL here:

`https://github.com/vincegwu/devops-micro-internship-interviews`

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

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
