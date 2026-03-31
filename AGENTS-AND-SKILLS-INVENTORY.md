# Agents & Skills Inventory — mbbmy-dbdo-app-nextgen

> **Generated:** 2026-03-31 | **Agents:** 76 files (73 agents + 3 governance) | **Skills:** 273+ | **Instructions:** 21

---

## Table of Contents

1. [Governance Files](#1-governance-files)
2. [Core Pipeline Agents (00–11)](#2-core-pipeline-agents)
3. [Standalone Agents (61)](#3-standalone-agents)
4. [Design Skills — .agents/skills/ (20)](#4-design-skills)
5. [Workflow Skills — .superpowers/skills/ (14)](#5-workflow-skills)
6. [Domain Skills — .github/skills/ (221+)](#6-domain-skills)
7. [User-Level Skills](#7-user-level-skills)
8. [Instruction Files (21)](#8-instruction-files)
9. [Full Dependency Map](#9-full-dependency-map)

---

## 1. Governance Files

Located in `.github/agents/`. These define the system's operating rules and override all other documents.

| File | Priority | Purpose |
|------|----------|---------|
| **CONTRACT.md** | HIGHEST | Universal I/O JSON schemas, status enums, artifact validation gates, `tasks.yaml` lifecycle, `status.json` schema, ASK_USER protocol, role boundaries |
| **WORKFLOW.md** | HIGH | State machine (INTAKE→DONE), full/lean modes, per-state agents/artifacts/gates, repair loops (max 3 retries), Definition of Done |
| **DISPATCH-REFERENCE.md** | HIGH | Exact dispatch template, pre-dispatch checklist, mandatory `context_files` per agent, subagent failure policy (2 retries then BLOCKED) |

**Precedence:** CONTRACT.md → Agent Specs → WORKFLOW.md / DISPATCH-REFERENCE.md → copilot-instructions.md (lowest)

---

## 2. Core Pipeline Agents

12 agents forming the orchestrated state machine in `.github/agents/`.

### 00 — Orchestrator

| Property | Value |
|----------|-------|
| **File** | `00-orchestrator.md` |
| **Model** | GPT-5.3-Codex |
| **Role** | State machine controller — **never writes code** |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo`, `read_file`, `ask_questions`, `runSubagent` |

**Subagents dispatched (11):**

| # | Subagent | When Dispatched |
|---|----------|----------------|
| 1 | SpecAgent | INTAKE / INTAKE_LEAN |
| 2 | Researcher | DESIGN (when task requires tech/pattern investigation) |
| 3 | Architect | DESIGN |
| 4 | Designer | DESIGN (when task involves UI/UX) |
| 5 | Planner | PLAN |
| 6 | Coder | IMPLEMENT_LOOP (for each ready task) |
| 7 | Reviewer | IMPLEMENT_LOOP (per-batch or final meta review) |
| 8 | QA | IMPLEMENT_LOOP (if task concerns behavior/logic) |
| 9 | Security | IMPLEMENT_LOOP (if risk_flags includes security) |
| 10 | Integrator | INTEGRATE |
| 11 | Docs | RELEASE |

**Skills:** None  
**Files read:** `DISPATCH-REFERENCE.md` (before EVERY dispatch), `copilot-instructions.md` (session start), `status.json`  
**Files written:** All session artifacts in `.agents-work/<session>/`

---

### 01 — SpecAgent

| Property | Value |
|----------|-------|
| **File** | `01-spec-agent.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Produces** | `spec.md`, `acceptance.json`, `status.json` (+`tasks.yaml` in lean mode) |
| **Recommends next** | Architect (full mode), Coder (lean mode) |

---

### 02 — Architect

| Property | Value |
|----------|-------|
| **File** | `02-architect.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | `spec.md`, `acceptance.json`, research report (if any) |
| **Produces** | `architecture.md`, `adr/ADR-XXX.md` |
| **Recommends next** | Planner |

---

### 03 — Planner

| Property | Value |
|----------|-------|
| **File** | `03-planner.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | `spec.md`, `acceptance.json`, `architecture.md` |
| **Produces** | `tasks.yaml` |
| **Recommends next** | Coder |

---

### 04 — Coder

| Property | Value |
|----------|-------|
| **File** | `04-coder.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo`, `apply_patch`, `run_cmd` |
| **Reads** | `tasks.yaml`, `spec.md`, `architecture.md`, `status.json`, design-spec (**mandatory if exists**) |
| **Produces** | Source code, test files; updates `tasks.yaml` status (→`implemented`), `status.json` |
| **Recommends next** | Reviewer |

---

### 05 — Reviewer

| Property | Value |
|----------|-------|
| **File** | `05-reviewer.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | `spec.md`, `architecture.md`, `tasks.yaml`, `design-specs/`, `session_changed_files` |
| **Produces** | Review findings (blocker/major/minor/nit) |
| **Recommends next** | QA, Coder (FIX_REVIEW), or Integrator |

---

### 06 — QA

| Property | Value |
|----------|-------|
| **File** | `06-qa.md` |
| **Model** | Gemini 3 Pro (Preview) |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo`, `run_cmd` |
| **Reads** | `spec.md`, `acceptance.json`, `tasks.yaml`, `design-specs/`, `copilot-instructions.md` |
| **Produces** | Test files in repo |
| **Recommends next** | Integrator or Coder (FIX_TESTS) |

---

### 07 — Security

| Property | Value |
|----------|-------|
| **File** | `07-security.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | Change diff, `task.risk_flags`, `project_type`, `architecture.md` |
| **Produces** | Security findings with severity (critical/high → BLOCKED, medium → NEEDS_DECISION) |
| **Recommends next** | Coder (FIX_SECURITY), Reviewer, Integrator, or Orchestrator |

---

### 08 — Integrator

| Property | Value |
|----------|-------|
| **File** | `08-integrator.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | `repo_state`, completed tasks, CI logs |
| **Produces** | Build/config fixes |
| **Recommends next** | Docs or Orchestrator |

---

### 09 — Docs

| Property | Value |
|----------|-------|
| **File** | `09-docs.md` |
| **Model** | Claude Haiku 4.5 |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | `repo_structure`, completed tasks, `acceptance_checks`, build/test commands |
| **Produces** | `README.md`, `report.md`, changelog/release notes |
| **Recommends next** | Integrator or Orchestrator |

---

### 10 — Designer

| Property | Value |
|----------|-------|
| **File** | `10-designer.md` |
| **Model** | Gemini 3 Pro (Preview) |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo` |
| **Reads** | `spec.md`, `architecture.md`, `acceptance.json`, `copilot-instructions.md`, existing UI files |
| **Produces** | `design-specs/design-spec-<feature-slug>.md` |
| **Recommends next** | Coder |

---

### 11 — Researcher

| Property | Value |
|----------|-------|
| **File** | `11-researcher.md` |
| **Model** | GPT-5.3-Codex |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode`, `execute`, `read`, `agent`, `edit`, `search`, `web`, `todo`, external sources |
| **Reads** | `tasks.yaml`, `repo_state` |
| **Produces** | `research/research-<topic-slug>.md` |
| **Recommends next** | Architect, Planner, Coder, SpecAgent, or Orchestrator |

---

## 3. Standalone Agents

61 specialized agents in `.github/agents/`. Each entry documents subagents, skills, escalations, and tools.

---

### 3.1 Code Quality & Review (6)

#### Code Reviewer
| Property | Value |
|----------|-------|
| **File** | `code-reviewer.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | None |
| **Escalates to** | None |
| **Tools** | `read`, `execute`, `search`, `todo` |

#### Clean Code
| Property | Value |
|----------|-------|
| **File** | `clean-code.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | Standard toolset |

#### Code Alchemist
| Property | Value |
|----------|-------|
| **File** | `code-alchemist.agent.md` |
| **Model** | Claude Sonnet 4 |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | Standard toolset |

#### Janitor
| Property | Value |
|----------|-------|
| **File** | `janitor.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `search/changes`, `search/codebase`, `edit/editFiles`, `web/fetch`, `findTestFiles`, `web/githubRepo`, `runTests`, `search`, `usages`, `vscodeAPI`, `github`, `microsoft.docs.mcp` |

#### Refactor Cleaner
| Property | Value |
|----------|-------|
| **File** | `refactor-cleaner.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | None |
| **Escalates to** | None |
| **Tools** | `read`, `edit`, `execute`, `search`, `agent`, `todo` |
| **External tools** | `npx knip`, `npx depcheck`, `npx ts-prune`, `npx eslint` |

#### Tech Debt Plan
| Property | Value |
|----------|-------|
| **File** | `tech-debt-remediation-plan.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `changes`, `codebase`, `edit/editFiles`, `web/fetch`, `findTestFiles`, `githubRepo`, `runTests`, `search`, `usages`, `vscodeAPI`, `github` |

---

### 3.2 Security & Compliance (4)

#### SE: Security Reviewer
| Property | Value |
|----------|-------|
| **File** | `se-security-reviewer.agent.md` |
| **Model** | GPT-5.1-Codex-Max |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `search/codebase`, `edit/editFiles`, `search`, `read/problems` |

#### Security Reviewer
| Property | Value |
|----------|-------|
| **File** | `security-reviewer.md` |
| **Model** | sonnet |
| **Subagents** | None |
| **Skills** | **`security-review`** |
| **Tools** | `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob` |
| **External tools** | `npm audit --audit-level=high`, `npx eslint . --plugin security` |

#### Compliance Auditor
| Property | Value |
|----------|-------|
| **File** | `compliance-auditor.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | Standard toolset |

#### CODE-SENTINEL
| Property | Value |
|----------|-------|
| **File** | `wg-code-sentinel.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `changes`, `codebase`, `edit/editFiles`, `fetch`, `findTestFiles`, `githubRepo`, `runCommands`, `runTasks`, `search`, `usages`, `vscodeAPI` |

---

### 3.3 Test-Driven Development (3)

#### TDD Guide
| Property | Value |
|----------|-------|
| **File** | `tdd-guide.md` |
| **Model** | sonnet |
| **Subagents** | None |
| **Skills** | **`tdd-workflow`** |
| **Tools** | `Read`, `Write`, `Edit`, `Bash`, `Grep` |

#### TDD Red Phase
| Property | Value |
|----------|-------|
| **File** | `tdd-red.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None (references `tdd-workflow` implicitly) |
| **Tools** | `github`, `findTestFiles`, `edit/editFiles`, `runTests`, `runCommands`, `codebase`, `filesystem`, `search`, `problems`, `testFailure`, `terminalLastCommand` |

#### TDD Green Phase
| Property | Value |
|----------|-------|
| **File** | `tdd-green.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | **`tdd-workflow`** |
| **Tools** | `github`, `findTestFiles`, `edit/editFiles`, `runTests`, `runCommands`, `codebase`, `filesystem`, `search`, `problems`, `testFailure`, `terminalLastCommand` |

---

### 3.4 Language-Specific Reviewers (4)

#### TypeScript Reviewer
| Property | Value |
|----------|-------|
| **File** | `typescript-reviewer.md` |
| **Model** | sonnet |
| **Subagents** | None |
| **Skills** | **`coding-standards`**, **`frontend-patterns`**, **`backend-patterns`** |
| **Tools** | `Read`, `Grep`, `Glob`, `Bash` |

#### Java Reviewer
| Property | Value |
|----------|-------|
| **File** | `java-reviewer.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | **`springboot-patterns`** |
| **Escalates to** | **`security-reviewer`** (on CRITICAL security issues) |
| **Tools** | `read`, `execute`, `search`, `todo` |

#### Kotlin Reviewer
| Property | Value |
|----------|-------|
| **File** | `kotlin-reviewer.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | None |
| **Escalates to** | **`security-reviewer`** (on CRITICAL security issues) |
| **Tools** | `read`, `execute`, `search`, `todo` |

#### Database Reviewer
| Property | Value |
|----------|-------|
| **File** | `database-reviewer.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | Standard toolset |

---

### 3.5 DevOps & Deployment (5)

#### Deployment
| Property | Value |
|----------|-------|
| **File** | `deployment.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | None |
| **Skills** | None |

#### DevOps Expert
| Property | Value |
|----------|-------|
| **File** | `devops-expert.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |

#### GitHub Actions Expert
| Property | Value |
|----------|-------|
| **File** | `github-actions-expert.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |

#### Java Build Resolver
| Property | Value |
|----------|-------|
| **File** | `java-build-resolver.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | **`springboot-patterns`** |
| **Tools** | `read`, `edit`, `execute`, `search`, `agent`, `todo` |

#### Kotlin Build Resolver
| Property | Value |
|----------|-------|
| **File** | `kotlin-build-resolver.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | **`kotlin-patterns`** |
| **Tools** | `read`, `edit`, `execute`, `search`, `agent`, `todo` |

---

### 3.6 Documentation & Knowledge (5)

#### Doc Updater v2
| Property | Value |
|----------|-------|
| **File** | `doc-updater.agent.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | None |

#### Doc Updater
| Property | Value |
|----------|-------|
| **File** | `doc-updater.md` |
| **Model** | Claude Haiku 4.5 |
| **Subagents** | None |
| **Skills** | None |

#### ADR Generator
| Property | Value |
|----------|-------|
| **File** | `adr-generator.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |

#### Instruction Generator
| Property | Value |
|----------|-------|
| **File** | `instruction-generator.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | None |
| **Skills** | None |

#### Mentor
| Property | Value |
|----------|-------|
| **File** | `mentor.agent.md` |
| **Model** | — |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `codebase`, `web/fetch`, `findTestFiles`, `githubRepo`, `search`, `usages` |

---

### 3.7 Software Engineering (7)

#### Principal SE
| Property | Value |
|----------|-------|
| **File** | `principal-software-engineer.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | Full VS Code toolset + `github` |

#### Software Engineer v1
| Property | Value |
|----------|-------|
| **File** | `software-engineer-agent-v1.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | Full VS Code toolset + `github` (23 tools) |

#### Expert React FE
| Property | Value |
|----------|-------|
| **File** | `expert-react-frontend-engineer.agent.md` |
| **Subagents** | None |
| **Skills** | None |

#### Implementation Plan
| Property | Value |
|----------|-------|
| **File** | `implementation-plan.agent.md` |
| **Subagents** | None |
| **Skills** | None |

#### Loop Operator
| Property | Value |
|----------|-------|
| **File** | `loop-operator.md` |
| **Model** | Claude Sonnet 4.6 |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `vscode/askQuestions`, `execute`, `read`, `agent`, `search`, `todo` |

#### Architect (standalone)
| Property | Value |
|----------|-------|
| **File** | `architect.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | None |
| **Skills** | None |

#### Planner (standalone)
| Property | Value |
|----------|-------|
| **File** | `planner.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | None |
| **Skills** | None |

---

### 3.8 Operational Excellence (10)

#### OE Command Center ⭐ (Supreme Orchestrator)

| Property | Value |
|----------|-------|
| **File** | `oe-command-center.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | **`agents: ["*"]`** — dispatches all available agents |
| **Tools** | `execute`, `read`, `edit/createFile`, `search`, `agent`, `edit/editFiles`, `edit/createDirectory`, `todo` |

**Subagents dispatched in parallel:**

| Subagent | OE Items |
|----------|:--------:|
| OE Architecture Auditor | 1-3 |
| OE Security Fortress | 4-7 |
| OE Resilience Guardian | 8-10, 31-43 |
| OE Code Quality Sentinel | 11-17 |
| OE Test Coverage Enforcer | 18-20 |
| OE Observability Hawk | 21-27 |
| OE Deployment Strategist | 28-30 |

**Mandatory skills (14):**

| # | Skill | Category |
|---|-------|----------|
| 1 | **`oe-compliance-engine`** | Core (ALWAYS loaded first) |
| 2 | **`reliability-strategy-builder`** | Resilience |
| 3 | **`scalability-playbook`** | Resilience |
| 4 | **`auth-security-reviewer`** | Security |
| 5 | **`api-design`** | Architecture |
| 6 | **`api-versioning-deprecation-planner`** | Architecture |
| 7 | **`api-security-hardener`** | Security |
| 8 | **`observability-setup`** | Observability |
| 9 | **`contract-testing-builder`** | Testing |
| 10 | **`input-validation-sanitization-auditor`** | Code Quality |
| 11 | **`secrets-scanner`** | Security |
| 12 | **`openapi-generator`** | Architecture |
| 13 | **`structured-logging-standardizer`** | Observability |
| 14 | **`threat-model-generator`** | Security |

---

#### OE Architecture Auditor

| Property | Value |
|----------|-------|
| **File** | `oe-architecture-auditor.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Skills (4)** | **`oe-compliance-engine`**, **`architecture-patterns`**, **`openapi-generator`**, **`api-design`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### OE Code Quality Sentinel

| Property | Value |
|----------|-------|
| **File** | `oe-code-quality-sentinel.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Skills (3)** | **`oe-compliance-engine`**, **`input-validation-sanitization-auditor`**, **`coding-standards`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### OE Deployment Strategist

| Property | Value |
|----------|-------|
| **File** | `oe-deployment-strategist.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Skills (5)** | **`oe-compliance-engine`**, **`deployment-checklist-generator`**, **`github-actions-pipeline-creator`**, **`rollback-workflow-builder`**, **`release-automation-builder`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### OE Observability Hawk

| Property | Value |
|----------|-------|
| **File** | `oe-observability-hawk.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Skills (3)** | **`oe-compliance-engine`**, **`observability-setup`**, **`structured-logging-standardizer`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### OE Resilience Guardian

| Property | Value |
|----------|-------|
| **File** | `oe-resilience-guardian.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Coordinates with** | **`oe-deployment-strategist`** (items 28-30 delegated) |
| **Skills (5)** | **`oe-compliance-engine`**, **`reliability-strategy-builder`**, **`scalability-playbook`**, **`api-design`**, **`api-versioning-deprecation-planner`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### OE Security Fortress

| Property | Value |
|----------|-------|
| **File** | `oe-security-fortress.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Skills (5)** | **`oe-compliance-engine`**, **`auth-security-reviewer`**, **`api-security-hardener`**, **`secrets-scanner`**, **`threat-model-generator`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### OE Test Coverage Enforcer

| Property | Value |
|----------|-------|
| **File** | `oe-test-coverage-enforcer.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | `agents: ["*"]` |
| **Skills (5)** | **`oe-compliance-engine`**, **`unit-testing`**, **`e2e-testing`**, **`contract-testing-builder`**, **`coverage-strategist`** |
| **Tools** | `read`, `search`, `execute`, `agent`, `todo` |

#### API OE Checklist

| Property | Value |
|----------|-------|
| **File** | `api-oe-checklist.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Subagents** | None |
| **Skills** | None |

#### Microservice API OE Checklist

| Property | Value |
|----------|-------|
| **File** | `microservice-api-oe-checklist.agent.md` |
| **Model** | Claude Opus 4.6 |
| **Tools** | `execute`, `read`, `edit/createFile`, `search`, `agent`, `edit/editFiles`, `edit/createDirectory`, `todo` |

**Subagents dispatched (8):**

| # | Subagent |
|---|----------|
| 1 | Architecture Review Agent |
| 2 | Security Agent |
| 3 | Code Reviewer |
| 4 | Software Quality Assurance Agent |
| 5 | Observability Agent |
| 6 | Deployment Agent |
| 7 | Database Reviewer |
| 8 | Compliance Auditor Agent |

**Mandatory skills (3):**
- **`reliability-strategy-builder`**
- **`scalability-playbook`**
- **`auth-security-reviewer`**

---

### 3.9 SpecKit Workflow (9)

Each SpecKit agent defines explicit **handoffs** (subagent routing) in its frontmatter.

#### speckit.specify
| Property | Value |
|----------|-------|
| **File** | `speckit.specify.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Hands off to** | **`speckit.plan`** ("Build Technical Plan"), **`speckit.clarify`** ("Clarify Spec Requirements") |
| **Scripts** | `.specify/scripts/bash/create-new-feature.sh --json` |

#### speckit.clarify
| Property | Value |
|----------|-------|
| **File** | `speckit.clarify.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Hands off to** | **`speckit.plan`** ("Build Technical Plan") |
| **Scripts** | `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` |

#### speckit.plan
| Property | Value |
|----------|-------|
| **File** | `speckit.plan.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Hands off to** | **`speckit.tasks`** ("Create Tasks"), **`speckit.checklist`** ("Create Checklist") |
| **Scripts** | `.specify/scripts/bash/setup-plan.sh --json`, `.specify/scripts/bash/update-agent-context.sh copilot` |

#### speckit.tasks
| Property | Value |
|----------|-------|
| **File** | `speckit.tasks.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Hands off to** | **`speckit.analyze`** ("Analyze For Consistency"), **`speckit.implement`** ("Implement Project") |
| **Scripts** | `.specify/scripts/bash/check-prerequisites.sh --json` |

#### speckit.analyze
| Property | Value |
|----------|-------|
| **File** | `speckit.analyze.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Recommends** | `/speckit.implement`, `/speckit.specify`, `/speckit.plan` for remediation |
| **Scripts** | `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` |

#### speckit.checklist
| Property | Value |
|----------|-------|
| **File** | `speckit.checklist.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Templates** | `.specify/templates/checklist-template.md` |

#### speckit.implement
| Property | Value |
|----------|-------|
| **File** | `speckit.implement.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Scripts** | `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` |

#### speckit.constitution
| Property | Value |
|----------|-------|
| **File** | `speckit.constitution.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Hands off to** | **`speckit.specify`** ("Build Specification") |
| **Writes** | `.specify/memory/constitution.md` |

#### speckit.taskstoissues
| Property | Value |
|----------|-------|
| **File** | `speckit.taskstoissues.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **Tools** | `github/github-mcp-server/issue_write` |

---

### 3.10 Accessibility & i18n (2)

#### Accessibility Expert
| Property | Value |
|----------|-------|
| **File** | `accessibility.agent.md` |
| **Model** | GPT-4.1 |
| **Subagents** | None |
| **Skills** | None |

#### Lingo.dev i18n
| Property | Value |
|----------|-------|
| **File** | `lingodotdev-i18n.agent.md` |
| **Subagents** | None |
| **Skills** | None |
| **MCP Server** | **`lingo`** (SSE at `https://mcp.lingo.dev/main`) |
| **Tools** | `shell`, `read`, `edit`, `search`, `lingo/*` |

---

### 3.11 Agent & Framework Tooling (4)

#### Agent Generator
| Property | Value |
|----------|-------|
| **File** | `agent-generator.agent.md` |
| **Subagents** | None |
| **Skills** | None |

#### AI Agent Framework
| Property | Value |
|----------|-------|
| **File** | `ai-agent-framework.agent.md` |
| **Subagents** | None |
| **Skills** | None |

#### Custom Agent Foundry
| Property | Value |
|----------|-------|
| **File** | `custom-agent-foundry.agent.md` |
| **Model** | Claude Sonnet 4.5 |
| **Subagents** | None |
| **Skills** | None |

#### TypeScript MCP Expert
| Property | Value |
|----------|-------|
| **File** | `typescript-mcp-expert.agent.md` |
| **Model** | GPT-4.1 |
| **Subagents** | None |
| **Skills** | None |

---

### 3.12 E2E Testing (1)

#### Playwright Tester
| Property | Value |
|----------|-------|
| **File** | `playwright-tester.agent.md` |
| **Model** | Claude Sonnet 4 |
| **Subagents** | None |
| **Skills** | None |
| **MCP** | `playwright` |
| **Tools** | `changes`, `codebase`, `edit/editFiles`, `fetch`, `findTestFiles`, `runTests`, `search`, `testFailure` |

---

## 4. Design Skills

Located in `.agents/skills/` — installed from `pbakaus/impeccable` (tracked in `skills-lock.json`):

| # | Skill | Category | Description | Depends On |
|---|-------|----------|-------------|------------|
| 1 | `animate` | Motion | Purposeful animations, micro-interactions | **`frontend-design`** |
| 2 | `arrange` | Layout | Spacing, visual rhythm, alignment | **`frontend-design`** |
| 3 | `audit` | QA | Technical quality (a11y, perf, theming) | None |
| 4 | `bolder` | Visual | Amplify safe designs for impact | **`frontend-design`** |
| 5 | `clarify` | UX Writing | Error messages, microcopy, labels | **`frontend-design`** |
| 6 | `colorize` | Color | Strategic color for monochrome UIs | **`frontend-design`** |
| 7 | `critique` | Review | UX evaluation with quantitative scoring | **`frontend-design`** |
| 8 | `delight` | UX | Joy, personality, memorable interactions | **`frontend-design`** |
| 9 | `distill` | Simplify | Strip to essence, remove complexity | **`frontend-design`** |
| 10 | `extract` | Systems | Consolidate reusable components/tokens | **`frontend-design`** |
| 11 | `frontend-design` | **CORE** | Distinctive production-grade interfaces | None |
| 12 | `harden` | Resilience | Error handling, i18n, edge cases | **`frontend-design`** |
| 13 | `normalize` | Systems | Realign to design system standards | **`frontend-design`** |
| 14 | `onboard` | UX | Onboarding flows, empty states, first-run | **`frontend-design`** |
| 15 | `optimize` | Performance | UI loading, rendering, animations, bundle | **`frontend-design`** |
| 16 | `overdrive` | Advanced | Shaders, spring physics, 60fps | **`frontend-design`** |
| 17 | `polish` | Quality | Final pass: alignment, spacing, micro-details | **`frontend-design`** |
| 18 | `quieter` | Visual | Tone down overstimulating designs | **`frontend-design`** |
| 19 | `teach-impeccable` | **SETUP** | One-time design context gathering | None |
| 20 | `typeset` | Typography | Fonts, hierarchy, sizing, readability | **`frontend-design`** |

---

## 5. Workflow Skills

Located in `.superpowers/skills/` (also symlinked to `~/.copilot/skills/`):

| # | Skill | Category | Description | Mandatory? |
|---|-------|----------|-------------|:----------:|
| 1 | `brainstorming` | Planning | Explore intent before implementation | **YES** (before creative work) |
| 2 | `dispatching-parallel-agents` | Multi-Agent | Launch 2+ independent tasks in parallel | No |
| 3 | `executing-plans` | Execution | Execute plans in separate sessions with reviews | No |
| 4 | `finishing-a-development-branch` | Git | Decide: merge, PR, or cleanup | No |
| 5 | `receiving-code-review` | Review | Handle review feedback with technical rigor | No |
| 6 | `requesting-code-review` | Review | Submit work for verification | No |
| 7 | `subagent-driven-development` | Multi-Agent | Fresh subagent per task in current session | No |
| 8 | `systematic-debugging` | Debugging | Structured bug investigation methodology | No |
| 9 | `test-driven-development` | Testing | TDD workflow enforcement | No |
| 10 | `using-git-worktrees` | Git | Isolated feature work via git worktrees | No |
| 11 | `using-superpowers` | META | Discover and use skills | **YES** (every session start) |
| 12 | `verification-before-completion` | QA | Evidence before completion assertions | No |
| 13 | `writing-plans` | Planning | Multi-step implementation plans | No |
| 14 | `writing-skills` | Development | Create/edit/verify skills | No |

---

## 6. Domain Skills

221+ skills in `.github/skills/`. Organized by category with the skills referenced by agents highlighted.

### Skills referenced by agents (marked with ⭐)

| Skill | Referenced By Agent(s) |
|-------|----------------------|
| ⭐ `oe-compliance-engine` | OE Command Center, OE Architecture Auditor, OE Code Quality Sentinel, OE Deployment Strategist, OE Observability Hawk, OE Resilience Guardian, OE Security Fortress, OE Test Coverage Enforcer |
| ⭐ `reliability-strategy-builder` | OE Command Center, OE Resilience Guardian, Microservice API OE |
| ⭐ `scalability-playbook` | OE Command Center, OE Resilience Guardian, Microservice API OE |
| ⭐ `auth-security-reviewer` | OE Command Center, OE Security Fortress, Microservice API OE |
| ⭐ `api-design` | OE Command Center, OE Architecture Auditor, OE Resilience Guardian |
| ⭐ `api-versioning-deprecation-planner` | OE Command Center, OE Resilience Guardian |
| ⭐ `api-security-hardener` | OE Command Center, OE Security Fortress |
| ⭐ `observability-setup` | OE Command Center, OE Observability Hawk |
| ⭐ `contract-testing-builder` | OE Command Center, OE Test Coverage Enforcer |
| ⭐ `input-validation-sanitization-auditor` | OE Command Center, OE Code Quality Sentinel |
| ⭐ `secrets-scanner` | OE Command Center, OE Security Fortress |
| ⭐ `openapi-generator` | OE Command Center, OE Architecture Auditor |
| ⭐ `structured-logging-standardizer` | OE Command Center, OE Observability Hawk |
| ⭐ `threat-model-generator` | OE Command Center, OE Security Fortress |
| ⭐ `deployment-checklist-generator` | OE Deployment Strategist |
| ⭐ `github-actions-pipeline-creator` | OE Deployment Strategist |
| ⭐ `rollback-workflow-builder` | OE Deployment Strategist |
| ⭐ `release-automation-builder` | OE Deployment Strategist |
| ⭐ `architecture-patterns` | OE Architecture Auditor |
| ⭐ `coding-standards` | OE Code Quality Sentinel |
| ⭐ `unit-testing` | OE Test Coverage Enforcer |
| ⭐ `e2e-testing` | OE Test Coverage Enforcer |
| ⭐ `coverage-strategist` | OE Test Coverage Enforcer |
| ⭐ `springboot-patterns` | Java Build Resolver, Java Reviewer |
| ⭐ `kotlin-patterns` | Kotlin Build Resolver |
| ⭐ `security-review` | Security Reviewer |
| ⭐ `tdd-workflow` | TDD Guide, TDD Green Phase |
| ⭐ `coding-standards` | TypeScript Reviewer |
| ⭐ `frontend-patterns` | TypeScript Reviewer |
| ⭐ `backend-patterns` | TypeScript Reviewer |

### Full skill listing by category

**Accessibility (2):** `a11y` · `accessibility-auditor`

**AI/ML & Agent Engineering (13):** `advanced-evaluation` · `agent-creation` · `agent-eval` · `agent-harness-construction` · `agentic-engineering` · `ai-first-engineering` · `ai-regression-testing` · `autonomous-loops` · `continuous-learning` · `continuous-learning-v2` · `evaluation` · `hosted-agents` · `multi-agent-patterns`

**API & Backend (23):** `api-design` ⭐ · `api-docs-generator` · `api-mock-server` · `api-security-hardener` ⭐ · `api-test-suite-generator` · `api-versioning-deprecation-planner` ⭐ · `backend-dev` · `backend-patterns` ⭐ · `contract-testing-builder` ⭐ · `cors-configuration` · `database-migrations` · `jpa-patterns` · `nodejs` · `oauth2-oidc-implementer` · `observability-setup` ⭐ · `openapi-generator` ⭐ · `postman-collection-generator` · `rate-limiting-abuse-protection` · `reliability-strategy-builder` ⭐ · `scalability-playbook` ⭐ · `springboot-patterns` ⭐ · `springboot-security` · `springboot-tdd`

**Architecture & Patterns (10):** `architecture-decision-records` · `architecture-patterns` ⭐ · `blueprint` · `coding-standards` ⭐ · `conventions` · `event-driven-atchitect` · `migration-planner` · `system-design-generator`

**Context & LLM (5):** `context-budget` · `context-compression` · `context-degradation` · `context-fundamentals` · `context-optimization`

**Database (3):** `clickhouse-io` · `content-hash-cache-pattern` · `postgres-patterns`

**DevOps & CI/CD (15):** `canary-watch` · `continuous-agent-loop` · `cost-aware-llm-pipeline` · `deployment-checklist-generator` ⭐ · `dockerfile-optimizer` · `github-actions-pipeline-creator` ⭐ · `monorepo-ci-optimizer` · `production-scheduling` · `release-automation-builder` ⭐ · `rollback-workflow-builder` ⭐ · `verify-loop` · `dmux-workflows` · `configure-ecc`

**Documentation (12):** `readme-generator` · `changelog-writer` · `api-docs-generator` · `codebase-summarizer` · `codebase-onboarding` · `mermaid-diagram-generator` · `conventional-commits` · `explaining-code` · `process-documentation` · `confluence-cli` · `jira-cli` · `technical-communication`

**Frontend & React (29):** `react` · `react-native` · `react-native-best-practices` · `expo` · `mui` · `redux-toolkit` · `formik` · `tailwindcss` · `css` · `html` · `vite` · `webpack` · `ag-grid` · `astro` · `component-scaffold-generator` · `design-to-component-translator` · `page-layout-builder` · `responsive-design-system` · `state-ux-flow-builder` · `animation-micro-interaction-pack` · `frontend-dev` · `frontend-patterns` ⭐ · `frontend-refactor-planner` · `frontend-slides` · `i18n-frontend-implementer` · `storybook-setup` · `react-hook-builder` · `react-native-brownfield-migration` · `upgrading-react-native`

**OE & Compliance (3):** `oe-compliance-engine` ⭐ · `quality-nonconformance` · `quality-gates-enforcer`

**Performance (4):** `core-web-vitals-tuner` · `performance-budget-setter` · `load-test-builder` · `load-test-scenario-builder`

**Security (18):** `security-review` ⭐ · `security-scan` · `auth-security-reviewer` ⭐ · `secrets-scanner` ⭐ · `secrets-env-manager` · `input-validation-sanitization-auditor` ⭐ · `threat-model-generator` ⭐ · `secure-headers-csp-builder` · `rbac-policy-tester` · `oauth2-oidc-implementer` · `pii-redaction-logging-policy-builder` · `security-incident-playbook-generator` · `security-pr-checklist-skill` · `safety-guard` · `env-secrets-manager` · `dependency-vulnerability-triage`

**Testing (26):** `jest` · `playwright` · `react-testing-library` · `react-native-testing-library` · `tdd-workflow` ⭐ · `unit-test-generator` · `e2e-test-builder` · `e2e-testing` ⭐ · `unit-testing` ⭐ · `coverage-strategist` ⭐ · `contract-testing-builder` ⭐ · `mocking-assistant` · `snapshot-test-refactorer` · `integration-test-builder` · `load-test-builder` · `load-test-scenario-builder` · `test-data-factory-builder` · `test-reporting-triage-skill` · `visual-regression-tester` · `click-path-audit` · `eval-harness` · `rbac-policy-tester` · `stagehand` · `springboot-tdd` · `springboot-verification`

**TypeScript & JavaScript (7):** `typescript` · `javascript` · `eslint` · `prettier` · `zod` · `yup` · `eslint-prettier-config`

**Utilities & Other (30+):** `brainstorming` · `critical-partner` · `dependency-doctor` · `systematic-debugging` · `tech-debt-prioritizer` · `search-first` · `using-git-worktrees` · `humanizer` · `android-clean-architecture` · `compose-multiplatform-patterns` · `curl-command-generator` · `dev-onboarding-builder` · `postmortem-writer` · `structured-logging-standardizer` ⭐ · `kotlin-patterns` ⭐ · `bdi-mental-states` · `prompt-creation` · `prompt-optimizer` · `skill-creation` · `skill-comply` · `skill-sync` · `strategic-compact` · `santa-method` · `plankton-code-quality` · `ralphinho-rfc-pipeline` · `project-development` · `filesystem-context` · `planning-with-files` · `tool-design` · `memory-systems` · `verification-loop`

---

## 7. User-Level Skills

### `~/.copilot/skills/` (15 symlinks → `.superpowers/skills/`)

`brainstorming` · `dispatching-parallel-agents` · `executing-plans` · `finishing-a-development-branch` · `receiving-code-review` · `requesting-code-review` · `subagent-driven-development` · `systematic-debugging` · `test-driven-development` · `using-git-worktrees` · `using-superpowers` · `verification-before-completion` · `writing-plans` · `writing-skills`

### `~/.agents/skills/` (1)

| Skill | Purpose |
|-------|---------|
| `find-skills` | Discover and install skills via `npx skills` CLI |

---

## 8. Instruction Files

21 files in `.github/instructions/` + `.github/copilot-instructions.md`:

### Global (`**`)

| File | Focus |
|------|-------|
| `common-agents.instructions.md` | Agent orchestration, parallel execution, multi-perspective analysis |
| `common-coding-style.instructions.md` | Immutability, file org (<800 lines), error handling, validation |
| `common-development-workflow.instructions.md` | Plan → TDD → Review → Commit pipeline |
| `common-git-workflow.instructions.md` | Conventional commits, PR process |
| `common-patterns.instructions.md` | Repository pattern, API response envelope, skeleton projects |
| `common-security.instructions.md` | Mandatory security checks, secret management, response protocol |
| `common-testing.instructions.md` | 80% coverage, TDD workflow, unit/integration/e2e |

### Scoped

| File | applyTo | Focus |
|------|---------|-------|
| `architecture.instructions.md` | `src/**/*.{tsx,ts}` | Components <300 lines, single responsibility |
| `components.instructions.md` | `src/components/**/*.{tsx,ts}` | UI/common/feature component rules |
| `hooks.instructions.md` | `src/hooks/**/*.{ts,tsx}` | Data fetching, form, UI state hook patterns |
| `quality.instructions.md` | `src/**/*.{tsx,ts,css}` | Quality checklist, never-ship rules |
| `reusable.instructions.md` | `src/screens/**` | Extract components >100 lines |
| `performance-optimization.instructions.md` | `*` | Frontend/backend/DB performance |
| `reactjs.instructions.md` | `**/*.{jsx,tsx,js,ts,css,scss}` | React 19+ development standards |
| `markdown.instructions.md` | `**/*.md` | Documentation standards |

### Unscoped (Persona/Methodology)

| File | Focus |
|------|-------|
| `openant-security.instructions.md` | Security vulnerability analysis methodology |
| `rn-security.instructions.md` | React Native security (Keychain, SSL pinning, Zod) |
| `general-security.instructions.md` | Code review security focus areas |
| `responsive-cleanup.instructions.md` | Auto-generated responsive code cleanup |
| `agentic-chat.instructions.md` | Task description assistant for Copilot agents |

### Core Protocol

| File | Focus |
|------|-------|
| `copilot-instructions.md` | SUPERPOWERS PROTOCOL — autonomous Loop of Autonomy |

---

## 9. Full Dependency Map

### 9.1 Core Pipeline (Orchestrator dispatches)

```
Orchestrator (00)
 ├── reads: CONTRACT.md, WORKFLOW.md, DISPATCH-REFERENCE.md (every dispatch)
 ├── reads: copilot-instructions.md (session start)
 │
 ├─▶ SpecAgent (01)
 │     └── produces: spec.md, acceptance.json, status.json
 │
 ├─▶ Researcher (11) [optional]
 │     └── produces: research/<topic>.md
 │
 ├─▶ Architect (02) ← spec.md, acceptance.json, research
 │     └── produces: architecture.md, adr/ADR-*.md
 │
 ├─▶ Designer (10) [optional: UI tasks] ← spec.md, architecture.md
 │     └── produces: design-specs/design-spec-*.md
 │
 ├── ◆ APPROVE_DESIGN gate (user approval required)
 │
 ├─▶ Planner (03) ← spec.md, acceptance.json, architecture.md
 │     └── produces: tasks.yaml
 │
 ├── ◆ REVIEW_STRATEGY gate (per-batch | single-final)
 │
 ├── IMPLEMENT_LOOP:
 │   ├─▶ Coder (04) ← tasks.yaml, architecture, design-spec
 │   ├─▶ Reviewer (05) ← changed files, session_changed_files
 │   ├─▶ QA (06) ← spec.md, acceptance.json
 │   └─▶ Security (07) ← tasks.yaml, architecture
 │       └── [FIX loops if BLOCKED, max 3 retries each]
 │
 ├─▶ Integrator (08) ← tasks.yaml, acceptance.json
 └─▶ Docs (09) ← all artifacts
       └── produces: README.md, report.md, changelog
```

### 9.2 OE Command Center Fan-Out

```
OE Command Center
 │  Skills: oe-compliance-engine, reliability-strategy-builder, scalability-playbook,
 │          auth-security-reviewer, api-design, api-versioning-deprecation-planner,
 │          api-security-hardener, observability-setup, contract-testing-builder,
 │          input-validation-sanitization-auditor, secrets-scanner, openapi-generator,
 │          structured-logging-standardizer, threat-model-generator
 │
 ├─▶ OE Architecture Auditor (items 1-3)
 │     Skills: oe-compliance-engine, architecture-patterns, openapi-generator, api-design
 │
 ├─▶ OE Security Fortress (items 4-7)
 │     Skills: oe-compliance-engine, auth-security-reviewer, api-security-hardener,
 │             secrets-scanner, threat-model-generator
 │
 ├─▶ OE Resilience Guardian (items 8-10, 31-43)
 │     Skills: oe-compliance-engine, reliability-strategy-builder, scalability-playbook,
 │             api-design, api-versioning-deprecation-planner
 │     Coordinates with: oe-deployment-strategist (items 28-30)
 │
 ├─▶ OE Code Quality Sentinel (items 11-17)
 │     Skills: oe-compliance-engine, input-validation-sanitization-auditor, coding-standards
 │
 ├─▶ OE Test Coverage Enforcer (items 18-20)
 │     Skills: oe-compliance-engine, unit-testing, e2e-testing, contract-testing-builder,
 │             coverage-strategist
 │
 ├─▶ OE Observability Hawk (items 21-27)
 │     Skills: oe-compliance-engine, observability-setup, structured-logging-standardizer
 │
 └─▶ OE Deployment Strategist (items 28-30)
       Skills: oe-compliance-engine, deployment-checklist-generator,
               github-actions-pipeline-creator, rollback-workflow-builder,
               release-automation-builder
```

### 9.3 Microservice API OE Checklist Fan-Out

```
Microservice API OE Checklist
 │  Skills: reliability-strategy-builder, scalability-playbook, auth-security-reviewer
 │
 ├─▶ Architecture Review Agent
 ├─▶ Security Agent
 ├─▶ Code Reviewer
 ├─▶ Software Quality Assurance Agent
 ├─▶ Observability Agent
 ├─▶ Deployment Agent
 ├─▶ Database Reviewer
 └─▶ Compliance Auditor Agent
```

### 9.4 SpecKit Workflow Handoffs

```
speckit.constitution ──▶ speckit.specify
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
             speckit.clarify     speckit.plan
                    │               │
                    └───▶ speckit.plan
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
            speckit.tasks       speckit.checklist
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   speckit.analyze    speckit.implement
          │
          ▼
  speckit.taskstoissues (GitHub Issues via MCP)
```

### 9.5 Agent → Skill Cross-Reference

```
java-build-resolver ────▶ springboot-patterns
java-reviewer ──────────▶ springboot-patterns
                    └───▶ security-reviewer (escalation on CRITICAL)

kotlin-build-resolver ──▶ kotlin-patterns
kotlin-reviewer ────────▶ security-reviewer (escalation on CRITICAL)

typescript-reviewer ────▶ coding-standards, frontend-patterns, backend-patterns
security-reviewer ──────▶ security-review
tdd-guide ──────────────▶ tdd-workflow
tdd-green ──────────────▶ tdd-workflow
```

### 9.6 Design Skills Dependency Tree

```
teach-impeccable (run FIRST — one-time setup)
  │
  └── frontend-design (CORE — required by 17 others)
        ├── animate, arrange, bolder, clarify, colorize
        ├── critique, delight, distill, extract
        ├── harden, normalize, onboard, optimize
        ├── overdrive, polish, quieter, typeset
        │
        └── audit (INDEPENDENT — no frontend-design dependency)
```

### 9.7 Workflow Skills Chains

```
using-superpowers (MANDATORY — every session start)

brainstorming (MANDATORY — before creative work)
  └──▶ writing-plans
        ├──▶ executing-plans (parallel sessions)
        └──▶ subagent-driven-development (current session)
              └──▶ dispatching-parallel-agents
                    └──▶ verification-before-completion
                          └──▶ requesting-code-review
                                └──▶ receiving-code-review
                                      └──▶ finishing-a-development-branch

systematic-debugging (standalone — bug investigation)
test-driven-development (standalone — RED/GREEN/REFACTOR)
using-git-worktrees (standalone — feature isolation)
writing-skills (standalone — skill development)
```

---

**End of inventory.**
