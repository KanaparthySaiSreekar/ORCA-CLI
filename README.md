# ORCA-CLI

**OpenRouter-Compatible Agentic Coding Assistant**

A CLI-native coding agent with autonomous planning, multi-file editing, sandbox execution, and RAG-powered retrieval. Built to work with any OpenRouter model, including free tiers.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Status](https://img.shields.io/badge/status-planning-yellow.svg)]()

---

## 🎯 Vision

ORCA-CLI is your **local engineering co-worker** that lives in the terminal. It autonomously plans multi-step tasks, edits code across multiple files, runs tests in isolated sandboxes, and verifies correctness - all while using cost-effective or free LLM models through OpenRouter.

**Unlike other coding assistants:**
- ✅ **Explicit Planning:** Generates reviewable multi-step plans before execution
- ✅ **OpenRouter Native:** Works with any model, including free tiers
- ✅ **Sandbox Execution:** Runs code safely in isolated environments
- ✅ **RAG-Powered:** Semantic search across your entire codebase
- ✅ **Autonomous Iteration:** Automatically fixes failures and retries
- ✅ **Multi-File AST Editing:** Syntax-aware code modifications

---

## 🚀 Planned Features

### Phase 1: Core MVP (Weeks 1-8)
- [x] Project planning and architecture
- [ ] Basic CLI (`init`, `config`, `plan`, `exec`, `run`, `test`)
- [ ] OpenRouter integration with multi-model support
- [ ] Autonomous task planning engine
- [ ] Multi-file AST-aware code editing
- [ ] Docker-based sandbox execution
- [ ] Python & JavaScript support

### Phase 2: Advanced Features (Weeks 9-16)
- [ ] RAG indexing and semantic code search
- [ ] Test runner integration (pytest, Jest, go test, JUnit)
- [ ] Verification engine (syntax, types, tests)
- [ ] Static analysis tools (ESLint, Pylint, etc.)
- [ ] State persistence and auto-resume

### Phase 3: Production Ready (Weeks 17-24)
- [ ] Comprehensive observability and monitoring
- [ ] Security hardening (command blocking, file protection)
- [ ] CI/CD integration templates
- [ ] Full documentation and examples
- [ ] PyPI package and Homebrew formula

### Phase 4: Future
- [ ] Plugin system
- [ ] Community marketplace
- [ ] Enterprise features

---

## 📖 Documentation

- **[AGENT_ARCHITECTURE_RESEARCH.md](./AGENT_ARCHITECTURE_RESEARCH.md)** - 🔥 **NEW:** Comprehensive research on coding agent patterns, tool design, and best practices
- **[IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md)** - Detailed roadmap with 180+ tasks across 20 work streams (Updated 2025-11-23)
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - System design, components, and data flow
- **[TECH_RESEARCH.md](./TECH_RESEARCH.md)** - Technology evaluation and selection criteria
- **[PROJECT_KICKOFF.md](./PROJECT_KICKOFF.md)** - Team onboarding and development workflow
- **[AGENT_IMPROVEMENTS.md](./AGENT_IMPROVEMENTS.md)** - Advanced capabilities and enhancement proposals

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│              CLI Interface (Typer)                  │
├─────────────────────────────────────────────────────┤
│  Agent Core: Planner → Executor → Iteration Loop   │
├──────────────┬──────────────┬──────────────────────┤
│  Multi-File  │   Sandbox    │   RAG/Retrieval      │
│  Editor      │   Execution  │   (Vector Search)    │
├──────────────┼──────────────┼──────────────────────┤
│  OpenRouter  │   State      │   Observability      │
│  Integration │   Management │   & Security         │
└──────────────────────────────────────────────────────┘
```

---

## 🛠️ Technology Stack

| Component | Technology | Status |
|---|---|---|
| **Language** | Python 3.10+ | ✅ Selected |
| **CLI Framework** | Typer | ✅ Selected |
| **LLM Gateway** | OpenRouter | ✅ Selected |
| **Package Manager** | Poetry | ✅ Selected |
| **AST Parsing** | tree-sitter | 🔍 Evaluating |
| **Sandbox** | Docker | 🔍 Evaluating |
| **Vector DB** | ChromaDB | 🔍 Evaluating |
| **Embeddings** | CodeBERT | 🔍 Evaluating |

---

## 💡 Example Usage (Planned)

```bash
# Initialize workspace
$ orca init

# Configure OpenRouter API key
$ orca config set-model openrouter/anthropic/claude-3.5-sonnet
$ orca config set-api-key sk-or-...

# Plan a complex task
$ orca plan "Add JWT authentication middleware to FastAPI app"

📋 Generated Plan (plan-abc123):
  Step 1: Create JWT utility functions (encode, decode, verify)
  Step 2: Add authentication middleware to FastAPI
  Step 3: Update protected routes to use middleware
  Step 4: Write unit tests for JWT utilities
  Step 5: Write integration tests for protected endpoints

🔍 Files to modify: 5 files
⏱️  Estimated time: 8 minutes

# Execute the plan
$ orca exec plan-abc123 --auto

✅ Step 1: Created src/auth/jwt_utils.py
✅ Step 2: Modified src/middleware/auth.py
✅ Step 3: Updated 3 route files
✅ Step 4: Created tests/test_jwt_utils.py (12 tests)
⚠️  Step 5: Integration tests failed (2/5 passed)

🔄 Auto-fixing failures...
   Issue: Missing import in test_auth.py
   Fix: Added 'from src.middleware import auth'
   Re-running tests...

✅ All tests passing (17/17)
✅ Plan completed successfully!

# Run tests on demand
$ orca test --changed-only

# Execute code in sandbox
$ orca run python script.py

# Interactive chat mode
$ orca chat
```

---

## 🎯 Design Principles

1. **Safety First** - All code execution in sandboxes, validation at every layer
2. **Transparency** - Explicit plans, detailed logs, human approval gates
3. **Deterministic Core** - LLMs enhance, but don't replace, deterministic operations
4. **Extensibility** - Plugin-friendly, easy to add languages/tools
5. **Cost-Effective** - Designed for free/low-cost models via OpenRouter

---

## 🧑‍💻 Development

### Prerequisites

- Python 3.10+
- Poetry
- Docker (for sandbox execution)
- OpenRouter API key

### Setup

```bash
# Clone repository
git clone https://github.com/KanaparthySaiSreekar/ORCA-CLI.git
cd ORCA-CLI

# Install dependencies
poetry install

# Activate virtual environment
poetry shell

# Run tests (when available)
pytest

# Type checking
mypy src/

# Linting
ruff check src/

# Formatting
black src/
```

---

## 📊 Project Status

**Current Phase:** Phase 0 - Research & Foundation

| Work Stream | Progress | Status |
|---|---|---|
| Project Setup | 0/11 | 🟡 Planning |
| Core CLI | 0/7 | ⚪ Not Started |
| OpenRouter Integration | 0/7 | ⚪ Not Started |
| Agent Core | 0/11 | ⚪ Not Started |
| Multi-File Editing | 0/7 | ⚪ Not Started |
| Sandbox Execution | 0/8 | ⚪ Not Started |
| RAG System | 0/9 | ⚪ Not Started |
| Test Integration | 0/8 | ⚪ Not Started |
| Verification | 0/6 | ⚪ Not Started |
| Tools | 0/7 | ⚪ Not Started |
| State Management | 0/5 | ⚪ Not Started |
| Observability | 0/6 | ⚪ Not Started |
| Security | 0/5 | ⚪ Not Started |

**Next Milestone:** Phase 1 MVP (Week 8)

---

## 🤝 Contributing

This project is in early planning stages. Contribution guidelines will be published in [CONTRIBUTING.md](./CONTRIBUTING.md) once the core structure is established.

**Current Focus:** Phase 0 research and technology selection
- See [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md) for detailed tasks
- See [TECH_RESEARCH.md](./TECH_RESEARCH.md) for evaluation criteria

---

## 📝 License

[MIT License](LICENSE) - See LICENSE file for details

---

## 🙏 Acknowledgments

Inspired by:
- [Aider](https://github.com/paul-gauthier/aider) - AI pair programming
- [GPT Engineer](https://github.com/gpt-engineer-org/gpt-engineer) - AI software engineer
- [Mentat](https://github.com/AbanteAI/mentat) - Coding assistant for command line

Built with:
- [OpenRouter](https://openrouter.ai) - Unified LLM API
- [Typer](https://typer.tiangolo.com/) - CLI framework
- [tree-sitter](https://tree-sitter.github.io/) - Code parsing (planned)

---

## 📮 Contact

**Project Lead:** TBD
**Repository:** [KanaparthySaiSreekar/ORCA-CLI](https://github.com/KanaparthySaiSreekar/ORCA-CLI)
**Issues:** [GitHub Issues](https://github.com/KanaparthySaiSreekar/ORCA-CLI/issues)

---

**Status:** 🏗️ Under active development | Last Updated: 2025-11-22
