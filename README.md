
[README (2).md](https://github.com/user-attachments/files/25756774/README.2.md)
# 🔄 Identity Lifecycle Automation

> Automated provisioning, deprovisioning, and governance of digital identities across enterprise systems.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![SCIM: 2.0](https://img.shields.io/badge/SCIM-2.0-blue.svg)]()
[![IGA: Enabled](https://img.shields.io/badge/IGA-Enabled-purple.svg)]()
[![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen.svg)]()

---

## 📋 Table of Contents

- [Overview](#overview)
- [Lifecycle Stages](#lifecycle-stages)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Joiner Workflow](#joiner-workflow)
- [Mover Workflow](#mover-workflow)
- [Leaver Workflow](#leaver-workflow)
- [SCIM Provisioning](#scim-provisioning)
- [Access Certification](#access-certification)
- [Connectors](#connectors)
- [API Reference](#api-reference)
- [Security & Compliance](#security--compliance)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## Overview

Identity Lifecycle Automation (ILA) manages the full **Joiner-Mover-Leaver (JML)** lifecycle of user identities across enterprise systems. It automates account provisioning, role assignment, access reviews, and deprovisioning — reducing manual overhead, enforcing least privilege, and maintaining a continuous audit trail for compliance (SOX, HIPAA, SOC 2, ISO 27001).

**Core capabilities:**
- **Joiner** — Automate onboarding: create accounts, assign roles, provision app access, send welcome credentials
- **Mover** — Detect role/department changes and reconcile entitlements automatically
- **Leaver** — Trigger full or staged deprovisioning on termination, including token revocation and access removal
- **Orphan Detection** — Surface accounts with no active owner
- **Access Certification** — Periodic entitlement reviews with manager attestation
- **SCIM 2.0** — Standardized provisioning across IdP, HR, and downstream apps

---

## Lifecycle Stages

```
         HR System / HRIS (Source of Truth)
                        │
         ┌──────────────┼──────────────┐
         │              │              │
         ▼              ▼              ▼
      JOINER          MOVER          LEAVER
         │              │              │
    New Hire       Role Change    Termination /
    Contractor     Transfer       Resignation /
    Rehire         Promotion      Leave of Absence
         │              │              │
         ▼              ▼              ▼
┌─────────────────────────────────────────────┐
│          Identity Lifecycle Engine           │
│    Provisioning · Reconciliation · Audit     │
└──────────────────────┬──────────────────────┘
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
   Active Dir /    App Connectors    Audit &
   LDAP / IdP      (SCIM / API)     SIEM Log
```

---

## Features

| Feature | Description |
|---|---|
| 🧩 **JML Automation** | Full Joiner-Mover-Leaver workflow engine |
| 📡 **SCIM 2.0** | Standardized provisioning protocol |
| 🔍 **Orphan Detection** | Identify accounts with no active owner |
| ✅ **Access Certification** | Periodic entitlement reviews with attestation |
| 🔐 **Least Privilege** | Role-based provisioning scoped to job function |
| 🕵️ **Drift Detection** | Alert on entitlements outside policy |
| 📊 **Audit Trail** | Immutable logs for all lifecycle events |
| 🔌 **Connectors** | AD, Entra ID, Okta, Google Workspace, Salesforce, ServiceNow, GitHub |
| ⏱️ **SLA Enforcement** | Configurable deprovisioning SLAs (e.g., T+0, T+24h) |
| 📬 **Notifications** | Slack, email, and ticketing system alerts |

---

## Prerequisites

```bash
# Runtime
Python >= 3.11  OR  Node.js >= 20.x
Docker >= 25.x
Kubernetes >= 1.29 (optional)

# Infrastructure
PostgreSQL >= 15      # identity store + audit log
Redis >= 7.2          # job queue + cache
Vault >= 1.15         # connector credentials (optional)

# Integrations (at least one HR source required)
Workday / BambooHR / SAP SuccessFactors / UKG
Active Directory / Entra ID / Okta
```

---

## Installation

### Docker

```bash
git clone https://github.com/your-org/identity-lifecycle.git
cd identity-lifecycle

cp .env.example .env
# Configure HR source, connector creds, and DB

docker compose up -d
```

### Manual

```bash
git clone https://github.com/your-org/identity-lifecycle.git
cd identity-lifecycle

pip install -r requirements.txt   # or: npm install

python manage.py db upgrade
python manage.py worker start &
python manage.py api start
```

### Kubernetes

```bash
helm repo add identity-lifecycle https://charts.your-org.com
helm install identity-lifecycle identity-lifecycle/identity-lifecycle \
  --namespace identity \
  --create-namespace \
  -f values.yaml
```

---

## Configuration

### Environment Variables

```env
# Application
APP_PORT=8080
APP_ENV=production
APP_SECRET=<32-byte-secret>
LOG_LEVEL=info

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=identity_lifecycle
DB_USER=ila_user
DB_PASS=<password>

# Redis (Job Queue)
REDIS_URL=redis://localhost:6379

# HR Source of Truth
HR_PROVIDER=workday                      # workday | bamboohr | successfactors | ukg
HR_API_URL=https://wd3-impl-services1.workday.com/ccx/service/your-tenant/Human_Resources/v40.0
HR_API_USER=<integration-user>
HR_API_PASS=<password>
HR_SYNC_INTERVAL=900                     # seconds (15 min default)
HR_EMPLOYEE_FILTER=status=Active         # optional filter

# Identity Provider
IDP_PROVIDER=okta                        # okta | entra | google | ping
IDP_BASE_URL=https://your-org.okta.com
IDP_API_TOKEN=<token>
IDP_SCIM_ENDPOINT=https://your-org.okta.com/scim/v2

# Active Directory / LDAP
AD_HOST=dc01.corp.example.com
AD_PORT=636
AD_USE_SSL=true
AD_BIND_DN=CN=svc-ila,OU=ServiceAccounts,DC=corp,DC=example,DC=com
AD_BIND_PASS=<password>
AD_BASE_DN=DC=corp,DC=example,DC=com
AD_USER_OU=OU=Users,DC=corp,DC=example,DC=com

# Lifecycle SLAs
LEAVER_DEPROVISION_SLA_HOURS=0           # 0 = immediate on termination
LEAVER_AD_DISABLE_SLA_HOURS=0
LEAVER_LICENSE_RECLAIM_SLA_HOURS=24
MOVER_RECONCILE_SLA_HOURS=4

# Notifications
NOTIFY_SLACK_WEBHOOK=https://hooks.slack.com/services/...
NOTIFY_EMAIL_FROM=iam-automation@corp.example.com
NOTIFY_MANAGER_ON_LEAVER=true
NOTIFY_IT_ON_LEAVER=true
TICKET_PROVIDER=servicenow               # servicenow | jira | none
SNOW_INSTANCE=https://your-org.service-now.com
SNOW_API_USER=<user>
SNOW_API_PASS=<password>

# Audit / SIEM
AUDIT_SIEM_ENDPOINT=https://siem.corp.example.com/ingest
AUDIT_SIEM_TOKEN=<token>
```

---

## Joiner Workflow

Triggered when a new hire record appears in the HRIS.

### Default Flow

```
HRIS New Hire Event
        │
        ▼
[1] Identity Created
    └─ Generate unique username (policy: first.last / fl+last)
    └─ Set temp password (force-reset on first login)
    └─ Assign UPN / email alias

        │
        ▼
[2] Role & Group Assignment
    └─ Map job_code → role_profile → AD groups + IdP groups
    └─ Apply least-privilege baseline entitlements
    └─ Assign licenses (M365 / Google Workspace)

        │
        ▼
[3] App Provisioning (SCIM / API)
    └─ Provision to IdP (Okta / Entra)
    └─ Provision downstream apps per role profile
    └─ Set manager relationship

        │
        ▼
[4] Notifications
    └─ Welcome email with credentials
    └─ Manager notified with access summary
    └─ ServiceNow ticket created and closed
    └─ Audit event logged
```

### Role Profile Example (`config/role_profiles.yaml`)

```yaml
role_profiles:
  SWE_L1:
    display_name: Software Engineer L1
    ad_groups:
      - GRP_Dev_ReadOnly
      - GRP_VPN_Users
      - GRP_GitHub_Developers
    app_assignments:
      - github
      - jira
      - confluence
      - datadog
    licenses:
      - m365_e3
    mfa_required: true
    privileged: false

  IT_ADMIN:
    display_name: IT Administrator
    ad_groups:
      - GRP_IT_Admins
      - GRP_VPN_Privileged
      - GRP_ServiceDesk
    app_assignments:
      - github
      - jira
      - servicenow
      - aws_console
    licenses:
      - m365_e5
    mfa_required: true
    privileged: true
    pam_required: true              # requires PAM checkout for privileged actions
```

### Trigger via API

```bash
POST /api/lifecycle/joiner
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "employee_id": "EMP-10042",
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane.doe@corp.example.com",
  "job_code": "SWE_L1",
  "department": "Engineering",
  "manager_id": "EMP-00012",
  "start_date": "2025-09-01",
  "employment_type": "full_time"       // full_time | contractor | intern
}
```

---

## Mover Workflow

Triggered when an HR record change is detected (role, department, manager, location).

### Change Detection

```yaml
# config/mover_triggers.yaml
mover_triggers:
  - field: job_code
    action: reconcile_role_profile
  - field: department
    action: reconcile_ad_ou + reconcile_groups
  - field: manager_id
    action: update_manager_relationship
  - field: cost_center
    action: update_attributes
  - field: location
    action: update_attributes + check_geo_policy
```

### Reconciliation Logic

```
Detect HR Change
      │
      ▼
Fetch current entitlements
      │
      ▼
Compute delta:
  └─ New role profile entitlements (desired)
  └─ Current entitlements (actual)
  └─ diff = (add: [...], remove: [...])
      │
      ▼
Apply delta:
  └─ Add new group memberships
  └─ Remove stale group memberships       ← enforces least privilege
  └─ Update app assignments
  └─ Reclaim or assign licenses
      │
      ▼
Notify manager + log audit event
```

### Trigger via API

```bash
POST /api/lifecycle/mover
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "employee_id": "EMP-10042",
  "change_type": "role_change",
  "previous": { "job_code": "SWE_L1", "department": "Engineering" },
  "current":  { "job_code": "SWE_L3", "department": "Platform" },
  "effective_date": "2025-10-01"
}
```

---

## Leaver Workflow

Triggered on termination, resignation, or extended leave. Designed for **zero-trust deprovisioning** — access removal begins at T+0 by default.

### Default Flow

```
HR Termination Event (T+0)
        │
        ▼
[1] Immediate Actions (T+0)
    └─ Disable AD account
    └─ Revoke all active SSO sessions + tokens
    └─ Disable IdP account (Okta / Entra)
    └─ Remove MFA devices
    └─ Suspend email (auto-forward to manager if configured)

        │
        ▼
[2] Within SLA Window (configurable, default T+24h)
    └─ Remove all AD group memberships
    └─ Deprovision downstream app accounts (SCIM DELETE)
    └─ Reclaim SaaS licenses
    └─ Transfer owned resources (Google Drive / OneDrive)
    └─ Remove SSH keys / API tokens / PAT tokens

        │
        ▼
[3] Offboarding Complete
    └─ Archive account (retain per data retention policy)
    └─ Create offboarding summary ticket
    └─ Notify manager + IT + HR
    └─ Immutable audit record created
```

### Leave of Absence (LOA)

```bash
POST /api/lifecycle/leaver
{
  "employee_id": "EMP-10042",
  "termination_type": "loa",           // terminated | resigned | loa | contractor_end
  "last_day": "2025-11-15",
  "return_date": "2026-02-01",         // LOA only
  "transfer_resources_to": "EMP-00012"
}
```

### Full Termination

```bash
POST /api/lifecycle/leaver
{
  "employee_id": "EMP-10042",
  "termination_type": "terminated",
  "last_day": "2025-11-15",
  "transfer_resources_to": "EMP-00012",
  "retain_email_days": 30
}
```

---

## SCIM Provisioning

This service exposes a **SCIM 2.0** endpoint for IdP-driven provisioning, and acts as a SCIM client to downstream apps.

### Inbound SCIM (IdP → ILA)

```
SCIM Base URL: https://your-host/scim/v2

Supported Operations:
  POST   /scim/v2/Users          Create user
  GET    /scim/v2/Users/{id}     Get user
  PUT    /scim/v2/Users/{id}     Replace user
  PATCH  /scim/v2/Users/{id}     Update user
  DELETE /scim/v2/Users/{id}     Deprovision user
  GET    /scim/v2/Groups         List groups
  PATCH  /scim/v2/Groups/{id}    Update group membership
```

### SCIM User Schema Extensions

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User",
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
    "urn:corp:scim:schemas:extension:lifecycle:1.0"
  ],
  "urn:corp:scim:schemas:extension:lifecycle:1.0": {
    "employeeId": "EMP-10042",
    "jobCode": "SWE_L1",
    "costCenter": "CC-4200",
    "hireDate": "2025-09-01",
    "lifecycleStatus": "active"        // active | mover | leaver | archived
  }
}
```

---

## Access Certification

Periodic entitlement reviews where managers attest whether direct reports should retain access.

### Certification Campaign Config (`config/certification.yaml`)

```yaml
campaigns:
  quarterly_review:
    name: Quarterly Access Review
    schedule: "0 8 1 */3 *"           # quarterly, 1st of month at 08:00
    scope:
      - all_active_users
    reviewers: manager                 # manager | owner | admin
    items_per_reviewer: 50
    due_days: 14
    on_no_response: revoke             # revoke | escalate | approve
    notify_channel: email + slack

  privileged_access_review:
    name: Privileged Access Review
    schedule: "0 8 1 * *"             # monthly
    scope:
      - groups: [GRP_IT_Admins, GRP_VPN_Privileged]
    reviewers: admin
    due_days: 7
    on_no_response: revoke
```

### Certification API

```bash
# Launch a campaign manually
POST /api/certification/campaigns/quarterly_review/launch
Authorization: Bearer <admin-token>

# Get pending reviews for a manager
GET /api/certification/reviews?reviewer=EMP-00012&status=pending

# Submit attestation decision
POST /api/certification/reviews/{reviewId}/decision
{
  "decision": "revoke",              // approve | revoke
  "justification": "Employee changed teams, access no longer needed"
}
```

---

## Connectors

| Connector | Protocol | Joiner | Mover | Leaver | Certify |
|---|---|---|---|---|---|
| Active Directory | LDAP | ✅ | ✅ | ✅ | ✅ |
| Entra ID (Azure AD) | Graph API + SCIM | ✅ | ✅ | ✅ | ✅ |
| Okta | SCIM 2.0 + API | ✅ | ✅ | ✅ | ✅ |
| Google Workspace | Directory API | ✅ | ✅ | ✅ | ✅ |
| GitHub Enterprise | SCIM + REST | ✅ | ✅ | ✅ | ⬜ |
| Salesforce | SCIM 2.0 | ✅ | ✅ | ✅ | ⬜ |
| ServiceNow | REST API | ✅ | ⬜ | ✅ | ⬜ |
| AWS IAM Identity Center | SCIM 2.0 | ✅ | ✅ | ✅ | ⬜ |
| Workday (HR Source) | SOAP / RaaS | ✅ | ✅ | ✅ | — |
| BambooHR (HR Source) | REST API | ✅ | ✅ | ✅ | — |

### Writing a Custom Connector

```python
# connectors/my_app.py
from connectors.base import BaseConnector

class MyAppConnector(BaseConnector):

    def provision(self, user: IdentityRecord) -> ProvisionResult:
        # Create account in MyApp
        ...

    def update(self, user: IdentityRecord, delta: EntitlementDelta) -> UpdateResult:
        # Apply role/group changes
        ...

    def deprovision(self, user: IdentityRecord) -> DeprovisionResult:
        # Disable or delete account
        ...

    def health_check(self) -> bool:
        # Verify connectivity
        ...
```

Register in `config/connectors.yaml`:

```yaml
connectors:
  my_app:
    enabled: true
    class: connectors.my_app.MyAppConnector
    base_url: https://api.my-app.example.com
    auth_type: bearer
    token: ${MY_APP_TOKEN}
    provision_on: [joiner]
    deprovision_on: [leaver]
```

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/lifecycle/joiner` | Trigger joiner workflow |
| `POST` | `/api/lifecycle/mover` | Trigger mover workflow |
| `POST` | `/api/lifecycle/leaver` | Trigger leaver workflow |
| `GET` | `/api/identity/{employeeId}` | Get identity record |
| `GET` | `/api/identity/{employeeId}/entitlements` | Get all entitlements |
| `POST` | `/api/identity/{employeeId}/reconcile` | Force entitlement reconciliation |
| `GET` | `/api/orphans` | List orphaned accounts |
| `GET` | `/api/drift` | List entitlement drift violations |
| `POST` | `/api/certification/campaigns/{id}/launch` | Launch cert campaign |
| `GET` | `/api/certification/reviews` | List pending reviews |
| `POST` | `/api/certification/reviews/{id}/decision` | Submit attestation decision |
| `GET` | `/api/audit/events` | Query audit log |
| `GET` | `/api/connectors` | List connector status |
| `GET` | `/health` | Health check |

Full OpenAPI spec: [`docs/openapi.yaml`](docs/openapi.yaml)

---

## Security & Compliance

**Least Privilege by Design**
- Entitlements derived from job function, not manually assigned
- Mover workflow automatically removes stale access on role change
- Access Certification enforces periodic revalidation

**Deprovisioning SLA**
- Default: T+0 session revocation, T+24h full deprovisioning
- Configurable per connector and termination type
- SLA breach alerts sent to IT and compliance team

**Audit Trail**
- All lifecycle events logged immutably with: actor, target, action, timestamp, source system
- SIEM integration for real-time alerting
- Retention configurable per compliance requirement (default 7 years)

**Secrets Management**
- Connector credentials stored in Vault or environment-encrypted secrets
- No plaintext credentials in config files or logs
- Credential rotation supported via `/api/admin/connectors/{id}/rotate`

**Compliance Mappings**

| Control | Framework | Automation Coverage |
|---|---|---|
| User Access Reviews | SOC 2 CC6.3 / ISO 27001 A.9.2.5 | Access Certification campaigns |
| Termination Procedures | SOC 2 CC6.2 / NIST 800-53 AC-2 | Leaver workflow (T+0) |
| Least Privilege | SOC 2 CC6.3 / NIST 800-53 AC-6 | Role profiles + Mover reconciliation |
| Audit Logging | SOX ITGC / ISO 27001 A.12.4 | Immutable event log + SIEM |

---

## Troubleshooting

**HR sync not picking up new hires**
```bash
# Check HR connector health
GET /api/connectors/workday/health

# Force a manual HR sync
POST /api/hr/sync/trigger

# Check sync logs
docker logs identity-lifecycle-worker | grep "HR_SYNC"
```

**Leaver deprovisioning stalled**
```bash
# Check job queue
GET /api/jobs?status=pending&type=leaver

# Retry a stuck job
POST /api/jobs/{jobId}/retry

# Check specific connector failure
GET /api/audit/events?employee_id=EMP-10042&action=deprovision
```

**Entitlement drift detected**
```bash
# List all drift violations
GET /api/drift

# Reconcile a specific user
POST /api/identity/EMP-10042/reconcile

# Dry-run reconciliation (no changes applied)
POST /api/identity/EMP-10042/reconcile?dry_run=true
```

**SCIM provisioning failures**
```
- Verify SCIM bearer token is valid and not expired
- Check attribute mapping: GET /api/connectors/{id}/schema
- Enable debug logging: CONNECTOR_LOG_LEVEL=debug in .env
```

---

## Contributing

1. Fork the repo
2. Branch: `git checkout -b feat/connector-salesforce`
3. Commit: `git commit -m "feat: add Salesforce SCIM connector"`
4. Push + open PR

All PRs touching provisioning or deprovisioning logic require:
- Unit tests covering the happy path and failure/rollback scenarios
- Security review note in the PR description
- Updated connector table in this README

---

## License

[MIT](LICENSE) © Your Org
