# GitHub Actions CI/CD Deployment Guide (VPS)

## Overview

This document explains how to implement automated CI/CD deployment for a Node.js, Express.js, TypeScript, Prisma, and PM2 application using GitHub Actions and a VPS server.

---

# CI/CD Flow

```text
Developer
    ↓
git push origin main
    ↓
GitHub Actions Trigger
    ↓
Connect VPS via SSH
    ↓
Pull Latest Source Code
    ↓
Install Dependencies
    ↓
Generate Prisma Client
    ↓
Push Database Schema
    ↓
Build Application
    ↓
Restart PM2 Process
    ↓
Production Updated
```

---

# Prerequisites

* Ubuntu VPS
* Git Installed
* Node.js Installed
* PM2 Installed
* GitHub Repository
* Project Already Running on VPS
* SSH Access to VPS

---

# Step 1: Generate SSH Key on VPS

```bash
ssh-keygen -t ed25519 -C "github-actions"
```

Save location:

```bash
/root/.ssh/github_actions
```

Generated files:

```text
/root/.ssh/github_actions
/root/.ssh/github_actions.pub
```

---

# Step 2: Add Public Key to Authorized Keys

```bash
cat ~/.ssh/github_actions.pub >> ~/.ssh/authorized_keys
```

Verify:

```bash
grep github-actions ~/.ssh/authorized_keys
```

Expected output:

```text
ssh-ed25519 AAAAxxxxxxxxxxxxxxxx github-actions
```

---

# Step 3: Add GitHub Secrets

Repository → Settings → Secrets and Variables → Actions

Create the following secrets:

## VPS_HOST

```text
YOUR_SERVER_IP
```

Example:

```text
206.162.244.134
```

---

## VPS_USER

```text
root
```

---

## VPS_SSH_KEY

Copy private key:

```bash
cat ~/.ssh/github_actions
```

Paste entire content into:

```text
VPS_SSH_KEY
```

---

# Step 4: Create GitHub Workflow

Create file:

```text
.github/workflows/deploy.yml
```

---

# Generic Deployment Workflow

```yaml
name: Deploy Backend

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy Application
        uses: appleboy/ssh-action@v1.0.3

        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}

          script: |
            cd /var/www/YOUR_PROJECT_NAME

            git pull origin main

            npm install

            npx prisma generate

            npx prisma db push

            npm run build

            pm2 restart YOUR_PM2_NAME

            pm2 save
```

---

# Example Configuration

## Project Path

```text
/var/www/your-backend
```

## PM2 Process Name

```text
your-backend
```

---

# Deployment Commands

Developer only needs:

```bash
git add .
git commit -m "update feature"
git push origin main
```

Everything else is handled automatically.

---

# PM2 Commands

## Check Running Applications

```bash
pm2 list
```

## Restart Application

```bash
pm2 restart APP_NAME
```

## View Logs

```bash
pm2 logs APP_NAME
```

## Save PM2 State

```bash
pm2 save
```

---

# Common Issues

## SSH Authentication Failed

Check:

```bash
cat ~/.ssh/authorized_keys
```

Verify GitHub public key exists.

---

## Prisma Errors

```bash
npx prisma generate
npx prisma db push
```

Run manually on VPS to debug.

---

## PM2 Restart Failed

Verify process name:

```bash
pm2 list
```

Update workflow with the correct PM2 process name.

---

# Recommended Project Structure

```text
project-root
│
├── .github
│   └── workflows
│       └── deploy.yml
│
├── prisma
├── src
├── dist
├── package.json
├── tsconfig.json
└── README.md
```

---

# Benefits

* Automatic Deployment
* No Manual SSH Login
* Faster Release Process
* Consistent Production Deployment
* Easy Team Collaboration
* Reusable Across Multiple Projects

---

# Final Result

```text
Code Push
    ↓
GitHub Actions
    ↓
SSH VPS
    ↓
Deploy
    ↓
Build
    ↓
Restart PM2
    ↓
Production Updated
```
