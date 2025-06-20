# SFDX-Hardis Demo Files Reference Table

## 📋 Complete File Inventory

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `.sfdx-hardis.yml` | Configuration | Master configuration file controlling all DevOps behaviors, environments, automation rules, and JIRA integration | Manual/Template | All sfdx hardis commands |
| `sfdx-project.json` | Configuration | Standard Salesforce project definition with package directories and namespace | `sf project generate` | Salesforce CLI |
| `.gitignore` | Configuration | Git exclusion rules for Salesforce projects | Template | Git |
| `README.md` | Documentation | Project documentation and setup instructions | Manual | Developers |

## 🔧 Environment Configuration Files

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `config/project-scratch-def.json` | Scratch Org | Development environment definition with features and settings | Manual/Template | `sfdx hardis:org:create --config dev` |
| `config/project-scratch-def-integration.json` | Scratch Org | Integration environment definition for CI/CD testing | Manual/Template | `sfdx hardis:org:create --config integration` |
| `config/project-scratch-def-uat.json` | Scratch Org | UAT environment definition for user acceptance testing | Manual/Template | `sfdx hardis:org:create --config uat` |
| `config/pmd-ruleset.xml` | Quality | PMD static analysis rules for Apex code quality checks | Manual/Template | `sfdx hardis:lint:access`, PMD scanner |
| `config/.eslintrc.json` | Quality | ESLint configuration for Lightning Web Components linting | Manual/Template | ESLint, VS Code |

## 📝 Source Code Files

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `force-app/main/default/objects/Account/fields/Customer_Type__c.field-meta.xml` | Metadata | Custom picklist field definition for Account object | `sf sobject field create` or Manual | Salesforce deployment |
| `force-app/main/default/objects/Account/validationRules/Customer_Type_Required.validationRule-meta.xml` | Metadata | Validation rule ensuring Customer Type field is populated | Manual | Salesforce deployment |
| `force-app/main/default/classes/AccountController.cls` | Apex | Business logic controller for Account operations | Manual/IDE | Lightning components, Apex runtime |
| `force-app/main/default/classes/AccountController.cls-meta.xml` | Metadata | Metadata definition for AccountController class | Auto-generated | Salesforce deployment |
| `force-app/main/default/classes/AccountControllerTest.cls` | Apex Test | Unit tests for AccountController with code coverage | Manual/IDE | `sfdx hardis:apex:test:run` |
| `force-app/main/default/classes/AccountControllerTest.cls-meta.xml` | Metadata | Metadata definition for test class | Auto-generated | Salesforce deployment |
| `force-app/main/default/lwc/customerTypeSelector/customerTypeSelector.js` | LWC | Lightning Web Component JavaScript logic | Manual/IDE | Lightning runtime |
| `force-app/main/default/lwc/customerTypeSelector/customerTypeSelector.html` | LWC | Lightning Web Component HTML template | Manual/IDE | Lightning runtime |
| `force-app/main/default/lwc/customerTypeSelector/customerTypeSelector.js-meta.xml` | LWC | Lightning Web Component metadata configuration | Manual/IDE | Lightning runtime |

## 🗃️ Data Management Files

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `data/dev-data-plan.json` | Data Plan | Development environment data seeding configuration | Manual | `sfdx hardis:data:seed --plan dev` |
| `data/integration-data-plan.json` | Data Plan | Integration environment data seeding configuration | Manual | `sfdx hardis:data:seed --plan integration` |
| `data/uat-data-plan.json` | Data Plan | UAT environment data seeding configuration | Manual | `sfdx hardis:data:seed --plan uat` |
| `data/Account.csv` | Test Data | Sample Account records for testing and demos | Manual/Export | Data seeding operations |
| `data/Contact.csv` | Test Data | Sample Contact records linked to test Accounts | Manual/Export | Data seeding operations |

## 🔨 Automation Scripts

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `scripts/apex/setup-dev-data.apex` | Apex Script | Post-deployment script for development environment setup | Manual | `sfdx hardis:org:create --config dev` |
| `scripts/apex/setup-integration-data.apex` | Apex Script | Post-deployment script for integration environment setup | Manual | `sfdx hardis:org:create --config integration` |
| `scripts/apex/setup-uat-data.apex` | Apex Script | Post-deployment script for UAT environment setup | Manual | `sfdx hardis:org:create --config uat` |
| `scripts/apex/post-data-setup.apex` | Apex Script | Generic post-data processing script | Manual | Data plan post-processing |
| `scripts/shell/pre-deploy-validation.sh` | Shell Script | Pre-deployment validation checks and preparation | Manual | Pre-deployment hooks |
| `scripts/shell/post-deploy-notification.sh` | Shell Script | Post-deployment notifications and cleanup | Manual | Post-deployment hooks |

## 🔄 CI/CD Pipeline Files

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `.github/workflows/ci-cd.yml` | GitHub Actions | CI/CD pipeline definition with build, test, deploy jobs, and JIRA integration | Manual | GitHub Actions runner |
| `DEVHUB_SFDX_URL` | Secret | Encrypted Dev Hub authentication URL | GitHub Secrets | GitHub Actions workflow |
| `SLACK_WEBHOOK_URL` | Secret | Encrypted Slack webhook for notifications | GitHub Secrets | GitHub Actions workflow |
| `JIRA_SERVER_URL` | Secret | Encrypted JIRA server URL for API access | GitHub Secrets | GitHub Actions workflow, JIRA integration |
| `JIRA_USERNAME` | Secret | Encrypted JIRA username for authentication | GitHub Secrets | GitHub Actions workflow, JIRA integration |
| `JIRA_API_TOKEN` | Secret | Encrypted JIRA API token for secure access | GitHub Secrets | GitHub Actions workflow, JIRA integration |

## 📊 Generated/Output Files

| **File Path** | **Type** | **Purpose** | **Generated by** | **Used by** |
|---------------|----------|-------------|------------------|-------------|
| `docs/index.html` | Documentation | Auto-generated project documentation | `sfdx hardis:doc:generate` | Developers, stakeholders |
| `docs/apex/index.html` | Documentation | Auto-generated Apex class documentation | `sfdx hardis:doc:generate` | Developers |
| `docs/lwc/index.html` | Documentation | Auto-generated Lightning Web Component documentation | `sfdx hardis:doc:generate` | Developers |
| `reports/coverage.xml` | Report | Test coverage report in XML format | `sfdx hardis:apex:test:run --coverage` | CI/CD pipeline, SonarQube |
| `reports/pmd-report.html` | Report | PMD static analysis report | `sfdx hardis:lint:access` | Developers, QA team |
| `reports/security-scan.json` | Report | Security vulnerability scan results | `sfdx hardis:security:scan` | Security team, CI/CD |
| `reports/jira-integration.json` | Report | JIRA ticket updates and automation logs | `sfdx hardis:jira:*` commands | Project managers, JIRA administrators |

## 📁 File Organization Summary

### **Root Level (6 files)**
- Core configuration and project definition files

### **Config Directory (5 files)** 
- Environment definitions and quality rules

### **Source Code (9 files)**
- Salesforce metadata, Apex classes, and Lightning components

### **Data Directory (5 files)**
- Test data and seeding configurations

### **Scripts Directory (6 files)**
- Automation and deployment scripts

### **CI/CD Directory (6 items)**
- GitHub Actions workflow, secrets, and JIRA integration

### **Generated Content (7 files)**
- Auto-generated documentation, reports, and JIRA logs

## 🎯 Key File Relationships

1. **`.sfdx-hardis.yml`** → Controls behavior of 80% of other files
2. **Data Plans** → Reference CSV files and post-processing scripts
3. **CI/CD Workflow** → Triggers generation of reports and documentation
4. **Source Files** → Dependencies between components, classes, and tests
5. **Scripts** → Referenced by configuration files for automation hooks

**Total Files in Demo**: **48 files** across 7 categories, demonstrating the comprehensive nature of a professional Salesforce DevOps setup with SFDX-Hardis and complete JIRA integration for project management and traceability.
