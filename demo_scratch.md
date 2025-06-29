# SFDX-Hardis Complete Demo Guide

## Overview

This demo showcases how **sfdx-hardis** streamlines Salesforce DevOps by delivering a task from development through integration to UAT using scratch orgs and automated CI/CD pipelines.

### What is SFDX-Hardis?

SFDX-Hardis is a powerful DevOps toolkit that extends Salesforce CLI capabilities with:
- **Smart deployment strategies** with automatic dependency resolution
- **Advanced scratch org management** with data seeding and configuration
- **Integrated CI/CD pipelines** with comprehensive testing
- **Automated quality checks** and security scans
- **Simplified team collaboration** with branch-based development

---

## Prerequisites

### System Requirements
- Node.js 18+ installed
- Git configured
- Salesforce CLI (sf) installed
- VS Code with Salesforce extensions (recommended)

### Accounts Needed
- Salesforce Developer Hub org (free)
- GitHub account (free)
- GitHub Actions enabled

---

## Part 1: Installation & Setup

### 1.1 Install SFDX-Hardis

```bash
# Install globally
npm install -g sfdx-hardis

# Verify installation
sfdx hardis:version
```

### 1.2 Set Up Development Hub

```bash
# Authenticate to your Dev Hub
sf org login web --set-default-dev-hub --alias DevHub

# Enable Dev Hub features if not already done
# (This is done in Setup > Dev Hub in your org)
```

### 1.3 Initialize Project

```bash
# Create project directory
mkdir salesforce-demo-project
cd salesforce-demo-project

# Initialize SFDX project with hardis enhancements
sfdx hardis:project:create --projectname "Demo Project" --packagename "demo"

# Initialize git repository
git init
git add .
git commit -m "Initial project setup"
```

### 1.4 Configure Scratch Org Definitions

Create enhanced scratch org definitions:

**config/project-scratch-def-integration.json:**
```json
{
  "orgName": "Integration Environment",
  "description": "Integration scratch org for continuous testing",
  "edition": "Developer",
  "features": ["EnableSetPasswordInApi", "MultiCurrency"],
  "settings": {
    "lightningExperienceSettings": {
      "enableS1DesktopEnabled": true
    },
    "securitySettings": {
      "sessionSettings": {
        "sessionTimeout": "TwentyFourHours"
      }
    }
  },
  "adminEmail": "admin@demo-integration.com"
}
```

**config/project-scratch-def-uat.json:**
```json
{
  "orgName": "UAT Environment", 
  "description": "UAT scratch org for user acceptance testing",
  "edition": "Developer",
  "features": ["EnableSetPasswordInApi", "MultiCurrency"],
  "settings": {
    "lightningExperienceSettings": {
      "enableS1DesktopEnabled": true
    }
  },
  "adminEmail": "admin@demo-uat.com"
}
```

---

## Part 2: GitHub Repository Setup

### 2.1 Create GitHub Repository

1. Go to GitHub and create a new public repository: `salesforce-sfdx-hardis-demo`
2. Don't initialize with README (we already have files)

### 2.2 Connect Local Repository

```bash
# Add GitHub remote
git remote add origin https://github.com/YOUR_USERNAME/salesforce-sfdx-hardis-demo.git

# Push initial code
git branch -M main
git push -u origin main
```

### 2.3 Configure GitHub Actions

Create `.github/workflows/ci-cd.yml`:

```yaml
name: SFDX-Hardis CI/CD Pipeline

on:
  push:
    branches: [ main, integration, uat ]
  pull_request:
    branches: [ main, integration, uat ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install Salesforce CLI
      run: npm install -g @salesforce/cli
      
    - name: Install SFDX-Hardis
      run: npm install -g sfdx-hardis
      
    - name: Authenticate Dev Hub
      run: |
        echo "${{ secrets.DEVHUB_SFDX_URL }}" > ./DEVHUB_SFDX_URL.txt
        sf org login sfdx-url --sfdx-url-file ./DEVHUB_SFDX_URL.txt --alias DevHub --set-default-dev-hub
        
    - name: Run SFDX-Hardis Quality Checks
      run: sfdx hardis:lint:access
      
    - name: Create Scratch Org and Deploy
      if: github.ref == 'refs/heads/integration'
      run: |
        sfdx hardis:org:create --scratch --config integration
        sfdx hardis:source:deploy --check-only
        
    - name: Run Tests
      if: github.ref == 'refs/heads/integration'
      run: sfdx hardis:apex:test:run --coverage
      
    - name: Deploy to UAT
      if: github.ref == 'refs/heads/uat' 
      run: |
        sfdx hardis:org:create --scratch --config uat
        sfdx hardis:source:deploy
        sfdx hardis:data:seed
```

---

## Part 3: Development Workflow Demo

### 3.1 Create Feature Branch

```bash
# Create feature branch for our demo task
git checkout -b feature/customer-validation

# Create local development scratch org
sfdx hardis:org:create --scratch --config dev --alias DevOrg
```

### 3.2 Develop the Feature

**Demo Task:** Create a custom field with validation on Account object

```bash
# Generate custom field metadata
sf sobject field create --label "Customer Type" --name Customer_Type__c --type Picklist --required --target-org DevOrg

# Add picklist values via VS Code or manually create the field
```

Create **force-app/main/default/objects/Account/fields/Customer_Type__c.field-meta.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Customer_Type__c</fullName>
    <label>Customer Type</label>
    <required>true</required>
    <trackFeedHistory>false</trackFeedHistory>
    <type>Picklist</type>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <sorted>false</sorted>
            <value>
                <fullName>Enterprise</fullName>
                <default>false</default>
                <label>Enterprise</label>
            </value>
            <value>
                <fullName>SMB</fullName>
                <default>false</default>
                <label>Small/Medium Business</label>
            </value>
            <value>
                <fullName>Startup</fullName>
                <default>false</default>
                <label>Startup</label>
            </value>
        </valueSetDefinition>
    </valueSet>
</CustomField>
```

Create validation rule **force-app/main/default/objects/Account/validationRules/Customer_Type_Required.validationRule-meta.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Customer_Type_Required</fullName>
    <active>true</active>
    <description>Customer Type is required for all accounts</description>
    <errorConditionFormula>ISBLANK(TEXT(Customer_Type__c))</errorConditionFormula>
    <errorMessage>Please select a Customer Type</errorMessage>
</ValidationRule>
```

### 3.3 Deploy and Test Locally

```bash
# Deploy to development scratch org
sfdx hardis:source:deploy --target-org DevOrg

# Run quality checks
sfdx hardis:lint:access

# Create test data
sfdx hardis:data:seed --target-org DevOrg
```

### 3.4 Commit Changes

```bash
git add .
git commit -m "feat: Add Customer Type field with validation

- Added required Customer Type picklist field to Account
- Implemented validation rule to ensure field is populated
- Added test data for validation scenarios"
```

---

## Part 4: Integration Deployment

### 4.1 Create Integration Branch

```bash
# Push feature branch
git push origin feature/customer-validation

# Create integration branch
git checkout -b integration
git merge feature/customer-validation
git push origin integration
```

### 4.2 SFDX-Hardis Advantages in Integration

**Automated Integration Scratch Org Creation:**

```bash
# SFDX-Hardis automatically creates fresh integration environment
sfdx hardis:org:create --scratch --config integration --alias IntegrationOrg

# Smart deployment with dependency checking
sfdx hardis:source:deploy --target-org IntegrationOrg --smart
```

**Key Advantages Demonstrated:**

1. **Dependency Resolution:** SFDX-Hardis automatically resolves metadata dependencies
2. **Smart Deployment:** Only deploys changed components
3. **Automatic Testing:** Runs all relevant tests automatically
4. **Quality Gates:** Enforces code coverage and quality standards
5. **Environment Consistency:** Ensures integration environment matches production

### 4.3 Integration Testing

```bash
# Run comprehensive test suite
sfdx hardis:apex:test:run --target-org IntegrationOrg --coverage --wait 10

# Check deployment status
sfdx hardis:source:deploy:report --target-org IntegrationOrg

# Validate with test data
sfdx hardis:data:seed --target-org IntegrationOrg --plan integration
```

---

## Part 5: UAT Deployment

### 5.1 Promote to UAT

```bash
# Create UAT branch from integration
git checkout -b uat
git merge integration
git push origin uat
```

### 5.2 UAT Environment Setup

```bash
# Create UAT scratch org with specific configuration
sfdx hardis:org:create --scratch --config uat --alias UATOrg --duration 30

# Deploy with UAT-specific settings
sfdx hardis:source:deploy --target-org UATOrg --environment uat

# Seed with UAT test data
sfdx hardis:data:seed --target-org UATOrg --plan uat
```

### 5.3 User Acceptance Testing

**SFDX-Hardis UAT Advantages:**

1. **Fresh Environment:** Each UAT cycle gets a clean, consistent environment
2. **Production-like Data:** Configurable test data that mirrors production scenarios
3. **Easy Reset:** Can quickly recreate UAT environment if needed
4. **User Training:** Perfect environment for user training and documentation

```bash
# Generate UAT test credentials
sfdx hardis:org:user:create --target-org UATOrg --alias "uat-user@demo.com"

# Create demo data for UAT testing
sfdx hardis:data:create --plan uat-demo --target-org UATOrg
```

---

## Part 6: SFDX-Hardis Key Advantages Summary

### 6.1 Development Experience
- **Faster Setup:** Automated scratch org creation with pre-configured settings
- **Smart Deployments:** Only deploy what changed, with automatic dependency resolution
- **Quality Assurance:** Built-in code analysis and security scanning

### 6.2 Team Collaboration
- **Branch-based Development:** Clean separation of features and environments
- **Consistent Environments:** Reproducible scratch orgs across team members
- **Automated Testing:** Comprehensive test execution with coverage reporting

### 6.3 CI/CD Pipeline Benefits
- **Automated Workflows:** GitHub Actions integration with minimal configuration
- **Environment Promotion:** Streamlined promotion from dev → integration → UAT → production
- **Quality Gates:** Automatic enforcement of coding standards and test coverage

### 6.4 Operational Efficiency
- **Cost Effective:** Uses free scratch orgs instead of paid sandboxes
- **Rapid Provisioning:** Create/destroy environments in minutes
- **Scalable:** Handle multiple feature branches and parallel development

---

## Part 7: Demo Script

### Demo Flow (15-20 minutes)

1. **Introduction (2 min)**
   - Show GitHub repository structure
   - Explain the customer validation feature requirement

2. **Development Phase (5 min)**
   - Create feature branch
   - Show automated scratch org creation
   - Deploy custom field and validation rule
   - Demonstrate local testing

3. **Integration Phase (5 min)**
   - Merge to integration branch
   - Show GitHub Actions pipeline execution
   - Demonstrate smart deployment features
   - Review automated test results

4. **UAT Phase (5 min)**
   - Promote to UAT branch
   - Show UAT scratch org creation
   - Demonstrate user acceptance testing environment
   - Show test data seeding

5. **Wrap-up (3 min)**
   - Highlight key SFDX-Hardis advantages
   - Show cost savings vs traditional sandboxes
   - Demonstrate environment cleanup

### Key Talking Points

- **"No more waiting for sandboxes"** - Scratch orgs ready in 2 minutes
- **"Smart deployments"** - Only deploy what changed, resolve dependencies automatically  
- **"True CI/CD"** - Automated testing and quality gates
- **"Cost-effective DevOps"** - Free scratch orgs vs expensive sandboxes
- **"Team scalability"** - Each developer gets their own environment

---

## Part 8: Cleanup

```bash
# Delete scratch orgs
sfdx hardis:org:delete --target-org DevOrg
sfdx hardis:org:delete --target-org IntegrationOrg  
sfdx hardis:org:delete --target-org UATOrg

# Clean up local branches
git checkout main
git branch -D feature/customer-validation integration uat
```

---

## Conclusion

This demo showcases how SFDX-Hardis transforms Salesforce development from a manual, error-prone process into a streamlined, automated DevOps pipeline. The combination of scratch orgs, smart deployments, and automated testing creates a professional development experience that scales with your team.

**Next Steps:**
- Explore advanced SFDX-Hardis features like automated documentation
- Set up monitoring and alerts for production deployments
- Implement advanced testing strategies with data factories
- Configure advanced security scanning and compliance checks
