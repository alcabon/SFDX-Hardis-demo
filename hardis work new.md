# SFDX Hardis Work New Command - Detailed Steps and Prompts

## Overview
The `hardis:work:new` command is an interactive wizard that helps developers start new Salesforce tasks with proper git branching and org setup.

## Step-by-Step Process

### 1. **Initial Setup & Validation**
- Displays welcome message about the CI/CD assistance tool
- Checks that git status is clean (allows stash if needed)
- Loads project configuration from `.sfdx-hardis.yml`

### 2. **Target Branch Selection**
- Automatically selects target branch using `selectTargetBranch()` function
- Uses `developmentBranch` from config or prompts if multiple `availableTargetBranches` are configured

### 3. **Project Selection** (Optional)
**Prompt:** "Please select the project your task is for"
- **Condition:** Only appears if `availableProjects` is configured with multiple options
- **Values:** Configured project names from `.sfdx-hardis.yml`
- **Purpose:** Creates project prefix for branch name (e.g., `france/features/dev/TASK-123`)

### 4. **Task Type Selection**
**Prompt:** "What is the type of the task you want to do?"
- **Default Options:**
  - `🏗️ Feature` (value: `features`) - "New feature, evolution of an existing feature... If you don't know, just select Feature"
  - `🛠️ Debug` (value: `fixes`) - "A bug has been identified and you are the right person to solve it!"
- **Customizable:** Can be overridden via `branchPrefixChoices` in config

### 5. **Source Type Selection**
**Prompt:** "What type(s) of Salesforce updates will you have to perform for this task?"
- **Options:**
  - `🥸 Configuration` (value: `config`) - "You will update anything in the setup except apex code :)"
  - `🤓 Development` (value: `dev`) - "You are a developer who will do magic with Apex or Javascript!"
  - `🥸🤓 Configuration + Development` (value: `dev`) - "Like the unicorn you are, you will update configuration but also write code :)"

### 6. **Task Name Input**
**Prompt:** "What is the name of your new task? (example: MYPROJECT-123 Update account status validation rule). Please avoid accents or special characters"
- **Input Type:** Text input
- **Validation:** Optional regex validation via `newTaskNameRegex` config
- **Processing:** Removes special characters and replaces spaces with dashes
- **Retry:** Re-prompts if validation fails

### 7. **Git Operations**
- Checks out target branch from remote
- Pulls latest changes
- Creates new branch with format: `[project/]taskType/sourceType/taskName`
- Updates local user config to track branch target

### 8. **Default Branch Update** (Conditional)
**Prompt:** "Do you want to update your default target git branch to [SELECTED_BRANCH]?"
- **Condition:** Only if selected branch differs from `developmentBranch` and no `availableTargetBranches` configured
- **Type:** Confirmation (yes/no)
- **Default:** false

### 9. **Org Type Selection**
**Prompt:** "Do you want to use a scratch org or a tracked sandbox org?"
- **Options depend on `allowedOrgTypes` config:**
  - `🌎 Sandbox org with source tracking` (value: `sandbox`) - "Release manager told me that I can work on Sandboxes on my project so let's use fresh dedicated one"
  - `🪐 Scratch org` (value: `scratch`) - "Scratch orgs are configured on my project so I want to create or reuse one"
  - `😎 Current org [URL]` (value: `currentOrg`) - "Your default org with username [USERNAME] is already the org you want to use to work :)" (only if target-org flag is set)
  - `🤠 I'm hardcore, I don't need an org!` (value: `noOrg`) - "You just want to play with XML and sfdx-hardis configuration, and you know what you are doing!"

## Scratch Org Path (if scratch selected)

### 10a. **Scratch Org Selection**
**Prompt:** "Please select a scratch org to use for your branch [BRANCH_NAME]"
- **Options:**
  - `🆕 Create new scratch org` (value: `newScratchOrg`) - "This will generate a new scratch org, and in a few minutes you'll be ready to work"
  - `♻️ Reuse current org` (if current org exists) - "This will reuse current org [URL]. Beware of conflicts if others merged merge requests :)"
  - List of existing scratch orgs: `☁️ Reuse scratch org [ALIAS]`

### 11a. **Scratch Org Creation/Selection**
- If "new scratch org": Creates new scratch org via `ScratchCreate.run()`
- If existing org: Sets as default and opens in browser

## Sandbox Path (if sandbox or currentOrg selected)

### 10b. **Sandbox Selection** (if sandbox selected)
**Prompt:** "Please select a sandbox org to use for your branch [BRANCH_NAME] (if you want to avoid conflicts, you should often refresh your sandbox)"
- **Options:**
  - `🌐 Connect to a sandbox not appearing in this list` (value: `connectSandbox`) - "Login in web browser to your source-tracked sandbox"
  - List of existing sandboxes: `☁️ Use sandbox org [USERNAME]`

### 11b. **Sandbox Initialization** (if not sharedDevSandboxes)
**Prompt:** "Do you want to update the sandbox according to git branch '[TARGET_BRANCH]' current state ? (packages,SOURCES,permission set assignments,apex scripts,initial data)"
- **Options:**
  - `🧑‍🤝‍🧑 No, continue working on my current sandbox state` (value: `no`) - "Use if you are multiple users in the same SB, or have have uncommitted changes in your sandbox"
  - `☢️ Yes, please try to update my sandbox !` (value: `init`) - "Integrate new updates from the parent branch '[TARGET_BRANCH]' before working on your new task. WARNING: Will overwrite uncommitted changes in your org!"

### 12b. **Confirmation for Sandbox Update** (if init selected)
**Prompt:** "Are you really sure you want to update the dev sandbox with the state of git branch [TARGET_BRANCH]? This will overwrite setup updates that you or other users have not committed yet"
- **Type:** Confirmation (yes/no)
- **Action:** If confirmed, performs full sandbox initialization

## Final Steps
- Opens selected org in browser (if applicable)
- Sends WebSocket refresh message to VS Code
- Displays success message: "You are now ready to work in branch [BRANCH_NAME] :)"

## Configuration Options in .sfdx-hardis.yml

### Branch and Project Configuration
```yaml
# Custom branch prefix choices
branchPrefixChoices:
  - title: "🏗️ Feature"
    value: "features"
    description: "New feature development"

# Multiple target branches
availableTargetBranches:
  - integration
  - preprod

# Project selection
availableProjects:
  - build
  - run
  - france
  - uk

# Task name validation
newTaskNameRegex: '^[A-Z]+-[0-9]+ .*'
newTaskNameRegexExample: 'MYPROJECT-123 Update account status validation rule'

# Org type restrictions
allowedOrgTypes:
  - scratch
  - sandbox

# Shared sandbox configuration
sharedDevSandboxes: true
```

### Initialization Configuration
```yaml
# Package installation
installedPackages:
  - packageId: "04t..."
    packageName: "My Package"

# Permission set assignments
initPermissionSets:
  - "MyPermissionSet"

# Apex initialization scripts
scratchOrgInitApexScripts:
  - "MyInitScript.apex"

# Data packages
dataPackages:
  - "Account"
  - "Contact"
```

## Branch Naming Convention
Final branch name format: `[project/]taskType/sourceType/taskName`

Examples:
- `features/dev/PROJ-123-new-validation-rule`
- `france/fixes/config/TICKET-456-fix-workflow`
- `build/features/dev/STORY-789-implement-api`
