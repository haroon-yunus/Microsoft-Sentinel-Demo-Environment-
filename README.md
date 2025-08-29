
# 🚨 Microsoft Sentinel Demo Environment — Terraform Deployment

This repository provisions a complete Microsoft Sentinel demo using Terraform, designed for security analytics, threat detection, and infrastructure observability. It includes:

- Azure Resource Group and Log Analytics Workspace
- Storage Account with diagnostic settings
- Sentinel onboarding
- Scheduled Alert Rule using KQL
- Modular, reusable Terraform code

---

## 📐 Architecture Overview

```text
+-----------------------------+
| Resource Group              |
|  sentinel-demo-rg          |
+-----------------------------+
        |         |         |
        v         v         v
+--------+   +----------+   +------------------+
| Storage|   | Log      |   | Sentinel         |
| Account|-->| Analytics|-->| Alert Rule: KV   |
|        |   | Workspace|   | Suspicious Access|
+--------+   +----------+   +------------------+
```

---

## 📦 Resources Deployed

| Resource Type                            | Terraform Resource                                            | Purpose                                                                 |
|-----------------------------------------|------------------------------------------------------------    |-------------------------------------------------------------------------|
| Resource Group                          | `azurerm_resource_group.rg`                                    | Logical container for all resources                                    |
| Log Analytics Workspace                 | `azurerm_log_analytics_workspace.law`                          | Centralized logging and analytics engine                               |
| Storage Account                         | `azurerm_storage_account.sa`                                   | Simulated data source for diagnostics                                  |
| Diagnostic Settings                     | `azurerm_monitor_diagnostic_setting.sa_diag`                   | Enables log/metric flow to Sentinel                                    |
| Sentinel Onboarding                     | `azurerm_sentinel_log_analytics_workspace_onboarding.sentinel` | Activates Sentinel on workspace               |
| Scheduled Alert Rule                    | `azurerm_sentinel_alert_rule_scheduled.kv_access_alert`        | Detects suspicious Key Vault access using KQL |

---

## 🔧 Terraform Modules & Logic

### Diagnostic Settings

Azure diagnostic categories vary by resource type. This demo uses:

```hcl
enabled_log {
  category = "StorageRead"
}
enabled_log {
  category = "StorageWrite"
}
enabled_log {
  category = "StorageDelete"
}
```

These categories ensure visibility into blob operations. Metrics are also enabled:

```hcl
metric {
  category = "AllMetrics"
  enabled  = true
}
```

> ⚠️ Use `az monitor diagnostic-settings categories list` to validate supported categories per resource.

---

### Sentinel Alert Rule

The alert rule uses KQL to detect unauthorized access to Key Vault secrets:

```kql
KQL Query

AzureDiagnostics
| where Category == "AuditEvent" and ResourceType == "VAULTS"
| sort by TimeGenerated desc
| take 50
```

**Terraform Configuration Highlights:**

```hcl
query_frequency  = "PT1H"
query_period     = "PT1H"
trigger_operator = "GreaterThan"
trigger_threshold= 0
incident_configuration {
  create_incident = true
}
```

This configuration ensures the rule runs hourly and triggers an incident if any suspicious access is detected.

---

## 🧪 Testing & Validation

1. **Simulate Key Vault Access**
   - Use Azure CLI or Portal to access secrets from an untrusted IP
   - Ensure Key Vault diagnostic settings are enabled and pointed to the same workspace

2. **Verify Data Ingestion**
   - Go to Log Analytics → Logs → Run the KQL query manually
   - Confirm `KeyVaultAuditEvents` table exists and returns results

3. **Check Sentinel Incidents**
   - Navigate to Microsoft Sentinel → Incidents
   - Confirm alert rule has triggered and incident is created

---

## 🛡️ Security & Operational Notes

- **Least Privilege:** Terraform assumes contributor access to deploy resources. For production, use RBAC and scoped roles.
- **Data Retention:** Workspace retention is set to 30 days. Adjust via `retention_in_days`.
- **Modularity:** Resources are grouped logically. For CI/CD, split into modules: `network`, `monitoring`, `security`.

---

## 📚 References

- [Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/)
- [Terraform AzureRM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [KQL Language Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)

---

## 👨‍💻 Author

**Haroon Yunus**  
Cloud Security & Automation Specialist  
Focused on modular infrastructure, real-world troubleshooting, and demo excellence.




