# SFDX-Hardis POC Installation & Testing Plan

## Overview
This POC will demonstrate the integration of sfdx-hardis (an open-source Salesforce CI/CD toolbox that orchestrates base commands and assists users with interactive wizards) with GitHub Enterprise, Jira enterprise, and Grafana for monitoring.

## Prerequisites
- GitHub Enterprise instance with admin access
- Jira Enterprise instance with admin access
- Docker environment for Grafana installation
- Salesforce org(s) for testing
- Visual Studio Code
- Node.js (latest LTS version)
- Git access to repositories

## Phase 1: Core SFDX-Hardis Setup (Estimated: 2-4 hours)

### 1.1 Environment Preparation (30 minutes)
1. **Install Salesforce CLI (SF CLI v2)**
   ```bash
   npm install -g @salesforce/cli
   ```

2. **Install sfdx-hardis plugin**
   ```bash
   echo y | sf plugins install sfdx-hardis
   ```

3. **Install Visual Studio Code Extension**
   - Install "SFDX Hardis by Cloudity" from VS Code marketplace
   - Follow prompted installation messages for additional applications and settings

### 1.2 Project Initialization (1-2 hours)
1. **Create or configure existing Salesforce DX project**
   ```bash
   sf project generate --name sfdx-hardis-poc
   cd sfdx-hardis-poc
   ```

2. **Initialize sfdx-hardis configuration**
   - Create `.sfdx-hardis.yml` configuration file
   - Configure org connections (scratch orgs, sandboxes, production)
   - Set up authentication to Salesforce orgs

3. **Verify installation**
   - Test basic sfdx-hardis commands
   - Ensure VS Code extension is working properly

### 1.3 Basic CI/CD Setup (1-2 hours)
1. **Configure branch strategy**
   - Set up development, integration, UAT, and production branches
   - Define org mappings for each branch

2. **Test basic deployment workflow**
   - Create sample metadata changes
   - Test deployment to scratch org
   - Verify CI/CD pipeline basics

## Phase 2: GitHub Enterprise Integration (Estimated: 2-3 hours)

### 2.1 Repository Setup (30 minutes)
1. **Create repository in GitHub Enterprise**
   - Initialize with sfdx-hardis project structure
   - Configure branch protection rules
   - Set up required status checks

### 2.2 CI/CD Pipeline Configuration (1.5-2 hours)
1. **Create GitHub Actions workflow**
   ```yaml
   # .github/workflows/sfdx-hardis.yml
   name: SFDX-Hardis CI/CD
   on:
     pull_request:
       branches: [main, integration]
     push:
       branches: [main, integration]
   
   jobs:
     sfdx-hardis:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Setup Node.js
           uses: actions/setup-node@v3
           with:
             node-version: '18'
         - name: Install SFDX-Hardis
           run: |
             echo y | sf plugins install sfdx-hardis
         - name: Run SFDX-Hardis Deploy
           run: sf hardis:project:deploy:start
           env:
             GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
   ```

2. **Configure GitHub Integration**
   - Set up permissions for pull-requests: write in workflow
   - Configure GITHUB_TOKEN environment variable
   - Enable post-deployment status comments on Pull Requests

### 2.3 Testing GitHub Integration (1 hour)
1. **Create test Pull Request**
   - Make metadata changes
   - Verify automated deployment checks
   - Confirm status comments appear in PR
   - Test Quick Deploy functionality

## Phase 3: Jira Enterprise Integration (Estimated: 1-2 hours)

### 3.1 Jira Configuration (30 minutes)
1. **Generate Jira API credentials**
   - Create API token for service account
   - Configure proper permissions for ticket access

2. **Configure Jira integration variables**
   ```yaml
   # In CI/CD environment variables
   JIRA_HOST: https://your-enterprise-jira.com
   JIRA_USERNAME: service-account@company.com
   JIRA_TOKEN: your-api-token
   JIRA_TICKET_REGEX: "(PROJ-[0-9]+)"  # Customize for your project
   ```

### 3.2 Integration Setup (1 hour)
1. **Configure ticket detection**
   - Set up automatic analysis of commits and PR/MR descriptions to collect JIRA ticket URLs
   - Configure JIRA_TICKET_REGEX for proper ticket identification
   - Use full URLs format: https://your-jira.com/browse/TICKET-123

2. **Configure deployment tagging**
   - Set up automatic comments and tags on JIRA tickets when deployed to major orgs
   - Configure DEPLOYED_TAG_TEMPLATE environment variable

### 3.3 Testing Jira Integration (30 minutes)
1. **Test ticket detection**
   - Create commits with Jira ticket references
   - Verify tickets appear in PR comments
   - Test deployment notifications to Jira tickets

## Phase 4: Grafana Open Source Docker Installation (Estimated: 2-3 hours)

### 4.1 Grafana Docker Setup (1 hour)
1. **Install Grafana with Docker Compose**
   ```yaml
   # docker-compose.yml
   version: '3.8'
   services:
     grafana:
       image: grafana/grafana-oss:latest
       container_name: grafana-sfdx-hardis
       restart: unless-stopped
       ports:
         - "3000:3000"
       environment:
         - GF_SECURITY_ADMIN_PASSWORD=admin123
         - GF_USERS_ALLOW_SIGN_UP=false
       volumes:
         - grafana-storage:/var/lib/grafana
         - ./grafana/provisioning:/etc/grafana/provisioning
       networks:
         - monitoring
   
     loki:
       image: grafana/loki:latest
       container_name: loki
       ports:
         - "3100:3100"
       command: -config.file=/etc/loki/local-config.yaml
       networks:
         - monitoring
   
     prometheus:
       image: prom/prometheus:latest
       container_name: prometheus
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus:/etc/prometheus
       networks:
         - monitoring
   
   volumes:
     grafana-storage:
   
   networks:
     monitoring:
       driver: bridge
   ```

2. **Start the stack**
   ```bash
   docker-compose up -d
   ```

### 4.2 Grafana Configuration (1-1.5 hours)
1. **Access Grafana**
   - Navigate to http://localhost:3000
   - Login with admin/admin123
   - Change default password

2. **Configure Data Sources**
   - Add Loki data source (http://loki:3100)
   - Add Prometheus data source (http://prometheus:9090)
   - Test connections

3. **Import sfdx-hardis Dashboards**
   - Download all sfdx-hardis Dashboard JSON files from the sfdx-hardis repo folder
   - Go to Dashboards menu, click New, then New folder
   - Upload Dashboard JSON files for each downloaded dashboard
   - Select appropriate Loki or Prometheus data sources

### 4.3 SFDX-Hardis Monitoring Integration (30-60 minutes)
1. **Configure API notifications**
   ```yaml
   # Environment variables for sfdx-hardis
   NOTIF_API_URL: http://localhost:3100/loki/api/v1/push
   NOTIF_API_BASIC_AUTH_USERNAME: ""  # if authentication required
   NOTIF_API_BASIC_AUTH_PASSWORD: ""
   NOTIF_API_METRICS_URL: http://localhost:9090/api/v1/write
   NOTIF_API_DEBUG: true  # for troubleshooting
   ```

2. **Set up monitoring jobs**
   - Configure daily metadata backup and monitoring
   - Set up Apex tests, code quality, and legacy API checks
   - Configure notification channels

## Phase 5: Integration Testing & Validation (Estimated: 2-3 hours)

### 5.1 End-to-End Workflow Testing (1.5-2 hours)
1. **Create test deployment scenario**
   - Create feature branch with metadata changes
   - Include Jira ticket references in commits
   - Create Pull Request to integration branch

2. **Verify integrations**
   - GitHub: Check PR status comments and deployment results
   - Jira: Verify ticket linking and deployment notifications
   - Grafana: Monitor deployment metrics and logs

3. **Test monitoring capabilities**
   - Trigger metadata backup job
   - Monitor org health checks
   - Verify dashboard data population

### 5.2 Dashboard Configuration (30-60 minutes)
1. **Customize Grafana dashboards**
   - Configure org-specific metrics
   - Set up alerting rules
   - Create custom panels for business metrics

2. **Test alerting**
   - Configure notification channels (email, Slack, etc.)
   - Test alert triggers
   - Verify notification delivery

## Phase 6: Documentation & Handover (Estimated: 1-2 hours)

### 6.1 Documentation Creation (1 hour)
1. **Create setup documentation**
   - Environment variables reference
   - Deployment procedures
   - Troubleshooting guide

2. **Create user guides**
   - Developer workflow documentation
   - Dashboard usage instructions
   - Monitoring procedures

### 6.2 Team Training (1 hour)
1. **Demonstrate workflows**
   - Show development process
   - Explain monitoring dashboards
   - Cover troubleshooting procedures

## Total Estimated Time: 10-17 hours

### Time Breakdown by Role:
- **DevOps Engineer**: 8-12 hours (majority of setup)
- **Salesforce Developer**: 2-3 hours (testing and validation)
- **System Administrator**: 2-4 hours (integrations and permissions)

## Key Success Criteria

1. **Functional CI/CD Pipeline**
   - Automated deployments via GitHub Actions
   - Pull Request status updates
   - Integration with branch strategy

2. **Jira Integration**
   - Automatic ticket detection in commits/PRs
   - Deployment notifications to Jira tickets
   - Proper ticket linking

3. **Monitoring & Observability**
   - Grafana dashboards showing org health
   - Automated backup and monitoring jobs
   - Alert notifications for issues

4. **User Experience**
   - VS Code extension working properly
   - Clear deployment feedback
   - Accessible monitoring dashboards

## Risk Mitigation

- **Backup Plan**: Keep existing deployment process during POC
- **Rollback Strategy**: Document how to disable integrations
- **Security**: Use service accounts with minimal required permissions
- **Testing**: Use non-production orgs for initial testing

## Next Steps Post-POC

1. Production deployment planning
2. Team training and adoption
3. Custom dashboard development
4. Advanced monitoring configuration
5. Integration with additional tools (Slack, MS Teams)

---

*This POC plan provides a comprehensive approach to evaluating sfdx-hardis with enterprise integrations while maintaining security and operational best practices.*
