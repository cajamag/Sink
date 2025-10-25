# n8n Workflows

This directory contains n8n workflow definitions for automation tasks.

## Security Audit Workflow

**File:** `security-audit-configurable.json`

### Description

An automated security auditing workflow that scans web applications for exposed sensitive directories and files. This workflow is designed to detect potential security vulnerabilities in Laravel-based applications and other web servers.

### Features

- **Scheduled Execution**: Runs automatically every 24 hours
- **Configurable**: All settings (hosts, paths, emails, Slack channels) can be modified without changing code
- **Comprehensive Scanning**: Checks for common Laravel vulnerabilities including:
  - Exposed `.env` files
  - Accessible `.git` directories
  - Public storage directories
  - Database configuration files
  - Vendor directories
  - Composer files
- **Severity Classification**: Categorizes findings as CRÍTICA, ALTA, MEDIA, or BAJA
- **Multiple Notification Channels**:
  - Critical alerts sent via email and Slack
  - Regular reports sent via email
- **Beautiful HTML Reports**: Generates detailed, responsive HTML reports with:
  - Executive summary with key metrics
  - Top 5 most vulnerable hosts
  - Detailed findings per host
  - Color-coded severity indicators

### Workflow Nodes

1. **Schedule Trigger**: Initiates the workflow every 24 hours
2. **⚙️ Configuración**: Central configuration node - modify this to change:
   - List of hosts to scan
   - Paths to check
   - Email addresses (from/to)
   - Slack channel
   - Scan timeout
   - Delay between requests
3. **🔍 Security Scanner**: Core scanning engine that:
   - Performs HTTP HEAD requests to each URL
   - Classifies vulnerabilities by severity
   - Collects statistics
4. **🚨 Hay Críticas?**: Conditional node that routes based on critical vulnerabilities
5. **📊 Generate Report**: Creates HTML and text reports
6. **📧 Email Crítico**: Sends critical alert emails
7. **💬 Slack Crítico**: Sends critical alerts to Slack
8. **📧 Email Normal**: Sends regular status reports

### Installation

1. Open n8n
2. Click on "Workflows" → "Import from File"
3. Select `security-audit-configurable.json`
4. Configure the following credentials:
   - Email Send credentials (SMTP)
   - Slack credentials (OAuth2 or API Token)
5. Adjust the configuration in the "⚙️ Configuración" node:
   - Update `hosts` array with your domains
   - Update `emailFrom` and `emailTo` addresses
   - Update `slackChannel` with your Slack channel

### Configuration

All configuration is done in the **⚙️ Configuración** node. No code changes needed!

**Configurable Parameters:**

- `hosts`: Array of hostnames to scan
- `paths`: Array of paths to check on each host
- `emailFrom`: Sender email address
- `emailTo`: Recipient email address
- `slackChannel`: Slack channel for notifications (e.g., "#seguridad")
- `scanTimeout`: Timeout for each request in milliseconds (default: 10000)
- `delayBetweenRequests`: Delay between requests in milliseconds (default: 100)

### Security Best Practices

- Keep the `delayBetweenRequests` at a reasonable value to avoid overwhelming servers
- Use a dedicated service account for email notifications
- Restrict access to the Slack channel to security personnel
- Store n8n credentials securely
- Review and act on critical vulnerabilities immediately

### Severity Levels

- **CRÍTICA**: Exposed .env files, .git directories, database configs, or config directories
- **ALTA**: Accessible storage directories (except public/storage)
- **MEDIA**: Exposed vendor or composer files
- **BAJA**: Other accessible paths or non-200 responses

### Example Output

The workflow generates:
- HTML report with detailed findings
- Text summary for email body
- Slack notification with key metrics
- Metrics on total hosts, vulnerabilities, and secure hosts

### Customization

To scan different types of applications:
1. Update the `paths` array in the configuration node
2. Modify the `getSeverity()` function in the Security Scanner node if needed
3. Adjust the report template in the Generate Report node

### Support

For issues or questions about this workflow:
- Check n8n documentation: https://docs.n8n.io/
- Review n8n community forum: https://community.n8n.io/
