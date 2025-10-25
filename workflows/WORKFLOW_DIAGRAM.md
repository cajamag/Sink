# Security Audit Workflow - Visual Flow

```
┌─────────────────────┐
│  Schedule Trigger   │  Runs every 24 hours
│    (Every Day)      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  ⚙️ Configuración   │  Configure: hosts, paths, emails, Slack
│   (Configuration)   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 🔍 Security Scanner │  Scan all hosts and paths
│   (JavaScript)      │  Classify vulnerabilities
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  🚨 Hay Críticas?   │  Check for critical vulnerabilities
│  (If Condition)     │
└─────┬───────┬───────┘
      │       │
  YES │       │ NO
      │       │
      └───┬───┘
          │
          ▼
┌─────────────────────┐
│ 📊 Generate Report  │  Create HTML + Text reports
│   (JavaScript)      │
└─────────┬───────────┘
          │
          ├──────────────────┬──────────────────┐
          ▼                  ▼                  ▼
┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
│ 📧 Email Crítico │  │💬 Slack      │  │📧 Email      │
│ (Critical Alert) │  │  Crítico     │  │  Normal      │
└──────────────────┘  └──────────────┘  └──────────────┘
```

## Flow Logic

1. **Schedule Trigger** starts the workflow every 24 hours
2. **Configuration** node sets up all parameters (hosts, paths, notifications)
3. **Security Scanner** performs HTTP HEAD requests to check each URL
4. **Conditional Check** evaluates if critical vulnerabilities were found
5. **Generate Report** creates HTML and text reports regardless of critical status
6. **Notifications** are sent via:
   - **Email Crítico** - Sent when critical vulnerabilities detected
   - **Slack Crítico** - Sent when critical vulnerabilities detected
   - **Email Normal** - Always sent with full report

## Node Details

### Input/Output Flow

```
Schedule Trigger
  └─> ⚙️ Configuración
       Output: { hosts[], paths[], emailFrom, emailTo, slackChannel, scanTimeout, delayBetweenRequests }
       
       └─> 🔍 Security Scanner
            Output: { timestamp, summary, topVulnerable[], hosts[], config }
            
            └─> 🚨 Hay Críticas? (Condition: criticalVulnerabilities > 0)
                 Output [TRUE]: Passes full data to Generate Report
                 Output [FALSE]: Passes full data to Generate Report
                 
                 └─> 📊 Generate Report
                      Output: { html, textSummary, data, fileName, isCritical, hasWarnings }
                      
                      ├─> 📧 Email Crítico (Subject: "🚨 ALERTA CRÍTICA")
                      ├─> 💬 Slack Crítico (Message: Critical alert with metrics)
                      └─> 📧 Email Normal (Subject: "✅ Reporte de Seguridad")
```

## Severity Classification

| Severity | Examples | Color |
|----------|----------|-------|
| **CRÍTICA** | .env, .git, database, config | 🔴 Red |
| **ALTA** | storage/* (non-public) | 🟠 Orange |
| **MEDIA** | vendor, composer.json/lock | 🟡 Yellow |
| **BAJA** | Other paths, non-200 responses | ⚪ Gray |

## Notification Matrix

| Scenario | Email Crítico | Slack Crítico | Email Normal |
|----------|---------------|---------------|--------------|
| Critical vulnerabilities found | ✅ Sent | ✅ Sent | ✅ Sent |
| Only warnings found | ❌ Not sent | ❌ Not sent | ✅ Sent |
| All secure | ❌ Not sent | ❌ Not sent | ✅ Sent |

## Configuration Tips

1. **Performance Tuning**:
   - Increase `delayBetweenRequests` (e.g., 200ms) for slower networks
   - Increase `scanTimeout` (e.g., 15000ms) for slow servers
   
2. **Scanning Scope**:
   - Add more paths to check in the `paths` array
   - Remove paths that are not relevant to your tech stack
   
3. **Notifications**:
   - Use different email addresses for critical vs normal reports
   - Set up multiple Slack channels for different severity levels

## Security Considerations

⚠️ **Important**: This workflow performs HTTP requests to check for exposed files. Ensure:
- You have authorization to scan the target hosts
- The delay between requests is reasonable to avoid DoS
- Credentials are stored securely in n8n
- Reports contain sensitive information - protect them accordingly
