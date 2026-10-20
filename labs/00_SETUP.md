# Lab 0 — Participant Setup

Before you start Lab 1, set up **one Jenkins pipeline** that you'll reuse across all five labs. Each lab adds a new stage to `Jenkinsfile` on your branch; you push, click **Build Now**, and watch Bob do its thing.

## Table of Contents

- [Prerequisites](#prerequisites)
- [What your instructor should have given you](#what-your-instructor-should-have-given-you)
- [Step 1 — Create a GitHub PAT](#step-1--create-a-github-pat)
- [Step 2 — Fork the repository and create your branch](#step-2--fork-the-repository-and-create-your-branch)
  - [2a. Fork the repository](#2a-fork-the-repository)
  - [2b. Clone your fork](#2b-clone-your-fork)
  - [2c. Verify your clone](#2c-verify-your-clone)
  - [2d. Open in Bob IDE](#2d-open-in-bob-ide)
  - [2e. Configure Git (if needed)](#2e-configure-git-if-needed)
  - [2f. Cache your GitHub credentials](#2f-cache-your-github-credentials)
  - [2g. Create your working branch](#2g-create-your-working-branch)
- [Step 3 — Create your Jenkins pipeline](#step-3--create-your-jenkins-pipeline)
- [Step 4 — Run it once to verify](#step-4--run-it-once-to-verify)
- [You're set — the workflow from here](#youre-set--the-workflow-from-here)
- [Stuck?](#stuck)

---

## Prerequisites

Before starting, complete the [Prerequisites](../PREREQUISITES.md) checklist.

**Required:**
- Git installed and configured
- Bob IDE installed
- GitHub account (or access to your organization's Git platform)
- Repository URL (provided by your instructor)

---

## What your instructor should have given you

- [ ] **Jenkins URL** (e.g., `https://jenkins-jenkins.<cluster>.techzone.ibm.com`)
- [ ] **Jenkins username** — something like `user1`
- [ ] **Jenkins password** — something like `user1Workshop2024!` (the instructor picked a base password and your password is `<username><base>`)
- [ ] **Git repo URL** — the public repo where this workshop's code lives (you'll fork it)

---

## Step 1 — Create a GitHub PAT

You need a GitHub Personal Access Token (PAT) to authenticate when you `git push` your changes to your fork. You don't need it to clone (the repo is public), and you don't need to add it to Jenkins (Jenkins reads your public fork without credentials).

1. Go to your GitHub tokens page — `https://github.com/settings/tokens` for github.com, or `https://github.ibm.com/settings/tokens` for GitHub Enterprise (your instructor will tell you which)
2. Click **Generate new token → Generate new token (classic)**
3. Set:
   - **Note:** `bob-a-thon workshop`
   - **Expiration:** 7 or 30 days is fine
   - **Select scopes:** check only **`repo`**
4. Click **Generate token** at the bottom
5. **Copy the token now** — GitHub won't show it to you again. Save it somewhere secure (a password manager is recommended).

When Git prompts for credentials on `git push`, use your GitHub username and paste the PAT as the password (not your actual GitHub password). Don't paste the PAT into Bob prompts or save it in files in the repo. When the workshop is over, revoke it at the same tokens page.

---

## Step 2 — Fork the repository and create your branch

### 2a. Fork the repository

Your instructor will provide the repository URL for this workshop.

**GitHub / GitHub Enterprise:**

1. Navigate to the repository URL provided by your instructor
2. Click the **Fork** button in the top-right corner
3. Select your account or organization as the destination
4. Wait for the fork to complete

**GitLab / other Git platforms:** follow your platform's forking process, or ask your instructor for help.

### 2b. Clone your fork

Clone your fork to your local machine.

Your fork is public, so no credentials are needed to clone:

```bash
# Replace <YOUR_USERNAME> with your actual GitHub username
git clone https://github.com/<YOUR_USERNAME>/us-fsm-ce-bob-a-thon-assets.git
cd us-fsm-ce-bob-a-thon-assets
```

### 2c. Verify your clone

```bash
# Check you're in the right directory
pwd
# Should show: /path/to/us-fsm-ce-bob-a-thon-assets

# Verify the remote
git remote -v
# Should show your fork as 'origin'

# Check the repository structure
ls -la
# Should see: labs/, order-service/, Jenkinsfile, README.md, etc.
```

### 2d. Open in Bob IDE

1. Launch Bob IDE
2. Open the cloned repository: **File → Open Folder** (or **File → Open** on macOS), select the `us-fsm-ce-bob-a-thon-assets` directory, and click **Open**
3. Verify Bob can see the repository files in the Explorer sidebar

### 2e. Configure Git (if needed)

If this is your first time using Git on this machine, configure your identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### 2f. Cache your GitHub credentials

Each `git push` in the labs asks for your GitHub username and your PAT (as the password). To avoid re-entering the PAT on every push, run this once **before** your first push:

```bash
git config --global credential.helper 'cache --timeout=10800'
```

Git will now remember your credentials for 3 hours. You enter the PAT on your first push, and later pushes won't ask again.

> **Note:** The cache is held in memory only. Nothing is written to disk. It's lost on reboot or after 3 hours, and you'll just be prompted for the PAT again.

**Prefer not to cache?** Skip the command above. Git will then ask for your username and PAT on **every** `git push` (about 7–8 times across the labs), and you paste the PAT as the password each time.

### 2g. Create your working branch

```bash
# Create your branch and push it
git checkout -b user1-labs
git push -u origin user1-labs
```

Git prompts for your username and password. Enter your GitHub username, and paste your PAT as the password.

Replace `user1-labs` with your actual username (e.g., `user7-labs`). You'll push to this branch after every lab exercise.

---

## Step 3 — Create your Jenkins pipeline

**Create the pipeline inside your `userN` folder, not at the Jenkins root.** If you create it at the root, you won't have permission to save it.

From the Jenkins homepage:

1. Click your `userN` folder.
2. Inside the folder, click **New Item** in the left sidebar.
3. Enter a name — something like `user1-pipeline`.
4. Select **Pipeline** from the type list, then click **OK**.
5. You land on the config page. Scroll to the **Pipeline** section at the bottom and fill in:

   | Field | Value |
   |---|---|
   | **Definition** | `Pipeline script from SCM` |
   | **SCM** | `Git` |
   | **Repository URL** | paste the GitHub repo URL from your fork |
   | **Credentials** | leave as `- none -` (your fork is public) |
   | **Branch Specifier** | `*/user1-labs` — replace with **your** branch name |
   | **Script Path** | `Jenkinsfile` |

6. Click **Save**. You land on your new pipeline's page.

---

## Step 4 — Run it once to verify

Click **Build Now** in the left sidebar. A build appears under "Build History" — click its number, then **Console Output**.

**Expected:** the `Checkout` stage completes and the console prints your workspace contents (`.bob/`, `Jenkinsfile`, `order-service/`, etc.). The pipeline finishes `SUCCESS` with no other stages yet — that's correct; you'll add stages lab by lab.

If something fails, see the [stuck?](#stuck) section at the bottom.

---

## You're set — the workflow from here

The same pipeline runs through all 5 labs. Your loop for each lab:

1. Edit `Jenkinsfile` on your branch to add the new stage the lab describes.
2. If the lab calls for it, edit `.bob/custom_modes.yaml` to add a Bob custom mode.
3. Commit and push:
   ```bash
   git add .
   git commit -m "Lab N changes"
   git push
   ```
4. In Jenkins, click **Build Now** on your pipeline.
5. Watch the new stage run.

Ready? Open [LAB1_PR_REVIEW.md](lab1/LAB1_PR_REVIEW.md).

---

## Stuck?

- **Build fails with `ImagePullBackOff`**: the Jenkinsfile's Bob image URL doesn't match your cluster. Ask your instructor to confirm the Bob image is pushed to the `jenkins` namespace.
- **"Delete Folder" button is missing from my folder**: that's intentional. Matrix auth doesn't grant delete rights on the folder itself so you can't accidentally wipe your workshop work.
- **`git push` fails with auth**: the PAT may have expired or lack the `repo` scope. Create a new one (Step 1) and enter it when Git prompts. If the old one is cached, clear it with `git credential-cache exit`.
- **"Repository not found"** when cloning or building: double-check the repository URL, and that your fork exists and is public.
- **Git asks for username/password repeatedly**: you haven't enabled credential caching, or the 3-hour cache expired. See [2f](#2f-cache-your-github-credentials).
- **Can't see files in Bob IDE**: make sure you opened the repository root directory (`us-fsm-ce-bob-a-thon-assets`), not a subdirectory.

---

## Additional Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Forking Guide](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
- [Bob IDE Documentation](https://bob.ibm.com/docs/ide)
