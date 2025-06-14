# SFDX-Hardis Demo Project Folder Structure

```
salesforce-sfdx-hardis-demo/
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 ci-cd.yml                                    # GitHub Actions CI/CD pipeline
│
├── 📁 .sfdx/                                              # Salesforce CLI cache (auto-generated)
│   ├── 📄 sfdx-config.json
│   └── 📁 orgs/
│       └── 📄 ...                                         # Org cache files
│
├── 📁 .vscode/                                            # VS Code workspace settings
│   ├── 📄 extensions.json                                 # Recommended extensions
│   ├── 📄 launch.json                                     # Debug configurations
│   ├── 📄 settings.json                                   # Workspace settings
│   └── 📄 tasks.json                                      # Build tasks
│
├── 📁 config/                                             # Environment & Quality Configurations
│   ├── 📄 project-scratch-def.json                       # 🔧 Default dev scratch org definition
│   ├── 📄 project-scratch-def-integration.json           # 🔧 Integration environment definition
│   ├── 📄 project-scratch-def-uat.json                   # 🔧 UAT environment definition
│   ├── 📄 pmd-ruleset.xml                                # 🔍 PMD code quality rules for Apex
│   └── 📄 .eslintrc.json                                 # ⚡ ESLint rules for Lightning components
│
├── 📁 data/                                              # Test Data & Seeding Plans
│   ├── 📄 dev-data-plan.json                            # 🗃️ Development data seeding plan
│   ├── 📄 integration-data-plan.json                    # 🗃️ Integration data seeding plan
│   ├── 📄 uat-data-plan.json                            # 🗃️ UAT data seeding plan
│   ├── 📄 Account.csv                                    # 📊 Sample Account test data
│   ├── 📄 Contact.csv                                    # 📊 Sample Contact test data
│   └── 📄 Opportunity.csv                                # 📊 Sample Opportunity test data
│
├── 📁 docs/                                              # Generated Documentation
│   ├── 📄 index.html                                     # 📚 Main project documentation
│   ├── 📁 apex/
│   │   ├── 📄 index.html                                 # 📖 Apex classes documentation
│   │   └── 📄 AccountController.html                     # 📖 Specific class documentation
│   ├── 📁 lwc/
│   │   ├── 📄 index.html                                 # ⚡ Lightning Web Components docs
│   │   └── 📄 customerTypeSelector.html                  # ⚡ Component documentation
│   └── 📁 assets/
│       ├── 📄 style.css                                  # 🎨 Documentation styling
│       └── 📄 logo.png                                   # 🖼️ Project logo
│
├── 📁 force-app/                                         # Salesforce Source Code
│   └── 📁 main/
│       └── 📁 default/
│           ├── 📁 applications/                          # 📱 Lightning Apps
│           │   └── 📄 Demo_App.app-meta.xml
│           │
│           ├── 📁 classes/                               # 🧠 Apex Classes
│           │   ├── 📄 AccountController.cls              # 🧠 Main business logic
│           │   ├── 📄 AccountController.cls-meta.xml     # 📋 Class metadata
│           │   ├── 📄 AccountControllerTest.cls          # 🧪 Unit test class
│           │   ├── 📄 AccountControllerTest.cls-meta.xml # 📋 Test class metadata
│           │   ├── 📄 CustomerTypeService.cls            # 🧠 Service layer class
│           │   ├── 📄 CustomerTypeService.cls-meta.xml   # 📋 Service metadata
│           │   ├── 📄 TestDataFactory.cls                # 🏭 Test data factory
│           │   └── 📄 TestDataFactory.cls-meta.xml       # 📋 Factory metadata
│           │
│           ├── 📁 lwc/                                   # ⚡ Lightning Web Components
│           │   ├── 📁 customerTypeSelector/
│           │   │   ├── 📄 customerTypeSelector.js        # ⚡ Component JavaScript
│           │   │   ├── 📄 customerTypeSelector.html      # 🎨 Component template
│           │   │   ├── 📄 customerTypeSelector.css       # 🎨 Component styles
│           │   │   ├── 📄 customerTypeSelector.js-meta.xml # 📋 Component metadata
│           │   │   └── 📄 __tests__/                     # 🧪 Jest tests folder
│           │   │       └── 📄 customerTypeSelector.test.js
│           │   │
│           │   └── 📁 customerTypeUtils/
│           │       ├── 📄 customerTypeUtils.js           # ⚡ Utility component
│           │       ├── 📄 customerTypeUtils.html
│           │       ├── 📄 customerTypeUtils.css
│           │       └── 📄 customerTypeUtils.js-meta.xml
│           │
│           ├── 📁 objects/                               # 📋 Custom Objects & Fields
│           │   └── 📁 Account/
│           │       ├── 📁 fields/
│           │       │   ├── 📄 Customer_Type__c.field-meta.xml # 📝 Custom picklist field
│           │       │   └── 📄 Customer_Segment__c.field-meta.xml # 📝 Additional field
│           │       ├── 📁 validationRules/
│           │       │   ├── 📄 Customer_Type_Required.validationRule-meta.xml # ✅ Main validation
│           │       │   └── 📄 Customer_Segment_Logic.validationRule-meta.xml # ✅ Additional validation
│           │       ├── 📁 listViews/
│           │       │   └── 📄 Accounts_by_Customer_Type.listView-meta.xml
│           │       └── 📁 recordTypes/
│           │           └── 📄 Enterprise_Account.recordType-meta.xml
│           │
│           ├── 📁 permissionsets/                        # 🔐 Permission Sets
│           │   ├── 📄 Customer_Type_Manager.permissionset-meta.xml
│           │   └── 📄 Customer_Type_Viewer.permissionset-meta.xml
│           │
│           ├── 📁 tabs/                                  # 📑 Custom Tabs
│           │   └── 📄 Customer_Analytics.tab-meta.xml
│           │
│           ├── 📁 flows/                                 # 🔄 Process Automation
│           │   └── 📄 Customer_Type_Assignment.flow-meta.xml
│           │
│           ├── 📁 triggers/                              # ⚡ Apex Triggers
│           │   ├── 📄 AccountTrigger.trigger
│           │   ├── 📄 AccountTrigger.trigger-meta.xml
│           │   ├── 📄 AccountTriggerHandler.cls
│           │   └── 📄 AccountTriggerHandler.cls-meta.xml
│           │
│           └── 📁 staticresources/                       # 📦 Static Resources
│               ├── 📄 CustomerTypeIcons.resource-meta.xml
│               └── 📄 CustomerTypeIcons.zip
│
├── 📁 manifest/                                          # Deployment Manifests
│   ├── 📄 package.xml                                    # 📦 Full deployment manifest
│   ├── 📄 package-dev.xml                               # 📦 Development-only components
│   └── 📄 destructiveChanges.xml                        # 🗑️ Components to delete
│
├── 📁 reports/                                           # Generated Reports & Analysis
│   ├── 📄 coverage.xml                                   # 📊 Test coverage report (JUnit format)
│   ├── 📄 coverage.html                                  # 📊 Test coverage HTML report
│   ├── 📄 pmd-report.html                               # 🔍 PMD code quality report
│   ├── 📄 pmd-report.xml                                # 🔍 PMD XML report
│   ├── 📄 eslint-report.html                            # ⚡ ESLint report for LWC
│   ├── 📄 security-scan.json                            # 🛡️ Security vulnerability scan
│   ├── 📄 dependency-analysis.json                      # 🔗 Metadata dependency analysis
│   ├── 📄 deployment-report.html                        # 🚀 Deployment summary report
│   └── 📄 jira-integration.json                         # 🎫 JIRA automation logs
│
├── 📁 scripts/                                           # Automation Scripts
│   ├── 📁 apex/                                         # 🎯 Apex Scripts
│   │   ├── 📄 setup-dev-data.apex                       # 🔧 Development environment setup
│   │   ├── 📄 setup-integration-data.apex               # 🔧 Integration environment setup
│   │   ├── 📄 setup-uat-data.apex                       # 🔧 UAT environment setup
│   │   ├── 📄 post-data-setup.apex                      # 📤 Post-deployment data processing
│   │   ├── 📄 cleanup-dev-data.apex                     # 🧹 Development data cleanup
│   │   └── 📄 configure-permissions.apex                # 🔐 Permission setup script
│   │
│   ├── 📁 shell/                                        # 🐚 Shell Scripts
│   │   ├── 📄 pre-deploy-validation.sh                  # ✅ Pre-deployment validation
│   │   ├── 📄 post-deploy-notification.sh               # 📧 Post-deployment notifications
│   │   ├── 📄 backup-production.sh                      # 💾 Production backup script
│   │   ├── 📄 environment-cleanup.sh                    # 🧹 Environment cleanup
│   │   └── 📄 jira-integration.sh                       # 🎫 JIRA API integration
│   │
│   └── 📁 python/                                       # 🐍 Python Scripts (optional)
│       ├── 📄 data-migration.py                         # 📊 Data migration utilities
│       ├── 📄 report-generator.py                       # 📈 Custom report generation
│       └── 📄 requirements.txt                          # 📦 Python dependencies
│
├── 📁 tests/                                            # Additional Testing Files
│   ├── 📁 jest/                                         # ⚡ Jest tests for LWC
│   │   ├── 📄 jest.config.js                           # ⚡ Jest configuration
│   │   └── 📁 __tests__/
│   │       └── 📄 test-utils.js                        # 🧪 Test utilities
│   │
│   ├── 📁 selenium/                                     # 🌐 UI automation tests
│   │   ├── 📄 customer-type-e2e.js                     # 🧪 End-to-end tests
│   │   └── 📄 selenium-config.js                       # ⚙️ Selenium configuration
│   │
│   └── 📁 data/                                         # 🧪 Test-specific data
│       ├── 📄 test-accounts.csv                        # 📊 Test Account data
│       └── 📄 test-scenarios.json                      # 📋 Test scenarios definition
│
├── 📁 tools/                                            # Development Tools & Utilities
│   ├── 📄 package.json                                  # 📦 Node.js dependencies for tools
│   ├── 📄 package-lock.json                            # 🔒 Dependency lock file
│   ├── 📁 node_modules/                                # 📦 Node.js modules (git-ignored)
│   └── 📁 custom-scripts/
│       ├── 📄 metadata-analyzer.js                     # 🔍 Custom metadata analysis
│       └── 📄 org-compare.js                           # ⚖️ Org comparison utility
│
├── 📄 .sfdx-hardis.yml                                  # 🎯 Master SFDX-Hardis configuration
├── 📄 sfdx-project.json                                 # 📦 Salesforce project definition
├── 📄 .gitignore                                        # 🚫 Git exclusion rules
├── 📄 .forceignore                                      # 🚫 Salesforce deployment exclusions
├── 📄 .eslintignore                                     # 🚫 ESLint exclusion rules
├── 📄 .prettierrc                                       # 🎨 Code formatting rules
├── 📄 package.json                                      # 📦 Node.js project configuration
├── 📄 README.md                                         # 📖 Project documentation
├── 📄 CHANGELOG.md                                      # 📝 Version history
├── 📄 LICENSE                                           # ⚖️ Project license
└── 📄 .env.example                                      # 🔐 Environment variables template

```

## 📊 **Folder Structure Summary**

| **Folder** | **Purpose** | **File Count** | **Auto-Generated** |
|------------|-------------|----------------|-------------------|
| 📁 `.github/` | CI/CD pipeline configuration | 1 | ❌ Manual |
| 📁 `.sfdx/` | Salesforce CLI cache | ~5 | ✅ Auto |
| 📁 `.vscode/` | VS Code workspace settings | 4 | ❌ Manual |
| 📁 `config/` | Environment & quality configurations | 5 | ❌ Manual |
| 📁 `data/` | Test data and seeding plans | 6 | ❌ Manual |
| 📁 `docs/` | Generated documentation | ~8 | ✅ Auto |
| 📁 `force-app/` | Salesforce source code | ~25 | ❌ Manual |
| 📁 `manifest/` | Deployment manifests | 3 | ❌ Manual |
| 📁 `reports/` | Analysis and quality reports | 8 | ✅ Auto |
| 📁 `scripts/` | Automation scripts | 12 | ❌ Manual |
| 📁 `tests/` | Additional testing files | 6 | ❌ Manual |
| 📁 `tools/` | Development utilities | ~4 | ❌ Manual |
| **Root files** | Project configuration | 9 | ❌ Manual |

## 🎯 **Key Structure Benefits**

### **🗂️ Clear Separation of Concerns**
- **Source code** clearly separated from **configuration**
- **Generated content** isolated from **manual content**
- **Environment-specific** configurations organized logically

### **🔄 SFDX-Hardis Integration**
- **`.sfdx-hardis.yml`** controls behavior across all folders
- **Scratch org definitions** support multiple environments
- **Data plans** reference CSV files in organized structure
- **Scripts** are automatically discovered and executed

### **👥 Team Collaboration**
- **VS Code settings** shared across team
- **Git configuration** optimized for Salesforce development
- **Quality rules** enforced consistently
- **Documentation** auto-generated and always current

### **🚀 CI/CD Ready**
- **GitHub Actions** workflows in standard location
- **Reports** folder for CI/CD artifact collection
- **Manifest** folder for deployment variations
- **Scripts** folder for automation hooks

This structure provides a **professional, scalable foundation** for Salesforce DevOps with SFDX-Hardis, supporting everything from single developer projects to enterprise-scale team development! 🏗️
