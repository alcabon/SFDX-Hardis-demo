# SFDX-Hardis Configuration Reference (.sfdx-hardis.yml)

## 📋 **Complete Configuration Properties Reference**

*Based on official SFDX-Hardis documentation and schema analysis*

---

## 🏗️ **Project Configuration**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `projectName` | string | Manual | Project display name for documentation and reports | `"MyClient"` |
| `devHubAlias` | string | Auto/Manual | Dev Hub org alias for scratch org creation | `"DevHub_MyClient"` |
| `devHubUsername` | string | Auto | Dev Hub username for authentication | `"user@company.com"` |
| `developmentBranch` | string | Manual | Main development branch name | `"integration"` |
| `productionBranch` | string | Manual | Production branch name for retrofit operations | `"master"` |
| `retrofitBranch` | string | Manual | Branch used as merge target for retrofit | `"develop"` |

---

## 🌍 **Organization Management**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `allowedOrgTypes` | array | Manual | Allowed org types for operations | `["sandbox", "scratch"]` |
| `availableTargetBranches` | array | Manual | Available branches for new task creation | `["develop", "preprod"]` |
| `availableProjects` | array | Manual | Business projects for branch naming | `["core", "integration", "ui"]` |
| `mergeTargets` | array | Manual | Branch-scoped merge target branches | `["uat", "preprod"]` |

---

## 📦 **Package Management**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `installedPackages` | array | Auto/Manual | List of installed packages with metadata | See detailed structure below |
| `installPackagesDuringCheckDeploy` | boolean | Manual | Install packages during check-only deployments | `true` |
| `installOnScratchOrgs` | boolean | Manual | Install packages when creating scratch orgs | `true` |
| `installDuringDeployments` | boolean | Manual | Install packages during actual deployments | `true` |

### **Package Structure Example:**
```yaml
installedPackages:
  - Id: "0A37Z000000AtDYSA0"
    SubscriberPackageId: "033i0000000LVMYAA4"
    SubscriberPackageName: "Marketing Cloud"
    SubscriberPackageNamespace: "et4ae5"
    SubscriberPackageVersionId: "04t6S000001UjutQAC"
    SubscriberPackageVersionName: "Marketing Cloud"
    SubscriberPackageVersionNumber: "238.3.0.2"
    installOnScratchOrgs: true
    installDuringDeployments: true
```

---

## 🧹 **Automatic Cleaning & Maintenance**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `autoCleanTypes` | array | Manual | Automatic cleaning operations to perform | `["destructivechanges", "minimizeProfiles"]` |
| `autoRemoveUserPermissions` | array | Manual | User permissions to remove automatically | `["FieldServiceAccess", "ViewDataLeakageEvents"]` |
| `autoRetrieveWhenPull` | array | Manual | Metadata to auto-retrieve during pull operations | `["CustomApplication:MyApp", "CustomMetadata"]` |
| `retrofitIgnoredFiles` | array | Manual | Files to ignore during retrofit operations | `["force-app/.../MyApp.app-meta.xml"]` |
| `listViewsToSetToMine` | array | Manual | ListViews to set to "Mine" after deployment | `["Operation__c:MyCurrentOperations"]` |

---

## 🚀 **Deployment Configuration**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `useDeltaDeployment` | boolean | Manual | Enable git delta deployment optimization | `true` |
| `deploymentPlan` | object | Manual | Multi-step deployment configuration | See structure below |
| `testLevel` | string | Manual | Test execution level for deployments | `"RunLocalTests"` |
| `testsToRun` | array | Manual | Specific tests to run (use with caution) | `["AccountTest", "ContactTest"]` |
| `skipCodeCoverageCheck` | boolean | Manual | Skip code coverage validation | `false` |
| `apexTestsMinCoverageOrgWide` | number | Manual | Minimum required org-wide coverage % | `75` |

### **Deployment Plan Structure Example:**
```yaml
deploymentPlan:
  packages:
    - label: "Deploy Flow-Workflow"
      packageXmlFile: "manifest/splits/packageXmlFlowWorkflow.xml"
      order: 6
    - label: "Deploy SharingRules"
      packageXmlFile: "manifest/splits/packageXmlSharingRulesCase.xml"
      order: 30
      waitAfter: 30
```

---

## 👥 **User & Permission Management**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `initPermissionSets` | array | Manual | Permission sets to assign after org creation | `["AdminDefault", "ApiUserPS"]` |
| `profilesNotToMinimize` | array | Manual | Profiles excluded from minimization | `["Admin", "SystemAdministrator"]` |

---

## 🎯 **Branch & Task Management**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `branchPrefixChoices` | array | Manual | Available branch prefixes for new tasks | `["feature", "bugfix", "hotfix"]` |
| `newTaskNameRegex` | string | Manual | Regex pattern for task name validation | `"^[A-Z]+-[0-9]+ .*"` |
| `newTaskNameRegexExample` | string | Manual | Example task name for user guidance | `"PROJ-123 Update validation rules"` |

---

## 📊 **Monitoring & Commands**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `monitoringCommands` | array | Auto | List of monitoring commands to execute | See structure below |
| `customCommands` | array | Manual | Custom VS Code menu commands | See structure below |
| `monitoringAllowedSectionsActions` | object | Manual | Allowed monitoring actions per section | `{"backup": true, "health": true}` |

### **Monitoring Commands Structure:**
```yaml
monitoringCommands:
  - title: "Detect calls to deprecated API versions"
    key: "LEGACYAPI"
    command: "sf hardis:org:diagnose:legacyapi"
    frequency: "weekly"
  - title: "My custom command"
    key: "MY_CUSTOM_KEY"
    command: "sf my:custom:command"
    frequency: "daily"
```

### **Custom Commands Structure:**
```yaml
customCommands:
  - id: "custom-menu"
    label: "Custom commands"
    commands:
      - id: "generate-manifest-xml"
        label: "Generate manifest"
        icon: "file.svg"
        tooltip: "Generates a manifest package.xml"
        command: "sf project generate manifest --source-path force-app"
```

---

## 🔄 **Data Management**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `dataSeeding` | object | Manual | SFDMU data seeding configuration | See SFDMU integration |
| `sourcesToRetrofit` | array | Manual | Metadata types for retrofit operations | `["CustomField", "Layout", "PermissionSet"]` |

---

## 🎨 **Overwrite Management**

| **Property** | **Type** | **Setting** | **Description** | **Example** |
|--------------|----------|-------------|-----------------|-------------|
| `overwriteConfig` | object | Manual | Configuration for handling metadata overwrites | Complex structure for conflict resolution |

---

## 🛠️ **Environment Variables Integration**

Many properties can be overridden by environment variables. Common patterns:

| **Environment Variable** | **Config Property** | **Description** |
|--------------------------|-------------------|-----------------|
| `SFDX_HARDIS_DEPLOY_ERR_COLORS` | N/A | Disable error coloring in deployment output |
| `INSTALL_PACKAGES_DURING_CHECK_DEPLOY` | `installPackagesDuringCheckDeploy` | Override package installation during check |
| `SFDX_HARDIS_DEPLOY_IGNORE_SPLIT_PACKAGES` | N/A | Ignore package.xml splitting |
| `CI_SOURCES_TO_RETROFIT` | `sourcesToRetrofit` | Override retrofit metadata list |

---

## 📋 **Configuration Management Tips**

### **🤖 Automatically Set Properties**
Many properties are **automatically configured** during:
- `sf hardis project create` - Initial setup
- CI/CD setup operations - Pipeline configuration
- Maintenance operations - Ongoing optimization

### **✏️ Manual Configuration Required**
Properties that typically require manual configuration:
- Business logic: `availableProjects`, `branchPrefixChoices`
- Security: `autoRemoveUserPermissions`, `initPermissionSets`
- Performance: `useDeltaDeployment`, `deploymentPlan`
- Quality: `apexTestsMinCoverageOrgWide`, `testLevel`

### **🔄 Branch-Scoped Configuration**
Some properties can be overridden in branch-specific files:
- `config/branches/.sfdx-hardis-BRANCHNAME.yml`
- Useful for environment-specific settings
- Example: Different test levels for integration vs. production

---

## 💡 **Best Practices**

1. **Start Simple**: Use `sf hardis project create` for initial configuration
2. **Iterate Gradually**: Add complexity as team needs evolve
3. **Document Changes**: Comment complex configurations
4. **Environment Separation**: Use branch-scoped configs for environment differences
5. **Version Control**: Keep configuration changes in git history
6. **Team Alignment**: Ensure all team members understand configuration choices

---

## 🔗 **Related Resources**

- **Official Schema**: [Complete list of configuration properties](https://sfdx-hardis.cloudity.com/schema/sfdx-hardis-json-schema-parameters.html)
- **Environment Variables**: [Complete list of environment variables](https://sfdx-hardis.cloudity.com/all-env-variables/)
- **CI/CD Setup**: [Configuration guide for CI/CD](https://sfdx-hardis.cloudity.com/salesforce-ci-cd-config-home/)

---

*This reference table provides a comprehensive overview of SFDX-Hardis configuration options. The official documentation continues to evolve, so always check the latest schema for new properties and capabilities.*
