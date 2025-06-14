# SFDX-Hardis Auto-Generated Template Files

## 🎯 **Files Provided by `sf hardis:project:create`**

When you run `sf hardis:project:create`, SFDX-Hardis automatically generates a comprehensive project structure with multiple template files and configurations.

---

## 📋 **Core Configuration Files (Auto-Generated)**

| **File** | **Description** | **Customizable** | **Template Source** |
|----------|-----------------|------------------|-------------------|
| `.sfdx-hardis.yml` | Master configuration file with default settings | ✅ Yes | Built-in template |
| `sfdx-project.json` | Enhanced Salesforce project definition | ✅ Yes | Extended from SF CLI |
| `.gitignore` | Optimized Git exclusions for Salesforce projects | ✅ Yes | SFDX-Hardis template |
| `.forceignore` | Salesforce deployment exclusions | ✅ Yes | SFDX-Hardis template |
| `README.md` | Project documentation with SFDX-Hardis instructions | ✅ Yes | Generated template |

---

## 🔄 **CI/CD Pipeline Files (Platform-Specific)**

### **GitHub Actions** (when selected)
| **File** | **Description** | **Purpose** |
|----------|-----------------|-------------|
| `.github/workflows/` | Complete workflow folder | CI/CD automation |
| `├── ci-cd.yml` | Main CI/CD pipeline | Build, test, deploy |
| `├── backup.yml` | Automated org backup | Data protection |
| `└── monitoring.yml` | Org health monitoring | Quality assurance |

### **GitLab CI** (when selected)
| **File** | **Description** | **Purpose** |
|----------|-----------------|-------------|
| `gitlab-ci.yml` | Main GitLab pipeline | CI/CD automation |
| `gitlab-ci-config.yml` | Configuration variables | Pipeline settings |

### **Azure DevOps** (when selected)
| **File** | **Description** | **Purpose** |
|----------|-----------------|-------------|
| `azure-pipelines-checks.yml` | Quality checks pipeline | Code validation |
| `azure-pipelines-deployment.yml` | Deployment pipeline | Release automation |

### **Bitbucket Pipelines** (when selected)
| **File** | **Description** | **Purpose** |
|----------|-----------------|-------------|
| `bitbucket-pipelines.yml` | Bitbucket CI/CD pipeline | Build and deploy |

---

## 🔧 **Environment Configuration Templates**

| **File** | **Description** | **Auto-Generated Content** |
|----------|-----------------|---------------------------|
| `config/project-scratch-def.json` | Default scratch org definition | Standard features, settings |
| `config/.sfdx-hardis-integration.yml` | Integration branch configuration | Environment-specific rules |
| `config/.sfdx-hardis-uat.yml` | UAT branch configuration | Testing environment setup |
| `config/pmd-ruleset.xml` | PMD code quality rules | Best practice rules |

---

## 📦 **Package Management Files**

| **File** | **Description** | **Generated When** |
|----------|-----------------|-------------------|
| `manifest/package.xml` | Deployment manifest | Always (with basic template) |
| `manifest/destructiveChanges.xml` | Component deletion manifest | On request |
| `config/packageAliases.json` | Package version tracking | When using packages |

---

## 📊 **VS Code Integration Files**

| **File** | **Description** | **Content** |
|----------|-----------------|-------------|
| `.vscode/extensions.json` | Recommended VS Code extensions | SFDX-Hardis extension list |
| `.vscode/settings.json` | Workspace settings | Optimized for Salesforce dev |
| `.vscode/launch.json` | Debug configurations | Apex debugging setup |
| `.vscode/tasks.json` | Build tasks | SFDX-Hardis commands |

---

## 🧪 **Testing & Quality Templates**

| **File** | **Description** | **Purpose** |
|----------|-----------------|-------------|
| `config/.eslintrc.json` | ESLint configuration for LWC | Code quality |
| `.prettierrc` | Code formatting rules | Consistent styling |
| `jest.config.js` | Jest testing configuration | LWC unit tests |
| `tests/jest/test-utils.js` | Testing utilities | Test helpers |

---

## 📝 **Documentation Templates**

| **File** | **Description** | **Auto-Generated** |
|----------|-----------------|-------------------|
| `mkdocs.yml` | Documentation site configuration | ✅ |
| `docs/index.md` | Main documentation page | ✅ |
| `docs/setup.md` | Setup instructions | ✅ |
| `docs/contributing.md` | Contribution guidelines | ✅ |

---

## 🔨 **Script Templates**

| **File** | **Description** | **Language** |
|----------|-----------------|-------------|
| `scripts/shell/pre-deploy-validation.sh` | Pre-deployment checks | Shell |
| `scripts/shell/post-deploy-notification.sh` | Post-deployment actions | Shell |
| `scripts/apex/setup-dev-data.apex` | Development data setup | Apex |
| `scripts/python/data-migration.py` | Data migration utilities | Python |

---

## 🎨 **Customization and Interactive Setup**

### **Interactive Wizard Features**
When running `sf hardis:project:create`, you get an interactive wizard that asks:

1. **Project Type**
   - Scratch orgs only
   - Sandboxes only  
   - Mixed environment

2. **Git Provider**
   - GitHub
   - GitLab
   - Azure DevOps
   - Bitbucket

3. **CI/CD Features**
   - Automated testing
   - Quality gates
   - Deployment automation
   - Monitoring & backup

4. **Quality Tools**
   - PMD static analysis
   - ESLint for LWC
   - Security scanning
   - Code coverage

### **Template Customization**
- All generated files include comments explaining customization options
- Configuration files contain "additional configuration instructions"
- Templates are designed to be modified for specific project needs

---

## 🚀 **Auto-Generated Content Benefits**

### **⚡ Instant Professional Setup**
- Complete DevOps pipeline in minutes
- Industry best practices included
- No manual configuration required

### **🔧 Platform Optimized**
- Templates specific to chosen Git provider
- CI/CD workflows tailored to platform capabilities
- Secrets management configured

### **📈 Scalable Architecture**
- Templates support team collaboration
- Environment separation built-in
- Quality gates enforced

### **🎯 Salesforce Specialized**
- Optimized for Salesforce metadata
- Scratch org automation included
- Security and compliance focused

---

## 💡 **Template Evolution & Updates**

### **Version Management**
- Templates updated with each SFDX-Hardis release
- Backward compatibility maintained
- Migration guides provided for major changes

### **Community Contributions**
- Templates improved based on community feedback
- Best practices incorporated from real-world usage
- Platform-specific optimizations added

### **AI Enhancement**
- AI integration for documentation generation
- Automated code analysis and suggestions
- Smart template recommendations

---

## 🎯 **Key Advantages of SFDX-Hardis Templates**

1. **🚀 Zero-Configuration Start**: Professional setup in one command
2. **🏗️ Best Practices Built-In**: Industry standards automatically applied
3. **🔄 Platform Agnostic**: Works with all major Git providers
4. **📊 Quality Focused**: Code analysis and testing included
5. **👥 Team Ready**: Collaboration tools pre-configured
6. **🔧 Customizable**: Templates designed for modification
7. **📚 Well Documented**: Each file includes usage instructions
8. **🔄 Continuously Updated**: Templates evolve with community needs

**Bottom Line**: SFDX-Hardis provides "working GitHub workflows" that are "copied in your repo when initializing hardis config", giving you a complete, production-ready Salesforce DevOps setup instantly! 🎉
