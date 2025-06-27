# Grafana Docker Setup for GitHub Enterprise with SFDX-Hardis Delivery Monitoring

## Overview
This document provides a comprehensive guide for setting up a custom Grafana Docker image for GitHub Enterprise with specialized monitoring capabilities for SFDX-Hardis delivery jobs. The setup includes building a custom OCI-compliant image, pushing to GitHub Container Registry, and configuring specialized dashboards for deployment monitoring.

## Prerequisites
- GitHub Enterprise instance with Container Registry enabled
- Docker installed and configured
- GitHub Personal Access Token with appropriate permissions
- Admin access to GitHub Enterprise
- Knowledge of Grafana configuration
- Understanding of SFDX-Hardis deployment pipelines

## Phase 1: Custom Grafana Image Creation (Estimated: 2-3 hours)

### 1.1 Dockerfile Creation (45 minutes)

Create a custom Dockerfile that extends the official Grafana image with enterprise-specific configurations:

```dockerfile
# Dockerfile
FROM grafana/grafana-oss:latest

# Metadata
LABEL org.opencontainers.image.source="https://github.com/your-org/grafana-sfdx-hardis"
LABEL org.opencontainers.image.description="Custom Grafana for SFDX-Hardis monitoring"
LABEL org.opencontainers.image.licenses="Apache-2.0"

# Switch to root for installation
USER root

# Install additional plugins
RUN grafana cli --pluginsDir "/var/lib/grafana/plugins" plugins install \
    grafana-piechart-panel \
    grafana-worldmap-panel \
    grafana-clock-panel \
    camptocamp-prometheus-alertmanager-datasource \
    yesoreyeram-boomtable-panel

# Create directories for custom configurations
RUN mkdir -p /etc/grafana/provisioning/dashboards \
             /etc/grafana/provisioning/datasources \
             /etc/grafana/provisioning/notifiers \
             /var/lib/grafana/dashboards/sfdx-hardis

# Copy custom configurations
COPY config/grafana.ini /etc/grafana/grafana.ini
COPY provisioning/ /etc/grafana/provisioning/
COPY dashboards/ /var/lib/grafana/dashboards/sfdx-hardis/

# Set proper permissions
RUN chown -R grafana:grafana /etc/grafana /var/lib/grafana
RUN chmod 644 /etc/grafana/grafana.ini

# Switch back to grafana user
USER grafana

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/api/health || exit 1
```

### 1.2 Grafana Configuration (30 minutes)

Create the custom Grafana configuration file:

```ini
# config/grafana.ini
[analytics]
reporting_enabled = false
check_for_updates = false

[auth]
disable_login_form = false
disable_signout_menu = false

[auth.github]
enabled = true
allow_sign_up = true
client_id = YOUR_GITHUB_CLIENT_ID
client_secret = YOUR_GITHUB_CLIENT_SECRET
scopes = user:email,read:org
auth_url = https://your-github-enterprise.com/login/oauth/authorize
token_url = https://your-github-enterprise.com/login/oauth/access_token
api_url = https://your-github-enterprise.com/api/v3/user
team_ids = 123,456
allowed_organizations = your-org

[security]
admin_user = admin
admin_password = ${GF_SECURITY_ADMIN_PASSWORD}
secret_key = ${GF_SECURITY_SECRET_KEY}
disable_gravatar = true

[server]
http_port = 3000
domain = grafana.your-company.com
root_url = https://grafana.your-company.com/
serve_from_sub_path = false

[database]
type = postgres
host = ${GF_DATABASE_HOST}
name = ${GF_DATABASE_NAME}
user = ${GF_DATABASE_USER}
password = ${GF_DATABASE_PASSWORD}

[session]
provider = postgres
provider_config = user=${GF_DATABASE_USER} password=${GF_DATABASE_PASSWORD} host=${GF_DATABASE_HOST} port=5432 dbname=${GF_DATABASE_NAME} sslmode=disable

[log]
mode = console
level = info

[alerting]
enabled = true
execute_alerts = true

[unified_alerting]
enabled = true

[feature_toggles]
enable = ngalert
```

### 1.3 Data Source Provisioning (30 minutes)

Create data source configurations for SFDX-Hardis monitoring:

```yaml
# provisioning/datasources/datasources.yml
apiVersion: 1

datasources:
  # Loki for log aggregation
  - name: Loki-SFDX-Hardis
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true
    editable: true
    jsonData:
      maxLines: 1000
      derivedFields:
        - name: "TraceID"
          matcherRegex: "trace_id=(\\w+)"
          url: "http://jaeger:16686/trace/$${__value.raw}"

  # Prometheus for metrics
  - name: Prometheus-SFDX-Hardis
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: false
    editable: true
    jsonData:
      httpMethod: POST
      queryTimeout: 60s
      timeInterval: 30s

  # GitHub API for deployment data
  - name: GitHub-Enterprise
    type: grafana-github-datasource
    access: proxy
    url: https://your-github-enterprise.com/api/v3
    jsonData:
      gitHubUrl: https://your-github-enterprise.com
    secureJsonData:
      accessToken: ${GITHUB_ACCESS_TOKEN}
```

### 1.4 Dashboard Provisioning (45 minutes)

Create dashboard provisioning configuration:

```yaml
# provisioning/dashboards/dashboards.yml
apiVersion: 1

providers:
  - name: 'SFDX-Hardis Dashboards'
    orgId: 1
    folder: 'SFDX-Hardis'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards/sfdx-hardis
```

## Phase 2: SFDX-Hardis Delivery Monitoring Dashboards (Estimated: 3-4 hours)

### 2.1 Main Delivery Dashboard (1.5 hours)

Create the primary dashboard for monitoring SFDX-Hardis deliveries:

```json
{
  "dashboard": {
    "id": null,
    "title": "SFDX-Hardis Delivery Monitoring",
    "tags": ["sfdx-hardis", "deployments", "salesforce"],
    "style": "dark",
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Deployment Success Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "rate(sfdx_hardis_deployment_success_total[24h]) / rate(sfdx_hardis_deployment_total[24h]) * 100",
            "legendFormat": "Success Rate %"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 80},
                {"color": "green", "value": 95}
              ]
            },
            "unit": "percent"
          }
        },
        "gridPos": {"h": 8, "w": 6, "x": 0, "y": 0}
      },
      {
        "id": 2,
        "title": "Recent Deployments",
        "type": "table",
        "targets": [
          {
            "expr": "sfdx_hardis_deployment_info",
            "legendFormat": "{{org}} - {{branch}}"
          }
        ],
        "transformations": [
          {
            "id": "organize",
            "options": {
              "excludeByName": {},
              "indexByName": {},
              "renameByName": {
                "org": "Org",
                "branch": "Branch",
                "status": "Status",
                "duration": "Duration",
                "timestamp": "Time"
              }
            }
          }
        ],
        "gridPos": {"h": 8, "w": 18, "x": 6, "y": 0}
      },
      {
        "id": 3,
        "title": "Deployment Duration Trends",
        "type": "timeseries",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(sfdx_hardis_deployment_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          },
          {
            "expr": "histogram_quantile(0.50, rate(sfdx_hardis_deployment_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s"
          }
        },
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 8}
      },
      {
        "id": 4,
        "title": "Failed Deployments by Org",
        "type": "piechart",
        "targets": [
          {
            "expr": "sum by (org) (rate(sfdx_hardis_deployment_failures_total[24h]))",
            "legendFormat": "{{org}}"
          }
        ],
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 8}
      }
    ],
    "time": {
      "from": "now-24h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
```

### 2.2 Org Health Dashboard (1 hour)

Create dashboard for monitoring Salesforce org health:

```json
{
  "dashboard": {
    "id": null,
    "title": "SFDX-Hardis Org Health Monitoring",
    "tags": ["sfdx-hardis", "org-health", "salesforce"],
    "panels": [
      {
        "id": 1,
        "title": "Org Backup Status",
        "type": "table",
        "targets": [
          {
            "expr": "sfdx_hardis_backup_status",
            "legendFormat": "{{org}}"
          }
        ],
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 0}
      },
      {
        "id": 2,
        "title": "Apex Test Coverage",
        "type": "gauge",
        "targets": [
          {
            "expr": "sfdx_hardis_apex_coverage_percentage",
            "legendFormat": "{{org}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "min": 0,
            "max": 100,
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 75},
                {"color": "green", "value": 85}
              ]
            },
            "unit": "percent"
          }
        },
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 8}
      },
      {
        "id": 3,
        "title": "Metadata Changes (Last 7 Days)",
        "type": "timeseries",
        "targets": [
          {
            "expr": "increase(sfdx_hardis_metadata_changes_total[1d])",
            "legendFormat": "{{org}} - {{type}}"
          }
        ],
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 8}
      }
    ]
  }
}
```

### 2.3 GitHub Actions Integration Dashboard (30 minutes)

Dashboard for monitoring GitHub Actions workflows:

```json
{
  "dashboard": {
    "id": null,
    "title": "GitHub Actions - SFDX-Hardis Workflows",
    "tags": ["github-actions", "sfdx-hardis", "ci-cd"],
    "panels": [
      {
        "id": 1,
        "title": "Workflow Run Status",
        "type": "stat",
        "targets": [
          {
            "queryType": "github",
            "query": "workflows/sfdx-hardis-ci/runs?status=completed&per_page=50"
          }
        ],
        "gridPos": {"h": 6, "w": 8, "x": 0, "y": 0}
      },
      {
        "id": 2,
        "title": "Workflow Duration Trends",
        "type": "timeseries",
        "targets": [
          {
            "queryType": "github",
            "query": "workflows/runs?event=push&status=completed"
          }
        ],
        "gridPos": {"h": 8, "w": 16, "x": 8, "y": 0}
      }
    ]
  }
}
```

## Phase 3: Container Registry Setup (Estimated: 1-2 hours)

### 3.1 GitHub Actions Workflow for Image Build (45 minutes)

Create GitHub Actions workflow to build and push the custom image:

```yaml
# .github/workflows/build-grafana-image.yml
name: Build and Push Grafana Image

on:
  push:
    branches: [main]
    paths: 
      - 'docker/grafana/**'
  pull_request:
    branches: [main]
    paths: 
      - 'docker/grafana/**'
  workflow_dispatch:

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: grafana-sfdx-hardis

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ github.repository_owner }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./docker/grafana
          file: ./docker/grafana/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          image: ${{ env.REGISTRY }}/${{ github.repository_owner }}/${{ env.IMAGE_NAME }}:latest
          format: spdx-json
          output-file: sbom.spdx.json

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.spdx.json
```

### 3.2 Docker Compose for Production Deployment (30 minutes)

Create production-ready Docker Compose configuration:

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  grafana:
    image: ghcr.io/your-org/grafana-sfdx-hardis:latest
    container_name: grafana-sfdx-hardis
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
      - GF_SECURITY_SECRET_KEY=${GRAFANA_SECRET_KEY}
      - GF_DATABASE_HOST=${DB_HOST}
      - GF_DATABASE_NAME=${DB_NAME}
      - GF_DATABASE_USER=${DB_USER}
      - GF_DATABASE_PASSWORD=${DB_PASSWORD}
      - GITHUB_ACCESS_TOKEN=${GITHUB_ACCESS_TOKEN}
    volumes:
      - grafana-data:/var/lib/grafana
      - grafana-logs:/var/log/grafana
    networks:
      - monitoring
    depends_on:
      - postgres
      - loki
      - prometheus

  postgres:
    image: postgres:15-alpine
    container_name: grafana-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - monitoring

  loki:
    image: grafana/loki:latest
    container_name: loki-sfdx-hardis
    restart: unless-stopped
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - loki-data:/loki
      - ./config/loki.yml:/etc/loki/local-config.yaml:ro
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus-sfdx-hardis
    restart: unless-stopped
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    volumes:
      - prometheus-data:/prometheus
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml:ro
    networks:
      - monitoring

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager-sfdx-hardis
    restart: unless-stopped
    ports:
      - "9093:9093"
    volumes:
      - alertmanager-data:/alertmanager
      - ./config/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    networks:
      - monitoring

  nginx:
    image: nginx:alpine
    container_name: nginx-grafana
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    networks:
      - monitoring
    depends_on:
      - grafana

volumes:
  grafana-data:
  grafana-logs:
  postgres-data:
  loki-data:
  prometheus-data:
  alertmanager-data:

networks:
  monitoring:
    driver: bridge
```

## Phase 4: SFDX-Hardis Configuration for Monitoring (Estimated: 2 hours)

### 4.1 Environment Variables Configuration (30 minutes)

Configure SFDX-Hardis to send monitoring data to Grafana:

```bash
# Environment variables for GitHub Actions
NOTIF_API_URL=http://loki:3100/loki/api/v1/push
NOTIF_API_METRICS_URL=http://prometheus:9090/api/v1/write
NOTIF_API_DEBUG=true

# Grafana configuration
GRAFANA_API_KEY=${GRAFANA_SERVICE_ACCOUNT_TOKEN}
GRAFANA_URL=https://grafana.your-company.com

# GitHub Enterprise integration
GITHUB_TOKEN=${GITHUB_TOKEN}
GITHUB_ENTERPRISE_URL=https://your-github-enterprise.com

# Jira integration (optional)
JIRA_HOST=https://your-company.atlassian.net
JIRA_USERNAME=${JIRA_USERNAME}
JIRA_TOKEN=${JIRA_TOKEN}
```

### 4.2 Custom Monitoring Script (45 minutes)

Create a custom script to enhance SFDX-Hardis monitoring:

```bash
#!/bin/bash
# monitor-sfdx-deployment.sh

# Function to send metrics to Prometheus
send_metric() {
    local metric_name=$1
    local metric_value=$2
    local labels=$3
    local timestamp=$(date +%s)
    
    curl -X POST "${NOTIF_API_METRICS_URL}" \
        -H "Content-Type: application/x-protobuf" \
        -H "Content-Encoding: snappy" \
        --data-binary "@-" << EOF
${metric_name}{${labels}} ${metric_value} ${timestamp}000
EOF
}

# Function to send logs to Loki
send_log() {
    local log_level=$1
    local message=$2
    local labels=$3
    local timestamp=$(date +%s)
    
    curl -X POST "${NOTIF_API_URL}" \
        -H "Content-Type: application/json" \
        --data-raw "{
            \"streams\": [{
                \"stream\": {
                    \"job\": \"sfdx-hardis\",
                    \"level\": \"${log_level}\",
                    ${labels}
                },
                \"values\": [[\"${timestamp}000000000\", \"${message}\"]]
            }]
        }"
}

# Monitor deployment function
monitor_deployment() {
    local org=$1
    local branch=$2
    local start_time=$(date +%s)
    
    # Send deployment start metric
    send_metric "sfdx_hardis_deployment_started" "1" "org=\"${org}\",branch=\"${branch}\""
    send_log "info" "Deployment started for org: ${org}, branch: ${branch}" "\"org\":\"${org}\",\"branch\":\"${branch}\""
    
    # Run SFDX-Hardis deployment
    if sf hardis:project:deploy:start --target-org ${org}; then
        local end_time=$(date +%s)
        local duration=$((end_time - start_time))
        
        # Send success metrics
        send_metric "sfdx_hardis_deployment_success" "1" "org=\"${org}\",branch=\"${branch}\""
        send_metric "sfdx_hardis_deployment_duration_seconds" "${duration}" "org=\"${org}\",branch=\"${branch}\""
        send_log "info" "Deployment successful for org: ${org}, duration: ${duration}s" "\"org\":\"${org}\",\"branch\":\"${branch}\",\"status\":\"success\""
        
        return 0
    else
        local end_time=$(date +%s)
        local duration=$((end_time - start_time))
        
        # Send failure metrics
        send_metric "sfdx_hardis_deployment_failure" "1" "org=\"${org}\",branch=\"${branch}\""
        send_metric "sfdx_hardis_deployment_duration_seconds" "${duration}" "org=\"${org}\",branch=\"${branch}\""
        send_log "error" "Deployment failed for org: ${org}, duration: ${duration}s" "\"org\":\"${org}\",\"branch\":\"${branch}\",\"status\":\"failure\""
        
        return 1
    fi
}

# Monitor org health
monitor_org_health() {
    local org=$1
    
    # Run backup and monitoring
    if sf hardis:org:monitor:backup --target-org ${org}; then
        send_metric "sfdx_hardis_backup_success" "1" "org=\"${org}\""
        send_log "info" "Backup successful for org: ${org}" "\"org\":\"${org}\",\"type\":\"backup\""
    else
        send_metric "sfdx_hardis_backup_failure" "1" "org=\"${org}\""
        send_log "error" "Backup failed for org: ${org}" "\"org\":\"${org}\",\"type\":\"backup\""
    fi
    
    # Run apex tests and capture coverage
    if test_result=$(sf hardis:org:test:run --target-org ${org} --json); then
        coverage=$(echo $test_result | jq -r '.result.summary.testRunCoverage')
        send_metric "sfdx_hardis_apex_coverage_percentage" "${coverage}" "org=\"${org}\""
        send_log "info" "Apex test coverage: ${coverage}% for org: ${org}" "\"org\":\"${org}\",\"type\":\"test_coverage\""
    fi
}

# Main execution
case "${1}" in
    "deploy")
        monitor_deployment "${2}" "${3}"
        ;;
    "health")
        monitor_org_health "${2}"
        ;;
    *)
        echo "Usage: $0 {deploy|health} [org] [branch]"
        exit 1
        ;;
esac
```

### 4.3 GitHub Actions Integration (45 minutes)

Update GitHub Actions workflow to include monitoring:

```yaml
# .github/workflows/sfdx-hardis-monitored.yml
name: SFDX-Hardis Monitored Deployment

on:
  push:
    branches: [main, develop, release/*]
  pull_request:
    branches: [main, develop]

jobs:
  deploy-with-monitoring:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        org: [integration, uat, production]
        
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install SFDX-Hardis
        run: echo y | sf plugins install sfdx-hardis

      - name: Authenticate to Salesforce
        run: |
          echo "${{ secrets.SFDX_AUTH_URL_${{ matrix.org }} }}" > ./authfile
          sf org login sfdx-url --sfdx-url-file ./authfile --alias ${{ matrix.org }}

      - name: Deploy with Monitoring
        env:
          NOTIF_API_URL: ${{ vars.LOKI_ENDPOINT }}
          NOTIF_API_METRICS_URL: ${{ vars.PROMETHEUS_ENDPOINT }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ORG_NAME: ${{ matrix.org }}
          BRANCH_NAME: ${{ github.ref_name }}
        run: |
          chmod +x ./scripts/monitor-sfdx-deployment.sh
          ./scripts/monitor-sfdx-deployment.sh deploy ${{ matrix.org }} ${{ github.ref_name }}

      - name: Health Check
        if: success()
        env:
          NOTIF_API_URL: ${{ vars.LOKI_ENDPOINT }}
          NOTIF_API_METRICS_URL: ${{ vars.PROMETHEUS_ENDPOINT }}
        run: |
          ./scripts/monitor-sfdx-deployment.sh health ${{ matrix.org }}

      - name: Upload Deployment Logs
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: deployment-logs-${{ matrix.org }}
          path: |
            .sfdx/logs/
            deployment-results.json
          retention-days: 30
```

## Phase 5: Alerting Configuration (Estimated: 1 hour)

### 5.1 Prometheus Alerting Rules (30 minutes)

Create alerting rules for deployment monitoring:

```yaml
# config/alert-rules.yml
groups:
  - name: sfdx-hardis-alerts
    rules:
      - alert: DeploymentFailureRate
        expr: rate(sfdx_hardis_deployment_failure[5m]) > 0.1
        for: 2m
        labels:
          severity: warning
          service: sfdx-hardis
        annotations:
          summary: "High deployment failure rate detected"
          description: "Deployment failure rate is {{ $value }} for org {{ $labels.org }}"

      - alert: DeploymentDuration
        expr: histogram_quantile(0.95, rate(sfdx_hardis_deployment_duration_seconds_bucket[5m])) > 600
        for: 5m
        labels:
          severity: warning
          service: sfdx-hardis
        annotations:
          summary: "Deployment taking too long"
          description: "95th percentile deployment duration is {{ $value }}s for org {{ $labels.org }}"

      - alert: ApexCoverageLow
        expr: sfdx_hardis_apex_coverage_percentage < 75
        for: 1m
        labels:
          severity: critical
          service: sfdx-hardis
        annotations:
          summary: "Low Apex test coverage detected"
          description: "Apex test coverage is {{ $value }}% for org {{ $labels.org }}"

      - alert: BackupFailure
        expr: increase(sfdx_hardis_backup_failure[1h]) > 0
        for: 1m
        labels:
          severity: critical
          service: sfdx-hardis
        annotations:
          summary: "Org backup failed"
          description: "Backup failed for org {{ $labels.org }}"
```

### 5.2 Alertmanager Configuration (30 minutes)

Configure notification channels:

```yaml
# config/alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@your-company.com'

route:
  group_by: ['alertname', 'org']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'
    - match:
        service: sfdx-hardis
      receiver: 'sfdx-team'

receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://localhost:5001/'

  - name: 'critical-alerts'
    email_configs:
      - to: 'ops-team@your-company.com'
        subject: 'CRITICAL: SFDX-Hardis Alert'
        body: |
          {{ range .Alerts }}
          Alert: {{ .Annotations.summary }}
          Description: {{ .Annotations.description }}
          {{ end }}
    slack_configs:
      - api_url: '{{ .SlackWebhookURL }}'
        channel: '#ops-alerts'
        text: 'CRITICAL SFDX-Hardis Alert: {{ .CommonAnnotations.summary }}'

  - name: 'sfdx-team'
    email_configs:
      - to: 'sfdx-team@your-company.com'
        subject: 'SFDX-Hardis Deployment Alert'
    teams_configs:
      - webhook_url: '{{ .TeamsWebhookURL }}'
        title: 'SFDX-Hardis Alert'
        text: '{{ .CommonAnnotations.summary }}'
```

## Deployment Instructions

### 1. Build and Deploy the Custom Image

```bash
# Clone the repository
git clone https://github.com/your-org/grafana-sfdx-hardis.git
cd grafana-sfdx-hardis

# Build the image locally (optional)
docker build -t grafana-sfdx-hardis:local ./docker/grafana

# Push to GitHub Container Registry (via GitHub Actions)
git add .
git commit -m "Add custom Grafana configuration"
git push origin main
```

### 2. Deploy the Stack

```bash
# Create environment file
cat > .env << EOF
GRAFANA_ADMIN_PASSWORD=your-secure-password
GRAFANA_SECRET_KEY=your-secret-key
DB_HOST=postgres
DB_NAME=grafana
DB_USER=grafana
DB_PASSWORD=your-db-password
GITHUB_ACCESS_TOKEN=your-github-token
EOF

# Deploy with Docker Compose
docker-compose -f docker-compose.prod.yml up -d

# Verify deployment
docker-compose -f docker-compose.prod.yml ps
```

### 3. Configure SFDX-Hardis Projects

```bash
# Add monitoring script to your SFDX projects
cp scripts/monitor-sfdx-deployment.sh your-sfdx-project/scripts/
chmod +x your-sfdx-project/scripts/monitor-sfdx-deployment.sh

# Update GitHub Actions workflows
cp .github/workflows/sfdx-hardis-monitored.yml your-sfdx-project/.github/workflows/
```

## Success Criteria

1. **Custom Grafana Image**
   - ✅ Successfully built and pushed to GitHub Container Registry
   - ✅ Includes all required plugins and configurations
   - ✅ Proper security and authentication setup

2. **Delivery Monitoring**
   - ✅ Real-time deployment status tracking
   - ✅ Performance metrics and trends
   - ✅ Failure detection and alerting

3. **Integration Quality**
   - ✅ GitHub Enterprise authentication working
   - ✅ SFDX-Hardis metrics flowing to Grafana
   - ✅ Alerts triggering appropriately
   - ✅ Dashboards updating in real-time

## Maintenance and Operations

- **Regular Updates**: Update the base Grafana image monthly
- **Backup Strategy**: Backup Grafana configuration and dashboards
- **Log Rotation**: Configure log retention policies
- **Security Patches**: Monitor for security updates
- **Performance Monitoring**: Monitor Grafana resource usage

This setup provides comprehensive monitoring for SFDX-Hardis delivery jobs with enterprise-grade security and scalability.

## Summary Tables

### Phase Overview & Time Estimation

| Phase | Component | Key Deliverables | Time Estimate | Prerequisites | Technical Complexity |
|-------|-----------|------------------|---------------|---------------|---------------------|
| **1** | **Custom Grafana Image** | • Dockerfile with enterprise plugins<br>• Custom configuration files<br>• Provisioning setup<br>• Health checks | **2-3 hours** | • Docker knowledge<br>• GitHub Enterprise access<br>• Container registry permissions | **Medium** |
| **2** | **SFDX-Hardis Dashboards** | • Delivery monitoring dashboard<br>• Org health dashboard<br>• GitHub Actions dashboard<br>• Custom panels & metrics | **3-4 hours** | • Grafana dashboard experience<br>• SFDX-Hardis knowledge<br>• JSON configuration skills | **Medium-High** |
| **3** | **Container Registry** | • GitHub Actions build workflow<br>• Multi-arch image support<br>• Production Docker Compose<br>• SBOM generation | **1-2 hours** | • GitHub Actions knowledge<br>• Container registry access<br>• CI/CD experience | **Medium** |
| **4** | **SFDX-Hardis Integration** | • Monitoring script<br>• Environment configuration<br>• GitHub Actions integration<br>• Metrics collection | **2 hours** | • SFDX-Hardis expertise<br>• Bash scripting<br>• API integration knowledge | **Medium-High** |
| **5** | **Alerting Setup** | • Prometheus alert rules<br>• Alertmanager configuration<br>• Notification channels<br>• Alert testing | **1 hour** | • Prometheus knowledge<br>• Alert management experience<br>• Communication channels access | **Medium** |

### Technical Components & Architecture

| Component | Image/Version | Purpose | Ports | Storage Requirements | Dependencies |
|-----------|---------------|---------|-------|---------------------|-------------|
| **Custom Grafana** | `ghcr.io/org/grafana-sfdx-hardis:latest` | Visualization & dashboards | 3000 | 2GB persistent | PostgreSQL, Loki, Prometheus |
| **PostgreSQL** | `postgres:15-alpine` | Grafana database | 5432 | 1GB persistent | None |
| **Loki** | `grafana/loki:latest` | Log aggregation | 3100 | 5GB persistent | None |
| **Prometheus** | `prom/prometheus:latest` | Metrics storage | 9090 | 10GB persistent | None |
| **Alertmanager** | `prom/alertmanager:latest` | Alert management | 9093 | 1GB persistent | Prometheus |
| **Nginx** | `nginx:alpine` | Reverse proxy & SSL | 80, 443 | 100MB persistent | Grafana |

### Resource Requirements

| Resource Type | Specification | Purpose | Scalability Notes |
|---------------|---------------|---------|-------------------|
| **CPU** | 4+ vCPUs (8+ recommended) | Container orchestration & processing | Scale horizontally with replica sets |
| **Memory** | 8GB RAM (16GB recommended) | In-memory processing & caching | Increase based on metric volume |
| **Storage** | 50GB SSD (100GB+ recommended) | Persistent data & logs | Use high-performance storage for metrics |
| **Network** | 1Gbps (enterprise bandwidth) | Data ingestion & API calls | Consider CDN for dashboard delivery |
| **GitHub Enterprise** | Admin access + Container Registry | Image storage & authentication | Unlimited private repositories |

### Security & Access Requirements

| Access Type | Level Required | Purpose | Security Notes |
|-------------|----------------|---------|----------------|
| **GitHub Enterprise Admin** | Organization Owner | Container registry & OAuth apps | Use service accounts where possible |
| **Docker Registry** | Read/Write packages | Image push/pull operations | Implement image scanning |
| **Salesforce Orgs** | API access per org | Deployment monitoring | Use dedicated integration users |
| **SMTP/Communication** | Send permissions | Alert notifications | Secure credentials in secrets |
| **SSL Certificates** | Valid domain certificates | HTTPS termination | Automate renewal process |

### Integration Matrix

| Integration | Data Flow | Update Frequency | Alert Triggers | Monitoring Scope |
|-------------|-----------|------------------|----------------|------------------|
| **SFDX-Hardis → Loki** | Deployment logs | Real-time | Log level errors | All deployment activities |
| **SFDX-Hardis → Prometheus** | Metrics & counters | Every 30s | Threshold breaches | Performance & success rates |
| **GitHub Actions → Grafana** | Workflow status | Per workflow run | Workflow failures | CI/CD pipeline health |
| **Salesforce Orgs → Monitoring** | Health checks | Daily/Weekly | Backup failures, low coverage | Org configuration & quality |

### Risk Assessment & Mitigation

| Risk Category | Probability | Impact | Mitigation Strategy | Monitoring |
|---------------|-------------|---------|-------------------|-------------|
| **Container Registry Limits** | Low | Medium | Use multiple registries, implement cleanup policies | Monitor storage quotas |
| **GitHub Enterprise API Limits** | Medium | Medium | Implement rate limiting, use service accounts | Track API usage metrics |
| **Metrics Volume Explosion** | Medium | High | Configure retention policies, implement sampling | Monitor storage growth |
| **SSL Certificate Expiry** | Low | High | Automate renewal, implement cert monitoring | 30-day expiry alerts |
| **Database Performance** | Medium | Medium | Optimize queries, scale database resources | Monitor query performance |
| **Security Vulnerabilities** | Medium | High | Regular image scanning, dependency updates | Automated security scans |

### Operational Procedures

| Procedure | Frequency | Responsibility | Automation Level | Documentation |
|-----------|-----------|----------------|------------------|---------------|
| **Image Updates** | Monthly | DevOps Team | Fully Automated | GitHub Actions workflow |
| **Dashboard Backup** | Weekly | Platform Team | Semi-Automated | Backup scripts + manual verification |
| **Security Scanning** | Daily | Security Team | Fully Automated | Integrated in CI/CD pipeline |
| **Performance Review** | Monthly | Operations Team | Manual | Performance dashboard reviews |
| **Alert Rule Updates** | As needed | DevOps + SFDX Teams | Manual | Change management process |
| **Certificate Renewal** | Every 90 days | Platform Team | Fully Automated | Let's Encrypt automation |

### Success Metrics & KPIs

| Metric | Target Value | Measurement Method | Review Frequency |
|--------|-------------|-------------------|------------------|
| **Dashboard Load Time** | < 3 seconds | Grafana performance metrics | Daily |
| **Deployment Visibility** | 100% coverage | SFDX-Hardis integration logs | Weekly |
| **Alert Accuracy** | > 95% (low false positives) | Alert validation tracking | Monthly |
| **System Uptime** | > 99.5% | Monitoring stack availability | Daily |
| **Mean Time to Resolution** | < 30 minutes | Incident tracking | Monthly |
| **User Adoption** | > 80% of development teams | Dashboard usage analytics | Quarterly |

### Troubleshooting Quick Reference

| Issue | Symptoms | First Steps | Escalation Path |
|-------|-----------|-------------|-----------------|
| **Dashboard Not Loading** | HTTP 502/503 errors | Check container status, restart Grafana | Platform Team → Infrastructure Team |
| **Missing Metrics** | Empty panels, no data | Verify Prometheus targets, check SFDX-Hardis config | DevOps Team → SFDX Team |
| **Alert Spam** | Too many notifications | Review alert thresholds, adjust sensitivity | Operations Team → Alert Rule Review |
| **Login Issues** | GitHub auth failures | Check OAuth app config, verify enterprise settings | Security Team → GitHub Admin |
| **Performance Degradation** | Slow queries, timeouts | Check resource usage, optimize queries | Platform Team → Database Team |

---

**Total Implementation Time: 8-10 hours**

**Recommended Team Size: 2-3 engineers (DevOps, Platform, SFDX specialist)**

**Go-Live Timeline: 2-3 weeks (including testing and validation)**
