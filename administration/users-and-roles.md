# Users and Roles

Proscan uses role-based access control (RBAC) to manage what each user can see and do in the platform.

## Roles

| Role | Access Level |
|------|-------------|
| Super Admin | Full access to everything, including system settings and user management |
| Admin | Manage projects, scans, findings, integrations, and users |
| Security Engineer | Run scans, triage findings, manage projects |
| Developer | View findings and apply fixes for assigned projects |
| Analyst | View scan results and reports, no write access |
| Auditor | Read-only access to compliance reports and evidence |
| Viewer | Read-only access to dashboards and findings |

## Permissions

Each role has a set of granular permissions covering:

- Scan management (create, run, abort, delete)
- Finding management (triage, assign, change status)
- Project management (create, configure, delete)
- Report generation and export
- Integration and webhook configuration
- User administration
- System settings

You can see the full permission matrix in **Settings > Roles**.

## Managing Users

### Adding Users

1. Go to **Settings > Users**
2. Click **Add User**
3. Enter email, name, and assign a role
4. The user receives an invitation to set up their account

### Deactivating Users

User accounts can be deactivated without deletion, preserving their activity history in the audit log.

## Multi-Tenancy

For organizations managing multiple teams or clients, Proscan supports tenant isolation. Each tenant operates in a separate environment with independent data, users, and configurations. Cross-tenant access is not possible.
