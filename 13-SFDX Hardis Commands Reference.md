# SFDX Hardis Commands Reference

SFDX Hardis is a comprehensive Salesforce DevOps toolkit that provides over 100 specialized commands organized into 17 main categories. This plugin serves as a "Swiss-army-knife toolbox" for Salesforce development, automating complex CI/CD operations and providing intelligent workflows that would typically require extensive manual work.

## Authentication and Access Management

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:auth:*` | Authentication | Manage Salesforce org connections and authentication tokens | Handle org authentication and user access management |

## Configuration and Setup

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:config:*` | Configuration | Manage project and org configuration settings | Configure SFDX Hardis behaviors and project parameters |
| `hardis:project:create` | Project Setup | Create new SFDX Hardis projects with CI/CD pipelines | Initialize new Salesforce projects with complete DevOps setup |
| `hardis:project:configure:auth` | Project Config | Configure project authentication settings | Set up authentication for project environments |

## Source Code Management

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:source:push` | Source Control | Push source changes to orgs | Deploy local changes to target org |
| `hardis:source:retrieve` | Source Control | Retrieve source changes from orgs | Pull latest changes from org to local project |
| `hardis:git:*` | Git Integration | Integrate Git operations with Salesforce workflow | Manage Git repositories and branching strategies |
| `hardis:org:retrieve:sources:dx` | Source Retrieval | Retrieve sources in DX format | Extract metadata in modern DX format |
| `hardis:org:retrieve:sources:metadata` | Source Retrieval | Retrieve sources in metadata format | Extract metadata in traditional format |

## Deployment and Validation

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:deploy:start` | Deployment | Initiate deployment processes | Start deployment to target environments |
| `hardis:deploy:validate` | Deployment | Validate deployments before execution | Check deployment validity without deploying |
| `hardis:project:deploy:smart` | Smart Deployment | Intelligent deployment with delta detection | Deploy only changed metadata for efficiency |
| `hardis:project:deploy:quick` | Fast Deployment | Quick deployment operations | Leverage Salesforce Quick Deploy functionality |
| `hardis:project:deploy:simulate` | Deployment Testing | Simulate deployments without executing | Test deployment scenarios safely |
| `hardis:project:deploy:sources:dx` | DX Deployment | Deploy DX format sources | Deploy using modern DX source format |
| `hardis:project:deploy:sources:metadata` | Metadata Deployment | Deploy metadata format sources | Deploy using traditional metadata format |
| `hardis:project:deploy:notify` | Deployment Communication | Send deployment notifications | Notify teams about deployment status |
| `hardis:project:generate:gitdelta` | Delta Generation | Generate Git delta packages | Create deployment packages with only changed items |

## Code Quality and Linting

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:lint:metadatastatus` | Quality Assurance | Check metadata status and compliance | Validate metadata against standards |
| `hardis:lint:missingattributes` | Quality Check | Identify missing metadata attributes | Find incomplete metadata definitions |
| `hardis:lint:unusedmetadatas` | Cleanup Detection | Find unused metadata components | Identify metadata that can be removed |
| `hardis:project:lint` | Project Quality | Comprehensive project linting | Run all quality checks on project |

## Project Auditing and Analysis

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:project:audit:apiversion` | Project Audit | Audit API versions in use | Ensure consistent API version usage |
| `hardis:project:audit:callincallout` | Security Audit | Audit callout configurations | Review external system connections |
| `hardis:project:audit:duplicatefiles` | File Management | Find duplicate files in project | Identify and remove duplicate resources |
| `hardis:project:audit:remotesites` | Security Audit | Audit remote site settings | Review external site access permissions |
| `hardis:project:metadata:findduplicates` | Metadata Analysis | Find duplicate metadata components | Identify redundant metadata elements |

## Project Cleaning and Optimization

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:project:clean:emptyitems` | Cleanup | Remove empty metadata items | Clean up incomplete metadata |
| `hardis:project:clean:flowpositions` | Flow Optimization | Clean flow element positions | Optimize flow layout and positioning |
| `hardis:project:clean:hiddenitems` | Cleanup | Remove hidden metadata items | Clean up inaccessible components |
| `hardis:project:clean:minimizeprofiles` | Profile Optimization | Minimize profile definitions | Remove redundant profile permissions |
| `hardis:project:clean:references` | Reference Cleanup | Clean metadata cross-references | Fix broken or unnecessary references |
| `hardis:project:clean:sensitive-metadatas` | Security Cleanup | Handle sensitive metadata | Remove or secure sensitive information |
| `hardis:project:clean:xml` | XML Optimization | General XML file cleaning | Optimize and standardize XML format |
| `hardis:project:clean:systemdebug` | Debug Cleanup | Remove system debug information | Clean up development artifacts |
| `hardis:misc:purge-references` | Reference Management | Clean up metadata references | Remove obsolete cross-references |

## Package Management

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:package:install` | Package Deployment | Install packages in target orgs | Deploy managed or unmanaged packages |
| `hardis:package:version:create` | Package Versioning | Create new package versions | Generate new version of existing package |
| `hardis:package:version:list` | Package Information | List available package versions | View package version history |
| `hardis:package:version:promote` | Package Lifecycle | Promote package versions | Move packages through release stages |
| `hardis:packagexml:remove` | Package Management | Remove items from package.xml | Edit package manifest files |
| `hardis:org:retrieve:packageconfig` | Package Config | Retrieve installed package configurations | Get current package installation details |

## Scratch Org Management

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:scratch:delete` | Scratch Org Lifecycle | Delete scratch orgs | Clean up development environments |
| `hardis:scratch:pool:create` | Pool Management | Create scratch org pools | Set up shared development environments |
| `hardis:scratch:pool:refresh` | Pool Maintenance | Refresh scratch org pools | Update pool with latest changes |
| `hardis:scratch:pool:view` | Pool Monitoring | View scratch org pool status | Monitor pool health and usage |
| `hardis:scratch:push` | Scratch Org Sync | Push changes to scratch orgs | Deploy local changes to scratch environment |
| `hardis:scratch:pull` | Scratch Org Sync | Pull changes from scratch orgs | Retrieve changes from scratch environment |

## Org Management and Monitoring

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:org:configure:monitoring` | Org Monitoring | Set up automated org monitoring | Configure health checks and alerts |
| `hardis:org:purge:flow` | Org Maintenance | Clean up obsolete flow versions | Address 50 flow version limit |
| `hardis:cache:*` | Performance | Manage caching operations | Optimize performance through caching |

## Documentation and Reporting

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:doc:*` | Documentation | Generate project documentation | Create AI-enhanced project docs |
| `hardis:project:generate:flow-git-diff` | Visual Documentation | Generate flow visual diffs for Git | Create visual flow change documentation |
| `hardis:misc:servicenow-report` | Integration Reporting | Generate ServiceNow integration reports | Report on ServiceNow system integration |

## Workflow and Workspace Management

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:work:refresh` | Workspace | Refresh work environment | Update development workspace |
| `hardis:work:save` | Workspace | Save work progress with automated cleaning | Persist work state with cleanup |
| `hardis:work:ws` | Workspace | Manage development workspace | Control workspace configuration |

## Advanced Features and Fixes

| Command | Category | Description | Primary Use Case |
|---------|----------|-------------|------------------|
| `hardis:project:convert:profilestopermsets` | Migration | Convert profiles to permission sets | Modernize security model |
| `hardis:project:fix:profiletabs` | Bug Fixes | Fix profile tab configurations | Resolve profile tab issues |
| `hardis:project:fix:v53flexipages` | Version Fixes | Fix v53 FlexiPage issues | Address specific API version problems |
| `hardis:project:generate:bypass` | Configuration | Generate bypass configurations | Create exception handling configs |
| `hardis:misc:toml2csv` | Data Conversion | Convert TOML files to CSV format | Transform configuration data |

## Key capabilities that set SFDX Hardis apart

**Smart deployment features** include delta deployments that deploy only changed metadata, intelligent test execution that skips unnecessary test runs, and automated source cleaning before deployments. The toolkit provides **comprehensive CI/CD integration** with ready-to-use pipeline configurations for major platforms, pull request integration, and flow visual diff generation for code reviews.

**Monitoring and maintenance capabilities** offer automated daily backups, configurable org health monitoring, and intelligent cleanup of unused metadata and obsolete flow versions. The **developer experience** is enhanced through Visual Studio Code integration, interactive wizards for complex operations, and automated profile optimization.

Most commands support standard parameters including `--targetusername` for org targeting, `--debug` for detailed logging, `--json` for structured output, and `--loglevel` for logging control. Configuration is managed through `.sfdx-hardis.yml` files with branch-specific overrides and extensive environment variable support for CI/CD customization.
