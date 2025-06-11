Perfect! Now I have the actual VSCode extension code that shows the complete command structure. This gives us the real implementation details. Here's the refined breakdown with the actual SFDX Hardis commands from the source code:

I'll extract all the tooltips from the sfdx-hardis extension code for you.I've extracted all the tooltips from the SFDX Hardis extension code and organized them by category. The document shows each command with its tooltip description, providing a comprehensive overview of what each command does.

The tooltips are organized into the following categories:
- **CI/CD (simple)** - Basic development workflow commands
- **CI/CD (advanced)** - More advanced deployment and configuration commands  
- **CI/CD (misc)** - Miscellaneous CI/CD utilities
- **Data Import/Export** - Commands for managing data with SFDMU
- **Files Import/Export** - Commands for managing file transfers
- **Debugger** - Apex debugging and logging utilities
- **Org Operations** - Commands for managing org users and flows
- **Org Monitoring** - Commands for monitoring org health and compliance
- **Metadata Analysis** - Commands for analyzing project metadata
- **Setup Configuration** - Commands for configuring authentication and monitoring
- **Packaging** - Commands for managing Salesforce packages
- **Documentation Generation** - Commands for generating project documentation
- **Nerdy stuff** - Advanced technical commands for package management and metadata retrieval

Each tooltip provides clear guidance on what the command does and when to use it, making it easier for developers to understand the purpose of each SFDX Hardis command.

## **CI/CD (Simple)**
*Basic development workflow commands*
- `sf hardis:work:new` - Start a new task (creates git branch + org)
- `sf hardis:scratch:pull` - Pull from SF Org to local Git files
- `sf hardis:work:save` - Save/Publish current task (creates PR)
- `sf hardis:work:resetselection` - Reset selected list of items to merge

## **CI/CD (Advanced)**
*Advanced deployment and package management*
- `sf hardis:scratch:push` - Push from local files to Salesforce org
- `sf hardis:package:install` - Install a package
- `sf hardis:org:retrieve:packageconfig` - Retrieve packages configuration from org
- `sf hardis:project:clean:references` - Clean SFDX project sources
- `sf project delete tracking` - Clear local sfdx tracking files

## **CI/CD (Misc)**
*Additional CI/CD utilities*
- `sf hardis:scratch:create` - Create scratch org (or resume creation)
- `sf hardis:scratch:create --forcenew` - Create scratch org (force new)
- `sf org generate password` - Generate new password
- `sf hardis:org:connect` - Connect to a Salesforce org
- `sf hardis:source:retrieve` - Select and retrieve sfdx sources from org
- `sf hardis:org:retrieve:sources:analytics` - Retrieve all CRM analytics sources

## **Data Import/Export**
*SFDMU-based data operations*
- `sf hardis:org:data:export` - Export data from org with SFDMU
- `sf hardis:org:data:import` - Import data to org with SFDMU
- `sf hardis:org:data:delete` - Delete data from org with SFDMU
- `sf hardis:org:configure:data` - Create data import/export configuration
- `sf hardis:org:multi-org-query` - Multi-org SOQL Query & Report

## **Files Import/Export**
*File management operations*
- `sf hardis:org:files:export` - Export files from org
- `sf hardis:org:files:import` - Import files into org
- `sf hardis:org:configure:files` - Create files export configuration

## **Debugger**
*Debug and logging tools*
- `vscode-sfdx-hardis.debug.launch` - Run debugger (VSCode command)
- `vscode-sfdx-hardis.debug.activate` - Activate debug logs tracing
- `sf hardis:org:purge:apexlog` - Purge Apex Logs
- `vscode-sfdx-hardis.debug.logtail` - Display live logs in terminal

## **Org Operations**
*User and org management*
- `sf hardis:org:user:freeze` - Freeze users (for safe deployment)
- `sf hardis:org:user:unfreeze` - Unfreeze users
- `sf hardis:org:purge:flow` - Purge obsolete flows versions
- `sf hardis:scratch:delete` - Delete scratch org(s)
- `sf hardis:org:user:activateinvalid` - Activate .invalid user emails in sandbox

## **Org Monitoring**
*Health checks and diagnostics*
- `sf hardis:org:monitor:backup` - Retrieve all metadatas
- `sf hardis:org:diagnose:audittrail` - Suspicious Audit Trail Activities
- `sf hardis:org:test:apex` - Run Apex tests
- `sf hardis:org:monitor:limits` - Check Org Limits
- `sf hardis:org:diagnose:legacyapi` - Legacy API versions usage
- `sf hardis:org:diagnose:unusedusers` - Unused Users
- `sf hardis:lint:access` - Metadata without access

## **Metadata Analysis**
*Code quality and analysis tools*
- `sf hardis:project:audit:duplicatefiles` - Detect duplicate sfdx files
- `sf hardis:project:metadata:findduplicates` - Detect duplicate values in metadata
- `sf hardis:project:audit:apiversion` - Extract API versions of sources
- `sf hardis:project:audit:callincallout` - List call'in and call'outs
- `sf hardis:misc:custom-label-translations` - Extract Custom Label Translations

## **Setup Configuration**
*Project and authentication setup*
- `sf hardis:project:configure:auth` - Configure Org CI authentication
- `sf hardis:project:configure:auth --devhub` - Configure DevHub CI authentication
- `sf hardis:org:configure:monitoring` - Configure Org Monitoring
- `sf hardis:scratch:pool:create` - Configure scratch orgs pool
- `sf hardis:project:create` - Create a new SFDX project

## **Packaging**
*2GP package management*
- `sf hardis:package:create` - Create a new package
- `sf hardis:package:version:list` - List package versions
- `sf hardis:package:version:create` - Create a new package version

## **Documentation Generation**
*Generate project documentation*
- `sf hardis:doc:project2markdown` - Project Documentation
- `sf hardis:doc:project2markdown --pdf` - Project Documentation + PDF
- `sf hardis:doc:flow2markdown` - Flows Documentation
- `sf hardis:project:generate:flow-git-diff` - Single Flow Visual Git Diff

## **Nerdy Stuff**
*Advanced technical operations*
- `sf hardis:project:generate:gitdelta` - Generate package.xml git delta
- `sf hardis:org:generate:packagexmlfull` - Generate org full package.xml
- `sf hardis:org:retrieve:sources:dx` - Retrieve ALL DX sources from an org
- `sf hardis:org:retrieve:sources:metadata` - Retrieve ALL Metadata sources
- `sf hardis:package:mergexml` - Merge package.xml files

## **Help**
*Documentation and support*
- Open external URLs for documentation, GitHub issues, and contact forms
- Links to CI/CD docs, org monitoring docs, and command documentation

This structure shows how the VSCode extension organizes all commands into logical groups, with some commands being VSCode-specific (`vscode-sfdx-hardis.*`) while most are direct SFDX Hardis CLI commands (`sf hardis:*`).

# SFDX Hardis Extension - Command Tooltips

## CI/CD (simple)

**Start a new task** (`sf hardis:work:new`)
- Start to work, it will:
  - Create a new git branch where it is needed to 
  - Allow to select or create a Salesforce org to work in (sandbox or scratch)

**Pull from SF Org to local Git files** (`sf hardis:scratch:pull`)
- Once you have made your configuration in your org Setup, click here to download your updates.
  Then, you can commit the updates you want to publish (use VsCode Git extension)
  Then you can run command "Save / Publish my current task"
  Note: if you don't see all your updates, you can manually retrieve it using VsCode Extension "Org Browser"(Salesforce logo)

**Save / Publish my current task** (`sf hardis:work:save`)
- Once you performed your commit(s), click here to prepare your Pull Request. It will:
  - Automatically update package.xml and destructiveChanges.xml
  - Clean metadatas before publishing them (for example remove flow positions or remove object & field access from Profiles)

**Reset selected list of items to merge** (`sf hardis:work:resetselection`)
- You already pushed a commit but you selected updates that you do not want to deploy ?
  In that case, click here and you'll be able to select again what you want to commit.
  After creating new commits, click on "Save / Publish my current task"

## CI/CD (advanced)

**Push from local files to Salesforce org** (`sf hardis:scratch:push`)
- Propagates your local updates within Vs Code into your remote Salesforce scratch org

**Install a package** (`sf hardis:package:install`)
- This will update project .sfdx-hardis.yml so the package will always be installed in new scratch orgs and future deployments

**Retrieve packages configuration from org** (`sf hardis:org:retrieve:packageconfig`)
- Retrieve package configuration from an org and propose to update project sfdx-hardis configuration

**Clean SFDX project sources** (`sf hardis:project:clean:references`)
- Select and apply lots of cleaning commands provided by sfdx-hardis

**Clear local sfdx tracking files** (`sf project delete tracking`)
- Removes all local information about updates you already pulled from org

## CI/CD (misc)

**Create scratch org (or resume creation)** (`sf hardis:scratch:create`)
- If during Work:New you had an error, you can resume the scratch org creation

**Create scratch org (force new)** (`sf hardis:scratch:create --forcenew`)
- Create a new scratch org for the current work

**Generate new password** (`sf org generate password`)
- Generates a new password for your current scratch org user

**Connect to a Salesforce org** (`sf hardis:org:connect`)
- Connects to a Salesforce org without setting it as defaultusername

**Select and retrieve sfdx sources from org** (`sf hardis:source:retrieve`)
- Allows user to select a list of metadata types and process the retrieve from an org

**Retrieve all CRM analytics sources** (`sf hardis:org:retrieve:sources:analytics`)
- Allows user to select a list of metadata types and process the retrieve from an org

**Clear local and remote sfdx tracking files** (`sf project reset tracking`)
- Removes all local and remote information about updates you already pulled from org

**Logout from current Org and DevHub**
- Log out from orgs :)

**Login again to git**
- Use this command in case you have git login errors

**Extract pull requests** (`sf hardis:git:pull-requests:extract`)
- Extract Pull Requests and associated ticketing system references in a CSV / Excel file

**Generate bypass custom permissions and permission sets** (`sf hardis:project:generate:bypass`)
- Generates bypass custom permissions and permission sets for specified sObjects and automations (Flows, Triggers, and Validation Rules)

## Data Import/Export

**Export data from org with SFDMU** (`sf hardis:org:data:export`)
- Export data from org and store it in project files, so it can be imported during each scratch org initialization or deployment to org

**Import data to org with SFDMU** (`sf hardis:org:data:import`)
- Import data into org from project files

**Delete data from org with SFDMU** (`sf hardis:org:data:delete`)
- Delete data from org using SFDMU config files

**Create data import/export configuration** (`sf hardis:org:configure:data`)
- Initializes a new SFDMU project

**Multi-org SOQL Query & Report** (`sf hardis:org:multi-org-query`)
- Executes a SOQL query in multiple orgs and generate a single report from it

## Files Import/Export

**Export files from org** (`sf hardis:org:files:export`)
- Export files from org based on a configuration

**Import files into org** (`sf hardis:org:files:import`)
- Import files into org based on a configuration

**Create files export configuration** (`sf hardis:org:configure:files`)
- Initializes a new file export project

## Debugger

**Run debugger** (VS Code command)
- Run debugger on an apex log file

**Activate debug logs tracing** (VS Code command)
- Activate tracing of logs to use the local replay debugger

**Deactivate debug logs tracing** (VS Code command)
- Deactivate tracing of logs to use the local replay debugger

**Purge Apex Logs** (`sf hardis:org:purge:apexlog`)
- Purge all apex logs of default org

**Display live logs in terminal** (VS Code command)
- Display apex logs in console while they are generated

**Retrieve Apex sources from org**
- Retrieve sources from your org so you can use the replay debugger

## Org Operations

**Freeze users** (`sf hardis:org:user:freeze`)
- Freeze all users of an org except admins to deploy safely

**Unfreeze users** (`sf hardis:org:user:unfreeze`)
- Unfreeze all users of an org after a safe deployment

**Purge obsolete flows versions** (`sf hardis:org:purge:flow`)
- Purge all flows with status Obsolete in your org, so you are not bothered by the 50 versions limits

**Delete scratch org(s)** (`sf hardis:scratch:delete`)
- Prompts user for scratch orgs to mark for deletion

**Activate .invalid user emails in sandbox** (`sf hardis:org:user:activateinvalid`)
- Removes the .invalid of all users emails in a sandbox

## Org Monitoring

**Retrieve all metadatas** (`sf hardis:org:monitor:backup`)
- Retrieves all relevant Metadata of an org

**Suspicious Audit Trail Activities** (`sf hardis:org:diagnose:audittrail`)
- Detect setup actions in major orgs that are identified as Suspicious

**Run Apex tests** (`sf hardis:org:test:apex`)
- Runs all apex tests on the selected org. Will trigger error if minimum apex code coverage is not reached

**Check Org Limits** (`sf hardis:org:monitor:limits`)
- Checks if limits are reached or soon reached in the default Salesforce org

**Check Release Updates** (`sf hardis:org:diagnose:releaseupdates`)
- Checks if some Release Updates must be verified in the org

**Legacy API versions usage** (`sf hardis:org:diagnose:legacyapi`)
- Detects if deprected APIs are your in a production org

**Unused Users** (`sf hardis:org:diagnose:unusedusers`)
- Identify active users who haven't logged in recently to the org

**Unused PS Licenses (beta)** (`sf hardis:org:diagnose:unusedlicenses`)
- Detects if there are unused permission set licenses in the org, and offers to delete them

**Unused Apex Classes** (`sf hardis:org:diagnose:unused-apex-classes`)
- List all async Apex classes (Batch,Queueable,Schedulable) that has not been called for more than 365 days, and could probably be deleted

**Metadata without access** (`sf hardis:lint:access`)
- Detects if custom fields or apex classes are existing in source but not authorized on any Profile or Permission Set

**Unused Metadatas** (`sf hardis:lint:unusedmetadatas`)
- Check if elements (custom labels and custom permissions) are not used in the project

**Inactive Metadatas** (`sf hardis:lint:metadatastatus`)
- Check if flows or validation rules are inactive, so should be deleted

**Missing descriptions** (`sf hardis:lint:missingattributes`)
- Check if metadatas have missing descriptions

## Metadata Analysis

**Detect duplicate sfdx files** (`sf hardis:project:audit:duplicatefiles`)
- Detects if duplicate files are within in your SFDX project

**Detect duplicate values in metadata files** (`sf hardis:project:metadata:findduplicates`)
- Detects if duplicate values for given keys are within in your SFDX metadata files

**Extract API versions of sources** (`sf hardis:project:audit:apiversion`)
- Browse all project files and summarize API versions of elements

**List call'in and call'outs** (`sf hardis:project:audit:callincallout`)
- Browse sources to list inbound and outbound calls

**List remote sites** (`sf hardis:project:audit:remotesites`)
- Browse sources to list remote sites

**Extract Custom Label Translations** (`sf hardis:misc:custom-label-translations`)
- Extract selected custom labels, or of a given Lightning Web Component (LWC), from all language translation files

## Setup Configuration

**Configure Org CI authentication** (`sf hardis:project:configure:auth`)
- Assisted configuration to connect a protected branch and its related release org during CI

**Configure DevHub CI authentication** (`sf hardis:project:configure:auth --devhub`)
- Assisted configuration to connect to a Dev Hub org during CI

**Configure Org Monitoring** (`sf hardis:org:configure:monitoring`)
- To run only on a repo dedicated to monitoring (start from a blank repo)

**Configure scratch orgs pool** (`sf hardis:scratch:pool:create`)
- Define a scratch org pool to have scratch orgs ready to be used for development or CI

**Create a new SFDX project** (`sf hardis:project:create`)
- Create and initialize a new SFDX project

## Packaging

**Create a new package** (`sf hardis:package:create`)
- Second generation packages, unlocked or managed

**List package versions** (`sf hardis:package:version:list`)
- List all package versions associated to Dev Hub org

**Create a new package version** (`sf hardis:package:version:create`)
- Create a new version of a package

## Documentation Generation

**Project Documentation** (`sf hardis:doc:project2markdown`)
- Generates markdown pages with SF Project content: List of metadatas, installed packages...

**Project Documentation + PDF** (`sf hardis:doc:project2markdown --pdf`)
- Generates markdown pages with SF Project content: List of metadatas, installed packages... + PDF files

**Project Documentation (with history)** (`sf hardis:doc:project2markdown --with-history`)
- Generates markdown pages with SF Project content: List of metadatas, installed packages..., with Flow Diff History

**Run Local HTML Doc Pages**
- Run Documentation local web server, then open http://127.0.0.1:8000/ . You need Python on your computer :)

**Upload HTML Doc to Salesforce** (`sf hardis:doc:mkdocs-to-salesforce`)
- Generates HTML Doc and Uploads it to Salesforce as a static resource with a VfPage & a CustomTab. You need Python on your computer :)

**Flows Documentation** (`sf hardis:doc:flow2markdown`)
- Generates Visual Documentation for a Flow

**Flows Documentation + PDF** (`sf hardis:doc:flow2markdown --pdf`)
- Generates Visual Documentation for a Flow + PDF file

**Single Flow Visual Git Diff** (`sf hardis:project:generate:flow-git-diff`)
- Generates Visual Documentation of the differences between versions of a Flow

## Nerdy stuff

**Generate package.xml git delta** (`sf hardis:project:generate:gitdelta`)
- Generate package.xml & destructiveChanges.xml using git delta between 2 commit hashes

**Generate org full package.xml** (`sf hardis:org:generate:packagexmlfull`)
- Generate full package.xml from any org

**Retrieve DX sources from an org (package.xml)** (`sf hardis:org:retrieve:sources:dx2`)
- Retrieve locally the SFDX sources of an org, using a package.xml

**Retrieve ALL DX sources from an org** (`sf hardis:org:retrieve:sources:dx`)
- Retrieve locally all the metadatas of a remote salesforce org, in DX project format

**Retrieve ALL Metadata sources from an org** (`sf hardis:org:retrieve:sources:metadata`)
- Retrieve locally all the metadatas of a remote salesforce org, in metadata format

**Merge package.xml files** (`sf hardis:package:mergexml`)
- Merge package.xml files located in manifest folder
