# Lab 08: Creating a "Hello World" Jenkins Job

## Objective

Created a Jenkins Freestyle project that pulls a shell script from a GitHub repository and executes it. Demonstrated Jenkins job configuration, source code management (SCM) integration, and build execution.

---

## Scenario

You are a DevOps engineer setting up your first Jenkins job. The goal is to:
1. Create a simple shell script (`hello-world.sh`) and push it to GitHub.
2. Configure a Jenkins Freestyle project to pull the script and execute it.
3. Verify the build output in Jenkins console.

---

## Prerequisites

- **Jenkins** installed and running (via Docker or local install)
- **GitHub** account with a public repository
- **Git** configured locally
- **Personal Access Token (PAT)** for GitHub authentication

---

## Directory Structure

```
08-HelloWorld-Jenkins/
├── hello-world.sh     # Simple shell script to run in Jenkins
├── screenshot/        # Deployment and verification screenshots
└── README.md
```

---

## Step-by-Step Execution

### Step 1: Create a GitHub Repository

1. Log in to your [GitHub account](https://www.github.com).
2. Click the **+** icon (top-right) → **New Repository**.
3. Fill in the repository details:
   - **Repository Name:** `devops-sample-code`
   - **Description:** A demo repository for Jenkins scripting.
   - **Visibility:** Public
4. Click **Create Repository**.

---

### Step 2: Create a Fine-Grained Personal Access Token

Follow GitHub's official guide:
[Creating a fine-grained personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)

---

### Step 3: Create the Script Locally

1. Create the shell script:
   ```bash
   touch hello-world.sh
   ```

2. Add the following content to `hello-world.sh`:
   ```bash
   #!/bin/bash
   echo "Hello, Jenkins!"
   ```

3. Make the script executable:
   ```bash
   chmod +x hello-world.sh
   ```

---

### Step 4: Initialize a Local Git Repository

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git init
```

---

### Step 5: Add and Commit the Script

1. Stage the file:
   ```bash
   git add hello-world.sh
   ```

2. Verify status:
   ```bash
   git status
   ```
   Expected output:
   ```
   On branch master

   No commits yet

   Changes to be committed:
     (use "git rm --cached <file>..." to unstage)
           new file:   hello-world.sh
   ```

3. Commit:
   ```bash
   git commit -m "Add hello-world.sh"
   ```
   Expected output:
   ```
   [master (root-commit) 2e1b718] Add hello-world.sh
    1 file changed, 2 insertions(+)
    create mode 100644 hello-world.sh
   ```

---

### Step 6: Link and Push to GitHub

1. Add the remote:
   ```bash
   git remote add origin https://github.com/<your-GitHub-username>/devops-sample-code.git
   ```

2. Push to GitHub:
   ```bash
   git push -u origin main
   ```
   It will prompt for your GitHub username and Personal Access Token (PAT).

---

### Step 7: Verify on GitHub

Navigate to `https://github.com/<your-GitHub-username>/devops-sample-code` and confirm `hello-world.sh` is present.

---

## Jenkins Job Configuration

### Step 1: Access Jenkins

1. Open your browser and navigate to `http://localhost:8080`.
2. Log in using your credentials.

---

### Step 2: Create a New Job

1. On the Jenkins dashboard, click **"New Item"**.
2. Enter a name: `HelloWorld`.
3. Select **Freestyle project**.
4. Click **OK**.

---

### Step 3: Configure the Job

#### General Settings
- **Description:**
  ```
  Hello World! Jenkins job.
  ```

#### Source Code Management
- Select **Git**.
- Enter the repository URL:
  ```
  https://github.com/<your-username>/devops-sample-code.git
  ```

#### Build Steps
- Under **Build**, click **Add build step** → **Execute shell**.
- Add the following command:
  ```bash
  sh hello-world.sh
  ```

---

### Step 4: Save and Run the Job

1. Click **Save**.
2. On the job page, click **Build Now**.

---

### Step 5: View the Build Output

1. Navigate to the **Build History** section on the left-hand side.
2. Click the build number (e.g., `#1`).
3. Click **Console Output** to see the build logs.

**Expected Console Output:**
```
Started by user Admin
Building in workspace /var/jenkins_home/workspace/HelloWorld
[HelloWorld] $ /bin/sh -xe /tmp/jenkins1281930102.sh
+ echo 'Hello, Jenkins!'
Hello, Jenkins!
Finished: SUCCESS
```

---

## Expected Outputs

1. **GitHub Repository**: `hello-world.sh` file visible in the repository.
2. **Jenkins Build**: Successful build with "Hello, Jenkins!" in the console output.

---

## Deployment Evidence

![Jenkins Job Configuration](./screenshot/jenkins-config.png)

![Jenkins Build Console Output](./screenshot/build-output.png)

---

## Files

| File | Description |
|------|-------------|
| `hello-world.sh` | Shell script that prints "Hello, Jenkins!" |
