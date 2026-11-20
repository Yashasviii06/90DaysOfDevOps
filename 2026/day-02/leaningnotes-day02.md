# Day 02: What is DevOps? Setup, Architecture and Security Tools

> Notes style: easy language, real-life examples, interview answers and hands-on tasks.
> Goal of today: understand **what DevOps is, why it exists, how a pipeline looks, how to set up my machine, and what the security tools in a pipeline do.**

---

## Table of Contents
1. [What is DevOps?](#1-what-is-devops)
2. [DevOps Lifecycle and Team Structure](#2-devops-lifecycle-and-team-structure)
3. [Architecture of a Modern DevOps Pipeline](#3-architecture-of-a-modern-devops-pipeline)
4. [Developer vs DevOps Engineer](#4-developer-vs-devops-engineer)
5. [Setup: Windows, Linux and AWS](#5-setup-windows-linux-and-aws)
6. [Pipeline Tools: Compiler, Gitleaks, SonarQube, Dependency-Check, Vault, SBOM, Trivy, Pentesting](#6-pipeline-tools-explained)
7. [What I Will Learn and Benefits](#7-what-i-will-learn-and-benefits)
8. [Interview Questions and Answers](#8-interview-questions-and-answers)
9. [Quick Revision Cheat Sheet](#9-quick-revision-cheat-sheet)
10. [Day 02 Hands-on Tasks and Checklist](#10-day-02-hands-on-tasks-and-checklist)

---

## 1. What is DevOps?

### 1.1 Meaning: Dev + Ops
**DevOps = Development + Operations.**

| Word | Who | What they do |
|------|-----|--------------|
| **Dev** (Development) | Programmers | Write code, build new features, fix bugs |
| **Ops** (Operations) | System / infra / support team | Run the servers, deploy the app, keep it up 24x7, handle incidents |

**One-line definition:** DevOps is a culture + a set of practices + tools that bring Dev and Ops together so that software can be **built, tested, released and run quickly, safely and repeatedly**, mostly using automation.

### 1.2 Real-life story (restaurant)
- **Developer = chef.** Creates a new dish.
- **Operations = waiters + kitchen manager.** Serve the dish, keep the kitchen running, handle complaints.

**Before DevOps:** The chef cooks and says, "My work is done, now you serve it." The waiter says, "This dish is cold and the recipe is unclear!" Both blame each other. The customer waits a long time.

**With DevOps:** Chef and waiters work as one team. They agree on the recipe, check the food together, use machines for repetitive work, and learn from every complaint. Customers get hot food quickly.

> DevOps is **not a tool** and **not a job title only**. It is a way of working. Tools just help.

### 1.3 Before DevOps vs With DevOps

| Topic | Before DevOps (traditional) | With DevOps |
|-------|-----------------------------|-------------|
| Teams | Dev and Ops are separate, blame game | One shared responsibility |
| Release speed | Months (big releases) | Days or hours (small, frequent releases) |
| Deployment | Manual, copy files to the server | Automated pipeline |
| Testing | At the end, by hand | Automatic on every commit |
| Bugs found | Late, expensive | Early, cheap |
| Failure | "It worked on my machine!" | Same environment everywhere (containers, IaC) |
| Recovery | Slow, panic | Quick rollback, monitoring and alerts |

### 1.4 What does a DevOps engineer actually build or do?
- **CI/CD pipelines**: code push, then test, build and deploy automatically.
- **Infrastructure** (servers, networks, databases) using code (Terraform, Ansible), not by clicking.
- **Containers and clusters** (Docker, Kubernetes) so apps run the same everywhere.
- **Monitoring and alerting** (logs, metrics) so problems are found before users complain.
- **Security checks inside the pipeline** (secret scanning, code scanning, image scanning). This is called DevSecOps.
- **Automation scripts** (Bash, Python) for repeated work.
- **Incident handling and documentation** (runbooks, post-mortems).

### 1.5 Why is DevOps needed?
1. Users expect **fast updates** and **zero downtime**.
2. Manual work causes **human errors**.
3. Big releases are **risky**. Small releases are safe.
4. Security problems found late are **very costly**.
5. Companies run on **cloud** (AWS, Azure, GCP), which needs automation to manage.

### 1.6 Benefits of DevOps
- ⚡ **Faster delivery** of features
- 🐞 **Fewer bugs** (automatic tests) and **faster fixes**
- 🔁 **Repeatable and reliable** deployments
- 🤝 **Better teamwork** and less blame
- 💰 **Lower cost** (automation saves time, cloud saves hardware)
- 🔐 **Better security** (checks run on every change)
- 😌 **Less stress** (rollback is easy, monitoring is ready)

### 1.7 Where is DevOps used?
Almost everywhere software is built: banking apps, food delivery, e-commerce, streaming, startups, government portals, and any company that deploys to the cloud. Example: a food delivery app releases a small fix at 3 PM. The pipeline tests it, scans it, deploys it and monitors it, and no user notices anything.

---

## 2. DevOps Lifecycle and Team Structure

### 2.1 The DevOps lifecycle (the infinity loop ♾️)

```text
 PLAN → CODE → BUILD → TEST → RELEASE → DEPLOY → OPERATE → MONITOR
   ↑                                                         │
   └──────────────── feedback goes back to PLAN ─────────────┘
```

| Stage | Meaning | Example tools |
|-------|---------|---------------|
| **Plan** | Decide what to build | Jira, Trello, GitHub Issues |
| **Code** | Write and store code | Git, GitHub, GitLab |
| **Build** | Convert code into a runnable package | Compiler, Maven, dotnet build, Docker |
| **Test** | Check that nothing is broken | Unit tests, SonarQube, Trivy |
| **Release** | Approve and version the package | GitHub Releases, registries |
| **Deploy** | Put it on servers | GitHub Actions, Jenkins, ArgoCD, Kubernetes |
| **Operate** | Keep the system running | Linux, AWS, Terraform, Ansible |
| **Monitor** | Watch health and learn | Prometheus, Grafana, ELK |

### 2.2 Core principles (remember as **CALMS**)
- **C**ulture: shared ownership, no blame
- **A**utomation: automate everything repeatable
- **L**ean: small batches, remove waste
- **M**easurement: measure speed and failures
- **S**haring: share knowledge and documentation

### 2.3 Team structure (how DevOps looks in a company)
- **Product / Dev team:** writes features.
- **DevOps / Platform team:** builds pipelines, infra and tooling.
- **QA:** automated and manual testing.
- **Security (SecOps):** policies, scans, pentests. In DevSecOps, security checks are part of the pipeline.
- **SRE (Site Reliability Engineer):** focuses on reliability, SLIs/SLOs and incident response.

---

## 3. Architecture of a Modern DevOps Pipeline

### 3.1 Big-picture diagram

```text
 Developer
    │  git push
    ▼
 ┌────────────┐
 │ Git Repo   │  (GitHub / GitLab)
 └─────┬──────┘
       │ triggers
       ▼
 ┌───────────────────────── CI (Continuous Integration) ─────────────────────────┐
 │ 1. Gitleaks            → no secrets (passwords/keys) in the code?              │
 │ 2. Compile / Build     → does the code build?                                  │
 │ 3. Unit tests          → does the logic work?                                  │
 │ 4. SonarQube           → code quality + security bugs (SAST)                   │
 │ 5. Dependency-Check    → known vulnerabilities in libraries (SCA)              │
 │ 6. Docker build        → package the app into an image                         │
 │ 7. Trivy               → scan the image for vulnerabilities                    │
 │ 8. SBOM                → list of everything inside the app                     │
 └───────────────────────────────┬─────────────────────────────────────────────────┘
                                 ▼
                       ┌──────────────────┐
                       │ Artifact/Registry │  (Docker Hub / ECR / GHCR)
                       └────────┬─────────┘
                                ▼
 ┌───────────────────── CD (Continuous Delivery / Deployment) ───────────────────┐
 │ Deploy to Staging → automated tests + DAST / pentest → approval → Production   │
 │ (Kubernetes / EC2)      Secrets come from HashiCorp Vault, not from code       │
 └───────────────────────────────┬─────────────────────────────────────────────────┘
                                 ▼
                    Monitoring + Logs + Alerts (Prometheus, Grafana)
                                 │
                                 └──► feedback to developers
```

### 3.2 Simple explanation of each block
- **Git repo:** the single source of truth for code (and for infra code).
- **CI server / runner:** a machine that runs your pipeline steps (GitHub Actions, Jenkins).
- **Artifact / registry:** a store for the built output (Docker images, packages).
- **Infrastructure:** servers or clusters on the cloud (AWS EC2, EKS).
- **Secrets manager:** safe place for passwords and keys (Vault, AWS Secrets Manager).
- **Monitoring:** tells you if the app is healthy.

### 3.3 CI vs CD
- **CI (Continuous Integration):** every code push is automatically built and tested.
- **Continuous Delivery:** the code is always ready to release, with a manual approval click.
- **Continuous Deployment:** every passing change goes to production automatically.

### 3.4 DevSecOps and "Shift Left"
**Shift left** = find problems as early (left) as possible in the lifecycle. Fixing a bug while coding is cheap. The same bug found in production is far more expensive (downtime, angry users, emergency fixes). So we put security tools (Gitleaks, SonarQube, Trivy) inside the pipeline from the start.

---

## 4. Developer vs DevOps Engineer

| Point | Developer | DevOps Engineer |
|-------|-----------|-----------------|
| Main focus | Write features and business logic | Build, deliver, run and secure the system around the code |
| Output | Source code, APIs, UI | Pipelines, infrastructure, automation, monitoring |
| Key question | "Does the feature work?" | "Can we ship it safely, repeatedly and keep it running?" |
| Tools | C#, Java, Python, SQL, frameworks, IDE | Linux, Git, Docker, Kubernetes, Terraform, CI/CD, AWS, monitoring |
| Works with | Product managers, QA | Developers, QA, security, cloud, support |
| Failure looks like | A bug in a feature | A failed deployment, downtime, slow pipeline |
| Typical day | Code, review, fix bugs | Fix pipelines, automate, scale servers, handle alerts |
| Mindset | Create | Automate, stabilize and improve |

**Easy example:** A developer writes the login feature. A DevOps engineer makes sure that when that code is pushed, it is tested, scanned, packaged, deployed to AWS, and monitored, with no secrets leaked and a one-click rollback ready.

> A good developer who knows DevOps is very powerful. The goal is to **work together**, not to compete.

---

## 5. Setup: Windows, Linux and AWS

### 5.1 What do I need? (prerequisites)

| Need | Why |
|------|-----|
| A laptop/PC with 8 GB RAM (16 GB is better), 30 GB free disk | Docker and Linux run on it |
| Internet connection | Downloading images, tools, packages |
| Git + GitHub account | Store notes and code, daily commits |
| Terminal (Bash) | DevOps work happens in the command line |
| Code editor (VS Code) | Edit files, markdown, YAML |
| Docker | Run tools and apps in containers |
| AWS account | Practice cloud: servers, networks, IAM |
| LinkedIn account | Learn in Public |

### 5.2 Why Linux? (very important)
- Most servers in the cloud run **Linux**.
- Almost every DevOps tool is built for Linux first.
- You will use the Linux terminal daily. So even on Windows, we set up a Linux environment.

### 5.3 If I have Windows 10/11: use WSL2 (Linux inside Windows)

1. Open **PowerShell as Administrator** and run:
   ```powershell
   wsl --install
   ```
   Restart the PC when asked. (This installs WSL2 and Ubuntu by default.)
2. Open **Ubuntu** from the Start menu, create a Linux username and password.
3. Check the version:
   ```powershell
   wsl -l -v
   ```
   The VERSION column should show `2`.
4. Install **VS Code** and the **WSL** extension, then run `code .` from the Ubuntu terminal.
5. Install **Docker Desktop** and turn on "Use the WSL 2 based engine".
6. Inside Ubuntu, install the basics (same as the Linux steps below).

> Tip: Keep your project files inside the Linux file system (`~/projects`), not in `/mnt/c/...`. It is much faster.

### 5.4 If I have Linux (Ubuntu/Debian)

```bash
# 1) update packages and install basics
sudo apt update && sudo apt install -y git curl unzip

# 2) check git
git --version

# 3) set your identity for commits
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"

# 4) install Docker (simple way for learning)
sudo apt install -y docker.io
sudo usermod -aG docker $USER      # log out and log in again after this
docker --version

# 5) install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### 5.5 macOS (short note)
Use Homebrew: `brew install git awscli` and install Docker Desktop.

### 5.6 AWS setup (step by step)

**Why AWS?** It is the most widely used cloud, and most DevOps jobs ask for it. You can create servers (EC2), networks (VPC), users (IAM), storage (S3) and Kubernetes (EKS) within minutes and pay only for what you use. Practicing here is what makes you job-ready.

**Money check (as of Nov 2026, always re-verify on aws.amazon.com/free):** For accounts created after 15 July 2025, AWS gives new customers $100 in credits at sign-up and up to $100 more by completing activities. You choose a **Free plan** (it ends after 6 months or when credits run out, whichever comes first) or a **Paid plan**. Old tutorials that say "12 months free tier" describe the older model.

**Steps:**
1. **Create an account** at aws.amazon.com (email, card for verification, phone).
2. **Secure the root user:** turn on **MFA** (multi-factor authentication) right away. Do not use root for daily work.
3. **Create an IAM user** (or use IAM Identity Center) for daily work, give only the permissions you need while learning, and turn on MFA for it.
4. **Set a billing alert / budget** (AWS Budgets) for a small amount, for example $5, so you get an email before any surprise.
5. **Install and configure the AWS CLI:**
   ```bash
   aws configure          # asks for Access Key, Secret Key, region, output format
   aws sts get-caller-identity   # shows who you are logged in as
   ```
   Never put access keys in Git. (Gitleaks will catch you. 😄)
6. **Launch your first EC2 server (practice):**
   - Console → EC2 → **Launch instance**
   - Image: **Ubuntu** (a small, free-tier-eligible instance type such as t3.micro or t2.micro, whichever is shown as eligible)
   - Create a **key pair** (.pem file) and keep it safe
   - Security group: allow **SSH (port 22) only from My IP**
   - Launch, then copy the public IP
7. **Connect:**
   ```bash
   chmod 400 mykey.pem
   ssh -i mykey.pem ubuntu@<PUBLIC_IP>
   ```
8. **CLEAN UP after practice:** Stop or **Terminate** the instance (and delete extra volumes or Elastic IPs). A forgotten resource is the number one cause of surprise bills.

### 5.7 Setup checklist

```text
[ ] Git installed and configured
[ ] GitHub account + repo forked
[ ] Linux terminal ready (WSL2 or Ubuntu)
[ ] VS Code installed
[ ] Docker installed and "docker run hello-world" works
[ ] AWS account created, MFA on, budget alert set
[ ] AWS CLI configured (aws sts get-caller-identity works)
```

---

## 6. Pipeline Tools Explained

Quick map of where each tool sits:

| Tool / Idea | Stage | Looks at | Type |
|-------------|-------|----------|------|
| Compiler | Build | Your source code | Build step |
| Gitleaks | Commit / CI | Git history and files | Secret scanning |
| SonarQube | CI | Your source code | SAST (static analysis) |
| Dependency-Check | CI | Libraries you use | SCA |
| Trivy | CI / Registry | Container images, files, IaC | Vulnerability scanner |
| SBOM | CI / Release | List of all components | Inventory |
| HashiCorp Vault | Runtime / CI | Secrets | Secrets management |
| Pentesting | Staging / Prod | Running application | DAST / manual testing |

> **Remember:** SAST = scan code **without running** it. DAST = test the **running** app from outside. SCA = check **third-party libraries**.

---

### 6.1 Compiler

**What is it?** A compiler is a program that translates the code you write (human-readable) into something the computer can run (machine code or intermediate code).

**Easy example:** You write a letter in Hindi, and a translator converts it into English for a foreign office. The computer only understands its own language, and the compiler is the translator.

**With C#:** `dotnet build` uses the compiler (Roslyn) to turn `.cs` files into IL code inside a `.dll`; the .NET runtime then turns it into machine code when the program runs. For Java, `javac` makes `.class` files. For C/C++, `gcc` makes a native binary.

**Compiler vs Interpreter**

| Compiler | Interpreter |
|----------|-------------|
| Translates the whole code first, then runs | Translates and runs line by line |
| Finds many errors before running | Errors appear while running |
| Examples: C, C++, Go, Java (to bytecode), C# | Examples: Python, JavaScript (mostly) |

**Why it matters in DevOps:**
- The **build stage** of CI runs the compiler. If compilation fails, the pipeline stops early. This is the first quality gate.
- Output = an **artifact** (DLL, JAR, binary) that can be packaged and deployed.
- Build tools (Maven, Gradle, MSBuild, `dotnet`, npm) call the compiler and manage dependencies.

**Key features:** syntax and type checking, error and warning messages, optimization, producing deployable artifacts, "treat warnings as errors" option.

```bash
dotnet build        # compile a C# project
javac Hello.java    # compile Java
gcc hello.c -o hello
```

**Interview line:** "A compiler converts source code into executable or intermediate code, and in CI it is the first gate: if the code does not compile, nothing else should run."

---

### 6.2 Gitleaks

**What is it?** An open-source tool that **scans Git repositories (code and history) for secrets**, such as passwords, API keys, tokens and private keys.

**Easy example:** Someone writes the house key's duplicate on the front door. Gitleaks is the guard who checks every note before it is posted.

**Why needed?** Developers sometimes commit `AWS_SECRET_KEY=...` by mistake. Bots scan GitHub all day and can misuse a leaked key within minutes. Even if you delete the file later, the secret stays in **Git history**.

**Key features**
- Scans the current files **and the full commit history**
- Many built-in rules (AWS, GitHub, Slack, private keys, etc.) and **custom rules** through a config file
- Can run as a **pre-commit hook** (stops the commit) and in **CI**
- Fast, simple, reports in JSON, SARIF, CSV
- **Baseline** support to ignore already-known findings

```bash
# using Docker (no install needed): scan the current folder
docker run --rm -v "$(pwd):/path" zricethezav/gitleaks:latest detect --source="/path" -v
# newer versions also offer: gitleaks git  and  gitleaks dir
```

**If a secret leaks:** (1) **revoke/rotate it immediately**, (2) then clean history if needed. Deleting the line alone is not enough.

---

### 6.3 SonarQube

**What is it?** A platform for **continuous code quality and security inspection**. It performs **static analysis (SAST)** on source code across many languages (including C#, Java, Python, JavaScript).

**Easy example:** A teacher who checks your answer sheet for spelling mistakes, messy handwriting and wrong answers **before** the exam is submitted.

**What it finds**
- **Bugs:** code that is likely to fail
- **Vulnerabilities:** security weaknesses (SQL injection patterns, hard-coded passwords)
- **Code smells:** hard-to-maintain code (duplicate code, very long methods)
- **Security hotspots:** places that need a human review
- **Coverage and duplication** numbers

**Key features**
- **Quality Gate:** pass/fail rules (for example "no new critical bugs, coverage above 80%"). If the gate fails, the pipeline fails.
- **"Clean as You Code":** focus on **new code** so teams are not overwhelmed by old problems
- Dashboard with trends, **pull request decoration**, CI integration (GitHub Actions, Jenkins, Azure DevOps)
- Free **Community** edition for learning

```bash
# run SonarQube locally for practice (check Docker Hub for the current community tag)
docker run -d --name sonarqube -p 9000:9000 sonarqube:community
# open http://localhost:9000  (first login: admin / admin, then change the password)
```

**Interview line:** "SonarQube is a SAST tool that scans source code for bugs, vulnerabilities and code smells, and its Quality Gate can block bad code from moving ahead in the pipeline."

---

### 6.4 Dependency-Check (OWASP Dependency-Check)

**What is it?** An open-source **Software Composition Analysis (SCA)** tool. It checks the **third-party libraries** your project uses and tells you if any has a **known vulnerability (CVE)**.

**Easy example:** You build a house with bricks bought from a shop. Dependency-Check tells you, "Brick batch #45 was recalled because it cracks."

**Why needed?** Modern apps are **mostly other people's code** (NuGet, Maven, npm, pip packages). You can write perfect code and still be hacked through a vulnerable library.

**Key features**
- Detects libraries and matches them against public vulnerability databases (like the NVD)
- Supports Java, .NET, JavaScript, Python and more
- Reports in HTML, JSON, XML, SARIF and others
- Can **fail the build** above a chosen severity (CVSS score)
- Plugins for Maven, Gradle, Jenkins and the CLI

```bash
dependency-check.sh --project "demo" --scan . --format HTML
# The first run downloads vulnerability data and can take a while.
```

**SAST vs SCA:** SonarQube checks **your** code. Dependency-Check checks **their** code (libraries).

---

### 6.5 HashiCorp Vault

**What is it?** A **secrets management** tool. It stores, protects and controls access to **secrets** (passwords, API keys, certificates, database credentials) and encrypts data.

**Easy example:** A bank locker room. You do not keep cash on the table. You go to the bank, show your ID, and get only what you are allowed to take, and the bank keeps a log of everything.

**Why needed?** Putting secrets in code, config files or environment variables is risky. Vault gives **one secure, audited place**.

**Key features**
- **Secure secret storage** (encrypted)
- **Authentication and policies:** who can read which secret
- **Dynamic secrets:** creates short-lived database/cloud credentials on demand, which auto-expire
- **Leases and rotation:** secrets can expire and be renewed
- **Encryption as a service** (Transit engine)
- **Audit logs:** records who accessed what
- Works with Kubernetes, AWS, CI tools

```bash
# practice only: dev mode (in-memory, NOT for production)
vault server -dev
export VAULT_ADDR='http://127.0.0.1:8200'
vault kv put -mount=secret demo password=abc123
vault kv get -mount=secret demo
```

**Gitleaks vs Vault:** Gitleaks **finds** leaked secrets. Vault **prevents** leaks by storing secrets safely.

---

### 6.6 SBOM (Software Bill of Materials)

**What is it?** A **complete list of all components, libraries and versions** that make up a piece of software.

**Easy example:** The **ingredients list** on a biscuit packet. If someone says "ingredient X is harmful", you can quickly check whether your product contains it.

**Why needed?** When a big vulnerability appears (for example **Log4Shell** in the Log4j library, December 2021), companies must answer, "Do we use it? Where?" Without an SBOM this takes days. With an SBOM it takes minutes. Customers and regulators also increasingly ask for SBOMs.

**Key features**
- Lists components, versions, licenses and relationships (dependencies)
- Standard formats: **SPDX** and **CycloneDX**
- Can be generated automatically in CI for code or container images
- Helps with **vulnerability tracking, license compliance and supply-chain security**

```bash
# generate an SBOM for an image with Syft
syft nginx:latest -o cyclonedx-json > sbom.json
# or with Trivy
trivy image --format cyclonedx --output sbom.json nginx:latest
```

---

### 6.7 Trivy

**What is it?** An open-source, all-in-one **security scanner** (from Aqua Security). It scans **container images, file systems, Git repos, Kubernetes, and Infrastructure as Code** (Terraform, Dockerfile) for problems.

**Easy example:** The airport security scanner. Every bag (image) passes through it before boarding (deployment).

**What it finds**
- **Vulnerabilities (CVEs)** in OS packages and app libraries
- **Misconfigurations** (insecure Dockerfile, Kubernetes YAML, Terraform)
- **Secrets** accidentally baked into files
- **License issues**
- Can also **generate an SBOM**

**Key features:** fast and easy (a single binary), works offline-friendly with a local DB, severity filters, JSON/table/SARIF output, `--exit-code` to fail the pipeline, great CI integration.

```bash
# scan an image (no install needed, using Docker)
docker run --rm aquasec/trivy:latest image nginx:latest

# only show serious problems and fail the build if found (CI style)
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:1.0

# scan a project folder (files + IaC)
trivy fs .
```

---

### 6.8 Pentesting (Penetration Testing)

**What is it?** An **authorized, simulated cyber-attack** on your own system, done by security experts (ethical hackers) to find weaknesses **before real attackers do**.

**Easy example:** You hire a "friendly thief" to try to break into your house and then tell you which window was open.

> ⚠️ **Only test systems you own or have written permission for.** Hacking without permission is illegal. For practice, use legal labs: OWASP Juice Shop, DVWA, TryHackMe, Hack The Box.

**Typical phases:** Planning and scope, Reconnaissance (information gathering), Scanning, Exploitation, Post-exploitation, Reporting and fixing, Re-test.

**Types:** **Black-box** (no inside knowledge), **White-box** (full knowledge, code access), **Grey-box** (partial).

**Common tools:** Nmap (port scan), Burp Suite and OWASP ZAP (web app testing), Metasploit (exploitation framework).

**Key points**
- Finds **real, exploitable** issues that scanners miss (logic flaws, chained attacks)
- Gives a report with risk level and fix advice
- Is done **periodically** and before big releases, and complements automated tools

**Vulnerability scan vs Pentest**

| Vulnerability scan | Penetration test |
|--------------------|------------------|
| Automated, fast, broad | Manual + automated, deep |
| Lists possible weaknesses | Actually tries to exploit them |
| Run on every build | Run periodically / before release |
| Example: Trivy | Example: Burp Suite, Metasploit |

**In a pipeline:** automated **DAST** (for example OWASP ZAP) can run against the **staging** app after deployment.

---

## 7. What I Will Learn and Benefits

### 7.1 What I will learn in this 90-day journey
Linux, Networking, Shell scripting, Git and GitHub, Docker, CI/CD (GitHub Actions), Kubernetes, Terraform, Ansible, Observability, Helm, AWS (EKS), GitOps (ArgoCD), and DevSecOps basics (today's security tools).

### 7.2 Skills I get
- Automating build, test and deploy
- Running and debugging apps on Linux and cloud
- Packaging apps with containers and running them on Kubernetes
- Writing infrastructure as code
- Reading logs and fixing incidents
- Adding security checks inside pipelines

### 7.3 Why it is worth it (career and practical benefits)
- DevOps and cloud skills are in demand across companies of all sizes
- Strong fit for freshers who have developer basics (C#, SQL, Web API) and want to understand the **full journey of code to production**
- Real projects and a GitHub history prove your skills better than only certificates
- Opens roles such as DevOps Engineer, Cloud Engineer, SRE, Platform Engineer and DevSecOps Engineer

---

## 8. Interview Questions and Answers

**Q1. What is DevOps?**
A culture and set of practices that unite development and operations, using automation to deliver software faster, more safely and more reliably.

**Q2. Why do we need DevOps?**
To release faster, reduce manual errors, find bugs early, improve teamwork between Dev and Ops, and keep systems reliable.

**Q3. What is the DevOps lifecycle?**
Plan, Code, Build, Test, Release, Deploy, Operate, Monitor, in a continuous loop where monitoring feedback goes back to planning.

**Q4. Difference between Developer and DevOps Engineer?**
A developer builds features. A DevOps engineer builds and maintains the pipelines, infrastructure, automation and monitoring that deliver and run those features.

**Q5. What is CI/CD?**
CI automatically builds and tests every code change. CD automatically delivers (or deploys) the tested change to environments, up to production.

**Q6. Continuous Delivery vs Continuous Deployment?**
Delivery needs a manual approval before production. Deployment goes to production automatically if all checks pass.

**Q7. What is DevSecOps and "shift left"?**
Adding security checks inside every stage of the pipeline, and doing them as early as possible so issues are cheaper to fix.

**Q8. Why is Linux important for DevOps?**
Most servers, containers and cloud machines run Linux, and most DevOps tools are built for it.

**Q9. What is a compiler?**
A program that translates source code into executable or intermediate code. In CI it is the first quality gate.

**Q10. What does Gitleaks do?**
It scans code and Git history for exposed secrets such as keys and passwords.

**Q11. What is SonarQube? What is a Quality Gate?**
A SAST platform that finds bugs, vulnerabilities and code smells. A Quality Gate is a set of pass/fail conditions that decide if code can move forward.

**Q12. SAST vs DAST vs SCA?**
SAST scans source code without running it. DAST tests the running application. SCA checks third-party libraries for known vulnerabilities.

**Q13. What does OWASP Dependency-Check do?**
It identifies libraries used by a project and reports known CVEs in them.

**Q14. Why use HashiCorp Vault?**
To store and control access to secrets securely, issue short-lived dynamic credentials, and keep audit logs, instead of hard-coding secrets.

**Q15. What is an SBOM and why is it useful?**
A list of all components and versions in software. It helps find affected systems quickly when a new vulnerability appears and supports license and supply-chain checks.

**Q16. What is Trivy?**
An open-source scanner for vulnerabilities, misconfigurations and secrets in container images, file systems, Git repos and IaC. It can also generate SBOMs.

**Q17. Vulnerability scanning vs Penetration testing?**
Scanning is automated and lists possible weaknesses. Pentesting is a deeper, often manual, authorized attack simulation that proves what is actually exploitable.

**Q18. What if a secret is pushed to GitHub?**
Revoke or rotate it immediately, then remove it from history if required, and add Gitleaks (pre-commit and CI) to prevent a repeat.

**Q19. Why should we not use the AWS root account for daily work?**
Root has unlimited power. Use an IAM user or role with least privilege, and keep MFA on for root.

**Q20. Explain a pipeline in 30 seconds.**
"A developer pushes code. The pipeline scans for secrets, compiles, tests, runs SonarQube and dependency checks, builds an image, scans it with Trivy, generates an SBOM, pushes it to a registry, deploys to staging, tests again, and then deploys to production with monitoring, while secrets come from Vault."

---

## 9. Quick Revision Cheat Sheet

| Term | One-line memory |
|------|-----------------|
| DevOps | Dev + Ops working together with automation |
| CI | Auto build + test on every push |
| CD | Auto deliver/deploy after CI passes |
| Compiler | Translator from code to runnable form |
| Gitleaks | Finds leaked secrets in Git |
| SonarQube | Code quality + security (SAST) |
| Dependency-Check | Finds vulnerable libraries (SCA) |
| Vault | Safe place for secrets |
| SBOM | Ingredients list of software |
| Trivy | Scans images, files and IaC |
| Pentest | Authorized "friendly hacking" |
| SAST / DAST / SCA | Code / running app / libraries |
| Shift left | Find issues early |

**Memory trick for the pipeline order:** **G**ive **C**ode **T**o **S**mart **D**evs **T**o **S**ecure **P**roduction
(Gitleaks → Compile → Test → SonarQube → Dependency-Check → Trivy → SBOM → Pentest/DAST)

---

## 10. Day 02 Hands-on Tasks and Checklist

### Tasks
1. Complete the **setup** (WSL2 or Linux, Git, Docker, VS Code, AWS account with MFA and budget alert).
2. Run: `docker run hello-world` and `aws sts get-caller-identity`.
3. **Trivy:** scan a public image: `docker run --rm aquasec/trivy:latest image nginx:latest`. Note how many HIGH/CRITICAL issues it shows.
4. **Gitleaks:** create a test repo, add a **fake** key (for example `AWS_KEY=AKIAFAKEFAKEFAKE1234`), run Gitleaks, and see it get caught. (Never use real secrets.)
5. (Optional) Start SonarQube with Docker and explore the dashboard.
6. (Optional) Run `vault server -dev` and store and read one secret.
7. Draw the pipeline diagram **by hand** from memory and compare it with Section 3.
8. Write **3 lines** in your own words: "What is DevOps?"

### Checklist
```text
[ ] I can explain Dev + Ops with a real-life example
[ ] I can draw the DevOps lifecycle and a CI/CD pipeline
[ ] I know the difference between a developer and a DevOps engineer
[ ] My machine setup works (Linux/WSL2, Git, Docker, AWS CLI)
[ ] I can explain what each tool does in one line:
    Compiler, Gitleaks, SonarQube, Dependency-Check, Vault, SBOM, Trivy, Pentest
[ ] I answered at least 10 interview questions aloud
[ ] I committed today's notes to GitHub
[ ] I posted my Day 02 update on LinkedIn
```

### Common mistakes to avoid
- Treating DevOps as only a tool list. It is mainly culture + automation.
- Using the AWS **root** account or committing **access keys** to Git.
- Forgetting to **stop/terminate** AWS resources after practice.
- Copy-pasting commands without understanding. Type them and break/fix things on purpose.
- Skipping notes. Write 3 lines every day, because they become your interview preparation.



