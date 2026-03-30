# Model Routing Reference

Khuym uses a hybrid model architecture to leverage the strengths of different AI models.

## Model Assignments

| Model             | Role         | Strengths                                  | Skills Using                                         |
| ----------------- | ------------ | ------------------------------------------ | ---------------------------------------------------- |
| **Opus (Claude)** | Orchestrator | Deep reasoning, planning, validation       | exploring, planning, validating, swarming, reviewing |
| **Gemini 2.5**    | Researcher   | 1M token context, web search, large docs   | planning (Phase 1 Discovery)                         |
| **Codex (GPT-5)** | Worker       | Fast coding, implementation, bounded tasks | executing                                            |

---

## Routing Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│                       TASK ARRIVES                              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
     Is it implementation?         Is it research?
              │                           │
     ┌────────┴────────┐         ┌────────┴────────┐
     │ YES             │         │ YES             │
     │ → CODEX         │         │                 │
     │ (executing)     │         │ External docs?  │
     └─────────────────┘         │     ▼           │
                                 │  YES → GEMINI   │
                                 │  NO → gkg/local │
                                 └─────────────────┘
                            │
                            ▼
                  Planning/Validation?
                            │
                     YES → OPUS
```

---

## Detailed Routing Rules

### Route to Opus (Claude)

- User intent extraction (exploring skill)
- Feature planning and approach synthesis
- Plan validation and verification
- Swarm orchestration and coordination
- Code review and quality assessment
- Any high-stakes decision requiring deep reasoning

### Route to Gemini

- **Web research**: `web_search`, `google_search`
- **Documentation fetching**: `read_url`
- **Large codebase analysis**: >500 files, need holistic view
- **Multi-source synthesis**: comparing 5+ documentation pages
- **External library research**: new integrations with no codebase precedent

### Route to Codex

- **Bead implementation**: write code, tests, configs
- **Bounded coding tasks**: single file or module changes
- **Test writing and fixing**: unit tests, integration tests
- **Refactoring**: mechanical code transformations
- **Bug fixes**: isolated issue resolution

### Keep Local (gkg, grep, Read)

- Quick symbol lookup in current codebase
- Pattern search in <100 files
- Reading specific known files
- Dependency graph within project
- File content verification

---

## MCP Configuration

All models are configured in `plugins/khuym/skills/planning/mcp.json`:

```json
{
  "gemini": {
    "type": "stdio",
    "command": "gemini",
    "args": ["mcp"],
    "includeTools": ["web_search", "google_search", "read_url", "analyze_codebase"]
  },
  "codex": {
    "type": "stdio",
    "command": "codex",
    "args": ["mcp"],
    "includeTools": ["spawn_worker", "execute_task"]
  }
}
```

---

## Skill → Model Mapping

| Skill               | Primary Model | Secondary Models           |
| ------------------- | ------------- | -------------------------- |
| `khuym-flow`       | Opus          | -                          |
| `khuym-exploring`   | Opus          | -                          |
| `khuym-planning`    | Opus          | Gemini (external research) |
| `khuym-validating`  | Opus          | -                          |
| `khuym-swarming`    | Opus          | Codex (workers)            |
| `khuym-executing`   | **Codex**     | -                          |
| `khuym-reviewing`   | Opus          | -                          |
| `khuym-debugging`   | Opus          | Codex (fix implementation) |
| `khuym-compounding` | Opus          | -                          |

---

## Cost Optimization

The hybrid model approach optimizes for both quality and cost:

1. **Opus** for high-stakes reasoning (highest quality, moderate cost)
2. **Gemini** for research (large context, efficient for doc synthesis)
3. **Codex** for implementation (fast, cost-effective for volume)

Estimated cost distribution in typical feature development:

- Planning phase: ~30% (Opus heavy)
- Research phase: ~10% (Gemini heavy)
- Implementation phase: ~50% (Codex heavy)
- Review phase: ~10% (Opus heavy)

---

## Fallback Behavior

If a model's MCP is unavailable:

| Missing Model | Fallback                             |
| ------------- | ------------------------------------ |
| Gemini        | Use `WebFetch` + local tools         |
| Codex         | Use current model for implementation |
| Opus          | (Should not happen - it's the host)  |

Workers should note in their reports if fallback was used.

---

## Cross-References

- `plugins/khuym/skills/planning/SKILL.md` - Model Routing for Research section
- `plugins/khuym/skills/swarming/SKILL.md` - Worker Model Configuration
- `plugins/khuym/skills/executing/SKILL.md` - Hybrid Model Context
- `plugins/khuym/skills/planning/mcp.json` - MCP configurations
