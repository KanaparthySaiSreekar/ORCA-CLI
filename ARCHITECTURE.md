# ORCA-CLI Architecture

**Version:** 2.0 (Enhanced)
**Date:** 2025-11-23 (Updated)
**Status:** Design Phase - Enhanced with Advanced Capabilities

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Principles](#architecture-principles)
3. [Component Architecture](#component-architecture)
4. [Enhanced Components (NEW)](#enhanced-components-new)
5. [Data Flow](#data-flow)
6. [Technology Decisions](#technology-decisions)
7. [Integration Points](#integration-points)
8. [Security Architecture](#security-architecture)
9. [Scalability & Performance](#scalability--performance)

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

## Enhanced Components (NEW)

**Added:** 2025-11-23

This section describes the advanced components added to the architecture to transform ORCA-CLI from a basic coding agent to a comprehensive AI-powered development platform.

### Enhanced Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                 Enhanced Service Layer                           │
├────────────────┬────────────────┬────────────────┬──────────────┤
│ Code           │ Refactoring &  │ AI Testing &   │ Plan         │
│ Intelligence   │ Quality        │ Analysis       │ Enhancement  │
│                │                │                │              │
│ - Call graphs  │ - Smart        │ - Test gen     │ - Learning   │
│ - Impact       │   refactor     │ - Coverage     │ - Templates  │
│   analysis     │ - Code review  │   gaps         │ - Parallel   │
│ - Complexity   │ - Doc gen      │ - Mutation     │ - Predict    │
│ - Dead code    │ - Arch smells  │ - Flaky det    │ - Context    │
└────────────────┴────────────────┴────────────────┴──────────────┘
```

### 5. Code Intelligence Layer

**Responsibility:** Deep code understanding and dependency analysis

#### 5.1 Call Graph & Dependency Analyzer

**Purpose:** Track and visualize code dependencies and call relationships

**Components:**

**Call Graph Generator:**
```python
@dataclass
class CallGraph:
    nodes: Dict[str, FunctionNode]  # function_id -> node
    edges: List[CallEdge]  # caller -> callee relationships
    entry_points: List[str]  # main, API endpoints, etc.

class CallGraphAnalyzer:
    def build_call_graph(self, files: List[str]) -> CallGraph:
        """Build complete call graph from source files."""
        pass

    def find_callers(self, function_name: str) -> List[FunctionNode]:
        """Find all functions that call the specified function."""
        pass

    def find_callees(self, function_name: str) -> List[FunctionNode]:
        """Find all functions called by the specified function."""
        pass

    def detect_cycles(self) -> List[List[str]]:
        """Detect circular call dependencies."""
        pass
```

**Technology Stack:**
- Python: `pycg`, `ast` module
- JavaScript: `madge`, custom analysis
- Go: `go-callvis`
- Cross-language: Custom tree-sitter queries

**Use Cases:**
- "Show me what breaks if I change this function"
- "Find all code paths that lead to this bug"
- "Visualize the dependency structure"

#### 5.2 Impact Analysis Engine

**Purpose:** Predict consequences of code changes

**Architecture:**
```python
@dataclass
class ImpactAnalysis:
    changed_files: List[str]
    direct_impacts: List[Impact]  # Immediately affected code
    indirect_impacts: List[Impact]  # Transitively affected
    breaking_changes: List[BreakingChange]
    affected_tests: List[str]
    risk_score: float  # 0-1, probability of issues

@dataclass
class Impact:
    file_path: str
    affected_symbols: List[str]  # functions, classes, vars
    impact_type: ImpactType  # SIGNATURE_CHANGE, BEHAVIOR_CHANGE, etc.
    severity: Severity  # CRITICAL, HIGH, MEDIUM, LOW

class ImpactAnalyzer:
    def analyze_changes(self, changes: List[FileChange]) -> ImpactAnalysis:
        """Analyze impact of proposed code changes."""
        # 1. Build call graph
        # 2. Identify changed symbols
        # 3. Propagate through dependency graph
        # 4. Classify impact severity
        # 5. Identify affected tests
        pass

    def predict_breaking_changes(self, changes: List[FileChange]) -> List[BreakingChange]:
        """Detect API breaking changes."""
        # - Function signature changes
        # - Removed public methods
        # - Changed return types
        # - Removed parameters
        pass
```

**Integration with Planning:**
```python
# Before executing a plan step, check impact
impact = analyzer.analyze_changes(plan_step.changes)
if impact.risk_score > 0.7:
    print(f"⚠️  High risk change detected!")
    print(f"Breaking changes: {len(impact.breaking_changes)}")
    print(f"Affected tests: {len(impact.affected_tests)}")
    user_approval = request_approval()
```

#### 5.3 Complexity & Quality Metrics

**Purpose:** Measure and track code quality

**Metrics Tracked:**

**Cyclomatic Complexity:**
- Measures number of independent paths through code
- Target: Keep functions under 10

**Cognitive Complexity:**
- Measures how hard code is to understand
- Accounts for nesting, breaks in flow

**Maintainability Index:**
- Combined metric (0-100)
- Considers volume, complexity, comment density

**Technical Debt Ratio:**
- Estimated time to fix vs time to build
- Track over time

**Implementation:**
```python
@dataclass
class QualityMetrics:
    cyclomatic_complexity: int
    cognitive_complexity: int
    maintainability_index: float
    lines_of_code: int
    comment_ratio: float
    test_coverage: float
    technical_debt_hours: float

class QualityAnalyzer:
    def analyze_file(self, file_path: str) -> QualityMetrics:
        """Compute quality metrics for a file."""
        pass

    def analyze_function(self, function: Function) -> QualityMetrics:
        """Compute quality metrics for a function."""
        pass

    def detect_hotspots(self, project: Project) -> List[Hotspot]:
        """Find areas with high complexity and frequent changes."""
        # High complexity + high churn = refactoring candidate
        pass
```

**Tools:**
- `radon` (Python)
- `gocyclo` (Go)
- `eslint-plugin-complexity` (JavaScript)

**Actions Based on Metrics:**
```python
if metrics.cyclomatic_complexity > 10:
    suggest_refactoring("Extract functions to reduce complexity")

if metrics.cognitive_complexity > 15:
    suggest_refactoring("Simplify logic flow, reduce nesting")

if metrics.maintainability_index < 20:
    flag_for_review("Low maintainability - consider rewrite")
```

#### 5.4 Dead Code Detection

**Purpose:** Find and remove unused code

**Detection Strategies:**

**Unused Functions:**
```python
class DeadCodeDetector:
    def find_unused_functions(self, project: Project) -> List[Function]:
        """Find functions never called."""
        call_graph = build_call_graph(project)
        all_functions = find_all_functions(project)
        called_functions = set(call_graph.nodes.keys())
        return [f for f in all_functions if f.name not in called_functions]
```

**Unreachable Code:**
```python
def detect_unreachable_code(function: Function) -> List[Statement]:
    """Find code after return/raise that can never execute."""
    # AST analysis to find statements after unconditional returns
    pass
```

**Unused Imports:**
```python
def find_unused_imports(file: File) -> List[Import]:
    """Find imported symbols never used."""
    imports = extract_imports(file)
    used_symbols = extract_symbol_usage(file)
    return [i for i in imports if i.symbol not in used_symbols]
```

**Safety:**
- Never auto-delete without approval
- Provide confidence scores
- Show evidence (call graph)
- Check for reflection/dynamic usage

### 6. Refactoring & Quality Engine

**Responsibility:** Intelligent code refactoring and quality improvement

#### 6.1 Refactoring Operations

**Supported Refactorings:**

**Extract Function/Method:**
```python
class RefactoringEngine:
    def extract_function(
        self,
        file: str,
        start_line: int,
        end_line: int,
        new_function_name: str
    ) -> RefactoringResult:
        """Extract code block into new function."""
        # 1. Analyze selected code
        # 2. Identify inputs (parameters needed)
        # 3. Identify outputs (return values)
        # 4. Generate new function
        # 5. Replace original with call
        # 6. Run tests to verify equivalence
        pass
```

**Rename Symbol:**
```python
def rename_symbol(
    self,
    old_name: str,
    new_name: str,
    scope: Scope = Scope.PROJECT
) -> RefactoringResult:
    """Rename symbol across all files."""
    # 1. Find all occurrences
    # 2. Update definitions
    # 3. Update references
    # 4. Update imports
    # 5. Update comments/docs
    # 6. Verify no conflicts
    pass
```

**Inline Function:**
```python
def inline_function(self, function_name: str) -> RefactoringResult:
    """Replace function calls with function body."""
    # Only safe if: function called once, no side effects
    pass
```

**Extract Interface/Protocol:**
```python
def extract_interface(
    self,
    class_name: str,
    interface_name: str,
    methods: List[str]
) -> RefactoringResult:
    """Extract interface from class."""
    # 1. Create protocol/interface
    # 2. Move method signatures
    # 3. Update class to implement interface
    pass
```

**Safety Guarantees:**
- Run full test suite before refactoring
- Run full test suite after refactoring
- Ensure tests pass both times
- Provide diff for review
- Rollback on failure

#### 6.2 Code Review Automation

**Purpose:** AI-powered code review

**Review Categories:**

**Code Quality:**
```python
@dataclass
class CodeReviewFinding:
    file: str
    line: int
    severity: Severity
    category: Category
    message: str
    suggestion: str
    auto_fixable: bool

class CodeReviewer:
    def review_code(self, changes: List[FileChange]) -> List[CodeReviewFinding]:
        """Perform automated code review."""
        findings = []
        findings.extend(self.check_naming_conventions(changes))
        findings.extend(self.check_complexity(changes))
        findings.extend(self.check_security(changes))
        findings.extend(self.check_performance(changes))
        findings.extend(self.check_best_practices(changes))
        return findings
```

**Security Checks:**
- SQL injection vulnerabilities
- XSS vulnerabilities
- Hardcoded secrets/credentials
- Insecure randomness
- Path traversal
- Command injection

**Performance Checks:**
- N+1 query patterns
- Inefficient algorithms (O(n²) when O(n) exists)
- Memory leaks
- Resource leaks (unclosed files, connections)

**Best Practices:**
- Missing error handling
- Missing logging
- Inconsistent formatting
- Missing docstrings
- Type hint coverage

**Output Format:**
```markdown
## Code Review: src/api/auth.py

### Critical Issues (2)
❌ Line 45: SQL injection vulnerability
   Current: f"SELECT * FROM users WHERE email = '{email}'"
   Fix: Use parameterized query
   ```python
   cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
   ```

❌ Line 78: Hardcoded API key
   Move to environment variable

### Warnings (3)
⚠️  Line 23: Function too complex (CC: 15)
   Suggestion: Extract validation logic to separate function

⚠️  Line 56: Missing error handling
   Add try/except for database operations

⚠️  Line 92: N+1 query detected
   Use join or batch loading

### Suggestions (5)
💡 Line 12: Consider using dataclass
💡 Line 34: Add type hints
💡 Line 67: Add docstring
```

#### 6.3 Documentation Generator

**Purpose:** Generate comprehensive documentation from code

**Documentation Types:**

**Docstrings:**
```python
class DocGenerator:
    def generate_docstring(self, function: Function) -> str:
        """Generate Google-style docstring."""
        # Analyze:
        # - Function signature
        # - Parameters and types
        # - Return type
        # - Exceptions raised
        # - Examples from tests
        # Use LLM to generate natural language description
        pass
```

**Example Output:**
```python
def authenticate_user(username: str, password: str) -> Optional[User]:
    """Authenticate user with username and password.

    Validates user credentials against the database and returns
    the user object if authentication succeeds.

    Args:
        username: The user's username
        password: The user's plain-text password (will be hashed)

    Returns:
        User object if authentication succeeds, None otherwise

    Raises:
        DatabaseError: If database connection fails
        ValidationError: If username or password is invalid format

    Examples:
        >>> user = authenticate_user("alice", "secret123")
        >>> user.email
        'alice@example.com'
    """
```

**API Documentation:**
```python
def generate_openapi_schema(app: FastAPIApp) -> OpenAPISchema:
    """Generate OpenAPI 3.0 schema from FastAPI application."""
    # Extract routes, parameters, responses
    # Generate schema JSON
    pass
```

**Project Documentation:**
```python
def generate_readme(project: Project) -> str:
    """Generate README.md from project structure."""
    # Analyze:
    # - Project structure
    # - Dependencies
    # - Entry points
    # - Configuration
    # Generate sections: Installation, Usage, API, etc.
    pass
```

### 7. AI Testing & Quality Assurance

**Responsibility:** Intelligent test generation and quality analysis

#### 7.1 Test Generation Engine

**Purpose:** Generate comprehensive test cases using AI

**Capabilities:**

**Unit Test Generation:**
```python
class TestGenerator:
    def generate_unit_tests(self, function: Function) -> List[Test]:
        """Generate comprehensive unit tests."""
        # 1. Analyze function signature and implementation
        # 2. Identify test scenarios:
        #    - Happy path
        #    - Edge cases (empty, null, boundary values)
        #    - Error conditions
        # 3. Use LLM to generate test code
        # 4. Verify tests compile and run
        # 5. Use mutation testing to verify quality
        pass
```

**Example Generated Tests:**
```python
# Original function
def divide(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Generated tests
def test_divide_positive_numbers():
    assert divide(10, 2) == 5.0

def test_divide_negative_numbers():
    assert divide(-10, 2) == -5.0

def test_divide_by_zero():
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)

def test_divide_float_precision():
    assert abs(divide(1, 3) - 0.333333) < 0.0001
```

**Integration Test Generation:**
```python
def generate_integration_tests(self, endpoint: APIEndpoint) -> List[Test]:
    """Generate API integration tests."""
    # 1. Analyze endpoint schema
    # 2. Generate test cases:
    #    - Valid requests
    #    - Invalid requests (validation)
    #    - Authentication/authorization
    #    - Error responses
    pass
```

#### 7.2 Coverage Gap Analysis

**Purpose:** Intelligent coverage improvement

**Architecture:**
```python
@dataclass
class CoverageGap:
    file: str
    line_range: Tuple[int, int]
    function: str
    gap_type: GapType  # UNCOVERED, BRANCH_MISSING, EXCEPTION_PATH
    priority: Priority  # CRITICAL, HIGH, MEDIUM, LOW
    suggested_tests: List[str]

class CoverageAnalyzer:
    def analyze_gaps(self, coverage_report: Coverage) -> List[CoverageGap]:
        """Find and prioritize coverage gaps."""
        gaps = []

        # Find uncovered lines
        gaps.extend(self.find_uncovered_lines(coverage_report))

        # Find missing branches
        gaps.extend(self.find_missing_branches(coverage_report))

        # Find uncovered exception paths
        gaps.extend(self.find_uncovered_exceptions(coverage_report))

        # Prioritize by:
        # - Complexity of uncovered code
        # - Criticality (auth, payment, etc.)
        # - Recent changes (high churn)
        return self.prioritize_gaps(gaps)
```

**Output:**
```
Coverage Gap Analysis for src/auth.py
Overall: 72% line, 58% branch coverage

Critical Gaps:
  ❌ Lines 45-52: Password reset flow (0% coverage)
     Priority: CRITICAL (authentication code)
     Suggested tests:
       - test_password_reset_valid_token()
       - test_password_reset_expired_token()
       - test_password_reset_invalid_token()

  ⚠️  Lines 78-85: Token refresh (50% branch coverage)
     Priority: HIGH
     Missing branch: refresh_token_revoked scenario
     Suggested test:
       - test_token_refresh_with_revoked_token()
```

#### 7.3 Mutation Testing

**Purpose:** Verify test quality by mutating code

**Implementation:**
```python
class MutationTester:
    def run_mutation_testing(self, file: str) -> MutationReport:
        """Run mutation testing on file."""
        # 1. Generate mutants (small code changes)
        # 2. Run test suite against each mutant
        # 3. Track which mutants survive (weak tests)
        # 4. Suggest test improvements
        pass
```

**Mutation Types:**
- Arithmetic operators: `+` → `-`, `*` → `/`
- Comparison operators: `>` → `>=`, `==` → `!=`
- Boolean logic: `and` → `or`, `not` → removed
- Return values: `return x` → `return None`
- Delete statements

**Example Output:**
```
Mutation Testing Results for src/calculator.py

Mutants: 32 generated
Killed: 28 (87.5%)
Survived: 4 (12.5%)

Surviving Mutants (weak tests):
  Line 42: Changed > to >= (no test detected)
    Suggestion: Add boundary test case
    ```python
    def test_threshold_boundary():
        assert process(10) == "high"  # exactly at threshold
        assert process(9) == "low"    # just below threshold
    ```

  Line 58: Removed error handling (no test detected)
    Suggestion: Add error scenario test
```

### 8. Enhanced Planning & Learning

**Responsibility:** Intelligent planning with learning from experience

#### 8.1 Learning System

**Purpose:** Learn from past executions to improve over time

**Architecture:**
```python
@dataclass
class PlanExecution:
    plan_id: str
    task_description: str
    steps: List[PlanStep]
    model_used: str
    success: bool
    execution_time: float
    failure_reasons: List[str]
    fixes_applied: List[str]
    timestamp: datetime

class LearningSystem:
    def __init__(self):
        self.execution_history: List[PlanExecution] = []
        self.success_patterns: Dict[str, SuccessPattern] = {}
        self.failure_patterns: Dict[str, FailurePattern] = {}

    def record_execution(self, execution: PlanExecution):
        """Record plan execution for learning."""
        self.execution_history.append(execution)
        self.update_patterns(execution)

    def suggest_improvements(self, plan: Plan) -> List[Suggestion]:
        """Suggest plan improvements based on history."""
        # Check if similar plans failed before
        # Suggest preventive measures
        # Recommend better step ordering
        pass
```

**Pattern Recognition:**
```python
# Example learned patterns
{
    "add_api_endpoint": {
        "success_rate": 0.85,
        "avg_time": 180,  # seconds
        "best_approach": {
            "order": ["create_schema", "create_route", "write_tests", "update_docs"],
            "common_pitfalls": ["forgetting_import", "missing_dependency"]
        }
    }
}
```

#### 8.2 Plan Templates

**Purpose:** Reusable plan patterns for common tasks

**Template System:**
```python
@dataclass
class PlanTemplate:
    name: str
    description: str
    variables: List[TemplateVariable]
    steps: List[PlanStepTemplate]
    prerequisites: List[str]
    estimated_time: int  # seconds

class TemplateEngine:
    def instantiate_template(
        self,
        template_name: str,
        variables: Dict[str, Any]
    ) -> Plan:
        """Create plan from template with variable substitution."""
        pass
```

**Built-in Templates:**
- `add-api-endpoint`
- `add-database-model`
- `refactor-extract-function`
- `add-authentication`
- `setup-ci-cd`
- `migrate-database`

**Usage:**
```bash
$ orca plan template add-api-endpoint \
    --path=/users/{id} \
    --method=GET \
    --model=User

Generated plan from template:
  1. Create Pydantic schema (UserResponse)
  2. Create route handler (get_user)
  3. Add database query
  4. Write unit tests
  5. Write integration tests
  6. Update OpenAPI schema
```

#### 8.3 Parallel Execution

**Purpose:** Execute independent steps concurrently

**Implementation:**
```python
class ParallelExecutor:
    def build_dependency_dag(self, plan: Plan) -> DAG:
        """Build directed acyclic graph of step dependencies."""
        pass

    def identify_parallel_groups(self, dag: DAG) -> List[List[PlanStep]]:
        """Group steps that can run in parallel."""
        # Use topological sort to identify levels
        # Steps at same level can run in parallel
        pass

    async def execute_parallel(self, plan: Plan) -> ExecutionResult:
        """Execute plan with parallelism."""
        dag = self.build_dependency_dag(plan)
        groups = self.identify_parallel_groups(dag)

        for group in groups:
            # Execute all steps in group concurrently
            tasks = [self.execute_step(step) for step in group]
            results = await asyncio.gather(*tasks)

            # Check for failures
            if any(not r.success for r in results):
                return self.handle_failure(results)

        return ExecutionResult(success=True)
```

**Benefits:**
- 2-5x faster for complex plans
- Better resource utilization
- Visual feedback on parallelism

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
