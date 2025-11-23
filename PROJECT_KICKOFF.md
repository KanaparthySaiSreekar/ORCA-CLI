# ORCA-CLI Project Kickoff Guide

**Date:** 2025-11-22
**Project:** OpenRouter-Compatible Agentic Coding Assistant
**Status:** Initializing

---

## 🎯 Project Vision

Build a **CLI-first coding agent** that autonomously plans, executes, and verifies software engineering tasks using OpenRouter's free models, with deep agentic capabilities including multi-file editing, sandbox execution, RAG retrieval, and comprehensive verification.

**Think:** "A local engineering co-worker that lives in your terminal"

---

## 📋 Quick Navigation

- **[IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md)** - Detailed task breakdown, timeline, dependencies
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - System design, component architecture, data flow
- **[TECH_RESEARCH.md](./TECH_RESEARCH.md)** - Technology evaluation and selection guide
- **[README.md](./README.md)** - Project overview (to be expanded)

---

## 🚀 Getting Started (First Week)

### Day 1: Environment Setup

**All Team Members:**

1. **Clone repository**
   ```bash
   git clone https://github.com/KanaparthySaiSreekar/ORCA-CLI.git
   cd ORCA-CLI
   ```

2. **Read documentation**
   - [ ] Read IMPLEMENTATION_PLAN.md (30 min)
   - [ ] Read ARCHITECTURE.md (45 min)
   - [ ] Skim TECH_RESEARCH.md (20 min)

3. **Setup development environment**
   ```bash
   # Install Python 3.10+
   python --version  # should be 3.10 or higher

   # Install Poetry
   curl -sSL https://install.python-poetry.org | python3 -

   # Install development tools
   poetry install --with dev

   # Activate virtual environment
   poetry shell
   ```

4. **Setup OpenRouter account**
   - Create account at https://openrouter.ai
   - Get API key
   - Test with curl:
     ```bash
     curl https://openrouter.ai/api/v1/chat/completions \
       -H "Authorization: Bearer $OPENROUTER_API_KEY" \
       -H "Content-Type: application/json" \
       -d '{
         "model": "openrouter/meta-llama/llama-3.1-8b-instruct:free",
         "messages": [{"role": "user", "content": "Hello!"}]
       }'
     ```

### Day 2-3: Research Phase (Assigned Tasks)

**Lead Developer:**
- [ ] INFRA-001: Research similar CLI agents (Aider, GPT Engineer, etc.)
- [ ] INFRA-007: Draft architecture decisions document

**Developer 1:**
- [ ] INFRA-002: Research OpenRouter API
- [ ] INFRA-003: Research AST parsing libraries
- [ ] Create prototype: Simple OpenRouter integration

**Developer 2:**
- [ ] INFRA-004: Research sandbox technologies
- [ ] INFRA-005: Research vector databases
- [ ] INFRA-006: Research code embedding models
- [ ] Create prototype: Simple Docker sandbox

### Day 4-5: Project Setup

**All Together:**
- [ ] INFRA-008: Create Python package structure
- [ ] INFRA-009: Setup dev environment (pytest, black, mypy, ruff)
- [ ] INFRA-010: Create project config files
- [ ] INFRA-011: Setup pre-commit hooks

**Deliverable:** Working development environment, initial package structure

---

## 📦 Project Structure (After Week 1)

```
orca-cli/
├── .github/
│   └── workflows/
│       └── ci.yml              # CI pipeline (Phase 3)
├── .orca/                      # Created by 'orca init' (user workspace)
├── docs/                       # Documentation
│   ├── IMPLEMENTATION_PLAN.md  # ✅ Already created
│   ├── ARCHITECTURE.md         # ✅ Already created
│   ├── TECH_RESEARCH.md        # ✅ Already created
│   ├── PROJECT_KICKOFF.md      # ✅ Already created
│   └── API.md                  # Phase 3
├── examples/                   # Example use cases (Phase 3)
├── src/
│   └── orca/
│       ├── __init__.py
│       ├── cli/                # CLI commands
│       ├── agent/              # Agent core (planner, executor)
│       ├── openrouter/         # OpenRouter integration
│       ├── edit/               # Multi-file editor
│       ├── sandbox/            # Execution environment
│       ├── rag/                # Retrieval & RAG
│       ├── test_runner/        # Test integration
│       ├── verify/             # Verification engine
│       ├── tools/              # Tool integrations
│       ├── state/              # State management
│       ├── observability/      # Logging & metrics
│       └── security/           # Safety & security
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── .gitignore
├── .editorconfig
├── .pre-commit-config.yaml
├── pyproject.toml              # Poetry config, dependencies
├── setup.py                    # Installation script
├── README.md
├── LICENSE
└── CHANGELOG.md
```

---

## 🛠️ Development Workflow

### Branching Strategy

```
main (protected)
  ↓
develop (integration branch)
  ↓
feature/TASK-ID-description
  - Example: feature/CLI-001-basic-entry-point
  - Example: feature/AGENT-003-plan-parser
```

### Commit Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples:**
```
feat(cli): add basic CLI entry point with Typer

Implements CLI-001. Creates main.py with Typer framework,
registers placeholder commands for init, config, plan, exec.

feat(openrouter): implement API client with retry logic

Implements OR-001 and OR-005. Adds OpenRouter client with
authentication, request/response handling, and exponential
backoff retry strategy.

test(agent): add unit tests for plan parser

Adds tests for AGENT-003 covering valid plans, malformed
JSON, and edge cases.
```

### Code Review Process

1. Create feature branch
2. Implement feature with tests
3. Ensure all checks pass:
   ```bash
   # Run tests
   pytest

   # Type checking
   mypy src/

   # Linting
   ruff check src/

   # Formatting
   black --check src/
   ```
4. Create pull request to `develop`
5. Request review from 1+ team members
6. Address feedback
7. Merge after approval

---

## 🧪 Testing Strategy

### Test Pyramid

```
    ╱◥◣   E2E Tests (10%)
   ╱    ◥◣  Integration Tests (30%)
  ╱________◥◣ Unit Tests (60%)
```

### Testing Guidelines

**Unit Tests:**
- Test individual functions/classes in isolation
- Mock external dependencies
- Fast execution (<1s total)
- 80%+ coverage target

**Integration Tests:**
- Test component interactions
- Use real dependencies when feasible
- Moderate execution time (<10s total)
- Cover critical paths

**E2E Tests:**
- Test full workflows (plan → exec → verify)
- Use Docker for isolation
- Slow execution (<2min total)
- Cover happy paths and key error scenarios

### Test Organization

```
tests/
├── unit/
│   ├── test_cli.py
│   ├── test_planner.py
│   ├── test_executor.py
│   └── ...
├── integration/
│   ├── test_openrouter_integration.py
│   ├── test_sandbox_execution.py
│   └── ...
├── e2e/
│   ├── test_full_workflow.py
│   └── ...
└── fixtures/
    ├── sample_projects/
    ├── mock_responses/
    └── ...
```

---

## 📊 Progress Tracking

### Task Status

Track progress in IMPLEMENTATION_PLAN.md:
- ❌ `PENDING`: Not started
- 🔄 `IN_PROGRESS`: Currently working
- ✅ `COMPLETED`: Done and tested
- ⏸️ `BLOCKED`: Waiting on dependency
- 🚫 `CANCELLED`: No longer needed

### Weekly Sync

**When:** Every Monday, 10:00 AM
**Duration:** 30 minutes
**Agenda:**
1. Last week accomplishments (5 min)
2. This week goals (5 min)
3. Blockers and dependencies (10 min)
4. Architecture/design discussions (10 min)

### Daily Standups (Optional)

**When:** Every day, 9:00 AM
**Duration:** 10 minutes
**Format:**
- What did you do yesterday?
- What will you do today?
- Any blockers?

---

## 🎓 Learning Resources

### Python Development

- **Typer Documentation:** https://typer.tiangolo.com/
- **Poetry Guide:** https://python-poetry.org/docs/
- **pytest Best Practices:** https://docs.pytest.org/en/stable/goodpractices.html
- **Type Hints:** https://docs.python.org/3/library/typing.html

### AI/LLM Integration

- **OpenRouter Docs:** https://openrouter.ai/docs
- **LangChain (for inspiration):** https://python.langchain.com/docs/get_started/introduction
- **Prompt Engineering Guide:** https://www.promptingguide.ai/

### Code Parsing

- **tree-sitter:** https://tree-sitter.github.io/tree-sitter/
- **Python AST:** https://docs.python.org/3/library/ast.html

### Sandboxing

- **Docker SDK for Python:** https://docker-py.readthedocs.io/
- **firejail:** https://firejail.wordpress.com/

### Vector Search

- **ChromaDB:** https://docs.trychroma.com/
- **Sentence Transformers:** https://www.sbert.net/

---

## 🎯 Success Metrics (Reminders)

### Phase 1 Exit Criteria
- [ ] CLI installable via pip
- [ ] Core commands functional (init, config, plan, exec, run, test)
- [ ] OpenRouter integration with 3+ models
- [ ] Multi-step task planning (5+ steps)
- [ ] Multi-file editing (3+ files atomically)
- [ ] Sandbox execution for Python and JavaScript
- [ ] 70%+ test coverage
- [ ] Basic documentation

### Quality Standards
- **Code Coverage:** 80%+ (Phase 2), 85%+ (Phase 3)
- **Type Coverage:** 100% (strict mypy)
- **Documentation:** Every public API documented
- **Performance:** Plan generation <10s (p95)

---

## 🔐 Security & Best Practices

### Security Checklist

- [ ] Never commit API keys or secrets
- [ ] Use environment variables for sensitive config
- [ ] Validate all user inputs
- [ ] Sanitize LLM outputs before execution
- [ ] Run all code in sandboxes
- [ ] Implement approval gates for destructive operations

### Code Quality Checklist

- [ ] Type hints on all functions
- [ ] Docstrings for public APIs
- [ ] Error handling with specific exceptions
- [ ] Logging at appropriate levels
- [ ] Tests for new functionality
- [ ] No TODO/FIXME in main branch

---

## ❓ FAQ

### Q: What if I'm blocked on a dependency?

**A:**
1. Check if you can work around it temporarily (mock, stub)
2. Update task status to BLOCKED in IMPLEMENTATION_PLAN.md
3. Notify team in Slack/email
4. Pick up another non-blocked task
5. Escalate if blocking for >1 day

### Q: How do we handle disagreements on technical decisions?

**A:**
1. Document options in TECH_RESEARCH.md
2. List pros/cons objectively
3. Create simple prototypes if needed
4. Team discussion (async or sync)
5. Lead makes final call if no consensus
6. Document decision and rationale

### Q: Can I refactor existing code while working on a feature?

**A:**
- **Small refactors:** Yes, include in same PR
- **Large refactors:** Create separate task/PR
- **Rule of thumb:** If refactor doubles PR size, split it

### Q: How often should I push to remote?

**A:**
- At least daily (end of day)
- After completing logical chunks
- Before switching to another task

### Q: What if tests fail in CI but pass locally?

**A:**
1. Check for environment differences (OS, Python version)
2. Check for race conditions (parallel execution)
3. Check for missing test dependencies
4. Reproduce locally using Docker:
   ```bash
   docker run -it python:3.10 bash
   # Clone repo and run tests
   ```

---

## 🎉 First Milestone Celebration

When we complete Phase 1 (Core MVP):
- Team demo of working prototype
- Retrospective meeting (what went well, what to improve)
- Celebrate! 🎊

---

## 📞 Contact & Communication

### Team Contacts

| Role | Name | Email | Timezone |
|---|---|---|---|
| Lead | TBD | TBD | TBD |
| Developer 1 | TBD | TBD | TBD |
| Developer 2 | TBD | TBD | TBD |

### Communication Channels

- **Slack/Discord:** For daily communication (create #orca-cli channel)
- **GitHub Issues:** For bug reports and feature requests
- **GitHub Discussions:** For design discussions
- **Email:** For formal updates and announcements

---

## 🚦 Next Immediate Actions

### This Week (Week 1)

**Monday:**
- [ ] All team members read documentation
- [ ] Setup development environments
- [ ] Create OpenRouter accounts

**Tuesday-Wednesday:**
- [ ] Complete research tasks (INFRA-001 to INFRA-006)
- [ ] Create research prototypes
- [ ] Document findings

**Thursday-Friday:**
- [ ] Team review of research findings
- [ ] Make technology decisions
- [ ] Create initial package structure
- [ ] Setup CI pipeline basics

**End of Week:**
- [ ] Retrospective: What worked? What didn't?
- [ ] Plan Week 2 tasks
- [ ] Update IMPLEMENTATION_PLAN.md with progress

---

## 📝 Document Updates

This is a living document. Update as project evolves:
- Add/remove team members
- Adjust timelines based on progress
- Update workflows based on learnings
- Add new sections as needed

**Last Updated:** 2025-11-22
**Next Review:** 2025-11-29

---

**Let's build something amazing! 🚀**
