# SFDX-Hardis Demo - Macro Steps Overview

## 🎯 **Demo Scenario: Customer Type Field Implementation**
**JIRA Ticket**: DEMO-123 "Add Customer Type Validation to Account Object"

---

## 📋 **STEP 1: Installation & Prerequisites** (5 minutes)

### **Environment Setup**
```bash
# Install SFDX-Hardis
npm install -g sfdx-hardis

# Verify installation
sf hardis --version

# Authenticate Dev Hub
sf org login web --set-default-dev-hub --alias DevHub
```

### **Accounts Required**
- ✅ Salesforce Developer Hub org (free)
- ✅ GitHub account (free) 
- ✅ JIRA instance (optional for demo)

---

## 🔧 **STEP 2: Project Configuration** (10 minutes)

### **2.1 Initialize Project**
```bash
# Create project structure
mkdir salesforce-sfdx-hardis-demo
cd salesforce-sfdx-hardis-demo

# Interactive project setup wizard
sf hardis project create
```

**Wizard Selections:**
- Git Provider: **GitHub**
- Environment Strategy: **Scratch Orgs**
- CI/CD Features: **All enabled**
- Quality Tools: **PMD + ESLint + Security**

### **2.2 Configure GitHub Repository**
```bash
# Initialize git and connect to GitHub
git init
git remote add origin https://github.com/YOUR_USERNAME/salesforce-sfdx-hardis-demo.git
git push -u origin main
```

### **2.3 Setup GitHub Secrets**
- `DEVHUB_SFDX_URL` - Dev Hub authentication
- `JIRA_SERVER_URL` - JIRA integration (optional)
- `JIRA_API_TOKEN` - JIRA authentication (optional)

---

## 🌳 **STEP 3: Git Branches & Development Workflow** (15 minutes)

### **3.1 Start New Task**
```bash
# Interactive wizard for new development task
sf hardis work new
```

**Wizard Prompts:**
1. **Target Branch**: `integration`
2. **Task Name**: `DEMO-123 Add Customer Type Validation`
3. **Environment**: `Scratch Org`
4. **Duration**: `7 days`

**Auto-Generated:**
- ✅ Feature branch: `feature/DEMO-123-customer-validation`
- ✅ Development scratch org created and configured
- ✅ Data seeded, packages installed
- ✅ JIRA ticket updated to "In Development"

### **3.2 Implement Feature**
```bash
# Deploy custom field and validation rule
sf project deploy start --source-dir force-app

# Run local tests
sf apex test run --test-level RunLocalTests
```

**Development Output:**
- ✅ Custom field: `Customer_Type__c` (picklist)
- ✅ Validation rule: `Customer_Type_Required`
- ✅ Test class: `CustomerTypeTest`

### **3.3 Commit Changes**
```bash
git add .
git commit -m "feat(DEMO-123): Add Customer Type field with validation

- Added required Customer Type picklist field to Account
- Implemented validation rule to ensure field is populated
- Added comprehensive test coverage

Resolves: DEMO-123"
```

---

## 🔄 **STEP 4: Pull Request & Integration** (10 minutes)

### **4.1 Push and Create PR**
```bash
# Push feature branch
git push origin feature/DEMO-123-customer-validation

# Create Pull Request (via GitHub UI or CLI)
gh pr create --title "feat(DEMO-123): Add Customer Type validation" \
  --body "Implements customer type field with validation for Account object"
```

### **4.2 Automated PR Validation**
**GitHub Actions Triggered:**
- ✅ Code quality analysis (PMD + ESLint)
- ✅ Security scanning
- ✅ Scratch org creation for PR validation
- ✅ Automated deployment to integration scratch org
- ✅ Test execution with coverage reporting
- ✅ JIRA ticket updated to "Code Review"

**PR Comments Show:**
- Test coverage: 85%
- No security vulnerabilities
- All quality gates passed
- Deployment ready for integration

### **4.3 Merge to Integration**
```bash
# After approval, merge PR
git checkout integration
git merge feature/DEMO-123-customer-validation
git push origin integration
```

---

## 🚀 **STEP 5: Environment Promotion & Deployment** (15 minutes)

### **5.1 Integration Environment**
**Auto-Triggered on Integration Branch:**
```bash
# GitHub Actions automatically:
# 1. Creates fresh integration scratch org
# 2. Deploys all changes with dependency resolution
# 3. Runs comprehensive test suite
# 4. Updates JIRA ticket to "Testing"
```

**Integration Results:**
- ✅ Fresh environment created in 2 minutes
- ✅ Smart deployment (only changed metadata)
- ✅ All tests passed (92% coverage)
- ✅ Quality gates satisfied

### **5.2 UAT Environment Promotion**
```bash
# Promote to UAT branch
git checkout uat
git merge integration
git push origin uat
```

**Auto-Triggered UAT Deployment:**
- ✅ UAT scratch org created with production-like configuration
- ✅ Test data seeded for user acceptance testing
- ✅ JIRA ticket transitioned to "UAT Testing"
- ✅ Stakeholder notifications sent

### **5.3 UAT Validation**
```bash
# Generate UAT user credentials
sf hardis org user create --target-org UATOrg --alias uat-user@demo.com

# Open UAT environment for testing
sf org open --target-org UATOrg
```

**UAT Activities:**
- ✅ Business user testing of Customer Type field
- ✅ Validation rule testing with various scenarios
- ✅ User training and documentation
- ✅ JIRA ticket moved to "UAT Approved"

---

## 📊 **STEP 6: Demo Highlights & Benefits** (5 minutes)

### **🎯 Key Achievements Demonstrated**

| **Traditional Approach** | **SFDX-Hardis Approach** | **Improvement** |
|--------------------------|---------------------------|-----------------|
| Manual sandbox requests (days) | Scratch orgs in 2 minutes | **99% faster** |
| Manual deployment scripts | Smart deployment with dependencies | **Zero errors** |
| Manual testing coordination | Automated test execution | **100% coverage** |
| Manual environment management | Automated environment lifecycle | **Zero overhead** |
| Manual JIRA updates | Automated ticket management | **Complete traceability** |

### **💡 Business Value Delivered**
- **🚀 Speed**: Complete feature delivery in 1 day vs. 1 week
- **💰 Cost**: Free scratch orgs vs. expensive sandboxes  
- **🔒 Quality**: Automated quality gates and security scanning
- **👥 Scale**: Each developer gets isolated environments
- **📊 Visibility**: Real-time progress tracking and reporting
- **🛡️ Risk**: Consistent, repeatable deployment process

### **🎖️ Professional DevOps Benefits**
- **Automated CI/CD pipeline** with GitHub Actions
- **Environment parity** across dev/integration/UAT
- **Complete audit trail** from JIRA ticket to production
- **Quality enforcement** with automated code analysis
- **Team collaboration** with VS Code integration
- **Documentation generation** with AI enhancement

---

## 🎬 **Demo Script Summary**

### **Opening (2 min)**
*"Today I'll show you how SFDX-Hardis transforms Salesforce development from a manual, error-prone process into a professional DevOps pipeline. We'll implement a complete feature from JIRA ticket to UAT deployment in under 30 minutes."*

### **Installation (3 min)**
*"One npm install command gives us a complete Salesforce DevOps toolkit..."*

### **Configuration (5 min)**  
*"The interactive wizard sets up our entire CI/CD pipeline with best practices built-in..."*

### **Development (8 min)**
*"Starting a new task is as simple as running 'sf hardis work new' - watch as it creates our branch, sets up our environment, and even updates JIRA..."*

### **Deployment (10 min)**
*"Our pull request triggers automated quality checks, creates fresh environments, and promotes our changes through integration to UAT - all with zero manual intervention..."*

### **Closing (2 min)**
*"What traditionally takes a week of manual work, SFDX-Hardis delivers in minutes with professional quality, complete traceability, and zero infrastructure costs."*

---

## 🏆 **Success Metrics**

- **⏱️ Demo Duration**: 30 minutes total
- **🎯 Key Messages**: 6 major benefits highlighted  
- **💻 Commands Shown**: ~10 simple commands
- **🔄 Automation**: 90% of process automated
- **👥 Audience Impact**: Professional DevOps transformation
- **💼 Business Case**: Clear ROI demonstration

**Bottom Line**: SFDX-Hardis transforms Salesforce development from craft to engineering with professional automation, quality enforcement, and team scalability! 🚀
