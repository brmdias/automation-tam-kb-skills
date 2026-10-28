---
name: Sandbox Environments
description: Your Rundeck/RBA test environments — fill in per-user. Read before recommending a test environment.
type: reference
---

# Sandbox Environments

> TEMPLATE — replace every `<placeholder>` with your own values. `/tam-kb-setup`
> can pre-fill this for you, or edit it by hand. Delete any environment you don't use.
> Do not commit secrets/tokens to a shared repo; reference a secrets path instead.

## Selection Guide

```
Issue Type?
├─ Production parity needed? → <prod-like env> (managed DB, Enterprise Runners)
├─ Database-specific? → match the DB (Aurora/MySQL/Postgres/H2)
├─ Latest SaaS feature? → <SaaS/PDT env>
├─ Quick reproduction? → <local env>
└─ Generic job testing? → <default env>
```

## <Environment name, e.g. AWS EKS>

**URL:** `<https://your-instance.example.com>`  
**Version:** `<Rundeck/RBA version>`  
**Database:** `<Aurora MySQL / MySQL / Postgres / H2>`  
**Runners:** `<runner name / id, or "built-in">`  
**Management:** `<Helm / Docker Compose / WAR / SaaS>`  
**API Token:** `<secrets path — never inline the token>`

### Projects

- `<project-name>` — `<purpose / when to use>`

### Connection

```bash
# <how you connect: kubectl context, ssh, local start command, etc.>
```

### When to Use

- `<scenario 1>`
- `<scenario 2>`

## <Add more environments as needed>
