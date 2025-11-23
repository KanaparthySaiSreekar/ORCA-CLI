# ORCA-CLI Implementation Plan

**Version:** 2.0
**Date:** 2025-11-23
**Status:** Planning Phase - Research-Driven Architecture

---

## ⚠️ CRITICAL: Agent Architecture First

**Research Finding (2025-11-23):** The effectiveness of a coding agent is primarily determined by its **architecture**, **tool design**, and **agent loop** rather than the LLM alone.

**Evidence:**
- SWE-agent achieved **12.5%** on SWE-bench (vs 3.8% previous SOTA) through Agent-Computer Interface (ACI) design, **not model improvements**
- ReAct pattern achieves **86%** success rate with proper implementation
- Aider handles enterprise codebases (2000+ files) through AST-based repository mapping
- Tool design has more impact than model selection

**👉 See [AGENT_ARCHITECTURE_RESEARCH.md](./AGENT_ARCHITECTURE_RESEARCH.md) for comprehensive research findings.**

---

## Executive Summary

This document outlines the complete implementation plan for ORCA-CLI, an OpenRouter-Compatible Agentic Coding Assistant. The project is divided into 4 major phases with **20 work streams** and approximately **180+ distinct tasks** (including enhancements).

**Estimated Total Effort:** 8-12 months (with team of 2-3 developers)
**Current Phase:** Phase 0 - Research & Foundation

**Architecture Philosophy (Updated 2025-11-23):**
1. **Agent-First Design**: Tools and architecture designed specifically for LLM agents
2. **Hybrid Architecture**: Deterministic core + non-deterministic reasoning
3. **Compact Interfaces**: Token-efficient tool design (SWE-agent principle)
4. **Context Intelligence**: Just-in-time loading with repository mapping
5. **Self-Correction Loops**: Built-in error recovery and iteration

### Major Enhancements (Version 2.0 - 2025-11-23)

This plan has been significantly enhanced with advanced capabilities based on comprehensive analysis of modern coding agent requirements:

**New Capabilities Added:**
- ✅ **Advanced Code Intelligence** - Call graphs, impact analysis, complexity metrics, dead code detection
- ✅ **Enhanced Planning** - Learning system, plan templates, parallel execution, predictive estimation
- ✅ **AI-Powered Testing** - Test generation, coverage gap analysis, mutation testing, flaky test detection
- ✅ **Intelligent Refactoring** - Pattern-based refactoring, code review automation, documentation generation
- ✅ **Developer Experience** - Interactive debugging, enhanced errors, plan visualization, IDE integration
- ✅ **Cloud & Infrastructure** - IaC generation, deployment automation, database migrations, API contracts
- ✅ **Enterprise Features** - Team collaboration, governance, multi-repo support

**See [AGENT_IMPROVEMENTS.md](./AGENT_IMPROVEMENTS.md) for detailed analysis and specifications.**

---

## Table of Contents

1. [Phase Overview](#phase-overview)
2. [Work Streams](#work-streams)
3. [Detailed Task Breakdown](#detailed-task-breakdown)
4. [Dependency Graph](#dependency-graph)
5. [Technology Stack](#technology-stack)
6. [Success Criteria](#success-criteria)
7. [Risk Assessment](#risk-assessment)

---

## Phase Overview

### Phase 0: Research & Foundation (Weeks 1-2)
**Goal:** Research technologies, define architecture, setup project infrastructure

- **Duration:** 2 weeks
- **Team Size:** 2-3 developers
- **Key Deliverables:**
  - Technology selection decisions
  - Project structure and architecture
  - Development environment setup
  - Initial package structure

### Phase 1: Core MVP (Weeks 3-8)
**Goal:** Build minimum viable product with basic CLI, planning, execution, and OpenRouter integration

- **Duration:** 6 weeks
- **Team Size:** 2-3 developers
- **Key Deliverables:**
  - Working CLI with core commands
  - OpenRouter integration
  - Basic planning and execution engine
  - Multi-file editing capabilities
  - Sandbox execution

### Phase 2: Advanced Features (Weeks 9-16)
**Goal:** Add RAG, testing, verification, and tool integrations

- **Duration:** 8 weeks
- **Team Size:** 3-4 developers
- **Key Deliverables:**
  - RAG indexing and retrieval
  - Test runner integration
  - Verification engine
  - Static analysis tools
  - State persistence

### Phase 3: Production Ready (Weeks 17-24)
**Goal:** Enterprise features, CI integration, observability, security

- **Duration:** 8 weeks
- **Team Size:** 3-4 developers
- **Key Deliverables:**
  - Advanced verification
  - Multi-model pipelines
  - CI integration
  - Comprehensive observability
  - Security hardening
  - Documentation

### Phase 4: Future (Post-MVP)
**Goal:** Plugins, marketplace, enterprise features

- **Duration:** Ongoing
- **Key Deliverables:**
  - Plugin system
  - Community marketplace
  - Enterprise governance features

---

## Work Streams

### Core Work Streams (Original)

| ID | Work Stream | Phase | Priority | Owner |
|---|---|---|---|---|
| WS-01 | Project Setup & Infrastructure | 0 | P0 | TBD |
| WS-02 | Core CLI Framework | 1 | P0 | TBD |
| WS-03 | OpenRouter Integration | 1 | P0 | TBD |
| WS-04 | Agent Core (Planner + Executor) | 1 | P0 | TBD |
| WS-05 | Multi-File Editing Layer | 1 | P0 | TBD |
| WS-06 | Sandbox Execution Environment | 1 | P0 | TBD |
| WS-07 | Retrieval & RAG System | 2 | P1 | TBD |
| WS-08 | Test Runner Integration | 2 | P1 | TBD |
| WS-09 | Verification Engine | 2 | P1 | TBD |
| WS-10 | Tool Integrations | 2 | P1 | TBD |
| WS-11 | State & Persistence | 2 | P1 | TBD |
| WS-12 | Observability & Logging | 3 | P2 | TBD |
| WS-13 | Safety & Security | 3 | P2 | TBD |

### Enhanced Work Streams (New - See AGENT_IMPROVEMENTS.md)

| ID | Work Stream | Phase | Priority | Owner |
|---|---|---|---|---|
| WS-14 | Advanced Code Intelligence | 2 | P1 | TBD |
| WS-15 | Enhanced Planning & Learning | 2 | P1 | TBD |
| WS-16 | AI-Powered Testing | 2 | P1 | TBD |
| WS-17 | Refactoring & Code Quality | 2-3 | P1 | TBD |
| WS-18 | Developer Experience | 3 | P1 | TBD |
| WS-19 | Cloud & Infrastructure | 3-4 | P2 | TBD |
| WS-20 | Enterprise Features | 4 | P3 | TBD |

**Priority Levels:**
- **P0:** Critical path, blocks other work
- **P1:** High priority, core functionality
- **P2:** Important but not blocking
- **P3:** Nice to have

**Note:** See [AGENT_IMPROVEMENTS.md](./AGENT_IMPROVEMENTS.md) for detailed breakdown of new work streams WS-14 through WS-20.

---

## Detailed Task Breakdown

### Phase 0: Research & Foundation

#### WS-01: Project Setup & Infrastructure

**Updated (2025-11-23):** Phase 0 research must prioritize **Agent-Computer Interface (ACI) design** based on SWE-agent findings.

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| **INFRA-001** | **Research coding agent architectures (ReAct, Plan-and-Execute, ReWOO)** | ✅ COMPLETE | P0 | - | 2d |
| **INFRA-001a** | **Research Agent-Computer Interface (ACI) design patterns** | ✅ COMPLETE | P0 | INFRA-001 | 1d |
| **INFRA-001b** | **Research tool design principles for LLM agents** | ✅ COMPLETE | P0 | INFRA-001 | 1d |
| INFRA-002 | Research OpenRouter API capabilities and patterns | PENDING | P0 | - | 1d |
| INFRA-003 | Research AST parsing libraries (tree-sitter, ast) | PENDING | P0 | - | 2d |
| **INFRA-003a** | **Research repository mapping strategies (Aider approach)** | PENDING | P0 | INFRA-003 | 1d |
| INFRA-004 | Research sandbox technologies (Docker, firejail, bubblewrap, gVisor) | PENDING | P0 | - | 2d |
| INFRA-005 | Research vector databases (ChromaDB, FAISS, Weaviate, Qdrant) | PENDING | P0 | - | 1d |
| INFRA-006 | Research code embedding models (CodeBERT, GraphCodeBERT) | PENDING | P0 | - | 1d |
| **INFRA-006a** | **Design custom ACI optimized for coding tasks** | PENDING | P0 | INFRA-001a, INFRA-003a | 2d |
| INFRA-007 | Define project structure and architecture | PENDING | P0 | INFRA-001 to 006a | 2d |
| INFRA-008 | Create Python package structure (pyproject.toml, setup.py) | PENDING | P0 | INFRA-007 | 1d |
| INFRA-009 | Setup dev environment (pytest, black, mypy, ruff) | PENDING | P0 | INFRA-008 | 1d |
| INFRA-010 | Create project config files (.gitignore, .editorconfig) | PENDING | P0 | INFRA-008 | 0.5d |
| INFRA-011 | Setup pre-commit hooks | PENDING | P1 | INFRA-009 | 0.5d |

**Total Estimate:** 2.5 weeks

**Key Deliverables:**
- ✅ [AGENT_ARCHITECTURE_RESEARCH.md](./AGENT_ARCHITECTURE_RESEARCH.md) - Comprehensive research on agent patterns
- 📋 Custom ACI design document
- 📋 Repository mapping implementation plan
- 📋 Tool specifications for Phase 1

---

### Phase 1: Core MVP

#### WS-02: Core CLI Framework

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| CLI-001 | Create basic CLI entry point (Click/Typer) | PENDING | P0 | INFRA-008 | 1d |
| CLI-002 | Implement 'orca init' command | PENDING | P0 | CLI-001 | 1d |
| CLI-003 | Implement 'orca config' command | PENDING | P0 | CLI-001 | 2d |
| CLI-004 | Create configuration schema and validation | PENDING | P0 | CLI-003 | 1d |
| CLI-005 | Implement config persistence to .orca/config.json | PENDING | P0 | CLI-004 | 1d |
| CLI-006 | Create event logging system (JSON output) | PENDING | P0 | CLI-001 | 2d |
| CLI-007 | Implement streaming output handler | PENDING | P1 | CLI-006 | 2d |

**Total Estimate:** 1.5 weeks

#### WS-03: OpenRouter Integration

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| OR-001 | Create OpenRouter API client with auth | PENDING | P0 | INFRA-002, CLI-003 | 2d |
| OR-002 | Implement model routing logic | PENDING | P0 | OR-001 | 2d |
| OR-003 | Create model performance profiles cache | PENDING | P1 | OR-001 | 1d |
| OR-004 | Implement automatic model selection by task type | PENDING | P1 | OR-002 | 2d |
| OR-005 | Add retry logic with exponential backoff | PENDING | P0 | OR-001 | 1d |
| OR-006 | Implement streaming response handling | PENDING | P1 | OR-001 | 2d |
| OR-007 | Create prompt templates for agent operations | PENDING | P0 | OR-001 | 2d |

**Total Estimate:** 2 weeks

#### WS-04: Agent Core (Planner + Executor)

**Updated (2025-11-23):** Implement **hybrid Plan-and-Execute + ReAct** architecture based on research findings.

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| AGENT-001 | Design planning schema (Plan, Step, Action, Verification) | PENDING | P0 | - | 2d |
| **AGENT-001a** | **Implement Plan-and-Execute pattern for well-defined tasks** | PENDING | P0 | AGENT-001 | 3d |
| **AGENT-001b** | **Implement ReAct pattern for exploratory tasks** | PENDING | P0 | AGENT-001 | 3d |
| **AGENT-001c** | **Add task classification (well-defined vs exploratory)** | PENDING | P0 | AGENT-001a, AGENT-001b | 1d |
| AGENT-002 | Implement 'orca plan' command | PENDING | P0 | CLI-001, OR-007 | 3d |
| AGENT-003 | Create plan parser (LLM output → structured plan) | PENDING | P0 | AGENT-001, OR-001 | 3d |
| AGENT-004 | Implement plan storage and versioning (.orca/plan.json) | PENDING | P0 | AGENT-001 | 2d |
| AGENT-005 | Create plan executor with step-by-step execution | PENDING | P0 | AGENT-004 | 3d |
| AGENT-006 | Implement 'orca exec' command | PENDING | P0 | AGENT-005 | 2d |
| AGENT-007 | Add approval gate mechanism (--approve vs --auto) | PENDING | P1 | AGENT-006 | 1d |
| AGENT-008 | Implement plan rollback on failure | PENDING | P1 | AGENT-006 | 2d |
| **AGENT-009** | **Create self-correction loop (detect → reflect → retry)** | PENDING | P0 | AGENT-006 | 4d |
| **AGENT-009a** | **Add error categorization and analysis** | PENDING | P0 | AGENT-009 | 2d |
| AGENT-010 | Implement failure pattern detection | PENDING | P1 | AGENT-009 | 2d |
| AGENT-011 | Add escalating retry strategy (max 3 retries) | PENDING | P1 | AGENT-010 | 1d |
| **AGENT-012** | **Implement max_iterations safety limit for loops** | PENDING | P0 | AGENT-009 | 0.5d |

**Total Estimate:** 4.5 weeks

**Research References:**
- Plan-and-Execute achieves better results for complex multi-step tasks
- ReAct provides 86% success rate for exploratory tasks
- Self-correction loops recover from 70%+ of initial failures
- Always enforce max_iterations limit to prevent infinite loops

**Implementation Notes:**
- Separate deterministic orchestration from non-deterministic reasoning
- Use LLM for planning and code generation; deterministic for validation and execution
- Implement proper error categorization for effective self-correction

#### WS-05: Multi-File Editing Layer

**Updated (2025-11-23):** Implement **ACI-optimized editing tools** based on SWE-agent research (12.5% performance gain).

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| EDIT-001 | Integrate tree-sitter for AST parsing | PENDING | P0 | INFRA-003 | 2d |
| **EDIT-001a** | **Build repository mapper (AST-based, Aider approach)** | PENDING | P0 | EDIT-001 | 3d |
| EDIT-002 | Create language-specific parsers (Python, JS/TS, Go) | PENDING | P0 | EDIT-001 | 3d |
| **EDIT-003** | **Implement line-based editing (NOT full-file replacement)** | PENDING | P0 | EDIT-002 | 4d |
| **EDIT-003a** | **Add edit operations: replace_lines, insert_lines, delete_lines** | PENDING | P0 | EDIT-003 | 2d |
| **EDIT-003b** | **Implement compact file viewer with line numbers** | PENDING | P0 | EDIT-002 | 2d |
| EDIT-004 | Create diff generation and visualization | PENDING | P0 | EDIT-003 | 2d |
| EDIT-005 | Implement multi-file atomic edit operations | PENDING | P0 | EDIT-003 | 3d |
| **EDIT-005a** | **Add automatic backup/restore on edit failure** | PENDING | P0 | EDIT-005 | 1d |
| EDIT-006 | Add dependency-aware editing (imports/usage tracking) | PENDING | P1 | EDIT-002 | 3d |
| EDIT-007 | Create undo/redo mechanism | PENDING | P1 | EDIT-005 | 2d |
| **EDIT-008** | **Implement syntax validation before file write** | PENDING | P0 | EDIT-003 | 1d |

**Total Estimate:** 3.5 weeks

**Research References:**
- SWE-agent: "Compact, efficient file editing is critical to performance"
- Line-based editing reduces token usage by 80%+ vs full-file replacement
- Repository mapping enables context-aware editing without loading all files

**ACI Design Principles:**
- ✅ Self-contained: Each tool validates inputs and handles errors
- ✅ Robust: Automatic backup/recovery on failure
- ✅ Clear: Descriptive output with line numbers and diffs
- ✅ Structured: Return dataclasses, not strings
- ✅ Compact: Minimal token usage in tool output

**Tool Specifications:**
1. `edit_file(path, operation, start_line, end_line, new_content)` - Surgical edits
2. `view_file(path, start_line, num_lines)` - Compact file viewer
3. `search_code(query, search_type)` - Semantic, regex, or symbol search

#### WS-06: Sandbox Execution Environment

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| SANDBOX-001 | Choose sandbox technology (Docker vs firejail vs bubblewrap) | PENDING | P0 | INFRA-004 | 1d |
| SANDBOX-002 | Implement sandbox initialization and teardown | PENDING | P0 | SANDBOX-001 | 3d |
| SANDBOX-003 | Create language runtime managers (Python, Node, Go, Java, C/C++) | PENDING | P0 | SANDBOX-002 | 4d |
| SANDBOX-004 | Implement resource limits (CPU, memory, disk, network) | PENDING | P0 | SANDBOX-002 | 2d |
| SANDBOX-005 | Add runtime error capture (stack traces, stderr, stdout) | PENDING | P0 | SANDBOX-003 | 2d |
| SANDBOX-006 | Create 'orca run' command for sandbox execution | PENDING | P0 | SANDBOX-005, CLI-001 | 2d |
| SANDBOX-007 | Implement REPL mode with project context | PENDING | P1 | SANDBOX-006 | 3d |
| SANDBOX-008 | Add performance monitoring and stats | PENDING | P1 | SANDBOX-005 | 2d |

**Total Estimate:** 2.5 weeks

**Phase 1 Total:** ~12 weeks (with parallel work streams)

---

### Phase 2: Advanced Features

#### WS-07: Retrieval & RAG System

**Updated (2025-11-23):** Implement **just-in-time context management** with hierarchical loading (Aider approach).

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| **RAG-000** | **Implement repository map (always in context, ~1-2K tokens)** | PENDING | P0 | EDIT-001a | 2d |
| RAG-001 | Integrate vector database (ChromaDB recommended) | PENDING | P1 | INFRA-005 | 2d |
| RAG-002 | Implement code embedding generation (CodeBERT) | PENDING | P1 | INFRA-006 | 3d |
| RAG-003 | Create workspace indexer (files, deps, configs) | PENDING | P1 | RAG-001, RAG-002 | 3d |
| RAG-004 | Implement incremental indexing on file changes | PENDING | P1 | RAG-003 | 2d |
| **RAG-004a** | **Implement just-in-time context loading (load on demand)** | PENDING | P0 | RAG-000 | 3d |
| RAG-005 | Add vector search with relevance scoring | PENDING | P1 | RAG-003 | 2d |
| **RAG-005a** | **Implement multi-factor relevance scoring (keywords + semantic + recency)** | PENDING | P1 | RAG-005 | 2d |
| RAG-006 | Implement symbol search (ctags/tree-sitter) | PENDING | P1 | EDIT-001 | 2d |
| RAG-007 | Add regex search with ripgrep | PENDING | P1 | - | 1d |
| RAG-008 | Create context window management and chunking | PENDING | P1 | OR-001 | 3d |
| **RAG-008a** | **Implement context cleanup (clear after task completion)** | PENDING | P1 | RAG-008 | 1d |
| RAG-009 | Implement file ranking by task relevance | PENDING | P1 | RAG-005 | 2d |
| **RAG-010** | **Implement hierarchical context (map → summaries → full content)** | PENDING | P0 | RAG-000, RAG-004a | 2d |

**Total Estimate:** 4 weeks

**Research References:**
- Aider handles 2000+ file codebases through AST-based repository mapping
- Just-in-time context reduces token usage by 90%+ vs loading all files
- Repository map (~1-2K tokens) provides structural awareness

**Context Management Strategy:**
```
Level 1: Repository Map (always in context)
  - File structure, symbols, dependencies
  - ~1-2K tokens for entire project

Level 2: File Summaries (loaded for relevant files)
  - Purpose, public API, key abstractions
  - ~500 tokens per file

Level 3: Full Content (loaded when editing)
  - Complete source code
  - ~500-5000 tokens per file
```

**Key Principles:**
- Maintain lightweight identifiers, not full content
- Load context dynamically based on task
- Prioritize by relevance (keywords, semantic, recency, size)
- Clear context between unrelated tasks

#### WS-08: Test Runner Integration

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| TEST-001 | Create test framework auto-detection | PENDING | P1 | - | 2d |
| TEST-002 | Implement pytest runner integration | PENDING | P1 | TEST-001, SANDBOX-006 | 2d |
| TEST-003 | Implement Jest runner integration | PENDING | P1 | TEST-001, SANDBOX-006 | 2d |
| TEST-004 | Implement go test runner integration | PENDING | P1 | TEST-001, SANDBOX-006 | 1d |
| TEST-005 | Implement JUnit runner integration | PENDING | P1 | TEST-001, SANDBOX-006 | 2d |
| TEST-006 | Create 'orca test' command with smart selection | PENDING | P1 | TEST-002 to 005, CLI-001 | 2d |
| TEST-007 | Add changed-files-only testing mode | PENDING | P1 | TEST-006, RAG-004 | 2d |
| TEST-008 | Implement test result parsing and failure analysis | PENDING | P1 | TEST-006 | 2d |

**Total Estimate:** 2 weeks

#### WS-09: Verification Engine

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| VERIFY-001 | Create verification schema (check types) | PENDING | P1 | - | 1d |
| VERIFY-002 | Implement syntax verification per language | PENDING | P1 | EDIT-002 | 2d |
| VERIFY-003 | Add type checking (mypy, tsc) | PENDING | P1 | SANDBOX-006 | 2d |
| VERIFY-004 | Implement unit test verification | PENDING | P1 | TEST-006 | 1d |
| VERIFY-005 | Add optional CodeQL integration | PENDING | P2 | SANDBOX-006 | 3d |
| VERIFY-006 | Create verification report generation | PENDING | P1 | VERIFY-001 to 004 | 2d |

**Total Estimate:** 1.5 weeks

#### WS-10: Tool Integrations

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| TOOL-001 | Integrate ESLint (JavaScript/TypeScript) | PENDING | P1 | SANDBOX-006 | 1d |
| TOOL-002 | Integrate Pylint/Flake8 (Python) | PENDING | P1 | SANDBOX-006 | 1d |
| TOOL-003 | Integrate formatters (Black, Prettier, gofmt) | PENDING | P1 | SANDBOX-006 | 2d |
| TOOL-004 | Add dependency vulnerability scanning | PENDING | P2 | SANDBOX-006 | 2d |
| TOOL-005 | Implement license checker and compliance | PENDING | P2 | - | 2d |
| TOOL-006 | Integrate ripgrep for code search | PENDING | P1 | - | 1d |
| TOOL-007 | Integrate universal-ctags | PENDING | P1 | - | 1d |

**Total Estimate:** 1.5 weeks

#### WS-11: State & Persistence

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| STATE-001 | Design workspace state schema | PENDING | P1 | AGENT-001 | 2d |
| STATE-002 | Implement short-term memory (.orca/workspace.json) | PENDING | P1 | STATE-001 | 2d |
| STATE-003 | Create long-term belief store (constraints, architecture) | PENDING | P1 | STATE-001 | 2d |
| STATE-004 | Implement auto-resume after interruptions | PENDING | P1 | STATE-002, AGENT-006 | 3d |
| STATE-005 | Add workspace cleanup and garbage collection | PENDING | P2 | STATE-002 | 1d |

**Total Estimate:** 1.5 weeks

**Phase 2 Total:** ~10 weeks (with parallel work streams)

#### WS-14: Advanced Code Intelligence (NEW)

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| CODE-001 | Implement call graph generator (pycg, madge) | PENDING | P1 | EDIT-002 | 3d |
| CODE-002 | Build impact analysis engine | PENDING | P1 | CODE-001 | 4d |
| CODE-003 | Integrate complexity metrics (radon, gocyclo) | PENDING | P1 | EDIT-002 | 2d |
| CODE-004 | Add dead code detection | PENDING | P2 | CODE-001 | 2d |
| CODE-005 | Implement code clone detection | PENDING | P2 | RAG-002 | 3d |
| CODE-006 | Create dependency graph visualizer | PENDING | P1 | CODE-001 | 2d |

**Total Estimate:** 2 weeks

#### WS-15: Enhanced Planning & Learning (NEW)

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| PLAN-001 | Build learning system (plan execution history DB) | PENDING | P1 | STATE-002 | 5d |
| PLAN-002 | Create plan template system | PENDING | P1 | AGENT-004 | 3d |
| PLAN-003 | Implement parallel execution engine | PENDING | P1 | AGENT-006 | 4d |
| PLAN-004 | Add predictive time/cost estimation | PENDING | P2 | PLAN-001, OBS-005 | 3d |
| PLAN-005 | Enhance context enrichment in belief store | PENDING | P1 | STATE-003, RAG-003 | 3d |
| PLAN-006 | Add plan visualization (ASCII/Mermaid) | PENDING | P1 | AGENT-004 | 2d |

**Total Estimate:** 3 weeks

#### WS-16: AI-Powered Testing (NEW)

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| AITEST-001 | Build intelligent test generation engine | PENDING | P1 | EDIT-002, OR-001 | 5d |
| AITEST-002 | Implement coverage gap analysis | PENDING | P1 | TEST-006 | 3d |
| AITEST-003 | Integrate mutation testing (mutmut, Stryker) | PENDING | P2 | TEST-006 | 3d |
| AITEST-004 | Add flaky test detection | PENDING | P2 | TEST-006 | 2d |
| AITEST-005 | Build test performance optimizer | PENDING | P2 | TEST-006 | 2d |
| AITEST-006 | Add property-based testing support | PENDING | P2 | AITEST-001 | 3d |

**Total Estimate:** 2.5 weeks

#### WS-17: Refactoring & Code Quality (NEW)

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| REFACTOR-001 | Build intelligent refactoring engine (extract, rename, inline) | PENDING | P1 | EDIT-003, CODE-001 | 5d |
| REFACTOR-002 | Implement automated code review system | PENDING | P1 | CODE-003, OR-001 | 4d |
| REFACTOR-003 | Add documentation generation (docstrings, API docs) | PENDING | P1 | EDIT-002 | 3d |
| REFACTOR-004 | Build architecture smell detection | PENDING | P2 | CODE-001, CODE-003 | 4d |
| REFACTOR-005 | Add SOLID principles violation detection | PENDING | P2 | CODE-001 | 3d |

**Total Estimate:** 2.5 weeks

**Phase 2 Enhanced Total:** ~20 weeks (with parallel work streams and new features)

---

### Phase 3: Production Ready

#### WS-12: Observability & Logging

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| OBS-001 | Create structured event logging (JSON) | PENDING | P2 | CLI-006 | 2d |
| OBS-002 | Implement human-readable log formatter | PENDING | P2 | OBS-001 | 1d |
| OBS-003 | Add event types (plan_created, file_edited, etc.) | PENDING | P2 | OBS-001, AGENT-006 | 2d |
| OBS-004 | Implement latency and performance metrics | PENDING | P2 | - | 2d |
| OBS-005 | Add token usage tracking and cost estimation | PENDING | P2 | OR-001 | 2d |
| OBS-006 | Create monitoring dashboard/CLI view | PENDING | P2 | OBS-004, OBS-005 | 3d |

**Total Estimate:** 2 weeks

#### WS-13: Safety & Security

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| SECURITY-001 | Implement harmful command detection and blocking | PENDING | P2 | SANDBOX-006 | 3d |
| SECURITY-002 | Add dangerous file write protection | PENDING | P2 | EDIT-005 | 2d |
| SECURITY-003 | Create --force flag for restricted operations | PENDING | P2 | CLI-001 | 1d |
| SECURITY-004 | Implement code provenance scanning | PENDING | P2 | RAG-003 | 2d |
| SECURITY-005 | Add license conflict detection | PENDING | P2 | TOOL-005 | 2d |

**Total Estimate:** 1.5 weeks

#### Additional Phase 3 Tasks

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| PHASE3-001 | Implement 'orca chat' command (interactive mode) | PENDING | P1 | OR-001, CLI-001 | 3d |
| PHASE3-002 | Create 'orca run agent' command (full autonomous) | PENDING | P1 | AGENT-009 | 2d |
| PHASE3-003 | Add CI integration (GitHub Actions templates) | PENDING | P2 | - | 3d |
| PHASE3-004 | Implement PR creation and summary generation | PENDING | P2 | PHASE3-003 | 2d |
| PHASE3-005 | Create comprehensive README | PENDING | P1 | All core features | 2d |
| PHASE3-006 | Write architecture documentation (ARCHITECTURE.md) | PENDING | P1 | All core features | 3d |
| PHASE3-007 | Create user guide and tutorials | PENDING | P1 | All core features | 3d |
| PHASE3-008 | Write API documentation for plugins | PENDING | P2 | All core features | 2d |
| PHASE3-009 | Add example projects and use cases | PENDING | P1 | All core features | 3d |
| PHASE3-010 | Create contribution guidelines (CONTRIBUTING.md) | PENDING | P2 | - | 1d |
| PHASE3-011 | Write comprehensive test suite | PENDING | P0 | All core features | 5d |
| PHASE3-012 | Add integration tests for E2E workflows | PENDING | P0 | All core features | 5d |
| PHASE3-013 | Create performance benchmarks | PENDING | P1 | All core features | 3d |
| PHASE3-014 | Setup GitHub Actions CI/CD pipeline | PENDING | P0 | PHASE3-011, PHASE3-012 | 2d |
| PHASE3-015 | Create PyPI package and publish workflow | PENDING | P0 | PHASE3-014 | 2d |
| PHASE3-016 | Add Homebrew formula for macOS | PENDING | P1 | PHASE3-015 | 2d |

**Total Estimate:** 6 weeks

#### WS-18: Developer Experience (NEW)

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| DX-001 | Integrate debugger support (pdb, ipdb integration) | PENDING | P2 | SANDBOX-006 | 4d |
| DX-002 | Enhance error messages with AI-powered suggestions | PENDING | P1 | OR-001, AGENT-009 | 2d |
| DX-003 | Improve plan visualization (web UI) | PENDING | P1 | PLAN-006 | 3d |
| DX-004 | Create VS Code extension | PENDING | P2 | CLI-001, AGENT-006 | 5d |
| DX-005 | Build web dashboard for monitoring | PENDING | P2 | OBS-001 to OBS-006 | 7d |
| DX-006 | Add interactive debugging with AI assistance | PENDING | P2 | DX-001, OR-001 | 3d |

**Total Estimate:** 3 weeks

#### WS-19: Cloud & Infrastructure (NEW)

| ID | Task | Status | Priority | Dependencies | Estimate |
|---|---|---|---|---|---|
| CLOUD-001 | Implement IaC generation (Terraform, CloudFormation) | PENDING | P2 | EDIT-003 | 5d |
| CLOUD-002 | Add cloud deployment automation (AWS, GCP, Azure) | PENDING | P2 | SANDBOX-006 | 5d |
| CLOUD-003 | Build database migration management | PENDING | P1 | EDIT-003, TEST-006 | 4d |
| CLOUD-004 | Implement API contract management (OpenAPI, GraphQL) | PENDING | P2 | REFACTOR-003 | 4d |
| CLOUD-005 | Add Kubernetes manifest generation | PENDING | P2 | CLOUD-001 | 3d |
| CLOUD-006 | Build CI/CD pipeline templates | PENDING | P1 | PHASE3-003 | 3d |

**Total Estimate:** 3 weeks

**Phase 3 Enhanced Total:** ~16 weeks (with new features)

---

## Dependency Graph

### Critical Path (Must complete in order)

```
INFRA-001 to 006 (Research)
    ↓
INFRA-007 (Architecture)
    ↓
INFRA-008 (Package Structure)
    ↓
CLI-001 (CLI Entry Point)
    ↓
OR-001 (OpenRouter Client) + AGENT-001 (Planning Schema)
    ↓
AGENT-002 (Plan Command) + AGENT-003 (Plan Parser)
    ↓
AGENT-006 (Exec Command)
    ↓
EDIT-003 (AST Editing) + SANDBOX-006 (Sandbox Execution)
    ↓
AGENT-009 (Autonomous Iteration)
    ↓
RAG-003 (Workspace Indexing) + TEST-006 (Test Command)
    ↓
VERIFY-006 (Verification Engine)
    ↓
PHASE3-011 (Test Suite) + PHASE3-012 (Integration Tests)
    ↓
PHASE3-014 (CI/CD)
    ↓
PHASE3-015 (PyPI Package)
```

### Parallel Work Streams

These can be developed concurrently by different team members:

**Stream A (Agent & Planning):**
- AGENT-001 → AGENT-011

**Stream B (Editing & Execution):**
- EDIT-001 → EDIT-007
- SANDBOX-001 → SANDBOX-008

**Stream C (Infrastructure & Integration):**
- OR-001 → OR-007
- RAG-001 → RAG-009

**Stream D (Testing & Verification):**
- TEST-001 → TEST-008
- VERIFY-001 → VERIFY-006

---

## Technology Stack

### Core Technologies (Decided)

| Component | Technology | Rationale |
|---|---|---|
| Language | Python 3.10+ | Rich ecosystem, AI/ML libraries, rapid development |
| CLI Framework | Typer | Modern, type-safe, auto-documentation |
| Package Manager | Poetry | Dependency management, packaging |
| LLM Gateway | OpenRouter | Multi-model support, free tier |
| Testing | pytest | Industry standard, rich plugin ecosystem |
| Code Quality | black, ruff, mypy | Formatting, linting, type checking |

### Technologies To Evaluate (Phase 0)

| Component | Options | Evaluation Criteria |
|---|---|---|
| AST Parsing | tree-sitter vs ast module | Multi-language support, performance |
| Sandbox | Docker vs firejail vs bubblewrap vs gVisor | Security, overhead, compatibility |
| Vector DB | ChromaDB vs FAISS vs Qdrant | Ease of use, performance, features |
| Embeddings | CodeBERT vs GraphCodeBERT vs StarCoder | Code understanding quality, speed |
| Config Format | TOML vs YAML vs JSON | Human readability, validation support |

### Additional Technologies (Phase 2+)

| Component | Technology | Purpose | Phase |
|---|---|---|---|
| **Call Graph Analysis** | pycg, madge | Dependency tracking | 2 |
| **Complexity Metrics** | radon, gocyclo | Code quality | 2 |
| **Refactoring** | rope, jscodeshift | AST transformations | 2 |
| **Mutation Testing** | mutmut, Stryker | Test quality | 2 |
| **Documentation** | pdoc, Sphinx, swagger-jsdoc | Auto-doc generation | 2-3 |
| **Debugging** | pdb, ipdb, node-inspect | Debug integration | 3 |
| **Visualization** | graphviz, mermaid | Plan/dependency viz | 2-3 |
| **IaC** | terraform, pulumi | Infrastructure | 3 |
| **Contract Testing** | pact, dredd | API contracts | 3 |
| **Code Review** | pylint-plugin-patterns, custom | Pattern detection | 2 |

---

## Success Criteria

### Phase 1 (Core MVP) Exit Criteria

- [ ] CLI installable via pip
- [ ] All core commands functional (init, config, plan, exec, run, test)
- [ ] OpenRouter integration working with 3+ models
- [ ] Can plan and execute multi-step tasks (5+ steps)
- [ ] Multi-file editing (3+ files atomically)
- [ ] Sandbox execution for Python and JavaScript
- [ ] Basic test suite with 70%+ coverage
- [ ] Documentation for core features

### Phase 2 (Advanced Features) Exit Criteria

**Core Features:**
- [ ] RAG indexing and retrieval working
- [ ] Test runner integration for 4+ frameworks
- [ ] Verification engine with syntax, type, and test checks
- [ ] Static analysis integration (ESLint, Pylint)
- [ ] Auto-resume after interruptions
- [ ] Test coverage 80%+
- [ ] User guide and tutorials

**Enhanced Features (NEW):**
- [ ] Call graph analysis functional
- [ ] Impact analysis for code changes
- [ ] Complexity metrics integrated
- [ ] Test generation working for Python & JavaScript
- [ ] Coverage gap analysis reporting
- [ ] Plan templates for common tasks
- [ ] Learning system tracking plan success/failure
- [ ] Code review automation functional
- [ ] Documentation generation working

### Phase 3 (Production Ready) Exit Criteria

**Core Features:**
- [ ] Full observability and monitoring
- [ ] Security features (command blocking, file protection)
- [ ] CI integration templates
- [ ] Comprehensive documentation
- [ ] Integration test suite
- [ ] Performance benchmarks
- [ ] PyPI package published
- [ ] Homebrew formula available
- [ ] Test coverage 85%+

**Enhanced Features (NEW):**
- [ ] Intelligent refactoring engine operational
- [ ] Architecture smell detection
- [ ] AI-powered error messages
- [ ] Plan visualization (web + terminal)
- [ ] Mutation testing integrated
- [ ] Flaky test detection
- [ ] Database migration management
- [ ] API contract management (OpenAPI)
- [ ] Debugger integration

### Overall Success Metrics (from PRD)

- [ ] Reduction in user time to complete multi-step tasks: **50%+**
- [ ] Tasks solved autonomously without correction: **70%+**
- [ ] Error rate after verification: **<10%**
- [ ] Multi-file commits per minute: **5+**
- [ ] Average time to plan, execute, verify: **<5 minutes** for typical tasks

---

## Risk Assessment

### High Risk Items

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Sandbox security vulnerabilities | HIGH | MEDIUM | Use battle-tested tech (Docker), security audit |
| LLM API rate limits / costs | MEDIUM | HIGH | Implement aggressive caching, local fallbacks |
| AST parsing complexity across languages | HIGH | MEDIUM | Start with 2-3 languages, expand incrementally |
| Vector DB performance at scale | MEDIUM | MEDIUM | Benchmark early, optimize indexing strategy |
| Complex dependency graph in plan execution | HIGH | HIGH | Extensive testing, failure recovery mechanisms |

### Medium Risk Items

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Tree-sitter grammar maintenance | MEDIUM | MEDIUM | Use stable grammars, vendor if needed |
| Test framework compatibility | MEDIUM | MEDIUM | Start with popular frameworks, extend gradually |
| State corruption during interruptions | MEDIUM | MEDIUM | Implement transactional updates, checksums |
| Model response parsing failures | MEDIUM | HIGH | Robust parsers, fallback to simpler formats |

### Low Risk Items

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Configuration schema changes | LOW | MEDIUM | Version config files, migration scripts |
| CLI UX issues | LOW | MEDIUM | User testing, iterative improvements |
| Documentation drift | LOW | HIGH | Auto-generate from code, regular reviews |

---

## Next Steps

### Immediate Actions (This Week)

1. **INFRA-001 to INFRA-006:** Complete research phase
   - Assign: Team lead + 1 developer
   - Deliverable: Technology decision document

2. **INFRA-007:** Architecture design
   - Assign: Team lead
   - Deliverable: ARCHITECTURE.md with diagrams

3. **INFRA-008 to INFRA-011:** Project setup
   - Assign: All developers
   - Deliverable: Working dev environment for all team members

### Week 2 Actions

1. Begin parallel work on:
   - CLI-001 to CLI-007 (Developer 1)
   - OR-001 to OR-007 (Developer 2)
   - AGENT-001 (Team Lead)

2. Weekly sync meetings to review progress and dependencies

---

## Appendix

### Folder Structure (Proposed)

```
orca-cli/
├── .orca/                    # User workspace metadata (created by init)
│   ├── config.json          # User configuration
│   ├── plan.json            # Current plan state
│   ├── workspace.json       # Short-term memory
│   └── beliefs.json         # Long-term belief store
├── src/
│   └── orca/
│       ├── __init__.py
│       ├── cli/             # CLI commands
│       │   ├── __init__.py
│       │   ├── main.py      # Entry point
│       │   ├── init.py
│       │   ├── config.py
│       │   ├── plan.py
│       │   ├── exec.py
│       │   ├── test.py
│       │   ├── run.py
│       │   └── chat.py
│       ├── agent/           # Agent core
│       │   ├── __init__.py
│       │   ├── planner.py
│       │   ├── executor.py
│       │   ├── iteration.py
│       │   └── schemas.py
│       ├── openrouter/      # OpenRouter integration
│       │   ├── __init__.py
│       │   ├── client.py
│       │   ├── router.py
│       │   └── prompts.py
│       ├── edit/            # Multi-file editing
│       │   ├── __init__.py
│       │   ├── parser.py
│       │   ├── editor.py
│       │   └── languages/
│       ├── sandbox/         # Execution environment
│       │   ├── __init__.py
│       │   ├── manager.py
│       │   └── runtimes/
│       ├── rag/             # Retrieval system
│       │   ├── __init__.py
│       │   ├── indexer.py
│       │   ├── embeddings.py
│       │   └── search.py
│       ├── test_runner/     # Test integration
│       │   ├── __init__.py
│       │   ├── detector.py
│       │   └── runners/
│       ├── verify/          # Verification engine
│       │   ├── __init__.py
│       │   └── checks.py
│       ├── tools/           # Tool integrations
│       │   ├── __init__.py
│       │   ├── linters.py
│       │   └── formatters.py
│       ├── state/           # State management
│       │   ├── __init__.py
│       │   └── persistence.py
│       ├── observability/   # Logging & monitoring
│       │   ├── __init__.py
│       │   ├── logger.py
│       │   └── metrics.py
│       └── security/        # Safety & security
│           ├── __init__.py
│           └── validators.py
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/
│   ├── README.md
│   ├── ARCHITECTURE.md
│   ├── USER_GUIDE.md
│   ├── API.md
│   └── CONTRIBUTING.md
├── examples/
│   ├── simple_task/
│   ├── multi_file_refactor/
│   └── full_feature/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── publish.yml
├── pyproject.toml
├── setup.py
├── README.md
├── LICENSE
└── IMPLEMENTATION_PLAN.md (this file)
```

### Glossary

- **AST:** Abstract Syntax Tree
- **RAG:** Retrieval-Augmented Generation
- **LLM:** Large Language Model
- **CLI:** Command Line Interface
- **CI/CD:** Continuous Integration/Continuous Deployment
- **REPL:** Read-Eval-Print Loop
- **P0/P1/P2/P3:** Priority levels (0=highest, 3=lowest)

---

**Document Status:** Living document, updated as project progresses
**Last Updated:** 2025-11-22
**Next Review:** Weekly during active development
