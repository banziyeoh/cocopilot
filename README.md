# cocopilot
For Prompt Engineer by Software Engineer

Modules Included
https://github.com/jonlwowski012/copilot-agent-factory/blob/main/agent-generator.md
https://github.com/github/spec-kit

Available for Github Copilot CLI
https://github.com/Dammyjay93/interface-design
https://github.com/earchibald/vsc-superpowers


# use as a collection
E2E Workflow Ochestrator
- 00-orchestrator
- 01-spec-agent
- 02-architect
- 03-planner
- 04-coder
- 05-reviwer
- 06-qa
- 07-security
- 08-integrator
- 09-docs
- 10-designer
- 11-researcher
- CONTRACT
- DISPATCH-REFERENCE
- WORKFLOW


# Microservice API OE Checklist [agents/microservice-api-oe-checklist.agent.md]

## Subagents
- Architecture Review Agent[agents/architecture-review.agent.md]
- Security Agent[agents/security.agent.md]
- Code Reviewer[agents/code-reviewer.md]
- Software Quality Assurance Agent[agents/qa.agent.md]
- Observability Agent[agents/observability.agent.md]
- Deployment Agent[agents/deployment.agent.md]
- Database Reviewer[agents/database-reviewer.md]
- Compliance Auditor Agent[agents/compliance-auditor.agent.md]

## Skills
- Reliability Strategy builder[skills/reliability-strategy-builder]
- Scalability Playbook[skills/scalability-playbook]
- Auth Security Reviewer[skills/auth-security-reviewer]



| **API OE Checklist Agent** | api-oe-checklist.agent.md | Agent (Orchestrator) |

**Skill Dependencies:** `oe-compliance-engine`, `reliability-strategy-builder`, `scalability-playbook`, `auth-security-reviewer`, `api-design`, `api-versioning-deprecation-planner`, `api-security-hardener`, `observability-setup`, `contract-testing-builder`, `input-validation-sanitization-auditor`, `secrets-scanner`, `openapi-generator`, `structured-logging-standardizer`, `threat-model-generator`

---

### Subagents (Dispatched by Entry Point)

| # | Name | File Path | Items | Skill Dependencies |
|---|------|-----------|-------|--------------------|
| 1 | **OE Architecture Auditor** | oe-architecture-auditor.agent.md | 1-3 | `oe-compliance-engine`, `architecture-patterns`, `openapi-generator`, `api-design` |
| 2 | **OE Security Fortress** | oe-security-fortress.agent.md | 4-7 | `oe-compliance-engine`, `auth-security-reviewer`, `api-security-hardener`, `secrets-scanner`, `threat-model-generator` |
| 3 | **OE Resilience Guardian** | oe-resilience-guardian.agent.md | 8-10 | `oe-compliance-engine`, `reliability-strategy-builder`, `scalability-playbook`, `api-design`, `api-versioning-deprecation-planner` |
| 4 | **OE Code Quality Sentinel** | oe-code-quality-sentinel.agent.md | 11-17 | `oe-compliance-engine`, `input-validation-sanitization-auditor` |
| 5 | **OE Test Coverage Enforcer** | oe-test-coverage-enforcer.agent.md | 18-20 | `oe-compliance-engine`, `unit-testing`, `e2e-testing`, `contract-testing-builder`, `coverage-strategist` |
| 6 | **OE Observability Hawk** | oe-observability-hawk.agent.md | 21-27 | `oe-compliance-engine`, `observability-setup`, `structured-logging-standardizer` |
| 7 | **OE Deployment Strategist** | oe-deployment-strategist.agent.md | 28-30 | `oe-compliance-engine`, `deployment-checklist-generator`, `github-actions-pipeline-creator`, `rollback-workflow-builder`, `release-automation-builder` |
| 8 | **OE Command Center** | oe-command-center.agent.md | 31-43 | `oe-compliance-engine`, `reliability-strategy-builder`, `scalability-playbook`, `auth-security-reviewer`, `api-design`, `api-versioning-deprecation-planner`, `api-security-hardener`, `observability-setup`, `contract-testing-builder`, `input-validation-sanitization-auditor`, `secrets-scanner`, `openapi-generator`, `structured-logging-standardizer`, `threat-model-generator` |

---

### Skill (Shared Scoring Engine)

| Name | File Path | Type | Skill Dependencies |
|------|-----------|------|--------------------|
| **OE Compliance Engine** | SKILL.md | Skill | `conventions`, `architecture-patterns`, `critical-partner`, `process-documentation`, `english-writing` |

---

### Dependency Graph

```
User invokes
    └── API OE Checklist Agent (entry point)
            ├── OE Architecture Auditor ──── Items 1-3
            ├── OE Security Fortress ─────── Items 4-7
            ├── OE Resilience Guardian ───── Items 8-10
            ├── OE Code Quality Sentinel ─── Items 11-17
            ├── OE Test Coverage Enforcer ── Items 18-20
            ├── OE Observability Hawk ────── Items 21-27
            ├── OE Deployment Strategist ─── Items 28-30
            └── OE Command Center ────────── Items 31-43
                                                 │
                               All load: oe-compliance-engine (SKILL)
```

**Total: 9 agents + 1 skill | 43 checklist items | 19 unique skill dependencies**