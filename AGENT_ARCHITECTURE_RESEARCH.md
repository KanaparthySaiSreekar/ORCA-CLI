# Coding Agent Architecture Research & Best Practices

**Version:** 1.0
**Date:** 2025-11-23
**Status:** Research Findings & Recommendations

---

## Executive Summary

This document synthesizes cutting-edge research on coding agent architectures, tool design, and implementation patterns based on analysis of state-of-the-art systems (Aider, SWE-agent, Devin) and recent academic research (2024-2025).

**Core Insight:** The effectiveness of a coding agent is primarily determined by its **architecture**, **tool design**, and **agent loop** rather than the LLM alone. A well-designed agent with clear tools and robust control flow will dramatically outperform a naive LLM-only approach.

**Key Findings:**
- **Agent Architecture Patterns**: ReAct achieves 86% success rate; Plan-and-Execute forces explicit reasoning
- **Tool Design**: SWE-agent's Agent-Computer Interface (ACI) increased performance from 3.8% → 12.5% on SWE-bench
- **Context Management**: Aider's hierarchical AST-based approach handles enterprise codebases (2000+ files)
- **Hybrid Architecture**: Deterministic orchestration + non-deterministic reasoning = production reliability
- **Self-Correction Loops**: Enable agents to recover from 70%+ of initial failures

---

## Table of Contents

1. [Agent Architecture Patterns](#agent-architecture-patterns)
2. [Tool Design Principles](#tool-design-principles)
3. [Context Management Strategies](#context-management-strategies)
4. [Agent Loop Patterns](#agent-loop-patterns)
5. [Deterministic vs Non-Deterministic Components](#deterministic-vs-non-deterministic-components)
6. [Case Studies: Successful Agents](#case-studies-successful-agents)
7. [Recommendations for ORCA-CLI](#recommendations-for-orca-cli)
8. [References](#references)

---

## Agent Architecture Patterns

### 1. ReAct (Reason + Act)

**Pattern:** Interleaved reasoning and action in a loop

**Structure:**
```
Loop:
  1. Thought: Reason about current state
  2. Action: Execute tool/command
  3. Observation: Observe results
  4. [Repeat until goal achieved]
```

**Performance:**
- 86% success rate for Claude Computer Use implementations
- Reduces hallucinations by grounding each step in observable results
- Best for: Exploratory tasks, debugging, investigation

**Strengths:**
- Flexible and adaptive
- Self-correcting through observation
- Natural reasoning traces

**Weaknesses:**
- Can be inefficient (many LLM calls)
- May get stuck in loops
- Harder to parallelize

**Implementation:**
```python
class ReActAgent:
    def execute(self, goal: str) -> Result:
        state = initial_state
        max_iterations = 10

        for i in range(max_iterations):
            # Thought: Reason about what to do
            thought = self.llm.generate(
                f"Goal: {goal}\nCurrent state: {state}\nWhat should I do next?"
            )

            # Action: Parse and execute action
            action = self.parse_action(thought)
            if action.is_terminal():
                return Result(success=True, output=action.output)

            # Observation: Execute and observe
            observation = self.execute_tool(action)
            state = self.update_state(state, observation)

            # Check if goal achieved
            if self.is_goal_met(goal, state):
                return Result(success=True, output=state)

        return Result(success=False, reason="Max iterations exceeded")
```

### 2. Plan-and-Execute

**Pattern:** Separate planning from execution

**Structure:**
```
Phase 1 (Planning):
  - Analyze goal
  - Generate complete multi-step plan
  - Validate plan feasibility

Phase 2 (Execution):
  - Execute steps sequentially
  - Verify each step
  - Replan if needed
```

**Performance:**
- Forces explicit "thinking through" of all steps
- Better for complex, multi-step tasks
- Easier to review and approve before execution

**Strengths:**
- Clear separation of concerns
- Easier to understand and audit
- Can optimize plan before execution
- Better parallelization opportunities

**Weaknesses:**
- Less flexible to changing conditions
- May need replanning
- Initial planning can be expensive

**Implementation:**
```python
class PlanAndExecuteAgent:
    def execute(self, goal: str) -> Result:
        # Phase 1: Planning
        plan = self.planner.generate_plan(goal)
        plan = self.planner.validate_plan(plan)
        plan = self.planner.optimize_plan(plan)  # Reorder, parallelize

        # Optional: Human approval
        if self.config.require_approval:
            plan = self.request_approval(plan)

        # Phase 2: Execution
        results = []
        for step in plan.steps:
            result = self.executor.execute_step(step)

            if not result.success:
                # Replan or retry
                if self.should_replan(result):
                    remaining = plan.remaining_steps(step)
                    new_plan = self.planner.replan(goal, results, remaining)
                    plan = new_plan
                else:
                    return Result(success=False, reason=result.error)

            results.append(result)

        return Result(success=True, output=results)
```

### 3. ReWOO (Reasoning Without Observation)

**Pattern:** Plan all actions upfront, execute in parallel, then reason on results

**Structure:**
```
Phase 1: Generate execution plan with variable placeholders
  tool1(args) -> #E1
  tool2(#E1, other_args) -> #E2
  tool3(#E2) -> #E3

Phase 2: Execute tools (potentially in parallel)

Phase 3: Reason on results
```

**Performance:**
- More efficient than ReAct (fewer LLM calls)
- Enables parallelization
- Better for independent operations

**Strengths:**
- Efficient execution
- Parallelizable
- Fewer LLM calls

**Weaknesses:**
- Less adaptive
- Can't handle dynamic dependencies
- Harder to debug

### 4. LLMCompiler

**Pattern:** Optimize execution plan for maximum parallelism

**Performance:**
- **3.6x faster** than sequential execution
- Automatically identifies parallel execution opportunities
- Builds dependency DAG and schedules optimally

**Use Case:** When you have many independent operations (e.g., analyzing multiple files, running multiple tests)

### Comparison Matrix

| Pattern | Flexibility | Efficiency | Parallelism | Complexity | Best For |
|---------|------------|------------|-------------|------------|----------|
| **ReAct** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐ | Exploratory, debugging |
| **Plan-and-Execute** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | Complex multi-step tasks |
| **ReWOO** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Independent operations |
| **LLMCompiler** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Parallel-heavy workloads |

---

## Tool Design Principles

### The Critical Insight: Tool Design > Model Selection

**Research Finding:** SWE-agent achieved **12.5%** on SWE-bench (vs 3.8% previous SOTA) primarily through better **Agent-Computer Interface (ACI)** design, not model improvements.

**Key Quote from SWE-agent Paper:**
> "Careful ACI design can substantially improve LM agent performance without modifying the underlying LM's weights."

### Core Principles

#### 1. Self-Contained and Robust

**Bad Tool:**
```python
def edit_file(path: str, new_content: str):
    """Edit a file."""
    with open(path, 'w') as f:
        f.write(new_content)
```

**Good Tool:**
```python
def edit_file(path: str, new_content: str) -> EditResult:
    """
    Edit a file with automatic backup and validation.

    Args:
        path: Absolute file path (e.g., /project/src/main.py)
        new_content: Complete new file content

    Returns:
        EditResult with success status, diff, and backup location

    Errors:
        - FileNotFoundError: File doesn't exist
        - PermissionError: No write permission
        - SyntaxError: Invalid syntax for code files
    """
    # Validate path exists
    if not os.path.exists(path):
        return EditResult(
            success=False,
            error=f"File not found: {path}. Did you mean {suggest_similar(path)}?"
        )

    # Create backup
    backup_path = create_backup(path)

    # Validate syntax if code file
    if is_code_file(path):
        syntax_errors = validate_syntax(new_content, get_language(path))
        if syntax_errors:
            return EditResult(
                success=False,
                error=f"Syntax errors:\n{format_errors(syntax_errors)}"
            )

    # Write file
    try:
        with open(path, 'w') as f:
            f.write(new_content)
    except Exception as e:
        # Restore backup
        restore_backup(backup_path, path)
        return EditResult(success=False, error=str(e))

    # Generate diff
    diff = generate_diff(backup_path, path)

    return EditResult(
        success=True,
        diff=diff,
        backup_path=backup_path,
        lines_added=diff.additions,
        lines_removed=diff.deletions
    )
```

**Why Good:**
- Clear, descriptive docstring
- Validates inputs
- Provides helpful error messages with suggestions
- Automatic backup/recovery
- Returns structured result
- Includes context (diff, line counts)

#### 2. Clear and Unambiguous

**Anthropic Best Practice:**
> "A common failure mode is bloated tool sets that cover too much functionality or lead to ambiguous decision points about which tool to use—if a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better."

**Anti-Pattern: Ambiguous Tool Set**
```python
# BAD: Too many overlapping tools
tools = [
    "read_file",          # Reads entire file
    "read_file_lines",    # Reads specific lines
    "read_file_range",    # Reads line range
    "peek_file",          # Reads first N lines
    "tail_file",          # Reads last N lines
    "grep_file",          # Searches file
]
```

**Better: Single Focused Tool**
```python
# GOOD: One clear tool with options
def read_file(
    path: str,
    *,
    start_line: Optional[int] = None,
    end_line: Optional[int] = None,
    max_lines: Optional[int] = None,
    search_pattern: Optional[str] = None
) -> FileContent:
    """
    Read file contents with optional filtering.

    Examples:
        read_file("/src/main.py")  # Read entire file
        read_file("/src/main.py", start_line=10, end_line=20)  # Read lines 10-20
        read_file("/src/main.py", search_pattern="def.*auth")  # Search for pattern
    """
```

#### 3. Compact Representation

**SWE-agent Insight:** Compact, efficient file editing is critical to performance.

**Anti-Pattern: Verbose Output**
```
$ ls /project
File: main.py
  - Path: /project/main.py
  - Size: 1024 bytes
  - Modified: 2025-11-23 10:30:00
  - Permissions: rw-r--r--
  - Owner: user

File: utils.py
  - Path: /project/utils.py
  - Size: 512 bytes
  - Modified: 2025-11-23 09:15:00
  - Permissions: rw-r--r--
  - Owner: user
```

**Better: Compact, Scannable**
```
$ ls /project
main.py (1.0K, 2025-11-23 10:30)
utils.py (512B, 2025-11-23 09:15)
```

#### 4. Structured Output

**Always return structured data, not raw text:**

```python
@dataclass
class TestResult:
    framework: str  # pytest, jest, etc.
    total_tests: int
    passed: int
    failed: int
    skipped: int
    duration_seconds: float
    failures: List[TestFailure]

@dataclass
class TestFailure:
    test_name: str
    file_path: str
    line_number: int
    error_type: str  # AssertionError, TypeError, etc.
    error_message: str
    stack_trace: str

    def to_human_readable(self) -> str:
        """Format for display"""
        return f"""
        ❌ {self.test_name}
        File: {self.file_path}:{self.line_number}
        Error: {self.error_type}: {self.error_message}
        """
```

**Why:** Agent can programmatically analyze failures, count errors, extract patterns, etc.

#### 5. Feedback for Self-Correction

**Tools should provide actionable feedback:**

```python
def run_linter(file_path: str) -> LintResult:
    """
    Run linter and return structured results.

    Returns not just errors, but:
    - Specific line numbers
    - Error codes and categories
    - Suggested fixes
    - Severity levels
    """
    return LintResult(
        issues=[
            LintIssue(
                line=45,
                column=10,
                code="E501",
                message="Line too long (95 > 88 characters)",
                severity="warning",
                suggested_fix="Break line at operator",
                category="style"
            ),
            LintIssue(
                line=78,
                column=5,
                code="F841",
                message="Local variable 'result' is assigned but never used",
                severity="error",
                suggested_fix="Remove unused variable or use it",
                category="unused-code"
            )
        ],
        summary=LintSummary(
            total_issues=2,
            errors=1,
            warnings=1,
            fixable=2
        )
    )
```

---

## Context Management Strategies

### The Challenge

**Problem:** GPT-4's 128k context window holds ~50 average Java files. Enterprise applications have 2000+ files.

**Solution:** Smart context management, not larger context windows.

### 1. Just-in-Time Context (Recommended)

**Pattern:** Maintain lightweight identifiers, load data dynamically.

```python
class ContextManager:
    def __init__(self):
        # Don't load files into context
        # Store references only
        self.file_index = {
            "path/to/file.py": FileMetadata(
                path="path/to/file.py",
                language="python",
                size=1024,
                last_modified="2025-11-23",
                symbols=["class User", "def authenticate", ...]
            )
        }

    def get_context_for_task(self, task: str) -> Context:
        """Load only relevant context for this specific task."""

        # 1. Identify relevant files (semantic search, symbol search)
        relevant_files = self.search_relevant(task)

        # 2. Load only those files
        context = Context()
        for file_path in relevant_files[:5]:  # Limit to top 5
            content = self.load_file(file_path)
            context.add_file(file_path, content)

        # 3. Add related context (imports, dependencies)
        for file_path in context.files:
            dependencies = self.get_direct_dependencies(file_path)
            context.add_metadata("dependencies", dependencies)

        return context
```

**Benefits:**
- Only load what you need
- Fresh data every time
- Scalable to huge codebases

### 2. Hierarchical Context (Aider's Approach)

**Levels of Detail:**

```
Level 1: Repository Map (always in context)
  - File tree structure
  - Symbol index (all classes, functions)
  - Dependency graph
  - ~1-2K tokens

Level 2: File Summaries (loaded on demand)
  - File purpose
  - Public API surface
  - Key abstractions
  - ~500 tokens per file

Level 3: Full File Content (loaded when editing)
  - Complete source code
  - ~500-5000 tokens per file
```

**Aider's AST-Based Approach:**

```python
class RepositoryMapper:
    def build_map(self, repo_path: str) -> RepoMap:
        """Build AST-based repository map."""

        map = RepoMap()

        for file in find_all_files(repo_path):
            # Parse with tree-sitter
            tree = self.parser.parse(file)

            # Extract structure (NOT implementation)
            symbols = extract_symbols(tree)  # Classes, functions, imports

            map.add_file(
                path=file.path,
                symbols=symbols,
                dependencies=extract_dependencies(tree)
            )

        return map

    def to_context_string(self, map: RepoMap) -> str:
        """
        Convert to compact string representation.

        Output:

        src/auth.py
          class AuthService:
            def authenticate(username: str, password: str) -> User
            def create_token(user: User) -> str

        src/models.py
          class User:
            id: int
            username: str
            email: str
        """
```

**Why This Works:**
- LLM knows project structure without seeing code
- Can identify relevant files before loading them
- Only loads implementation when needed

### 3. Context Prioritization

**Strategy:** Rank context by relevance, include highest-priority first.

```python
class ContextPrioritizer:
    def prioritize(self, task: str, available_files: List[str]) -> List[str]:
        """Rank files by relevance to task."""

        scores = []
        for file_path in available_files:
            score = self.calculate_relevance(task, file_path)
            scores.append((file_path, score))

        # Sort by score descending
        scores.sort(key=lambda x: x[1], reverse=True)

        return [path for path, _ in scores]

    def calculate_relevance(self, task: str, file_path: str) -> float:
        """Multi-factor relevance scoring."""

        score = 0.0

        # Factor 1: Keyword matching
        task_keywords = extract_keywords(task)
        file_content = self.file_index[file_path]
        keyword_matches = count_matches(task_keywords, file_content.symbols)
        score += keyword_matches * 2.0

        # Factor 2: Semantic similarity (embeddings)
        semantic_sim = cosine_similarity(
            embed(task),
            file_content.embedding
        )
        score += semantic_sim * 3.0

        # Factor 3: Recently modified (recency)
        days_since_modified = (now() - file_content.last_modified).days
        recency_score = 1.0 / (1.0 + days_since_modified / 30.0)
        score += recency_score * 1.0

        # Factor 4: File size (prefer smaller files)
        size_penalty = min(file_content.size / 10000, 1.0)
        score -= size_penalty * 0.5

        # Factor 5: Test files (boost test files for testing tasks)
        if "test" in task.lower() and "test" in file_path:
            score += 2.0

        return score
```

### 4. Context Window Maintenance

**Problem:** During long sessions, context fills with irrelevant information.

**Solution:** Periodic context cleanup.

```python
class SessionManager:
    def __init__(self):
        self.context_window = ContextWindow(max_tokens=100000)

    def should_clear_context(self) -> bool:
        """Decide when to clear context."""
        return (
            self.context_window.usage() > 0.8 or  # 80% full
            self.turns_since_last_clear > 20 or   # 20 turns
            self.task_completed()                 # Task finished
        )

    def clear_context(self):
        """Clear context but preserve essentials."""

        # Keep:
        # - Repository map (small, always useful)
        # - Current task/goal
        # - Recent errors (for context)

        # Remove:
        # - Old file contents
        # - Previous task history
        # - Intermediate results

        essentials = self.extract_essentials()
        self.context_window.clear()
        self.context_window.add(essentials)
```

---

## Agent Loop Patterns

### The Universal Pattern: LLM + Loop + Tools

**Core Insight:** All agents are fundamentally loops that combine reasoning (LLM) with actions (tools).

```python
def agent_loop(goal: str) -> Result:
    """Universal agent loop pattern."""

    state = initialize_state(goal)
    max_iterations = 20  # ALWAYS have a limit

    for iteration in range(max_iterations):
        # 1. Observe current state
        observation = observe(state)

        # 2. Reason about next action (LLM)
        action = llm.decide_next_action(
            goal=goal,
            state=state,
            observation=observation,
            available_tools=get_tools()
        )

        # 3. Check for termination
        if action.is_terminal():
            return Result(success=action.success, output=state)

        # 4. Execute action (Tool)
        result = execute_tool(action)

        # 5. Update state
        state = update_state(state, result)

        # 6. Check if goal achieved
        if is_goal_met(goal, state):
            return Result(success=True, output=state)

    # Fallback: max iterations reached
    return Result(success=False, reason="Max iterations exceeded")
```

### Self-Correction Loop

**Components:**
1. **Error Detection**: Identify failures
2. **Reflection**: Analyze what went wrong
3. **Retry**: Attempt fix

```python
class SelfCorrectingAgent:
    def execute_with_retries(self, step: PlanStep, max_retries: int = 3) -> Result:
        """Execute with automatic error recovery."""

        for attempt in range(max_retries):
            result = self.execute_step(step)

            if result.success:
                return result

            # Error detection
            error = self.analyze_error(result)

            # Reflection: Why did it fail?
            reflection = self.llm.reflect(
                f"""
                Task: {step.description}
                Attempt: {attempt + 1}
                Error: {error}

                Why did this fail? What should be done differently?
                """
            )

            # Generate corrective action
            fix = self.llm.generate_fix(
                task=step,
                error=error,
                reflection=reflection,
                previous_attempts=[...attempts_so_far...]
            )

            # Apply fix and retry
            step = self.apply_fix(step, fix)

        return Result(success=False, reason=f"Failed after {max_retries} attempts")

    def analyze_error(self, result: Result) -> ErrorAnalysis:
        """Categorize and analyze error."""

        if "SyntaxError" in result.error:
            return ErrorAnalysis(
                category="syntax_error",
                fixable=True,
                suggested_action="Parse error message, fix syntax"
            )
        elif "FileNotFoundError" in result.error:
            return ErrorAnalysis(
                category="missing_file",
                fixable=True,
                suggested_action="Create file or fix path"
            )
        elif "AssertionError" in result.error:
            return ErrorAnalysis(
                category="test_failure",
                fixable=True,
                suggested_action="Analyze test expectation vs actual"
            )
        else:
            return ErrorAnalysis(
                category="unknown",
                fixable=False,
                suggested_action="Request human intervention"
            )
```

### Loop Agent Pattern (Advanced)

**Use Case:** Tasks requiring iterative refinement (code generation, test writing, refactoring)

```python
class LoopAgent:
    """
    Repeatedly executes subagents until termination condition.

    Example: Code → Test → Fix → Test → Fix → Done
    """

    def __init__(self, subagents: List[Agent], termination_fn: Callable):
        self.subagents = subagents
        self.termination_fn = termination_fn
        self.max_iterations = 10  # Safety limit

    def execute(self, task: Task) -> Result:
        state = State(task=task)

        for iteration in range(self.max_iterations):
            # Execute each subagent in sequence
            for subagent in self.subagents:
                result = subagent.execute(state)
                state = state.update(result)

            # Check termination
            if self.termination_fn(state):
                return Result(success=True, state=state)

        return Result(success=False, reason="Max iterations")

# Example: Write-Test-Fix loop
agent = LoopAgent(
    subagents=[
        CodeWriterAgent(),   # Writes code
        TestRunnerAgent(),   # Runs tests
        DebuggerAgent()      # Fixes failures
    ],
    termination_fn=lambda state: state.all_tests_passing()
)
```

---

## Deterministic vs Non-Deterministic Components

### The Hybrid Architecture Principle

**Key Insight:** Production systems need both deterministic (reliable, auditable) and non-deterministic (flexible, intelligent) components.

```
┌─────────────────────────────────────────────────┐
│         Orchestration Layer                     │
│         (Deterministic)                         │
│  - Workflow execution                           │
│  - State management                             │
│  - Error handling                               │
│  - Audit trail                                  │
└────────────┬───────────────────────────────────┘
             │
    ┌────────┴─────────┐
    │                  │
┌───▼────────┐   ┌─────▼────────┐
│ Activity 1 │   │ Activity 2   │
│ (Non-Det)  │   │ (Non-Det)    │
│            │   │              │
│ - LLM call │   │ - Code gen   │
│ - Planning │   │ - Analysis   │
└────────────┘   └──────────────┘
```

### Deterministic Components

**Where you MUST be deterministic:**

1. **Parsing & Validation**
```python
# Deterministic: Always produces same AST for same code
def parse_code(source: str, language: str) -> AST:
    parser = get_parser(language)
    return parser.parse(source)

# Deterministic: Syntax check
def validate_syntax(code: str, language: str) -> ValidationResult:
    try:
        ast = parse_code(code, language)
        return ValidationResult(valid=True)
    except SyntaxError as e:
        return ValidationResult(valid=False, error=str(e))
```

2. **Execution & Testing**
```python
# Deterministic: Always runs same tests
def run_tests(test_files: List[str]) -> TestResult:
    runner = get_test_runner()
    return runner.run(test_files)
```

3. **Diff Generation**
```python
# Deterministic: Same files → same diff
def generate_diff(old_content: str, new_content: str) -> Diff:
    return unified_diff(old_content, new_content)
```

4. **State Persistence**
```python
# Deterministic: Transactional, atomic updates
def save_state(state: State) -> None:
    # Atomic write with checksum
    temp_file = f"{STATE_FILE}.tmp"
    with open(temp_file, 'w') as f:
        json.dump(state.to_dict(), f)

    # Atomic rename (POSIX guarantee)
    os.rename(temp_file, STATE_FILE)
```

### Non-Deterministic Components

**Where non-determinism is acceptable (and beneficial):**

1. **Planning**
```python
# Non-deterministic: Multiple valid plans
def generate_plan(goal: str, context: Context) -> Plan:
    # LLM can generate different plans each time
    # All might be valid approaches
    return llm.generate(f"Create plan for: {goal}")
```

2. **Code Generation**
```python
# Non-deterministic: Multiple valid implementations
def generate_function(spec: FunctionSpec) -> str:
    # Different syntactic forms, all correct
    return llm.generate(f"Implement: {spec}")
```

3. **Error Analysis**
```python
# Non-deterministic: Multiple valid interpretations
def analyze_error(error: Error, context: Context) -> ErrorAnalysis:
    return llm.analyze(f"Why did this fail? {error}")
```

### Separation Strategy

```python
class HybridAgent:
    """Combines deterministic orchestration with non-deterministic reasoning."""

    def __init__(self):
        # Deterministic components
        self.parser = ASTParser()
        self.validator = SyntaxValidator()
        self.test_runner = TestRunner()

        # Non-deterministic components
        self.planner = LLMPlanner()
        self.code_generator = LLMCodeGenerator()
        self.analyzer = LLMAnalyzer()

    def execute_task(self, task: Task) -> Result:
        """Orchestration is deterministic."""

        # Non-deterministic: Generate plan
        plan = self.planner.generate(task)

        # Deterministic: Validate plan structure
        if not self.validate_plan(plan):
            return Result(success=False, error="Invalid plan structure")

        # Execute steps
        for step in plan.steps:
            # Non-deterministic: Generate code
            code = self.code_generator.generate(step)

            # Deterministic: Validate syntax
            validation = self.validator.validate(code)
            if not validation.valid:
                return Result(success=False, error=validation.error)

            # Deterministic: Run tests
            test_result = self.test_runner.run(step.test_files)
            if not test_result.passed:
                # Non-deterministic: Analyze failures
                analysis = self.analyzer.analyze(test_result.failures)

                # Deterministic: Apply fixes
                code = self.apply_fixes(code, analysis.suggested_fixes)

        return Result(success=True)
```

---

## Case Studies: Successful Agents

### 1. Aider

**Performance:**
- Handles enterprise codebases (2000+ files)
- High user satisfaction for multi-file edits
- Strong git integration

**Architecture Insights:**

#### Repository Mapping with AST
```python
# Key Innovation: Don't send code, send structure
map = """
src/auth.py:
  class AuthService:
    def authenticate(username: str, password: str) -> User
    def create_token(user: User) -> str

src/models.py:
  class User:
    id: int
    username: str
"""

# LLM can reason: "I need to edit AuthService.authenticate"
# THEN load src/auth.py into context
```

**Why This Works:**
- Repository map is ~1-2K tokens for large projects
- LLM knows structure without seeing implementation
- Load implementation only when editing

#### Architect/Editor Split

```python
# Mode 1: Architect
# - Discuss design, architecture, approaches
# - No code changes
# - Cheap, fast iterations

# Mode 2: Editor (with architect reasoning)
# Step 1: Architect reasons about solution
architect_plan = llm.generate("How should I solve: {task}")

# Step 2: Editor generates well-formed code edits
code_edits = llm.generate(
    f"Given plan: {architect_plan}, generate code edits"
)

# Benefit: Separates reasoning from code generation
# - Better accuracy
# - More consistent edits
# - Can use different models (fast for editing, smart for architecture)
```

#### Hierarchical Context
- Level 1: Always in context (repo map)
- Level 2: On demand (file summaries)
- Level 3: When editing (full code)

**Takeaways for ORCA-CLI:**
- ✅ Implement AST-based repository mapping
- ✅ Use architect/editor pattern for complex tasks
- ✅ Hierarchical context loading
- ✅ Git-first workflow with auto-commits

### 2. SWE-agent

**Performance:**
- **12.5%** on SWE-bench (vs 3.8% previous SOTA)
- Achieved through ACI design, not model improvements

**Architecture Insights:**

#### Agent-Computer Interface (ACI) Design

**The Critical Finding:**
> "Careful ACI design can substantially improve LM agent performance without modifying the underlying LM's weights."

**Key Principles:**

1. **Consolidated Commands**
```python
# BAD: Too many tools
tools = ["open_file", "read_file", "edit_file", "save_file", "close_file"]

# GOOD: Single editor with modes
tool = "edit"  # Handles open, read, edit, save in one
```

2. **Efficient File Viewer**
```python
# BAD: Show entire file
def show_file(path: str) -> str:
    with open(path) as f:
        return f.read()  # 5000 lines → huge context

# GOOD: Compact view with line numbers, scrolling
def show_file(path: str, start: int = 1, num_lines: int = 100) -> str:
    with open(path) as f:
        lines = f.readlines()

    return format_with_line_numbers(
        lines[start-1:start+num_lines-1],
        start_line=start
    )
```

3. **Structured Edit Commands**
```python
# BAD: Replace entire file
edit_file(path, new_content)  # Huge token cost

# GOOD: Targeted edits
edit_file_lines(
    path="/src/auth.py",
    start_line=45,
    end_line=52,
    new_content="..."
)  # Minimal tokens
```

**Takeaways for ORCA-CLI:**
- ✅ Design custom ACI optimized for LLM agents
- ✅ Consolidate related operations into single tools
- ✅ Use compact, structured representations
- ✅ Optimize for token efficiency

### 3. Devin

**Performance:**
- **13.86%** on SWE-bench
- Multi-agent coordination
- Self-assessed confidence

**Architecture Insights:**

#### Sandboxed Environment
```
┌─────────────────────────────────────┐
│  Devin's Workspace                  │
├─────────────────────────────────────┤
│  Shell    │  Editor   │  Browser    │
│  - Run    │  - Edit   │  - Docs     │
│  - Test   │  - View   │  - Search   │
│  - Debug  │  - Diff   │  - Test UI  │
└─────────────────────────────────────┘
```

**Everything a developer needs, isolated.**

#### The Planner
```python
class DevinPlanner:
    def plan(self, task: str) -> Plan:
        """
        Break down task into concrete steps.

        Example:
        Task: "Add JWT authentication"

        Plan:
        1. Research JWT libraries (browse docs)
        2. Install dependencies (shell)
        3. Create auth module (editor)
        4. Write tests (editor)
        5. Run tests (shell)
        6. Fix failures (iterate)
        7. Commit changes (shell)
        """
```

#### Browser Integration
```python
# Agent can:
# - Search StackOverflow
# - Read API docs
# - Test web apps
# - Look up error messages

browser.navigate("https://stackoverflow.com/search?q=JWT+Python")
content = browser.get_text()
relevant_answer = llm.extract_answer(content)
```

#### Multi-Agent Dispatch (Advanced)
```python
# Later versions: Agent coordination
main_agent = MainAgent()
main_agent.dispatch_task("Write tests", TestWritingAgent())
main_agent.dispatch_task("Review code", CodeReviewAgent())
main_agent.dispatch_task("Update docs", DocumentationAgent())

# Agents can work in parallel on independent tasks
```

#### Self-Assessed Confidence
```python
class ConfidenceAgent:
    def execute(self, task: Task) -> Result:
        # Before executing, assess confidence
        confidence = self.llm.assess_confidence(task)

        if confidence < 0.7:
            # Ask for clarification
            return self.request_clarification(task)
        else:
            # Proceed autonomously
            return self.execute_autonomously(task)
```

**Takeaways for ORCA-CLI:**
- ✅ Integrated workspace (editor + shell + browser)
- ✅ Strong planning with concrete steps
- ✅ Browser/web search for docs and research
- ✅ Self-assessment and asking for help
- Future: Multi-agent coordination

---

## Recommendations for ORCA-CLI

### 1. Adopt Hybrid Architecture Pattern

**Recommendation:** Combine Plan-and-Execute (for clear tasks) with ReAct (for exploratory tasks)

```python
class ORCAAgent:
    def execute(self, goal: str, mode: str = "auto") -> Result:
        """
        Auto-select pattern based on task type.
        """

        task_type = self.classify_task(goal)

        if task_type == "well_defined":
            # Use Plan-and-Execute
            # - "Add JWT authentication"
            # - "Refactor UserService to use dependency injection"
            return self.plan_and_execute(goal)

        elif task_type == "exploratory":
            # Use ReAct
            # - "Debug why tests are failing"
            # - "Find all unused imports"
            return self.react_loop(goal)

        else:
            # Default: Plan-and-Execute with fallback to ReAct
            try:
                return self.plan_and_execute(goal)
            except PlanningFailure:
                return self.react_loop(goal)
```

### 2. Design Custom Agent-Computer Interface

**Priority: CRITICAL**

**Research shows:** ACI design has more impact than model selection.

**Recommendations:**

#### A. File Editor Tool
```python
@tool
def edit_file(
    path: str,
    operation: Literal["replace_lines", "insert_lines", "delete_lines"],
    start_line: int,
    end_line: Optional[int] = None,
    new_content: Optional[str] = None
) -> EditResult:
    """
    Edit file with surgical precision.

    Examples:
        # Replace lines 10-15
        edit_file("/src/auth.py", "replace_lines", 10, 15, "new code")

        # Insert after line 20
        edit_file("/src/auth.py", "insert_lines", 20, new_content="new code")

        # Delete lines 30-35
        edit_file("/src/auth.py", "delete_lines", 30, 35)

    Returns:
        EditResult with:
        - success status
        - diff
        - syntax validation result
        - backup location
    """
```

**Why:** More efficient than "replace entire file" pattern.

#### B. Compact File Viewer
```python
@tool
def view_file(
    path: str,
    start_line: int = 1,
    num_lines: int = 100
) -> FileView:
    """
    View file with line numbers and context.

    Output format:

    /src/auth.py (150 lines total)
    ─────────────────────────────────
    10 | def authenticate(username: str, password: str) -> User:
    11 |     user = db.query(User).filter_by(username=username).first()
    12 |     if not user:
    13 |         raise AuthenticationError("User not found")
    14 |
    15 |     if not verify_password(password, user.password_hash):
    16 |         raise AuthenticationError("Invalid password")
    17 |
    18 |     return user
    ─────────────────────────────────
    Showing lines 10-18 of 150

    Next: view_file("/src/auth.py", start_line=19)
    """
```

#### C. Search Tool
```python
@tool
def search_code(
    query: str,
    *,
    file_pattern: Optional[str] = None,
    search_type: Literal["semantic", "regex", "symbol"] = "semantic"
) -> SearchResults:
    """
    Search codebase with multiple strategies.

    Examples:
        # Semantic: "authentication functions"
        search_code("authentication functions")

        # Regex: find all TODO comments
        search_code("TODO:.*", search_type="regex")

        # Symbol: find class by name
        search_code("UserService", search_type="symbol")

    Returns compact results:

    Found 3 matches:

    1. /src/auth.py:45
       def authenticate(username: str, password: str)

    2. /src/middleware/auth.py:12
       class AuthMiddleware:

    3. /src/services/user_service.py:67
       def authenticate_user(self, credentials: dict)
    """
```

### 3. Implement Repository Mapping (Like Aider)

**Priority: HIGH**

```python
class RepositoryMapper:
    """Build and maintain AST-based repository map."""

    def build_map(self, repo_path: str) -> RepoMap:
        """
        Build lightweight map of entire repository.

        For each file:
        - Parse with tree-sitter
        - Extract symbols (classes, functions)
        - Extract dependencies (imports)
        - Store metadata (size, language, last modified)

        Result: ~1-2K tokens for large projects
        """

        map = RepoMap()

        for file in find_all_source_files(repo_path):
            tree = parse_with_treesitter(file)

            map.add_file(
                path=file.path,
                symbols=extract_symbols(tree),
                dependencies=extract_imports(tree),
                metadata=FileMetadata(
                    language=detect_language(file),
                    size=get_file_size(file),
                    last_modified=get_mtime(file)
                )
            )

        return map

    def to_context_string(self, map: RepoMap) -> str:
        """
        Convert to compact string for LLM context.

        Always include in context (cheap!)
        """
        return format_compact_map(map)
```

**Benefit:** LLM knows structure of entire codebase without token explosion.

### 4. Implement Self-Correction Loop

**Priority: HIGH**

```python
class SelfCorrectingExecutor:
    """Execute with automatic error recovery."""

    def execute_step(self, step: PlanStep) -> Result:
        """Execute with retries and self-correction."""

        max_retries = 3
        errors = []

        for attempt in range(max_retries):
            result = self.try_execute(step)

            if result.success:
                return result

            # Error detection
            error = self.categorize_error(result.error)
            errors.append(error)

            # Reflection
            reflection = self.llm.reflect(
                f"""
                Task: {step.description}
                Attempt: {attempt + 1}/{max_retries}
                Error: {error}
                Previous attempts: {errors}

                Analysis:
                - What went wrong?
                - Why did this error occur?
                - What should be different?
                - Is this fixable?

                Provide: corrective action
                """
            )

            if not reflection.is_fixable:
                break

            # Generate fix
            fix = self.llm.generate_fix(
                step=step,
                error=error,
                reflection=reflection
            )

            # Apply fix
            step = self.apply_fix(step, fix)

        return Result(
            success=False,
            reason=f"Failed after {max_retries} attempts",
            errors=errors
        )
```

### 5. Just-in-Time Context Loading

**Priority: HIGH**

```python
class ContextManager:
    """Manage context with just-in-time loading."""

    def __init__(self):
        # Always in context (lightweight)
        self.repo_map = load_repository_map()
        self.task_history = []

        # Loaded on demand
        self.loaded_files = {}

    def get_context(self, current_task: str) -> Context:
        """Build context for current task."""

        context = Context()

        # Always include
        context.add("repository_map", self.repo_map.to_string())
        context.add("current_task", current_task)
        context.add("recent_tasks", self.task_history[-5:])

        # Load relevant files (top 5)
        relevant_files = self.find_relevant_files(current_task)
        for file_path in relevant_files[:5]:
            if file_path not in self.loaded_files:
                content = load_file(file_path)
                self.loaded_files[file_path] = content

            context.add_file(file_path, self.loaded_files[file_path])

        return context

    def clear_cache(self):
        """Periodically clear loaded files to prevent memory bloat."""
        self.loaded_files.clear()
```

### 6. Deterministic Core, Non-Deterministic Reasoning

**Priority: HIGH**

```python
# Deterministic: Always works the same
class DeterministicCore:
    """Reliable, auditable operations."""

    def parse_code(self, source: str, language: str) -> AST
    def validate_syntax(self, code: str, language: str) -> bool
    def run_tests(self, test_files: List[str]) -> TestResult
    def generate_diff(self, old: str, new: str) -> Diff
    def commit_changes(self, files: List[str], message: str) -> CommitHash

# Non-deterministic: Uses LLM
class ReasoningLayer:
    """Flexible, intelligent decision making."""

    def plan_task(self, goal: str, context: Context) -> Plan
    def generate_code(self, spec: FunctionSpec) -> str
    def analyze_error(self, error: Error) -> ErrorAnalysis
    def suggest_fix(self, error: Error, context: Context) -> Fix

# Combine in Orchestrator
class ORCAOrchestrator:
    def __init__(self):
        self.deterministic = DeterministicCore()
        self.reasoning = ReasoningLayer()

    def execute(self, task: Task) -> Result:
        # Non-deterministic: Generate plan
        plan = self.reasoning.plan_task(task.goal, task.context)

        for step in plan.steps:
            # Non-deterministic: Generate code
            code = self.reasoning.generate_code(step.spec)

            # Deterministic: Validate
            if not self.deterministic.validate_syntax(code, step.language):
                return Result(success=False, error="Invalid syntax")

            # Deterministic: Test
            test_result = self.deterministic.run_tests(step.test_files)

            if not test_result.passed:
                # Non-deterministic: Analyze
                analysis = self.reasoning.analyze_error(test_result.error)

                # Non-deterministic: Fix
                fix = self.reasoning.suggest_fix(
                    test_result.error,
                    context={"code": code, "test": step.test_files}
                )

        return Result(success=True)
```

### 7. Tool Design Checklist

**For every tool in ORCA-CLI, ensure:**

- [ ] **Clear Purpose**: One tool, one job
- [ ] **Self-Contained**: No external state dependencies
- [ ] **Robust**: Validates inputs, handles errors gracefully
- [ ] **Descriptive**: Clear docstring with examples
- [ ] **Structured Output**: Returns dataclasses, not strings
- [ ] **Actionable Errors**: Error messages suggest fixes
- [ ] **Compact**: Minimal token usage
- [ ] **Feedback Loops**: Provides info for self-correction

### 8. Specific Architecture Decisions

| Component | Recommendation | Priority |
|-----------|---------------|----------|
| **Primary Pattern** | Plan-and-Execute with ReAct fallback | P0 |
| **Repository Mapping** | AST-based with tree-sitter | P0 |
| **Context Strategy** | Just-in-time with hierarchical loading | P0 |
| **File Editing** | Line-based edits, not full-file replacement | P0 |
| **Self-Correction** | 3-retry loop with reflection | P0 |
| **Tool Design** | Follow SWE-agent ACI principles | P0 |
| **Browser Integration** | Add for Phase 2 | P1 |
| **Multi-Agent** | Add for Phase 3 | P2 |
| **Confidence Assessment** | Add for Phase 2 | P1 |

---

## References

### Academic Papers

1. **SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering**
   https://arxiv.org/abs/2405.15793
   NeurIPS 2024 | Princeton & Stanford

2. **AI Agentic Programming: A Survey**
   https://arxiv.org/html/2508.11126v1
   2024 Survey with 152 references

### Blog Posts & Articles

3. **Claude Code: Best practices for agentic coding**
   https://www.anthropic.com/engineering/claude-code-best-practices
   Anthropic Engineering Blog

4. **Effective context engineering for AI agents**
   https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
   Anthropic Engineering Blog

5. **Building Effective AI Agents**
   https://www.anthropic.com/research/building-effective-agents
   Anthropic Research

6. **Agent Architectures: ReAct, Self-Ask, Plan-and-Execute**
   https://apxml.com/courses/langchain-production-llm/chapter-2-sophisticated-agents-tools/agent-architectures
   Comprehensive comparison

7. **Aider: Separating code reasoning and editing**
   https://aider.chat/2024/09/26/architect.html
   Aider blog post on architect/editor pattern

8. **Understanding AI Coding Agents Through Aider's Architecture**
   https://simranchawla.com/understanding-ai-coding-agents-through-aiders-architecture/
   Detailed Aider analysis

9. **Plan-and-Execute Agents**
   https://blog.langchain.com/planning-agents/
   LangChain blog post

10. **Designing agentic loops**
    https://simonwillison.net/2025/Sep/30/designing-agentic-loops/
    Simon Willison's analysis

11. **Function calling using LLMs**
    https://martinfowler.com/articles/function-call-LLM.html
    Martin Fowler's guide

12. **AI Agents Design Patterns: Complete Guide**
    https://pub.towardsai.net/ai-agents-design-patterns-complete-guide-to-agentic-ai-models-in-2025-b0fe49cd02d7
    Towards AI 2025

### Systems & Documentation

13. **Aider - AI Pair Programming**
    https://aider.chat/
    Open source CLI coding assistant

14. **SWE-agent GitHub**
    https://github.com/SWE-agent/SWE-agent
    Research implementation

15. **Devin AI**
    https://devin.ai/
    https://cognition.ai/blog/introducing-devin
    Commercial autonomous coding agent

16. **Coding Agents 101: The Art of Actually Getting Things Done**
    https://devin.ai/agents101
    Devin's agent design guide

### Additional Resources

17. **Berkeley Function Calling Leaderboard**
    Benchmark for function calling capabilities

18. **SWE-bench**
    Standard benchmark for coding agents

19. **Temporal for AI Agents**
    https://temporal.io/blog/of-course-you-can-build-dynamic-ai-agents-with-temporal
    Combining deterministic workflows with AI

---

**Document Status:** Complete - Ready for implementation
**Last Updated:** 2025-11-23
**Next Steps:** Incorporate recommendations into ARCHITECTURE.md and IMPLEMENTATION_PLAN.md
