# WebSocket Flow - Shows the interactive communication between User, VSCode, CLI, and WebSocket Server

```mermaid
sequenceDiagram
    participant User
    participant VSCode
    participant "sfdx-hardis CLI" as SFDXHardisCLI
    participant WebSocketServer
    
    User->>VSCode: Interacts with UI (e.g., clicks command)
    VSCode->>WebSocketServer: Ensure WebSocket server is active
    VSCode->>SFDXHardisCLI: Invoke CLI command with --websocket host:port
    SFDXHardisCLI->>WebSocketServer: Connect as WebSocket client
    activate SFDXHardisCLI
    activate WebSocketServer
    WebSocketServer-->>SFDXHardisCLI: Connection established
    loop Real-time Interaction
        SFDXHardisCLI->>WebSocketServer: Send status/progress/prompt
        WebSocketServer->>VSCode: Forward message to UI
        VSCode->>User: Display update/prompt
        alt User Input Required
            User->>VSCode: Provides input
            VSCode->>WebSocketServer: Send user input
            WebSocketServer->>SFDXHardisCLI: Forward user input
        end
    end
    SFDXHardisCLI->>WebSocketServer: Send completion/error
    WebSocketServer->>VSCode: Forward final message
    VSCode->>User: Display final status
    deactivate WebSocketServer
    deactivate SFDXHardisCLI
```

# CI/CD Flow - Shows the GitHub Actions deployment and validation process

```mermaid
sequenceDiagram
    participant Developer
    participant GitHubRepo
    participant GitHubActions
    participant SalesforceOrg
    
    Developer->>GitHubRepo: Push code / Create Pull Request
    GitHubRepo->>GitHubActions: Trigger workflow (on push/PR)
    activate GitHubActions
    GitHubActions->>GitHubActions: 1. Checkout code (fetch-depth: 0)
    GitHubActions->>GitHubActions: 2. Setup Node.js
    GitHubActions->>GitHubActions: 3. Install Salesforce CLI & sfdx-hardis plugin
    GitHubActions->>SalesforceOrg: 4. Authenticate (JWT Bearer Flow using GitHub Secrets)
    activate SalesforceOrg
    alt Pull Request Validation
        GitHubActions->>SalesforceOrg: 5. Run `sf hardis:project:deploy:smart --check` (Validate metadata, run tests)
        SalesforceOrg-->>GitHubActions: Validation result
        GitHubActions->>GitHubRepo: 6. Post validation status to PR
    else Deployment (on merge to main)
        GitHubActions->>SalesforceOrg: 5. Run `sf hardis:project:deploy:smart` (Deploy metadata)
        SalesforceOrg-->>GitHubActions: Deployment result
        GitHubActions->>GitHubRepo: 6. Post deployment status
    end
    deactivate SalesforceOrg
    deactivate GitHubActions
```

# Monitoring Flow - Shows the scheduled org monitoring via GitHub Actions

```mermaid
sequenceDiagram
    participant GitHubActions
    participant SalesforceOrg
    participant GitHubRepo
    
    Note over GitHubActions: Triggered by CRON schedule
    activate GitHubActions
    GitHubActions->>GitHubActions: 1. Checkout code
    GitHubActions->>GitHubActions: 2. Setup Node.js
    GitHubActions->>GitHubActions: 3. Install Salesforce CLI & sfdx-hardis
    GitHubActions->>SalesforceOrg: 4. Authenticate to Monitored Org (JWT using GitHub Secrets)
    activate SalesforceOrg
    GitHubActions->>SalesforceOrg: 5. Run `sf hardis:org:monitor:backup` or `sf hardis:org:monitor:all`
    SalesforceOrg-->>GitHubActions: Backup data/status
    alt Optional: Commit backup to repo
        GitHubActions->>GitHubRepo: Commit metadata backup
    end
    deactivate SalesforceOrg
    deactivate GitHubActions
```
