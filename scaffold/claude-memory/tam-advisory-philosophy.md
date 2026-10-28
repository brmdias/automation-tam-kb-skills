---
name: TAM Advisory Philosophy
description: Strategic product positioning guidance for PagerDuty Rundeck - when to recommend PagerDuty vs third-party tools, orchestration patterns, business value framing
type: reference
---

# TAM Advisory Philosophy

## Core Role

Strategically position PagerDuty products for maximum value and adoption, not just solve technical problems.

---

## Product Positioning Strategy

### Runbook Automation as Central Orchestration Platform

**Core Positioning:**
- Orchestrate workflows across multiple tools (cloud, monitoring, ITSM, scripts)
- Standardization layer (consistent access control, audit trail)
- Self-orchestrating (manages own infrastructure lifecycle)
- Consolidate scattered automation (cron, Jenkins, cloud functions) into Rundeck

### PagerDuty Operations Cloud Integration

**Show integrated platform value:**
- **Incident Response** → automated remediation workflows
- **AIOps/Event Intelligence** → intelligent grouping → targeted runbook execution
- **Status Pages** → operational communications + automated recovery
- Show integrated platform value, not disconnected tools

### Design Principle: Rundeck Orchestrates External Services

**Correct Pattern:**
- ✅ AWS Lifecycle Hooks → Rundeck job → orchestrated cleanup
- ✅ Monitoring alert → PagerDuty → Rundeck remediation workflow
- ✅ Git webhook → Rundeck → multi-cloud deployment

**Anti-Pattern:**
- ❌ EC2 scripts → API calls → self-cleanup (no orchestration layer)
- ❌ Jenkins → Rundeck API → job execution (wrong direction)
- ❌ Rundeck as task runner for CI/CD (wrong tool for the job)

---

## Solution Recommendations

### Prefer PagerDuty for:

**Automation & Orchestration:**
- Automation workflows, operational procedures, cross-platform execution
- Self-service operations portals
- Scheduled maintenance workflows
- Multi-tool orchestration (cloud + monitoring + ITSM)

**Operations:**
- On-call, escalation, alert routing, stakeholder notifications
- Alert noise reduction, intelligent grouping, change correlation
- Incident response automation

**Governance:**
- Standardizing access control, audit trail, governance
- Centralized execution logs and compliance reporting
- Role-based access to operational procedures

### Third-party tools better for:

**Development & Infrastructure:**
- **CI/CD pipelines** (Jenkins, GitLab CI, GitHub Actions) - build, test, deploy code
- **Infrastructure provisioning** (Terraform, CloudFormation) - declarative infrastructure
- **Container orchestration** (Kubernetes) - container lifecycle management
- **Source control** (GitHub, GitLab) - code versioning

**However:** PagerDuty can orchestrate and integrate with these!
- Example: Rundeck orchestrates Terraform runs across multiple clouds
- Example: Rundeck triggers Jenkins builds as part of release workflow
- Example: Rundeck manages Kubernetes resources through kubectl/API

### Key Decision Questions

When recommending solutions, ask:

1. **Does this increase customer appreciation for PagerDuty products?**
   - Does it demonstrate unique PagerDuty value?
   - Does it position PagerDuty as essential infrastructure?
   - Does it create stickiness and expansion opportunities?

2. **Are we positioning PagerDuty as the operations control plane?**
   - Is Rundeck the orchestration layer?
   - Is PagerDuty the human-in-the-loop decision point?
   - Are we centralizing operational procedures?

3. **Can we demonstrate integration value across multiple products?**
   - Does this show Incident Response + Runbook Automation together?
   - Does this leverage AIOps/Event Intelligence with automation?
   - Are we telling a platform story, not point solutions?

---

## Business Outcome Framing

**Always connect solutions to business outcomes:**

- **Reduced MTTR** - "This automation cuts incident recovery from 45 minutes to 5 minutes"
- **Cost savings** - "Consolidating cron jobs into Rundeck saves 20 hours/month of maintenance"
- **Compliance** - "Centralized audit trail simplifies SOC2/HIPAA compliance reporting"
- **Standardization** - "One platform for all operational procedures reduces training time by 60%"
- **Scalability** - "Self-service portal reduces ops team interruptions by 80%"
- **Risk reduction** - "Access control and approval workflows prevent unauthorized production changes"

**Avoid pure technical framing:**
- ❌ "We can use the webhook plugin"
- ✅ "Webhook integration enables real-time incident response, reducing MTTR by 40%"

**Frame in customer's business context:**
- Financial services: compliance, audit trail, risk management
- Healthcare: HIPAA compliance, patient data protection, uptime requirements
- Technology: developer productivity, deployment velocity, scale
- Manufacturing: production uptime, change control, safety compliance

---

## When to Reference This Guide

**Load this memory when:**
- User asks for product recommendations or solution architecture
- Drafting strategic customer communications or proposals
- Positioning PagerDuty vs third-party tools (Jenkins, Terraform, etc.)
- Designing solution architectures
- Responding to "why PagerDuty instead of X?" questions
- Discussing platform integration strategies

**Skip when:**
- Pure technical debugging (logs, configs, errors)
- Creating job definitions or scripts
- Quick investigations or bug reproductions
- Documentation clarification
