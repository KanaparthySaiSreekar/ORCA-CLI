# ORCA-CLI Architecture

**Version:** 1.0 (Draft)
**Date:** 2025-11-22
**Status:** Design Phase

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Principles](#architecture-principles)
3. [Component Architecture](#component-architecture)
4. [Data Flow](#data-flow)
5. [Technology Decisions](#technology-decisions)
6. [Integration Points](#integration-points)
7. [Security Architecture](#security-architecture)
8. [Scalability & Performance](#scalability--performance)

---

## System Overview

ORCA-CLI is a command-line coding agent that autonomously plans, executes, and verifies software engineering tasks. It combines LLM reasoning with deterministic execution, AST-aware code editing, sandboxed runtime environments, and retrieval-augmented generation.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLI Interface                            │
│                    (Typer-based Commands)                        │
└────────────┬────────────────────────────────────────────────────┘
             │
             ├─── orca init      (Workspace initialization)
             ├─── orca config    (Configuration management)
             ├─── orca plan      (Task planning)
             ├─── orca exec      (Plan execution)
             ├─── orca test      (Test runner)
             ├─── orca run       (Code execution)
             └─── orca chat      (Interactive mode)

┌─────────────────────────────────────────────────────────────────┐
│                        Agent Core Layer                          │
├─────────────────────┬───────────────────┬───────────────────────┤
│   Planner Engine    │  Executor Engine  │  Iteration Loop       │
│  - Parse goals      │  - Execute steps  │  - Observe results    │
│  - Generate plans   │  - Track state    │  - Detect failures    │
│  - Validate plans   │  - Handle errors  │  - Auto-fix & retry   │
└─────────────────────┴───────────────────┴───────────────────────┘
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
┌─────────────────────────────────┴───────────────────────────────┐
│                    Service Layer (Tools)                         │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│ Multi-File   │  Sandbox     │  RAG/        │  Test Runner      │
│ Editor       │  Execution   │  Retrieval   │  Integration      │
│              │              │              │                   │
│ - AST parse  │ - Runtime    │ - Embedding  │ - Framework       │
│ - Code edit  │   isolation  │ - Vector DB  │   detection       │
│ - Diff gen   │ - Resource   │ - Semantic   │ - Test exec       │
│ - Validation │   limits     │   search     │ - Result parse    │
└──────────────┴──────────────┴──────────────┴───────────────────┘
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
┌─────────────────────────────────┴───────────────────────────────┐
│                   Infrastructure Layer                           │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│ OpenRouter   │  State       │  Observa-    │  Security         │
│ Integration  │  Management  │  bility      │  Layer            │
│              │              │              │                   │
│ - API client │ - Workspace  │ - Logging    │ - Command block   │
│ - Model      │   state      │ - Metrics    │ - File protect    │
│   routing    │ - Belief     │ - Events     │ - Provenance      │
│ - Streaming  │   store      │ - Monitoring │ - License check   │
└──────────────┴──────────────┴──────────────┴───────────────────┘
```

---

## Architecture Principles

### 1. **Separation of Concerns**
- Clear boundaries between CLI, agent logic, services, and infrastructure
- Each layer has well-defined responsibilities
- Minimal coupling between components

### 2. **Deterministic Core with LLM Enhancement**
- Deterministic operations (parsing, execution, verification) in core logic
- LLM used for planning, reasoning, and code generation
- Always verify LLM output with deterministic checks

### 3. **Safety First**
- All code execution in isolated sandboxes
- Validation at every layer (input, plan, code, output)
- Explicit approval gates for destructive operations
- Comprehensive audit trail

### 4. **Modularity & Extensibility**
- Plugin-friendly architecture
- Easy to add new languages, test frameworks, tools
- Swappable LLM providers (not locked to OpenRouter)

### 5. **Observability by Default**
- Structured logging at every step
- Metrics collection built-in
- Event-driven architecture for easy monitoring

### 6. **Graceful Degradation**
- Fallback strategies when services unavailable
- Partial results better than complete failure
- Auto-recovery and resume capabilities

---

## Component Architecture

### 1. CLI Interface Layer

**Responsibility:** User interaction, command parsing, output formatting

**Components:**
- **Main Entry Point** (`main.py`)
  - Command registration and routing
  - Global option handling (--verbose, --config-path, etc.)
  - Error handling and user feedback

- **Command Modules** (`init.py`, `plan.py`, `exec.py`, etc.)
  - Individual command implementation
  - Input validation
  - Service orchestration
  - Output rendering

**Key Interfaces:**
```python
class Command(Protocol):
    def execute(self, args: CommandArgs) -> CommandResult
    def validate(self, args: CommandArgs) -> ValidationResult
```

**Technology:** Typer (Click-based, type-safe CLI framework)

---

### 2. Agent Core Layer

**Responsibility:** Autonomous planning, execution, and iteration

#### 2.1 Planner Engine

**Purpose:** Convert user goals into structured, executable plans

**Components:**
- **Goal Parser:** Extract intent, constraints, success criteria from user input
- **Plan Generator:** Use LLM to create multi-step plan with verification criteria
- **Plan Validator:** Verify plan feasibility, detect circular dependencies
- **Plan Optimizer:** Reorder steps for efficiency, identify parallelization opportunities

**Data Structures:**
```python
@dataclass
class Plan:
    id: str
    goal: str
    steps: List[PlanStep]
    created_at: datetime
    status: PlanStatus

@dataclass
class PlanStep:
    id: str
    action_type: ActionType  # edit, create, delete, run, test
    target_files: List[str]
    description: str
    verification_criteria: List[VerificationCheck]
    depends_on: List[str]  # step IDs
```

**Prompting Strategy:**
- System prompt with project context (architecture, constraints, patterns)
- Few-shot examples of good plans
- Chain-of-thought reasoning for complex tasks
- JSON schema enforcement for structured output

#### 2.2 Executor Engine

**Purpose:** Execute plan steps, manage state, handle errors

**Components:**
- **Step Executor:** Execute individual plan steps
- **State Tracker:** Maintain execution state, track progress
- **Rollback Manager:** Undo changes on failure
- **Approval Gate:** Implement manual approval workflow

**Execution Flow:**
```
For each step in plan:
  1. Check dependencies satisfied
  2. Request approval (if --approve mode)
  3. Execute action (edit, run, test, etc.)
  4. Verify success criteria
  5. Update state
  6. On failure: attempt retry or rollback
```

#### 2.3 Iteration Loop

**Purpose:** Autonomous failure recovery and self-correction

**Components:**
- **Observer:** Capture execution results (stdout, stderr, test failures, errors)
- **Failure Analyzer:** Categorize failures, extract patterns
- **Fix Generator:** Generate corrective actions using LLM
- **Retry Controller:** Escalating retry strategy with max attempts

**Iteration Strategy:**
```
attempt = 0
max_attempts = 3

while attempt < max_attempts:
  result = execute_step(step)

  if result.success:
    break

  failure_pattern = analyze_failure(result)
  corrective_action = generate_fix(failure_pattern, context)

  apply_fix(corrective_action)
  attempt += 1
```

---

### 3. Service Layer

#### 3.1 Multi-File Editor

**Responsibility:** AST-aware code parsing and editing

**Architecture:**
```
┌─────────────────────────────────────────┐
│         Editor Service                   │
├─────────────────────────────────────────┤
│  - parse_file(path, language)           │
│  - edit_code(file, edits)               │
│  - generate_diff(old, new)              │
│  - validate_syntax(code, language)      │
└─────────────┬───────────────────────────┘
              │
    ┌─────────┴──────────┐
    │                    │
┌───▼────┐        ┌──────▼──────┐
│  AST   │        │   Language  │
│ Parser │        │   Grammars  │
│        │        │             │
│tree-   │        │ - Python    │
│sitter  │        │ - JS/TS     │
│        │        │ - Go        │
└────────┘        └─────────────┘
```

**Key Features:**
- **Structural Awareness:** Edits respect language syntax and semantics
- **Dependency Tracking:** Track imports, function calls, type usage
- **Multi-File Atomicity:** Group edits across files as single transaction
- **Incremental Diff:** Generate minimal, readable diffs

**Editing Operations:**
- `insert_function(file, function_def, location)`
- `modify_function(file, function_name, new_body)`
- `delete_function(file, function_name)`
- `rename_symbol(files, old_name, new_name)`
- `add_import(file, import_spec)`

#### 3.2 Sandbox Execution

**Responsibility:** Safe, isolated code execution

**Architecture Options:**

**Option A: Docker-based (Recommended for MVP)**
```
┌─────────────────────────────────────────┐
│     Sandbox Manager                      │
├─────────────────────────────────────────┤
│  - create_container(runtime, limits)    │
│  - execute_code(code, env)              │
│  - capture_output(stdout, stderr)       │
│  - cleanup()                             │
└─────────────┬───────────────────────────┘
              │
    ┌─────────┴──────────┐
    │                    │
┌───▼────────┐   ┌───────▼────────┐
│  Runtime   │   │   Resource     │
│  Images    │   │   Limits       │
│            │   │                │
│ - python   │   │ - CPU: 2 cores │
│ - node     │   │ - Mem: 2GB     │
│ - go       │   │ - Time: 5min   │
│ - java     │   │ - Network: off │
└────────────┘   └────────────────┘
```

**Option B: Lightweight (firejail/bubblewrap)**
- Lower overhead, faster startup
- Linux-only, less portable
- Good for local development use case

**Security Measures:**
- No network access by default (configurable)
- Read-only filesystem except designated work dir
- CPU and memory limits enforced
- Execution timeout
- No privileged operations

#### 3.3 RAG / Retrieval System

**Responsibility:** Semantic code search and context retrieval

**Architecture:**
```
┌──────────────────────────────────────────────┐
│           Retrieval Service                   │
├──────────────────────────────────────────────┤
│  - index_workspace(root_path)                │
│  - search_semantic(query, top_k)             │
│  - search_symbol(symbol_name)                │
│  - search_regex(pattern)                     │
│  - rank_files_by_relevance(task)             │
└────────┬─────────────────────────────────────┘
         │
    ┌────┴────┬────────────┬───────────┐
    │         │            │           │
┌───▼────┐ ┌─▼────┐  ┌────▼───┐  ┌────▼────┐
│Vector  │ │Symbol│  │ Regex  │  │  File   │
│Search  │ │Index │  │ Engine │  │ Ranker  │
│        │ │      │  │        │  │         │
│Chroma  │ │ctags/│  │ripgrep │  │Relevance│
│DB      │ │tree- │  │        │  │Scoring  │
│        │ │sitter│  │        │  │         │
└────────┘ └──────┘  └────────┘  └─────────┘
```

**Indexing Strategy:**
- **Incremental:** Only re-index changed files
- **Hierarchical:** File → Class → Function → Statement
- **Metadata:** Language, LOC, dependencies, last modified

**Search Strategies:**
1. **Semantic Search:** Use embeddings for "files related to authentication"
2. **Symbol Search:** Fast lookup by function/class name
3. **Regex Search:** Pattern matching for specific code constructs
4. **Hybrid:** Combine multiple strategies, re-rank results

**Embedding Model:** CodeBERT or lightweight alternative (all-MiniLM-L6-v2)

#### 3.4 Test Runner Integration

**Responsibility:** Detect, execute, and analyze tests

**Architecture:**
```
┌──────────────────────────────────────────────┐
│           Test Runner Service                 │
├──────────────────────────────────────────────┤
│  - detect_framework(project_path)            │
│  - run_tests(framework, filters)             │
│  - parse_results(output)                     │
│  - analyze_failures(results)                 │
└────────┬─────────────────────────────────────┘
         │
    ┌────┴────┬────────┬────────┐
    │         │        │        │
┌───▼────┐ ┌─▼───┐ ┌──▼──┐ ┌───▼────┐
│ pytest │ │Jest │ │go   │ │ JUnit  │
│ Runner │ │     │ │test │ │        │
└────────┘ └─────┘ └─────┘ └────────┘
```

**Test Selection Strategies:**
- **All:** Run entire test suite
- **Changed Files:** Only tests for modified files
- **Failed Last Run:** Re-run previously failed tests
- **Affected:** Tests impacted by changes (via dependency analysis)

**Failure Analysis:**
- Parse stack traces
- Extract error messages
- Identify test categories (unit, integration, e2e)
- Detect patterns (e.g., all auth tests failing → likely auth system issue)

---

### 4. Infrastructure Layer

#### 4.1 OpenRouter Integration

**Responsibility:** LLM API abstraction and routing

**Architecture:**
```
┌──────────────────────────────────────────────┐
│        OpenRouter Service                     │
├──────────────────────────────────────────────┤
│  - send_request(prompt, model, options)      │
│  - stream_response(prompt, model)            │
│  - route_by_task(task_type)                  │
│  - estimate_cost(prompt, model)              │
└────────┬─────────────────────────────────────┘
         │
    ┌────┴────────┬─────────────┐
    │             │             │
┌───▼──────┐  ┌──▼────────┐ ┌──▼────────┐
│  Model   │  │ Prompt    │ │  Retry    │
│  Router  │  │ Templates │ │  Handler  │
│          │  │           │ │           │
│- Planning│  │- System   │ │- Exp.     │
│  → GPT-4 │  │- Few-shot │ │  backoff  │
│- Editing │  │- CoT      │ │- Fallback │
│  → Claude│  │           │ │  models   │
└──────────┘  └───────────┘ └───────────┘
```

**Model Selection Strategy:**
```python
def select_model(task_type: TaskType) -> str:
    routing_table = {
        TaskType.PLANNING: "openrouter/anthropic/claude-3.5-sonnet",
        TaskType.CODE_EDIT: "openrouter/meta-llama/llama-3.1-70b-instruct",
        TaskType.CODE_REVIEW: "openrouter/anthropic/claude-3.5-sonnet",
        TaskType.QUICK_FIX: "openrouter/meta-llama/llama-3.1-8b-instruct",
    }
    return routing_table.get(task_type, default_model)
```

**Caching Strategy:**
- Cache model responses for identical prompts (with TTL)
- Cache model metadata (context windows, pricing)
- Deduplicate prompts within same session

#### 4.2 State Management

**Responsibility:** Persist and manage agent state

**State Types:**

**1. Configuration State** (`.orca/config.json`)
```json
{
  "openrouter_api_key": "sk-...",
  "default_model": "openrouter/anthropic/claude-3.5-sonnet",
  "auto_approve": false,
  "sandbox_type": "docker",
  "log_level": "info"
}
```

**2. Plan State** (`.orca/plan.json`)
```json
{
  "current_plan_id": "plan-123",
  "plans": [
    {
      "id": "plan-123",
      "goal": "Add JWT middleware",
      "status": "in_progress",
      "current_step": 2,
      "steps": [...]
    }
  ]
}
```

**3. Workspace State** (`.orca/workspace.json`)
```json
{
  "modified_files": ["src/auth.py", "src/middleware.py"],
  "test_results": {...},
  "last_execution": "2025-11-22T10:30:00Z",
  "error_count": 0
}
```

**4. Belief Store** (`.orca/beliefs.json`)
```json
{
  "architecture": "FastAPI microservices",
  "auth_system": "JWT with RS256",
  "database": "PostgreSQL",
  "conventions": ["Use type hints", "100% test coverage for services"],
  "constraints": ["No external API calls in tests"]
}
```

**Persistence Strategy:**
- Atomic writes (write to temp file, then rename)
- Versioning for plan and workspace state
- Auto-backup before destructive operations
- Periodic cleanup of old state

#### 4.3 Observability

**Responsibility:** Logging, metrics, monitoring

**Event Types:**
```python
class EventType(Enum):
    PLAN_CREATED = "plan.created"
    PLAN_STARTED = "plan.started"
    STEP_STARTED = "step.started"
    STEP_COMPLETED = "step.completed"
    STEP_FAILED = "step.failed"
    FILE_EDITED = "file.edited"
    TEST_STARTED = "test.started"
    TEST_PASSED = "test.passed"
    TEST_FAILED = "test.failed"
    MODEL_CALL = "model.call"
    MODEL_RESPONSE = "model.response"
    ERROR = "error"
```

**Logging Outputs:**
- **JSON Logs:** `.orca/logs/events.jsonl` (structured, machine-readable)
- **Human Logs:** Console output (pretty-printed, colored)
- **Metrics:** `.orca/logs/metrics.jsonl` (performance stats)

**Metrics Tracked:**
- Latency per operation (planning, editing, execution, testing)
- Token usage per model call
- Cost estimation
- Success/failure rates
- Test coverage changes

#### 4.4 Security Layer

**Responsibility:** Safety validations and compliance

**Components:**

**1. Command Validator**
```python
BLOCKED_COMMANDS = [
    r"rm\s+-rf\s+/",
    r":(){ :|:& };:",  # fork bomb
    r"dd\s+if=/dev/zero",
    # ... more patterns
]

def is_safe_command(cmd: str) -> bool:
    for pattern in BLOCKED_COMMANDS:
        if re.search(pattern, cmd):
            return False
    return True
```

**2. File Write Protector**
```python
SENSITIVE_FILES = [
    ".env", "*.pem", "*.key", "*credentials*",
    ".aws/credentials", ".ssh/id_rsa"
]

def is_safe_file_write(path: str) -> bool:
    for pattern in SENSITIVE_FILES:
        if fnmatch.fnmatch(path, pattern):
            return False
    return True
```

**3. License Checker**
- Scan dependencies for licenses
- Detect GPL in MIT projects
- Alert on proprietary code inclusion

**4. Provenance Scanner**
- Track code source (LLM-generated vs human-written vs copied)
- Detect code from Stack Overflow, GitHub (via similarity)
- Maintain attribution metadata

---

## Data Flow

### Typical Task Execution Flow

```
User: "orca plan 'Add JWT authentication'"
   │
   ├─> CLI parses command, validates input
   │
   ├─> Planner Engine:
   │   ├─> RAG: Retrieve relevant files (auth.py, middleware.py, etc.)
   │   ├─> Belief Store: Load project architecture, conventions
   │   ├─> OpenRouter: Generate plan with GPT-4/Claude
   │   └─> Validate plan structure, save to .orca/plan.json
   │
   └─> Output plan to user, request approval

User: "orca exec --auto"
   │
   ├─> Executor Engine loads plan from .orca/plan.json
   │
   ├─> For each step:
   │   │
   │   ├─> Step 1: "Create JWT utility functions"
   │   │   ├─> Multi-File Editor: Generate code using LLM
   │   │   ├─> Syntax Validator: Check Python syntax
   │   │   ├─> Save file, generate diff
   │   │   └─> Log event: FILE_EDITED
   │   │
   │   ├─> Step 2: "Add middleware to FastAPI app"
   │   │   ├─> Multi-File Editor: AST-aware insertion
   │   │   ├─> Dependency Tracker: Update imports
   │   │   └─> Log event: FILE_EDITED
   │   │
   │   ├─> Step 3: "Write unit tests"
   │   │   ├─> Multi-File Editor: Generate test code
   │   │   ├─> Test Runner: Execute pytest
   │   │   ├─> Verify: All tests pass
   │   │   └─> Log event: TEST_PASSED
   │   │
   │   └─> Step 4: "Run integration tests"
   │       ├─> Test Runner: Execute full suite
   │       ├─> Result: 2 tests fail
   │       │
   │       ├─> Iteration Loop:
   │       │   ├─> Analyze failures (missing import)
   │       │   ├─> Generate fix (add import statement)
   │       │   ├─> Apply fix
   │       │   ├─> Re-run tests
   │       │   └─> Result: All pass ✓
   │       │
   │       └─> Log event: STEP_COMPLETED
   │
   ├─> Plan complete, generate summary
   └─> Output: "Successfully added JWT authentication. Modified 4 files, added 12 tests."
```

---

## Technology Decisions

### Finalized Decisions

| Component | Technology | Rationale |
|---|---|---|
| Language | Python 3.10+ | Rich ecosystem, type hints, asyncio |
| CLI Framework | Typer | Type-safe, auto-docs, modern |
| Packaging | Poetry | Dependency management, reproducible builds |
| Testing | pytest | Industry standard, rich plugins |
| Linting | Ruff | Fast, comprehensive |
| Formatting | Black | Opinionated, consistent |
| Type Checking | mypy | Strict type safety |

### To Be Decided (Phase 0 Research)

| Component | Leading Option | Alternatives |
|---|---|---|
| AST Parsing | tree-sitter | Language-specific parsers (ast, esprima) |
| Sandbox | Docker | firejail, bubblewrap, gVisor |
| Vector DB | ChromaDB | FAISS, Qdrant, Weaviate |
| Embeddings | CodeBERT | all-MiniLM-L6-v2, StarCoder |
| Config Format | TOML | YAML, JSON |

---

## Integration Points

### External Systems

**1. Version Control (Git)**
- Read current branch, diff, commit history
- Create commits with generated messages
- Detect uncommitted changes

**2. Package Managers**
- npm, pip, go mod, maven
- Dependency installation
- Vulnerability scanning

**3. CI/CD Systems**
- GitHub Actions (templates)
- GitLab CI (future)
- Jenkins (future)

**4. Code Hosting**
- GitHub (PR creation, issue linking)
- GitLab (future)

---

## Security Architecture

### Threat Model

**Assets:**
- User code and data
- API keys (OpenRouter, GitHub)
- Local filesystem

**Threats:**
1. **Malicious LLM Output:** Model generates harmful code
2. **Sandbox Escape:** Executed code breaks containment
3. **API Key Leakage:** Keys logged or transmitted insecurely
4. **Supply Chain:** Compromised dependencies
5. **File Overwrites:** Accidental deletion of important files

**Mitigations:**
1. **Code Validation:** Syntax, type, and security checks before execution
2. **Sandboxing:** Isolated execution environment with resource limits
3. **Secret Management:** Encrypted storage, never log keys
4. **Dependency Scanning:** Automated vulnerability checks
5. **Approval Gates:** Human review for destructive operations

---

## Scalability & Performance

### Performance Targets

| Operation | Target | Measurement |
|---|---|---|
| Plan generation | < 10s | 95th percentile |
| Single file edit | < 2s | Average |
| Test execution | < 30s | Typical project |
| RAG search | < 500ms | 95th percentile |
| Sandbox startup | < 3s | Average |

### Optimization Strategies

**1. Caching**
- LLM responses (same prompt → same response)
- Embeddings (file hash → vector)
- Test results (no code changes → same results)

**2. Parallelization**
- Independent plan steps executed concurrently
- Batch file embeddings
- Async I/O for API calls

**3. Incremental Operations**
- Only re-index changed files
- Only run tests for changed code
- Delta plans (modify existing plan vs create new)

**4. Resource Management**
- Connection pooling for API calls
- Lazy loading of embeddings
- Cleanup of unused sandboxes

---

## Future Considerations

### Plugin Architecture (Phase 4)

**Plugin Types:**
- Language support (Rust, Ruby, etc.)
- Test frameworks (RSpec, Mocha)
- LLM providers (OpenAI, Anthropic direct)
- Code analysis tools (SonarQube, Snyk)

**Plugin Interface:**
```python
class OrcaPlugin(Protocol):
    name: str
    version: str

    def register(self, registry: PluginRegistry) -> None: ...
    def on_plan_created(self, plan: Plan) -> None: ...
    def on_step_executed(self, step: PlanStep, result: StepResult) -> None: ...
```

### Multi-Agent Collaboration (Future)

- Specialized agents (backend, frontend, DevOps)
- Agent communication protocol
- Task decomposition across agents

---

**Document Status:** Living document, updated during implementation
**Next Steps:** Complete Phase 0 research, validate technology choices
**Reviewers:** Engineering team
