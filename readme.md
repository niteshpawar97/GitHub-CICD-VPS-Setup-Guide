# 🚀 GitHub CI/CD VPS Deployment Guide

[![GitHub Stars](https://img.shields.io/github/stars/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)
[![GitHub Forks](https://img.shields.io/github/forks/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

> **Complete step-by-step guide for GitHub CI/CD deployment to VPS servers with SSH keys, PM2 process management, and multiple repository automation.**

## 🎯 **What You'll Learn:**
- ✅ GitHub Actions CI/CD pipeline setup
- ✅ VPS server configuration for deployments  
- ✅ SSH deploy keys management
- ✅ Multiple repository deployment automation
- ✅ PM2 process manager integration
- ✅ Security best practices for production

## 🔥 **Popular Keywords Covered:**
`github-actions` `vps-deployment` `ssh-deploy-keys` `cicd-pipeline` `pm2-deployment` `server-automation` `devops-tutorial` `continuous-integration` `deployment-automation`

## 📋 Table of Contents
1. [GitHub Repository Secrets Setup](#1-github-repository-secrets-setup)
2. [VPS SSH Configuration](#2-vps-ssh-configuration)
3. [SSH Key Generation](#3-ssh-key-generation)
4. [Multiple Deploy Keys Setup](#4-multiple-deploy-keys-setup)
5. [PM2 Installation](#5-pm2-installation)
6. [Repository Cloning](#6-repository-cloning)
7. [GitHub Actions Workflow](#7-github-actions-workflow)
8. [Troubleshooting Common Issues](#8-troubleshooting-common-issues)
9. [Advanced Configuration](#9-advanced-configuration)
10. [Performance Optimization](#10-performance-optimization)

---

## 1. GitHub Repository Secrets Setup

Navigate to your GitHub repository:
```
Repository → Settings → Secrets and variables → Actions → Repository secrets
```

**Add the following secrets:**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `SSH_USER` | `your-vps-username` | VPS SSH username |
| `SSH_PASSWORD` | `your-vps-password` | VPS SSH password |
| `SSH_HOST` | `your-vps-ip-address` | VPS server IP address |

---

## 2. VPS SSH Configuration

### Step 1: Connect to VPS
```bash
ssh username@your-server-ip
```

### Step 2: Switch to Root User
```bash
sudo su
```

### Step 3: Enable Password Authentication
Edit SSH configuration file:
```bash
nano /etc/ssh/sshd_config
```

**Update or add these lines:**
```bash
PasswordAuthentication yes
PermitRootLogin yes
```

> **Note:** Remove `#` if lines are commented out.

### Step 4: Restart SSH Service
```bash
systemctl restart ssh
```

### Step 5: Test Password Login
```bash
ssh username@your-server-ip
```

---

## 3. SSH Key Generation

### Navigate to SSH Directory
```bash
cd ~/.ssh
ls
```

### Generate Deploy Key (Named)
For specific deploy key with custom name:
```bash
ssh-keygen -t ed25519 -C "deploy-key-your-project"
# Enter custom filename when prompted
```

### Generate Default Key
For default key name:
```bash
ssh-keygen
# Press Enter for default name and location
```

### View Generated Keys
```bash
ls -l ~/.ssh/
```

### Copy Public Key
```bash
cat ~/.ssh/id_rsa.pub
# or
cat ~/.ssh/id_ed25519.pub
```

**Add this key to GitHub:**
```
Repository → Settings → Deploy Keys → Add Deploy Key
```

---

## 4. Multiple Deploy Keys Setup

When you need to manage **multiple repositories** on the same VPS with different deploy keys.

### Scenario
- **Repository 1**: Already has access with `id_rsa` key
- **Repository 2**: `your-project-backend` needs new `id_ed25519` key

### Step 1: Keep Existing Keys
Your existing keys:
```
/home/niteshpawar97/.ssh/id_rsa
/home/niteshpawar97/.ssh/id_ed25519
```

### Step 2: Create SSH Config File
```bash
nano ~/.ssh/config
```

**Add the following configuration:**
```bash
# Default GitHub account (First repository)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa

# Your Project Backend repository
Host github-project
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519

# Your Project Frontend/Admin Panel
Host github-project-admin
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

**Save and exit:** `Ctrl + O` → `Enter` → `Ctrl + X`

### Step 3: Set Proper Permissions
```bash
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/id_rsa
```

### Step 4: Add Public Key to GitHub
Copy your public key:
```bash
cat ~/.ssh/id_ed25519.pub
```

Add to GitHub Repository:
```
Repository → Settings → Deploy Keys → Add Deploy Key
```

- **Title:** `Your Project VPS Deploy Key`
- **Key:** Paste the copied public key
- **✅ Allow write access** (if needed for CI/CD push)
- Click **Add key**

### Step 5: Clone Repository with Custom Host
**Standard GitHub URL:**
```
git@github.com:username/repo.git
```

**Replace with custom host:**
```
git@github-project:username/repo.git
```

**Example:**
```bash
git clone git@github-project:username/your-project-backend.git
git clone git@github-project-admin:username/your-project-frontend.git
```

### Step 6: Test SSH Connection
```bash
ssh -T git@github-project
ssh -T git@github-project-admin
```

**Expected output:**
```
Hi niteshpawar97! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## 5. PM2 Installation

Navigate to project directory:
```bash
cd ~/htdocs/your-project-backend
```

### Install PM2 Globally (Non-root user)
```bash
sudo npm install -g pm2 --unsafe-perm=true
```

---

## 6. Repository Cloning

### Clone Backend Repository
```bash
git clone git@github-project:username/your-project-backend.git
```

### Clone Frontend Admin Panel
```bash
git clone git@github-project-admin:username/your-project-frontend.git
```

---

---

## 7. GitHub Actions Workflow

### Create Workflow File
Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests
      run: npm test
      
    - name: Build application
      run: npm run build
      
    - name: Deploy to VPS
      uses: appleboy/ssh-action@v0.1.7
      with:
        host: ${{ secrets.SSH_HOST }}
        username: ${{ secrets.SSH_USER }}
        password: ${{ secrets.SSH_PASSWORD }}
        script: |
          cd ~/htdocs/your-project
          git pull origin main
          npm install --production
          npm run build
          pm2 restart your-app-name
          pm2 save
```

### Advanced Workflow with Multiple Environments

```yaml
name: Advanced Deploy Pipeline

on:
  push:
    branches: [ main, develop, staging ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Run Tests
      run: |
        npm ci
        npm run test:coverage
        npm run lint
        
  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to Staging
      uses: appleboy/ssh-action@v0.1.7
      with:
        host: ${{ secrets.STAGING_HOST }}
        username: ${{ secrets.SSH_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd ~/staging/your-project
          git pull origin staging
          npm install
          pm2 restart staging-app
          
  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to Production
      uses: appleboy/ssh-action@v0.1.7
      with:
        host: ${{ secrets.SSH_HOST }}
        username: ${{ secrets.SSH_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd ~/htdocs/your-project
          git pull origin main
          npm install --production
          npm run build
          pm2 restart production-app
          pm2 save
```

---

## 8. Troubleshooting Common Issues

### 🚨 Common Problems & Solutions

#### **1. SSH Connection Failed**
```bash
# Error: Permission denied (publickey)
# Solution: Check SSH key configuration
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com
```

#### **2. PM2 Process Not Starting**
```bash
# Check PM2 status
pm2 status
pm2 logs your-app-name

# Restart PM2
pm2 restart all
pm2 save
```

#### **3. Git Pull Permission Denied**
```bash
# Check SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Test GitHub connection
ssh -T git@github-project
```

#### **4. Node Modules Installation Failed**
```bash
# Clear npm cache
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

#### **5. Port Already in Use**
```bash
# Find process using port
lsof -i :3000
# Kill process
kill -9 PID_NUMBER
```

### 🔧 Debug Commands

```bash
# Check SSH configuration
cat ~/.ssh/config

# Verify SSH keys
ls -la ~/.ssh/

# Test SSH connection with verbose
ssh -Tv git@github-project

# Check PM2 logs
pm2 logs --lines 50

# Monitor system resources
htop
df -h
```

---

## 9. Advanced Configuration

### **Multiple Environment Setup**

#### **Development Environment**
```bash
# .env.development
NODE_ENV=development
PORT=3000
DB_HOST=localhost
```

#### **Staging Environment**
```bash
# .env.staging
NODE_ENV=staging
PORT=3001
DB_HOST=staging-db.example.com
```

#### **Production Environment**
```bash
# .env.production
NODE_ENV=production
PORT=80
DB_HOST=prod-db.example.com
```

### **PM2 Ecosystem File**
Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [
    {
      name: 'your-app-production',
      script: './app.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      error_file: './logs/err.log',
      out_file: './logs/out.log',
      log_file: './logs/combined.log',
      time: true
    },
    {
      name: 'your-app-staging',
      script: './app.js',
      instances: 1,
      env: {
        NODE_ENV: 'staging',
        PORT: 3001
      }
    }
  ]
};
```

### **Nginx Configuration**
```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### **SSL Configuration with Let's Encrypt**
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d your-domain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

---

## 10. Performance Optimization

### **Server Optimization**

#### **1. Enable Gzip Compression**
```nginx
# In nginx.conf
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
```

#### **2. PM2 Monitoring**
```bash
# Install PM2 monitoring
pm2 install pm2-server-monit

# Monitor with keymetrics
pm2 link your-secret-key your-public-key
```

#### **3. Database Optimization**
```bash
# MongoDB optimization
# Add to mongod.conf
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 1
```

### **Application Optimization**

#### **1. Code Splitting**
```javascript
// Lazy loading components
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

#### **2. Caching Strategy**
```javascript
// Redis caching
const redis = require('redis');
const client = redis.createClient();

app.get('/api/data', cache(300), (req, res) => {
  // Cached for 5 minutes
});
```

#### **3. Load Balancing**
```javascript
// PM2 cluster mode
module.exports = {
  apps: [{
    name: 'app',
    script: './app.js',
    instances: 'max',
    exec_mode: 'cluster'
  }]
};
```

## 📊 SSH Configuration Summary

| Repository | SSH Host | Key File | Git Clone Command |
|------------|----------|----------|-------------------|
| Default Repo | `github.com` | `~/.ssh/id_rsa` | `git@github.com:user/repo.git` |
| Your Project Backend | `github-project` | `~/.ssh/id_ed25519` | `git@github-project:username/your-project-backend.git` |
| Your Project Frontend | `github-project-admin` | `~/.ssh/id_ed25519` | `git@github-project-admin:username/your-project-frontend.git` |

---

## ✅ Verification Checklist

- [ ] GitHub repository secrets configured
- [ ] VPS SSH password authentication enabled
- [ ] SSH keys generated successfully
- [ ] SSH config file created with custom hosts
- [ ] Public keys added to GitHub deploy keys
- [ ] SSH connection test successful
- [ ] PM2 installed globally
- [ ] Repositories cloned successfully

---

---

## 🌟 **Star this Repository**
If this guide helped you, please ⭐ **star this repository** to help others discover it!

## 📚 **Additional Resources**

### **Official Documentation**
- 📖 [GitHub Actions Documentation](https://docs.github.com/en/actions)
- 🔧 [PM2 Process Manager Guide](https://pm2.keymetrics.io/)
- 🔐 [SSH Key Management](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- 🌐 [Nginx Configuration Guide](https://nginx.org/en/docs/)

### **Related Tutorials**
- 🚀 [Docker CI/CD with GitHub Actions](https://docs.docker.com/ci-cd/github-actions/)
- ☁️ [AWS EC2 Deployment Guide](https://aws.amazon.com/getting-started/hands-on/deploy-nodejs-web-app/)
- 🔄 [GitLab CI/CD Pipeline](https://docs.gitlab.com/ee/ci/)

### **Community Resources**
- 💬 [GitHub Discussions](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/discussions)
- 🐛 [Report Issues](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/issues)
- 📖 [Wiki Documentation](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/wiki)
- 🎥 [Video Tutorials](https://youtube.com/playlist/your-playlist)

## 🤝 **Contributing**

We welcome contributions! Here's how you can help:

### **Ways to Contribute:**
1. 🐛 **Report Bugs** - Found an issue? Let us know!
2. 💡 **Suggest Features** - Have ideas for improvements?
3. 📝 **Improve Documentation** - Help make guides clearer
4. 🔧 **Submit Pull Requests** - Fix bugs or add features
5. ⭐ **Star the Repository** - Show your support!

### **Contribution Guidelines:**
```bash
# Fork the repository
git fork niteshpawar97/GitHub-CICD-VPS-Setup-Guide

# Create feature branch
git checkout -b feature/amazing-feature

# Make your changes
git add .
git commit -m "Add amazing feature"

# Push to branch
git push origin feature/amazing-feature

# Create Pull Request
```

### **Code of Conduct:**
- ✅ Be respectful and inclusive
- ✅ Provide constructive feedback
- ✅ Help others learn and grow
- ✅ Follow best practices

## 📊 **Repository Stats**

![GitHub repo size](https://img.shields.io/github/repo-size/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)
![GitHub issues](https://img.shields.io/github/issues/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)
![GitHub pull requests](https://img.shields.io/github/issues-pr/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)
![GitHub last commit](https://img.shields.io/github/last-commit/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)

## 🏷️ **Tags & Topics**
```
github-actions, vps-deployment, ssh-keys, pm2, deployment-automation, 
devops, cicd-pipeline, continuous-integration, server-automation, 
nodejs-deployment, linux-server, nginx, ssl-certificate, docker-deployment
```

## � **Use Cases**

### **Perfect for:**
- 🚀 **Startups** deploying their first applications
- 👨‍💻 **Developers** learning DevOps practices  
- 🏢 **Small Teams** needing simple deployment solutions
- 🎓 **Students** studying CI/CD concepts
- 🔧 **DevOps Engineers** setting up new pipelines

### **Supported Technologies:**
- ✅ **Node.js** applications
- ✅ **React/Vue/Angular** frontends
- ✅ **Express.js** backends
- ✅ **MongoDB/MySQL** databases
- ✅ **Nginx** web server
- ✅ **PM2** process manager

## 📧 **Support & Contact**

### **Get Help:**
- 💬 [GitHub Discussions](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/discussions) - General questions
- 🐛 [GitHub Issues](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/issues) - Bug reports
- 📧 [Email Support](mailto:support@yourproject.com) - Direct assistance
- 💼 [LinkedIn](https://linkedin.com/in/your-profile) - Professional inquiries

### **Response Time:**
- 🟢 **Issues**: Within 24 hours
- 🟡 **Discussions**: Within 48 hours  
- 🔴 **Email**: Within 72 hours

## 🎯 **Roadmap**

### **Upcoming Features:**
- [ ] 🐳 Docker containerization guide
- [ ] ☁️ Cloud providers integration (AWS, DigitalOcean)
- [ ] 📊 Monitoring and logging setup
- [ ] 🔄 Blue-green deployment strategy
- [ ] 🧪 Automated testing integration
- [ ] 📱 Mobile app deployment guides

### **Version History:**
- **v2.0.0** - Advanced configurations added
- **v1.5.0** - Troubleshooting section enhanced  
- **v1.0.0** - Initial complete guide

## 📜 **License**

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

```
MIT License

Copyright (c) 2025 Open Source Community

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software...
```

## 🙏 **Acknowledgments**

Special thanks to:
- 🌟 **GitHub Actions Team** for amazing CI/CD platform
- 🚀 **PM2 Team** for excellent process management
- 🔧 **Open Source Community** for continuous improvements
- 👥 **Contributors** who made this project better
- 📚 **Documentation Writers** for clear guides

## 📈 **SEO Keywords**

`github actions tutorial` `vps deployment guide` `ssh deploy keys setup` `pm2 process manager` `cicd pipeline nodejs` `continuous integration deployment` `devops automation` `server deployment tutorial` `github secrets management` `multiple repository deployment` `nginx ssl setup` `linux server configuration`

## 🔒 Security Best Practices

1. **Never commit SSH private keys** to version control
2. **Use Deploy Keys** instead of personal SSH keys for repositories
3. **Limit Deploy Key permissions** (read-only if possible)
4. **Regularly rotate SSH keys** for enhanced security
5. **Use strong passwords** for VPS access
6. **Enable firewall** on VPS (UFW recommended)
7. **Keep PM2 and Node.js updated** to latest stable versions
8. **Use environment variables** for sensitive data
9. **Enable two-factor authentication** on GitHub
10. **Monitor server logs** regularly for suspicious activity

### **Security Checklist:**
```bash
# Enable UFW firewall
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443

# Update system packages
sudo apt update && sudo apt upgrade -y

# Set up fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

---

## 📝 Notes

- Replace `niteshpawar97` with your actual GitHub username
- Update `your-server-ip` with your actual VPS IP address
- Ensure proper file permissions for SSH keys (600)
- Test SSH connections before proceeding with deployment
- Customize project names according to your actual project structure

---

**Last Updated:** November 9, 2025  
**Maintained by:** Open Source Community  
**Repository:** [GitHub-CICD-VPS-Setup-Guide](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide)

---

<div align="center">

### ⚡ **Made with ❤️ by DevOps Community**

**[⭐ Star](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide) • [🍴 Fork](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/fork) • [🐛 Report Bug](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/issues) • [💡 Request Feature](https://github.com/niteshpawar97/GitHub-CICD-VPS-Setup-Guide/issues)**

</div>
