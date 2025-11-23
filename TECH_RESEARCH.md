# ORCA-CLI Technology Research Guide

**Version:** 1.0
**Date:** 2025-11-22
**Purpose:** Guide Phase 0 research and technology selection

---

## Research Tasks

This document outlines the key research areas for Phase 0. Each section includes evaluation criteria, leading candidates, and decision frameworks.

---

## 1. Similar CLI Coding Agents

### Research Objective
Understand existing solutions, identify best practices, and avoid common pitfalls.

### Agents to Analyze

| Tool | Focus | Key Features to Study |
|---|---|---|
| **Aider** | Pair programming | - Git integration<br>- Multi-file editing<br>- Model selection<br>- Cost tracking |
| **GitHub Copilot CLI** | Command suggestions | - Natural language to command<br>- Context awareness<br>- Safety features |
| **Cursor** | IDE integration | - Code understanding<br>- Edit operations<br>- Chat interface |
| **Mentat** | Multi-file editing | - AST awareness<br>- Diff generation<br>- Review workflow |
| **GPT Engineer** | Project generation | - Planning phase<br>- Iteration loop<br>- Self-improvement |
| **Smol Developer** | Autonomous coding | - Task decomposition<br>- Self-debugging<br>- File management |

### Evaluation Questions

1. **Planning Approach:**
   - How do they decompose complex tasks?
   - Do they use explicit planning or direct execution?
   - How is context managed across steps?

2. **Code Editing:**
   - Search-and-replace vs AST-aware?
   - Multi-file edit capabilities?
   - Diff visualization?

3. **Safety & Verification:**
   - Approval gates?
   - Rollback mechanisms?
   - Testing integration?

4. **User Experience:**
   - Command structure?
   - Output format?
   - Error handling?

### Decision Criteria

What to adopt:
- ✓ Proven UX patterns (command names, flags)
- ✓ Safety mechanisms (approval, rollback)
- ✓ Git integration patterns

What to differentiate:
- OpenRouter compatibility (vs locked to OpenAI)
- Explicit planning phase (vs implicit)
- Sandbox execution (vs trust local environment)
- RAG-based retrieval (vs full context)

---

## 2. OpenRouter API Integration

### Research Objective
Design robust, efficient OpenRouter integration with multi-model support.

### Key Research Areas

#### 2.1 API Capabilities

**Questions:**
- What models are available in free tier?
- What are rate limits?
- How does streaming work?
- How is cost calculated?
- What's the error handling strategy?

**Documentation:**
- Official Docs: https://openrouter.ai/docs
- API Reference: https://openrouter.ai/docs/api-reference
- Model List: https://openrouter.ai/models

#### 2.2 Model Selection

**Free Models to Test (as of 2025):**
```
openrouter/meta-llama/llama-3.1-8b-instruct:free
openrouter/meta-llama/llama-3.1-70b-instruct:free
openrouter/google/gemma-2-9b-it:free
openrouter/mistralai/mistral-7b-instruct:free
openrouter/openchat/openchat-7b:free
```

**Evaluation Metrics:**
- Code generation quality
- Instruction following
- JSON output reliability
- Latency
- Context window size

**Test Prompts:**
1. Simple function generation (Python)
2. Multi-step plan creation
3. Bug fix from error message
4. Code refactoring
5. Test generation

#### 2.3 Routing Strategy

**Task-to-Model Mapping:**

| Task Type | Requirements | Candidate Models |
|---|---|---|
| Planning | Reasoning, structure | Llama 3.1 70B, GPT-4 (if available) |
| Code Editing | Accuracy, speed | Llama 3.1 8B, Codestral |
| Code Review | Critical thinking | Claude, GPT-4, Llama 3.1 70B |
| Quick Fixes | Speed, cost | Llama 3.1 8B, Gemma 2 9B |
| Embeddings | Specialized | CodeBERT, all-MiniLM-L6-v2 |

**Fallback Strategy:**
```
Primary Model (free) → Secondary Model (free) → Paid Model (if configured) → Error
```

#### 2.4 Implementation Considerations

**Retry Logic:**
```python
@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=retry_if_exception_type(RateLimitError)
)
async def call_openrouter(prompt, model):
    # API call
```

**Rate Limiting:**
- Track requests per minute
- Implement token bucket algorithm
- Queue requests during high load

**Cost Tracking:**
```python
@dataclass
class UsageMetrics:
    prompt_tokens: int
    completion_tokens: int
    model: str
    cost_usd: float
    timestamp: datetime
```

---

## 3. AST Parsing & Code Editing

### Research Objective
Choose parsing technology that supports multi-language, accurate editing.

### Option A: tree-sitter (Recommended)

**Pros:**
- ✓ Supports 40+ languages
- ✓ Fast, incremental parsing
- ✓ Error-tolerant (works with invalid code)
- ✓ Query language for code search
- ✓ Active community

**Cons:**
- ✗ Python bindings can be tricky to install
- ✗ Grammar files need to be compiled
- ✗ Learning curve for query syntax

**Languages Supported:**
- Python, JavaScript, TypeScript, Go, Rust, Java, C, C++, Ruby, PHP, etc.

**Example Usage:**
```python
from tree_sitter import Language, Parser

# Load language grammar
PY_LANGUAGE = Language('build/my-languages.so', 'python')
parser = Parser()
parser.set_language(PY_LANGUAGE)

# Parse code
tree = parser.parse(bytes(source_code, "utf8"))
root_node = tree.root_node

# Query for functions
query = PY_LANGUAGE.query("""
(function_definition
  name: (identifier) @func.name
  parameters: (parameters) @func.params
  body: (block) @func.body)
""")

matches = query.matches(root_node)
```

### Option B: Language-Specific Parsers

**Python:** `ast` module (built-in)
```python
import ast

tree = ast.parse(source_code)
# Manipulate with ast.NodeTransformer
```

**JavaScript:** `esprima` or `@babel/parser`
```javascript
const esprima = require('esprima');
const tree = esprima.parseScript(code);
```

**Pros:**
- ✓ Official, well-maintained
- ✓ Native to language ecosystem
- ✓ Excellent documentation

**Cons:**
- ✗ Need separate library per language
- ✗ Different APIs for each language
- ✗ More complex integration

### Decision Framework

**Choose tree-sitter if:**
- Need to support 3+ languages from day one
- Want unified API across languages
- Require error-tolerant parsing
- Plan to implement code search queries

**Choose language-specific if:**
- Starting with single language (Python-only MVP)
- Need deep language-specific features
- Want simpler dependencies

### Recommendation
**tree-sitter** for production, with fallback to `ast` module for Python-only development mode.

---

## 4. Sandbox Execution

### Research Objective
Select sandbox technology that balances security, performance, and ease of use.

### Option A: Docker (Recommended for MVP)

**Pros:**
- ✓ Industry standard, well-documented
- ✓ Cross-platform (Linux, macOS, Windows)
- ✓ Pre-built images for all languages
- ✓ Strong isolation
- ✓ Resource limits easy to configure
- ✓ Large ecosystem of tools

**Cons:**
- ✗ Higher overhead (100-500MB per container)
- ✗ Slower startup (2-5 seconds)
- ✗ Requires Docker daemon running

**Example Usage:**
```python
import docker

client = docker.from_env()

container = client.containers.run(
    image="python:3.10-slim",
    command="python /code/script.py",
    volumes={'/path/to/code': {'bind': '/code', 'mode': 'ro'}},
    mem_limit="512m",
    cpu_period=100000,
    cpu_quota=50000,  # 50% of one CPU
    network_disabled=True,
    detach=False,
    stdout=True,
    stderr=True,
    remove=True
)

print(container.decode('utf-8'))
```

### Option B: firejail

**Pros:**
- ✓ Lightweight (minimal overhead)
- ✓ Fast startup (<100ms)
- ✓ Simple CLI interface
- ✓ Good for local development

**Cons:**
- ✗ Linux-only
- ✗ Less isolation than containers
- ✗ May require SUID binary

**Example Usage:**
```bash
firejail \
  --noprofile \
  --private=/tmp/sandbox \
  --net=none \
  --rlimit-cpu=60 \
  --rlimit-as=512M \
  python script.py
```

### Option C: bubblewrap

**Pros:**
- ✓ Lightweight, fast
- ✓ No daemon required
- ✓ Used by Flatpak

**Cons:**
- ✗ Linux-only
- ✗ More complex setup
- ✗ Less documentation

### Option D: gVisor

**Pros:**
- ✓ Strong security (kernel-level isolation)
- ✓ Compatible with Docker
- ✓ Google-backed

**Cons:**
- ✗ Performance overhead
- ✗ Complex setup
- ✗ Overkill for most use cases

### Evaluation Tests

**Security:**
- Attempt file system escape
- Attempt network access when disabled
- Attempt fork bomb
- Attempt resource exhaustion

**Performance:**
- Startup time
- Execution overhead
- Memory footprint
- Cleanup time

**Compatibility:**
- Python, Node.js, Go, Java, C/C++
- Dependency installation
- Multi-file projects

### Decision Matrix

| Criteria | Docker | firejail | bubblewrap | gVisor |
|---|---|---|---|---|
| Security | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Performance | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Ease of Use | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Cross-platform | ⭐⭐⭐⭐⭐ | ⭐ | ⭐ | ⭐⭐⭐ |
| Ecosystem | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |

### Recommendation

**Phase 1 (MVP):** Docker
- Prioritize cross-platform and ease of use
- Accept performance trade-off
- Can optimize later

**Phase 2 (Optimization):** Add firejail for Linux
- Detect OS, use firejail on Linux for better performance
- Fall back to Docker on macOS/Windows

---

## 5. Vector Database for RAG

### Research Objective
Choose vector DB that's easy to integrate, performant for code search.

### Option A: ChromaDB (Recommended)

**Pros:**
- ✓ Python-native, easy to install
- ✓ Embedded mode (no separate server)
- ✓ Simple API
- ✓ Good documentation
- ✓ Active development

**Cons:**
- ✗ Less mature than FAISS
- ✗ Smaller community

**Example Usage:**
```python
import chromadb

client = chromadb.Client()
collection = client.create_collection("code_embeddings")

# Add documents
collection.add(
    documents=["def hello(): print('hello')", "def goodbye(): print('bye')"],
    metadatas=[{"file": "utils.py", "type": "function"}, {"file": "utils.py", "type": "function"}],
    ids=["func1", "func2"]
)

# Search
results = collection.query(
    query_texts=["greeting function"],
    n_results=2
)
```

### Option B: FAISS (Facebook AI Similarity Search)

**Pros:**
- ✓ Battle-tested, used at scale
- ✓ Very fast
- ✓ Advanced indexing options
- ✓ CPU and GPU support

**Cons:**
- ✗ C++ library with Python bindings (harder to install)
- ✗ Lower-level API (more boilerplate)
- ✗ No built-in metadata storage

**Example Usage:**
```python
import faiss
import numpy as np

# Create index
dimension = 384  # embedding size
index = faiss.IndexFlatL2(dimension)

# Add vectors
embeddings = np.array([...])  # shape: (n, 384)
index.add(embeddings)

# Search
query_vector = np.array([[...]])  # shape: (1, 384)
distances, indices = index.search(query_vector, k=5)
```

### Option C: Qdrant

**Pros:**
- ✓ Built for production
- ✓ Excellent filtering capabilities
- ✓ Good performance
- ✓ REST API and Python client

**Cons:**
- ✗ Requires separate server (or Docker)
- ✗ More complex setup
- ✗ Overkill for small projects

### Option D: Weaviate

**Pros:**
- ✓ GraphQL API
- ✓ Hybrid search (vector + keyword)
- ✓ Built-in vectorization

**Cons:**
- ✗ Requires separate server
- ✗ Complex setup
- ✗ Heavyweight

### Decision Framework

**For ORCA-CLI:**
- Embedded mode preferred (no separate server)
- Simple installation (pip install)
- Metadata storage (file path, language, etc.)
- Good enough performance for typical repos (<100K files)

### Recommendation
**ChromaDB** for Phase 1 & 2
- Easy to get started
- Sufficient performance
- Can migrate to FAISS/Qdrant later if needed

---

## 6. Code Embeddings

### Research Objective
Select embedding model that understands code semantics.

### Option A: CodeBERT (Microsoft)

**Pros:**
- ✓ Trained specifically on code
- ✓ Supports 6 languages (Python, Java, JS, PHP, Ruby, Go)
- ✓ Good performance on code search tasks
- ✓ Available on Hugging Face

**Cons:**
- ✗ Larger model (125M parameters)
- ✗ Slower than general-purpose models

**Usage:**
```python
from transformers import AutoTokenizer, AutoModel

tokenizer = AutoTokenizer.from_pretrained("microsoft/codebert-base")
model = AutoModel.from_pretrained("microsoft/codebert-base")

code = "def hello(): print('hello world')"
inputs = tokenizer(code, return_tensors="pt")
outputs = model(**inputs)
embedding = outputs.last_hidden_state.mean(dim=1)  # shape: (1, 768)
```

### Option B: GraphCodeBERT (Microsoft)

**Pros:**
- ✓ Uses data flow graphs (more semantic understanding)
- ✓ Better performance than CodeBERT
- ✓ Same language support

**Cons:**
- ✗ More complex preprocessing
- ✗ Requires data flow analysis

### Option C: all-MiniLM-L6-v2 (General Purpose)

**Pros:**
- ✓ Very fast (22M parameters)
- ✓ Small model size (80MB)
- ✓ Good general-purpose embeddings
- ✓ Easy to use

**Cons:**
- ✗ Not trained on code specifically
- ✗ May miss code-specific patterns

**Usage:**
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
embedding = model.encode("def hello(): print('hello')")  # shape: (384,)
```

### Option D: StarCoder Embeddings

**Pros:**
- ✓ State-of-the-art code understanding
- ✓ Supports 80+ languages
- ✓ Trained on The Stack (3TB of code)

**Cons:**
- ✗ Larger model
- ✗ May require GPU for reasonable speed

### Evaluation Test

**Code Search Quality:**
```python
corpus = [
    "def authenticate(username, password): ...",
    "def login(user_id): ...",
    "def fetch_user_profile(user_id): ...",
    "def calculate_tax(amount): ...",
]

query = "user authentication"

# Expected: authenticate, login ranked highest
```

**Cross-Language:**
```python
corpus = [
    "def hello(): print('hello')",  # Python
    "function hello() { console.log('hello'); }",  # JS
    "func hello() { fmt.Println(\"hello\") }",  # Go
]

query = "function that prints greeting"

# Expected: All three ranked similarly
```

### Recommendation

**Phase 1:** all-MiniLM-L6-v2
- Fast, lightweight
- Good enough for MVP
- Easy to integrate

**Phase 2:** CodeBERT
- Better code understanding
- Worth the performance trade-off
- Can run in background for indexing

---

## 7. Configuration Format

### Research Objective
Choose config format that's human-friendly and validates well.

### Option A: TOML

**Pros:**
- ✓ Human-readable
- ✓ Python standard (used by Poetry, Black)
- ✓ Supports comments
- ✓ Strict types

**Cons:**
- ✗ Less flexible than YAML

**Example:**
```toml
[openrouter]
api_key = "sk-or-..."
default_model = "openrouter/anthropic/claude-3.5-sonnet"

[sandbox]
type = "docker"
cpu_limit = 2
memory_limit = "2GB"

[planning]
auto_approve = false
max_steps = 20
```

### Option B: YAML

**Pros:**
- ✓ Very flexible
- ✓ Widely used
- ✓ Supports comments
- ✓ Good for complex configs

**Cons:**
- ✗ Whitespace-sensitive (error-prone)
- ✗ Multiple ways to do same thing

### Option C: JSON

**Pros:**
- ✓ Simple, ubiquitous
- ✓ Easy to parse
- ✓ Good schema validation

**Cons:**
- ✗ No comments
- ✗ Less human-friendly

### Recommendation
**TOML** for user-facing config (`.orca/config.toml`)
**JSON** for internal state (`.orca/plan.json`, `.orca/workspace.json`)

---

## Research Timeline

| Week | Research Area | Deliverable |
|---|---|---|
| Week 1, Day 1-2 | Similar CLI agents | Competitive analysis doc |
| Week 1, Day 3 | OpenRouter API | API integration prototype |
| Week 1, Day 4-5 | AST parsing | tree-sitter vs ast comparison |
| Week 2, Day 1-2 | Sandbox execution | Docker vs firejail benchmark |
| Week 2, Day 3 | Vector DB | ChromaDB vs FAISS test |
| Week 2, Day 4 | Code embeddings | Model quality comparison |
| Week 2, Day 5 | Architecture review | Final technology decisions |

---

## Decision Log Template

For each technology decision, fill out:

```markdown
## Decision: [Technology Choice]

**Date:** YYYY-MM-DD
**Decided By:** [Names]
**Status:** Proposed | Accepted | Rejected

### Context
What problem are we solving?

### Options Considered
1. Option A
2. Option B
3. Option C

### Evaluation
| Criteria | Option A | Option B | Option C |
|---|---|---|---|
| Performance | ... | ... | ... |
| Ease of Use | ... | ... | ... |
| Maintainability | ... | ... | ... |

### Decision
We chose **Option X** because...

### Consequences
- Positive: ...
- Negative: ...
- Risks: ...

### Validation
How will we verify this was the right choice?
```

---

**Next Steps:**
1. Assign research tasks to team members
2. Setup evaluation repos for testing
3. Run benchmarks and document results
4. Review findings as team
5. Make final decisions
6. Update ARCHITECTURE.md with decisions
