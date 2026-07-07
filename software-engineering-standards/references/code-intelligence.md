# Code Intelligence & Memory Architecture

## Four-Layer Code Engine

To navigate and understand complex codebases efficiently, employ a four-layer intelligence model:

| Layer | Tool Type | Purpose | When to Use |
|-------|-----------|---------|-------------|
| 1. **Code Map** | AST Index | Structural overview | Find where functions, classes, or types are defined and imported. |
| 2. **Dependency Graph** | Graph Analysis | Understand relationships | See the impact of a change or find all files related to a specific module. |
| 3. **Knowledge RAG** | Vector Search | Unstructured knowledge | Find design rationale, requirements, ADRs, and past decisions. |
| 4. **Precision Search** | Regex / `grep` | Exact string matching | Find specific error messages, variable names, or hardcoded strings. |

### Debugging Workflow

When encountering an error, follow this sequence rather than guessing:
1. **Precision Search**: Find all occurrences of the exact error message string.
2. **Code Map**: Read the structural index of the file containing the error (imports, exports, functions).
3. **Dependency Graph**: Trace dependencies to identify the minimal set of files involved in the execution path.
4. **Knowledge RAG**: Search for related ADRs, requirements, or previously fixed bugs that match the error context.

### AST Symbol Taxonomy

When building code intelligence tools, extract and index these universal symbols:
`function`, `class`, `interface`, `type`, `enum`, `variable`, `component`, `hook`, `constant`.

## Memory Architecture

Systems and agents must maintain context across sessions using a tiered memory architecture:

| Layer | Scope | Description |
|-------|-------|-------------|
| **Session** | Current run | Short-term memory for the current task context. Cleared after task completion. |
| **Project** | Per-project | Persistent decisions, rules, and state specific to the project (e.g., `HANDOFF.md`, ADRs). |
| **Global** | All projects | Proven rules and patterns promoted from individual projects, applied to all new projects. |

### Context Management

- **Handoff Documents**: Maintain a living `HANDOFF.md` file that captures the current state, recent increments, and next steps. This is the primary mechanism for context transfer between sessions.
- **Context Compression**: When context windows fill up, summarize older interactions into dense state representations rather than truncating them.

## Self-Improvement & Evolution Loop

Systems should continuously improve their own rules and processes using the Evolution Loop:

**OBSERVE → ANALYZE → HYPOTHESIZE → SIMULATE → PROPOSE → APPROVE → APPLY → MEASURE → LEARN → REPEAT**

### Prevention and Auto-Fix Rules

1. **Failure Analysis**: When a task fails, generate a Prevention Rule stored in the project's memory.
2. **Auto-Fix Strategies**: For recurring issues, implement Auto-Fix Rules that trigger automated remediation strategies: `apply_known_fix`, `regenerate`, or `rollback`.
3. **Promotion**: When a Prevention Rule successfully prevents errors across multiple tasks, promote it from Project memory to Global memory.

## Skills Ecosystem

Reusable knowledge and capabilities should be packaged as "Skills".

### Three Roles

| Role | Responsibility |
|------|---------------|
| **Consumer** | Discover and install skills based on the project's technology stack and requirements. |
| **Producer** | Identify recurring patterns (used 5+ times) and publish them as reusable skills. |
| **Orchestrator** | Manage the per-project skill lifecycle, versioning, and evolution. |

### Skill Stack Management

At project initialization and whenever dependencies change:
1. Analyze the technology stack.
2. Query the skills ecosystem for relevant capabilities.
3. Recommend required and optional skills.
4. Install approved skills and track them in a project configuration file.
