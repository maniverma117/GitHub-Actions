# **GitHub Actions for Deploying to Different Branches via SSH**  

This repository is configured with GitHub Actions to automatically deploy updates to different environments (`dev`, `staging`, and `main`) when code is pushed to the respective branches.  

Each branch triggers a separate workflow that:  
- Authenticates via **SSH** to the target server.  
- Runs a **bash script** to deploy the latest changes.  

---

## **🚀 How It Works**  
Each branch has a dedicated **GitHub Actions workflow** that connects to a remote server via SSH and executes a deployment script.  

### **Branch-to-Server Mapping**  
| Branch  | Workflow File       | Target Server  | Script Executed  |
|---------|--------------------|---------------|------------------|
| `dev`   | `.github/workflows/dev.yml`   | Dev Server      | `/path/to/your-script.sh`  |
| `staging` | `.github/workflows/staging.yml` | Staging Server  | `/path/to/your-script.sh`  |
| `main`  | `.github/workflows/main.yml`  | Production Server | `/path/to/your-script.sh`  |

---

## **1️⃣ Setup SSH Access for GitHub Actions**  

### **Step 1: Generate an SSH Key Pair (If Not Already Created)**
Run the following command on your local machine:  

```sh
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```
- Save the key as `~/.ssh/github-actions-key`.  
- This creates:  
  - A **private key** (`github-actions-key`)  
  - A **public key** (`github-actions-key.pub`)  

### **Step 2: Add the Public Key to the Target Server**
Run this command to copy the public key to the remote server:  

```sh
ssh-copy-id -i ~/.ssh/github-actions-key user@server-ip
```
Or manually add the contents of `github-actions-key.pub` to the `/home/user/.ssh/authorized_keys` file on the remote server.  

### **Step 3: Store the Private Key as a GitHub Secret**
1. Go to your GitHub repository.  
2. Navigate to **Settings → Secrets and variables → Actions**.  
3. Click **"New repository secret"**.  
4. Add a new secret:  
   - **Name:** `SSH_PRIVATE_KEY`  
   - **Value:** Paste the contents of `github-actions-key` (private key).  

---

## **2️⃣ Configure GitHub Actions Workflows**  

### **🟢 Dev Deployment (`.github/workflows/dev.yml`)**  

```yaml
name: Deploy to Dev

on:
  push:
    branches:
      - dev

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup SSH Key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H server-ip >> ~/.ssh/known_hosts

      - name: SSH into Dev Server and Run Script
        run: |
          ssh user@server-ip "bash /path/to/your-script.sh"
```

---

### **🟠 Staging Deployment (`.github/workflows/staging.yml`)**  

```yaml
name: Deploy to Staging

on:
  push:
    branches:
      - staging

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup SSH Key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H server-ip >> ~/.ssh/known_hosts

      - name: SSH into Staging Server and Run Script
        run: |
          ssh user@server-ip "bash /path/to/your-script.sh"
```

---

### **🔴 Production Deployment (`.github/workflows/main.yml`)**  

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup SSH Key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H server-ip >> ~/.ssh/known_hosts

      - name: SSH into Production Server and Run Script
        run: |
          ssh user@server-ip "bash /path/to/your-script.sh"
```

---

## **3️⃣ How to Use**
### **1️⃣ Commit and Push Changes**
Whenever a commit is pushed to a specific branch, the corresponding workflow will be triggered.

```sh
git add .
git commit -m "Deploying latest changes"
git push origin dev  # Or staging / main
```

### **2️⃣ Monitor GitHub Actions**
- Go to your repository.  
- Click **"Actions"** in the GitHub UI.  
- Select the workflow for the branch (`dev.yml`, `staging.yml`, `main.yml`).  
- Check logs for errors or success messages.  

---

## **4️⃣ Troubleshooting**
### **1. GitHub Action Fails with "Permission Denied (Public Key)"**
- Ensure the public key is added to the **authorized_keys** file on the remote server:
  ```sh
  cat ~/.ssh/authorized_keys
  ```
- If missing, manually add the public key.

### **2. GitHub Action Fails at SSH Key Setup**
- Ensure `SSH_PRIVATE_KEY` is correctly set in GitHub Secrets.  
- Check permissions on `id_rsa`:  
  ```sh
  chmod 600 ~/.ssh/id_rsa
  ```

### **3. Deployment Script Fails**
- Verify the script path and permissions on the remote server:  
  ```sh
  ls -l /path/to/your-script.sh
  chmod +x /path/to/your-script.sh
  ```

---

## **🎯 Summary**
- GitHub Actions automatically deploys to different servers when pushing to respective branches.
- Uses **SSH authentication** for secure server access.
- Runs a **bash script** on the target machine.
- Separate **workflow YAML files** for `dev`, `staging`, and `main` ensure branch-specific deployments.

🚀 **This setup enables fully automated deployments with minimal manual intervention.** 🎯  

---

Let me know if you need any modifications! 🚀
