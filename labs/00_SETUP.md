# Lab 0 — Participant Setup

Before you start Lab 1, set up **one Jenkins pipeline** that you'll reuse across all five labs. Each lab adds a new stage to `Jenkinsfile` on your branch; you push, click **Build Now**, and watch Bob do its thing.

## Table of Contents

- [Prerequisites](#prerequisites)
- [What your instructor should have given you](#what-your-instructor-should-have-given-you)
- [Step 1 — Add a GitHub PAT (private repo only)](#step-1--add-a-github-pat-private-repo-only)
  - [1a. Create the PAT on GitHub](#1a-create-the-pat-on-github)
  - [1b. Add the PAT to your folder's credential store](#1b-add-the-pat-to-your-folders-credential-store)
- [Step 2 — Fork the repository and create your branch](#step-2--fork-the-repository-and-create-your-branch)
  - [2a. Fork the repository](#2a-fork-the-repository)
  - [2b. Clone your fork](#2b-clone-your-fork)
  - [2c. Verify your clone](#2c-verify-your-clone)
  - [2d. Open in Bob IDE](#2d-open-in-bob-ide)
  - [2e. Configure Git (if needed)](#2e-configure-git-if-needed)
  - [2f. Create your working branch](#2f-create-your-working-branch)
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
- [ ] **Git repo URL** — where this workshop's code lives
- [ ] **Whether the repo is public or private** — if private, you'll need to do [Step 1](#step-1--add-a-github-pat-private-repo-only) below. If public, skip it.

---

## Step 1 — Add a GitHub PAT (private repo only)

Skip this step if your instructor told you the repo is public.

You need a GitHub Personal Access Token (PAT) to authenticate when cloning and pushing to a private repo. You also need to add it to Jenkins **before** you configure your pipeline — the pipeline's Credentials dropdown won't show the PAT unless it's saved in Jenkins first, and it has to be saved in the right place.

### 1a. Create the PAT on GitHub

1. Go to your GitHub tokens page — `https://github.com/settings/tokens` for github.com, or `https://github.ibm.com/settings/tokens` for GitHub Enterprise (your instructor will tell you which)
2. Click **Generate new token → Generate new token (classic)**
3. Set:
   - **Note:** `bob-a-thon workshop`
   - **Expiration:** 7 or 30 days is fine
   - **Select scopes:** check only **`repo`**
4. Click **Generate token** at the bottom
5. **Copy the token now** — GitHub won't show it to you again. Save it somewhere secure (a password manager is recommended).

When Git prompts for credentials on a private repo, use your GitHub username and paste the PAT as the password (not your actual GitHub password). You can also embed it in the clone URL — see [2b](#2b-clone-your-fork).

### 1b. Add the PAT to your folder's credential store

**This step is finicky** — there are three places in Jenkins that look like a "Credentials" page, and only one of them makes your PAT visible to your pipeline jobs. Follow this navigation exactly.

Logged into Jenkins as `userN`:

1. From the **Jenkins homepage**, click your `userN` folder (the folder-icon item in the list). **Do not** click your username in the top-right — that goes to a different (wrong) credential store.
2. In the folder's **left sidebar**, click **Credentials**.
3. On the folder's Credentials page, under **Stores scoped to userN**, click **(global)**.
4. Click **Add Credentials**
5. Fill in:
   - **Kind:** `Username with password`
   - **Scope:** `Global` (see note below about what this means)
   - **Username:** your GitHub username
   - **Password:** paste the PAT from step 1a
   - **ID:** `userN-github-pat` — replace `userN` with your actual username, e.g. `user1-github-pat`. The exact ID matters; the pipeline references it by this name.
   - **Description:** anything
6. Click **Create**.

You should land back on the folder's Credentials page, with one row now showing your new `userN-github-pat` credential.

> **Note on "Scope: Global".** The word *Global* in this dialog is misleading — it refers to which **URLs** the credential applies to (it's a URL-matching setting), **not** who can see the credential. Visibility is locked by the store you're in (your folder), and other workshop users can't see it. You can safely pick `Global` as the scope.

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

**Public repository:**

```bash
# Replace <YOUR_USERNAME> with your actual GitHub username
git clone https://github.com/<YOUR_USERNAME>/us-fsm-ce-bob-a-thon-assets.git
cd us-fsm-ce-bob-a-thon-assets
```

**Private repository — Option A: embed the PAT in the URL**

```bash
# Replace <YOUR_PAT> and <YOUR_USERNAME> with your actual values
git clone https://<YOUR_PAT>@github.com/<YOUR_USERNAME>/us-fsm-ce-bob-a-thon-assets.git
cd us-fsm-ce-bob-a-thon-assets
```

**Private repository — Option B: enter the PAT when prompted**

```bash
git clone https://github.com/<YOUR_USERNAME>/us-fsm-ce-bob-a-thon-assets.git

# Git will prompt for credentials:
# Username: <your-github-username>
# Password: <paste-your-PAT-here>

cd us-fsm-ce-bob-a-thon-assets
```

**Private repository — Option C: SSH (if you have SSH keys configured)**

```bash
git clone git@github.com:<YOUR_USERNAME>/us-fsm-ce-bob-a-thon-assets.git
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

### 2f. Create your working branch

```bash
# Create your branch and push it
git checkout -b user1-labs
git push -u origin user1-labs
```

Replace `user1-labs` with your actual username (e.g., `user7-labs`). You'll push to this branch after every lab exercise.

---

## Step 3 — Create your Jenkins pipeline

**Create the pipeline inside your `userN` folder, not at the Jenkins root.** If you create it at the root, you won't have permission to save it and the PAT from Step 1 won't be reachable.

From the Jenkins homepage:

1. Click your `userN` folder (the same one you added the PAT to).
2. Inside the folder, click **New Item** in the left sidebar.
3. Enter a name — something like `user1-pipeline`.
4. Select **Pipeline** from the type list, then click **OK**.
5. You land on the config page. Scroll to the **Pipeline** section at the bottom and fill in:

   | Field | Value |
   |---|---|
   | **Definition** | `Pipeline script from SCM` |
   | **SCM** | `Git` |
   | **Repository URL** | paste the GitHub repo URL from your fork |
   | **Credentials** | for a public repo: leave as `- none -`. For a private repo: select the `userN-github-pat` credential you created in Step 1 |
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
- **"Credentials dropdown is empty" when configuring the pipeline**: your PAT went into the wrong store. Only **folder-scoped** credentials appear in the pipeline's Credentials dropdown — user-personal credentials (the one you get from the top-right username menu) don't. Re-check Step 1b: navigate from the homepage → your folder → **Credentials** in the folder sidebar (not the one under your top-right username).
- **"Credentials link is missing from the folder sidebar"**: your instructor needs to run the matrix-auth Groovy that grants `CredentialsProvider.VIEW` to each user. Ping them.
- **"Delete Folder" button is missing from my folder**: that's intentional. Matrix auth doesn't grant delete rights on the folder itself so you can't accidentally wipe your workshop work.
- **`git push` fails with auth**: regenerate the PAT (it may have expired) and re-add it in Step 1.
- **"Permission denied" when cloning**: verify your Git credentials or SSH keys are configured correctly. Contact your instructor for repository access.
- **"Repository not found"**: double-check the repository URL provided by your instructor and make sure you have access to it.
- **Git asks for username/password repeatedly**: use SSH keys or cache your credentials, e.g. `git config --global credential.helper 'cache --timeout=3600'`.
- **Can't see files in Bob IDE**: make sure you opened the repository root directory (`us-fsm-ce-bob-a-thon-assets`), not a subdirectory.

---

## Additional Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Forking Guide](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
- [Bob IDE Documentation](https://bob.ibm.com/docs/ide)
