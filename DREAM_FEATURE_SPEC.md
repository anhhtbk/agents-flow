# Dream: Memory Consolidation — Feature Specification

**Source:** Claude Code binary `2.1.83` (`/Users/themrb/.local/share/claude/versions/2.1.83`)
**Extracted:** By reverse-engineering embedded JavaScript via `strings` and `dd` extraction
**Purpose:** Automatic reflective pass over memory files to consolidate, prune, and index memories across sessions

---

## Overview

Dream is a background memory consolidation task that runs automatically after N hours and M sessions, or can be triggered manually. It reviews recent session transcripts, daily logs, and existing memories to produce updated, well-organized memories for future sessions.

---

## Core Prompt — `TT9` Function

**Offset:** ~78,820,174 in binary

```javascript
function TT9(_, T, q) {
  return `# Dream: Memory Consolidation
You are performing a dream — a reflective pass over your memory files. Synthesize what you've learned recently into durable, well-organized memories so that future sessions can orient quickly.

Memory directory: \`${_}\`
${oLq}

Session transcripts: \`${T}\` (large JSONL files — grep narrowly, don't read whole files)

## Phase 1 — Orient
- \`ls\` the memory directory to see what already exists
- Read \`${pw}\` to understand the current index
- Skim existing topic files so you improve them rather than creating duplicates
- If \`logs/\` or \`sessions/\` subdirectories exist (assistant-mode layout), review recent entries there

## Phase 2 — Gather recent signal
Look for new information worth persisting. Sources in rough priority order:
1. **Daily logs** (\`logs/YYYY/MM/YYYY-MM-DD.md\`) if present — these are the append-only stream
2. **Existing memories that drifted** — facts that contradict something you see in the codebase now
3. **Transcript search** — if you need specific context (e.g., "what was the error message from yesterday's build failure?"), grep the JSONL transcripts for narrow terms:
   \`grep -rn "<narrow term>" ${T}/ --include="*.jsonl" | tail -50\`
Don't exhaustively read transcripts. Look only for things you already suspect matter.

## Phase 3 — Consolidate
For each thing worth remembering, write or update a memory file at the top level of the memory directory. Use the memory file format and type conventions from your system prompt's auto-memory section — it's the source of truth for what to save, how to structure it, and what NOT to save.
Focus on:
- Merging new signal into existing topic files rather than creating near-duplicates
- Converting relative dates ("yesterday", "last week") to absolute dates so they remain interpretable after time passes
- Deleting contradicted facts — if today's investigation disproves an old memory, fix it at the source

## Phase 4 — Prune and index
Update \`${pw}\` so it stays under ${$d} lines AND under ~25KB. It's an **index**, not a dump — each entry should be one line under ~150 characters: \`- [Title](file.md) — one-line hook\`. Never write memory content directly into it.
- Remove pointers to memories that are now stale, wrong, or superseded
- Demote verbose entries: if an index line is over ~200 chars, it's carrying content that belongs in the topic file — shorten the line, move the detail
- Add pointers to newly important memories
- Resolve contradictions — if two files disagree, fix the wrong one

Return a brief summary of what you consolidated, updated, or pruned. If nothing changed (memories are already tight), say so.${q ? `
## Additional context
${q}` : ""}`;
}
```

### Variables Used in Prompt

| Variable | Value | Meaning |
|---|---|---|
| `_` | `BH()` | Memory directory path |
| `T` | `Aj(tq())` | Session transcripts directory path |
| `q` | additional context string | Optional extra context |
| `pw` | `"MEMORY.md"` | Memory index filename |
| `oLq` | `"This directory already exists — write to it directly with the Write tool (do not run mkdir or check for its existence)."` | Instruction hint |
| `$d` | `200` | Max lines for MEMORY.md |

---

## Constants & Configuration

```javascript
// Default thresholds (can be overridden via tengu_onyx_plover setting)
KT9 = {
  minHours: 24,      // Minimum hours since last consolidation
  minSessions: 5      // Minimum new sessions since last consolidation
};

// Lock file
SC$ = ".consolidate-lock";  // Filename in memory directory

// Lock staleness check
EC$ = 3600000;              // 1 hour in ms — lock considered stale after this

// Scan throttle
fb$ = 600000;               // 10 minutes in ms — minimum time between autoDream scans

// Task turn history
bC$ = 30;                   // Max turns to keep in task history
```

---

## Environment Variables / Settings

| Name | Type | Default | Description |
|---|---|---|---|
| `tengu_onyx_plover` | JSON `{minHours, minSessions, enabled}` | `{minHours: 24, minSessions: 5}` | Override autoDream timing |
| `tengu_onyx_plover.enabled` | bool | auto (from `autoDreamEnabled` setting) | Enable/disable autoDream |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | bool | false | Disables auto memory extraction |
| `tengu_herring_clock` | bool | false | Disables team memory directory |

---

## AutoDream Trigger Logic — `$T9` / `OT9`

```javascript
$T9 = async function(q, K) {
  let $ = wb$(), O = Pb$();
  if (!O && !Yb$()) return;  // Yb$() checks: not in forbidden group, not I7(), is cK(), YmT() enabled

  let R;
  try { R = await xIT() }    // Read last consolidation timestamp
  catch (D) { return; }

  let A = (Date.now() - R) / 3600000;  // hours since last
  if (!O && A < $.minHours) return;     // Not enough hours elapsed

  let H = Date.now() - _;
  if (!O && H < fb$) return;            // Throttled (10 min throttle)

  _ = Date.now();
  let j;
  try { j = await Vo7(R) }              // Get session IDs modified since R
  catch (D) { return; }

  let z = VT();                          // Current session ID
  j = j.filter((D) => D !== z);          // Exclude current session
  if (!O && j.length < $.minSessions) return;  // Not enough new sessions

  let f;
  if (O) f = R;
  else {
    f = await yo7();                     // Acquire lock
    if (f === null) return;              // Lock held by another process
  }

  // FORK THE DREAM
  let w = q.toolUseContext.setAppStateForTasks ?? q.toolUseContext.setAppState;
  let Y = new AbortController();
  let P = Co7(w, { sessionsReviewing: j.length, priorMtime: f, abortController: Y });

  try {
    let D = BH(), J = Aj(tq());
    let k = `**Tool constraints for this run:** Bash is restricted to read-only commands (\`ls\`, \`find\`, \`grep\`, \`cat\`, \`stat\`, \`wc\`, \`head\`, \`tail\`, and similar). Anything that writes, redirects to a file, or modifies state will be denied. Plan your exploration with this in mind — no need to probe.
Sessions since last consolidation (${j.length}):
${j.map((Z) => `- ${Z}`).join('\n')}`;

    let X = TT9(D, J, k);
    let W = await cW({
      promptMessages: [BT({ content: X })],
      cacheSafeParams: sy(q),
      canUseTool: Db$(D),
      querySource: "auto_dream",
      forkLabel: "auto_dream",
      skipTranscript: !0,
      overrides: { abortController: Y },
      onMessage: Jb$(P, w)
    });

    Eo7(P, w);  // Mark completed

    let h = q.toolUseContext.getAppState().tasks?.[P];
    if (K && Lo7(h) && h.filesTouched.length > 0)
      K({ ...JmT(h.filesTouched), verb: "Improved" });

  } catch (D) {
    if (Y.signal.aborted) { return; }
    bo7(P, w);  // Mark failed
    await uIT(f);  // Release lock
  }
};
```

### Enabling Checks (`Yb$`)

```javascript
function Yb$() {
  if (FG()) return false;        // Forbidden group check
  if (I7()) return false;        // I7 check
  if (!cK()) return false;       // cK check
  return YmT();                  // autoDreamEnabled from settings
}
```

---

## Lock Mechanism

### Acquire Lock — `yo7()`

```javascript
async function yo7() {
  let _ = rlq(), T, q;  // rlq() = path.join(BH(), ".consolidate-lock")
  try {
    let [$, O] = await Promise.all([zG.stat(_), zG.readFile(_, "utf8")]);
    T = $.mtimeMs;
    let R = parseInt(O.trim(), 10);
    q = Number.isFinite(R) ? R : undefined;
  } catch {}

  // If lock exists and is less than 1h old, check if PID is alive
  if (T !== undefined && Date.now() - T < EC$) {
    if (q !== undefined && uY_(q))  // PID still alive
      return null;  // Lock held
  }

  // Write our PID to lock file
  await zG.mkdir(BH(), { recursive: true });
  await zG.writeFile(_, String(process.pid));

  let K = await zG.readFile(_, "utf8");
  if (parseInt(K.trim(), 10) !== process.pid) return null;

  return T ?? 0;  // Return prior mtime
}
```

### Release Lock — `uIT(_)`

```javascript
async function uIT(_) {
  let T = rlq();
  try {
    if (_ === 0) {
      await zG.unlink(T);
      return;
    }
    await zG.writeFile(T, "");
    let q = _ / 1000;
    await zG.utimes(T, q, q);  // Restore prior mtime
  } catch (q) {
    N(`[autoDream] rollback failed: ${q.message} — next trigger delayed to minHours`);
  }
}
```

### Read Last Consolidation Time — `xIT()`

```javascript
async function xIT() {
  try {
    return (await zG.stat(rlq())).mtimeMs;
  } catch {
    return 0;
  }
}
```

---

## Session Discovery — `Vo7()` / `Zo7()`

```javascript
async function Zo7(_, T, q) {
  // Reads a directory looking for .jsonl session files
  let K = await bIT.readdir(_);
  return (await Promise.all(K.map(async (O) => {
    if (!O.endsWith(".jsonl")) return null;
    let R = oD8(O.slice(0, -6));  // Parse session ID from filename
    if (!R) return null;
    let A = nlq.join(_, O);
    if (!T) return { sessionId: R, filePath: A, mtime: 0, projectPath: q };
    try {
      let H = await bIT.stat(A);
      return { sessionId: R, filePath: A, mtime: H.mtime.getTime(), projectPath: q };
    } catch { return null; }
  }))).filter((O) => O !== null);
}

async function Vo7(_) {
  let T = Aj(tq());  // Session transcript directory
  return (await Zo7(T, true)).filter((K) => K.mtime > _).map((K) => K.sessionId);
}
```

---

## Task Management

### Create Dream Task — `Co7()`

```javascript
function Co7(_, T) {
  let q = qy("dream");  // Generate unique task ID
  let K = {
    ...hW(q, "dream", "dreaming"),
    type: "dream",
    status: "running",
    phase: "starting",
    sessionsReviewing: T.sessionsReviewing,
    filesTouched: [],
    turns: [],
    abortController: T.abortController,
    priorMtime: T.priorMtime
  };
  return BW(K, _), q;  // Register in app state
}
```

### Update Task (on each turn) — `So7()`

```javascript
function So7(_, T, q, K) {
  R$(_, K, ($) => {
    let O = new Set($.filesTouched);
    let R = q.filter((A) => !O.has(A) && O.add(A));
    if (T.text === "" && T.toolUseCount === 0 && R.length === 0) return $;
    return {
      ...$,
      phase: R.length > 0 ? "updating" : $.phase,
      filesTouched: R.length > 0 ? [...$.filesTouched, ...R] : $.filesTouched,
      turns: [...$.turns.slice(-(bC$ - 1)), T]
    };
  });
}
```

### Mark Completed — `Eo7()`

```javascript
function Eo7(_, T) {
  R$(_, T, (q) => ({
    ...q,
    status: "completed",
    endTime: Date.now(),
    notified: true,
    abortController: undefined
  }));
}
```

### Mark Failed — `bo7()`

```javascript
function bo7(_, T) {
  R$(_, T, (q) => ({
    ...q,
    status: "failed",
    endTime: Date.now(),
    notified: true,
    abortController: undefined
  }));
}
```

### Dream Task Type Definition

```javascript
mIT = {
  name: "DreamTask",
  type: "dream",
  async kill(_, T) {
    let q;
    R$(_, T, (K) => {
      if (K.status !== "running") return K;
      K.abortController?.abort();
      q = K.priorMtime;
      return { ...K, status: "killed", endTime: Date.now(), notified: true, abortController: undefined };
    });
    if (q !== undefined) await uIT(q);
  }
};
```

### Type Guard — `Lo7()`

```javascript
function Lo7(_) {
  return typeof _ === "object" && _ !== null && "type" in _ && _.type === "dream";
}
```

---

## Tool Constraints During Dream

```javascript
function Db$(_) {
  let T = PmT(_);  // Standard auto-memory tool policy
  return async (q, K, $, O, R, A) => {
    if (q.name === v6) {  // Bash tool
      let H = q.inputSchema.safeParse(K);
      if (H.success && q.isReadOnly(H.data)) return { behavior: "allow", updatedInput: K };
      let j = "Only read-only shell commands are permitted in this context (ls, find, grep, cat, stat, wc, head, tail, and similar)";
      return { behavior: "deny", message: j, decisionReason: { type: "other", reason: j } };
    }
    return T(q, K, $, O, R, A);  // Delegate to standard policy for other tools
  };
}
```

**Allowed Bash commands (implicit via `isReadOnly` check):**
- `ls`, `find`, `grep`, `cat`, `stat`, `wc`, `head`, `tail`
- Any command deemed "read-only" by the `isReadOnly()` schema check

**Write operations (Write, Edit, Bash with redirects/writes) are DENIED.**

---

## Message Handler — `Jb$()`

```javascript
function Jb$(_, T) {
  return (q) => {
    if (q.type !== "assistant") return;
    let K = "", $ = 0, O = [];
    for (let R of q.message.content) {
      if (R.type === "text") K += R.text;
      else if (R.type === "tool_use") {
        $++;
        if (R.name === V7 || R.name === j4) {  // Read/Edit file tools
          let A = R.input;
          if (typeof A.file_path === "string") O.push(A.file_path);
        }
      }
    }
    So7(_, { text: K.trim(), toolUseCount: $ }, O, T);
  };
}
```

---

## Key Directory Functions

| Function | Returns | Description |
|---|---|---|
| `BH()` | string | Memory directory path (e.g., `~/.claude/memory/`) |
| `Aj(tq())` | string | Session transcripts directory (e.g., `~/.claude/sessions/`) |
| `rlq()` | string | Lock file path = `BH()/.consolidate-lock` |
| `VT()` | string | Current session ID |
| `tq()` | string | Current working directory (project root) |

---

## Task State Machine

```
Co7() → status: "running", phase: "starting"
  ↓
on each message → So7() → phase: "updating" (if files touched)
  ↓
Eo7() → status: "completed", endTime: Date.now()
   OR
bo7() → status: "failed", endTime: Date.now()
   OR
kill() → status: "killed", endTime: Date.now()
```

---

## Module Initialization Order — `liq`

```javascript
var liq = M(() => {
  vy(); H8(); RT(); vT(); sq(); Lj(); ciq();
  l7(); XT(); DmT(); qT9(); IIT(); pIT(); Bj();
  KT9 = { minHours: 24, minSessions: 5 };
});
```

- `DmT()` — Initializes `TT9` (Dream prompt)
- `qT9()` — Initializes extraction memory subsystem
- `IIT()` — Initializes lock/session discovery (`yo7`, `Vo7`, `xIT`)
- `pIT()` — Registers `mIT` (DreamTask type) into task registry

---

## Memory File Format Reference

From the TT9 prompt and related code, memory files use this format:

```markdown
---
name: {{memory name}}
description: {{one-line description — used to decide relevance in future conversations, so be specific}}
type: {{user, feedback, project, reference}}
---

{{memory content — for feedback/project types, structure as: rule/fact, then **Why:** and **How to apply:** lines}}
```

### MEMORY.md Index Entry Format

```
- [Title](file.md) — one-line hook
```

Rules:
- Max ~150 characters per line
- Max 200 lines total in MEMORY.md
- Max ~25KB total size

### Memory Types

| Type | Description |
|---|---|
| `user` | User's role, preferences, knowledge |
| `feedback` | Guidance on what to avoid/repeat, with **Why:** and **How to apply:** |
| `project` | Project-specific state, goals, initiatives |
| `reference` | Pointers to external systems/resources |

---

## Telemetry Events

| Event | Fields | When |
|---|---|---|
| `tengu_auto_dream_fired` | `hours_since`, `sessions_since` | Dream starts |
| `tengu_auto_dream_completed` | `cache_read`, `cache_created`, `output`, `sessions_reviewed` | Dream finishes |
| `tengu_auto_dream_failed` | — | Dream throws |
| `tengu_extract_memories_skipped_direct_write` | `message_count` | Skipped due to direct memory writes |
| `tengu_memdir_loaded` | `total_file_count`, `total_subdir_count`, `content_length`, `memory_type` | Memory dir read |

---

## Dream vs. ExtractMemories

| | Dream (`TT9`) | ExtractMemories (`jb$`) |
|---|---|---|
| Trigger | Automatic (time/session based) OR manual | Automatic (after N messages in conversation) |
| Scope | All sessions since last consolidation | Current conversation only |
| Tools | Read-only Bash + memory file writes | Only `Write`, `Read`, `Edit` within memory dir |
| Goal | Full memory refresh/prune/index | Capture recent conversation into memory |

---

## Implementation Notes for Recreation

1. **Fork model**: Dream runs as a forked work session via `cW()`. It is NOT a tool call — it's a complete separate Claude session with its own prompt.

2. **Tool restrictions**: Only `Read`, `Write`, `Edit` for memory files, and read-only `Bash` (ls, grep, cat, head, tail, find, stat, wc). No `Bash` with redirections or write operations.

3. **Locking**: Uses a simple `.consolidate-lock` file containing the PID. Lock is checked for staleness at 1 hour.

4. **Transcript discovery**: Sessions are `.jsonl` files named by session ID. Filtered by modification time (`mtime > priorMtime`).

5. **No slash command found**: Based on binary analysis, Dream appears to run only via the auto-trigger mechanism (`$T9`). No explicit slash command registration (`/dream`) was found in the extracted strings — it may be a user-facing alias that maps to the same internal task type, or it may be registered via a generic slash command handler not visible in static strings analysis.

6. **The `cW` function**: The `cW()` call is the workflow fork function. It takes the prompt messages, tool constraints, query source label, and an on-message callback. Its exact definition is in a bundled/internal module not visible via static strings extraction. For recreation, this maps to "create a background task with a custom system prompt".
