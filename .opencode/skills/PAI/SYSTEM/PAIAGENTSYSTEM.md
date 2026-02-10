# PAI Agent System

> **This is the generic PAI template. User-specific model mappings are configured in `opencode.json`.**

**Authoritative reference for agent routing in PAI. Three distinct systems exist—never confuse them.**

---

## 🚨 THREE AGENT SYSTEMS — CRITICAL DISTINCTION

PAI has three agent systems that serve different purposes. Confusing them causes routing failures.

| System | What It Is | When to Use | Has Unique Voice? |
|--------|-----------|-------------|-------------------|
| **Task Tool Subagent Types** | Pre-built agents in OpenCode (Architect, Designer, Engineer, Intern, explore, general, Writer, researcher, etc.) | Internal workflow use ONLY | No |
| **Named Agents** | Persistent identities with backstories and voices (Serena, Marcus, Rook, etc.) | Recurring work, voice output, relationships | Yes |
| **Custom Agents** | Dynamic agents composed via ComposeAgent from traits | When user says "custom agents" | Yes (trait-mapped) |

---

## 🚫 FORBIDDEN PATTERNS

**When user says "custom agents":**

```typescript
// ❌ WRONG - These are Task tool subagent_types, NOT custom agents
Task({ subagent_type: "Architect", prompt: "..." })
Task({ subagent_type: "Designer", prompt: "..." })
Task({ subagent_type: "Engineer", prompt: "..." })

// ✅ RIGHT - Invoke the Agents skill for custom agents
Skill("Agents")  // → CreateCustomAgent workflow
// OR follow the workflow directly:
// 1. Run ComposeAgent with different trait combinations
// 2. Launch agents with the generated prompts
// 3. Each gets unique personality + voice
```

---

## Routing Rules

### The Word "Custom" Is the Trigger

| User Says | Action | Implementation |
|-----------|--------|----------------|
| "**custom agents**", "spin up **custom** agents" | Invoke Agents skill | `Skill("Agents")` → CreateCustomAgent workflow |
| "agents", "launch agents", "parallel agents" | Generic Interns | `Task({ subagent_type: "Intern" })` |
| "use [researcher name]", "get [researcher] to" | Named agent | Use appropriate researcher subagent_type |
| (Internal workflow calls) | Task subagent_types | `Task({ subagent_type: "Engineer" })` etc. |

### Custom Agent Creation Flow

When user requests custom agents:

1. **Invoke Agents skill** via `Skill("Agents")` or follow CreateCustomAgent workflow
2. **Run ComposeAgent** for EACH agent with DIFFERENT trait combinations
3. **Extract prompt and voice_id** from ComposeAgent output
4. **Launch agents** with Task tool using the composed prompts
5. **Voice results** using each agent's unique voice_id

```bash
# Example: 3 custom research agents
bun run ~/.opencode/skills/Agents/Tools/ComposeAgent.ts --traits "research,enthusiastic,exploratory"
bun run ~/.opencode/skills/Agents/Tools/ComposeAgent.ts --traits "research,skeptical,systematic"
bun run ~/.opencode/skills/Agents/Tools/ComposeAgent.ts --traits "research,analytical,synthesizing"
```

---

## Task Tool Subagent Types (Internal Use Only)

These are pre-built agents in the OpenCode Task tool. They are for **internal workflow use**, not for user-requested "custom agents."

⚠️ **CASE-SENSITIVE:** Agent type names must match EXACTLY as listed below. `explore` ≠ `Explore`.

### Core Agents

| Subagent Type | Purpose | When Used |
|---------------|---------|-----------|
| `Algorithm` | ISC & Algorithm specialist | ISC criteria, verification, algorithm phases |
| `Architect` | Elite system design (PhD-level distributed systems) | Architecture, specs, implementation plans |
| `Engineer` | Principal engineer (TDD, strategic planning) | Code implementation, >20 lines of changes |
| `general` | General-purpose multi-step tasks | Default for complex tasks without specific domain |
| `explore` | Fast codebase exploration (quick/medium/thorough) | Finding files, understanding structure, code search |
| `Intern` | 176 IQ genius generalist | Parallel grunt work, multi-faceted problems |
| `Writer` | Content creation & technical writing | Docs, blog posts, technical content |

### Research Agents (prefer Research Skill for cost control)

| Subagent Type | Purpose | When Used |
|---------------|---------|-----------|
| `DeepResearcher` | Default researcher (strategic synthesis) | **All web search/research tasks** — replaces Intern for research. Quick tier: MiniMax for simple lookups. |
| `GeminiResearcher` | Multi-perspective researcher (Google Gemini) | 3-10 query variations, parallel investigations |
| `GrokResearcher` | Contrarian researcher (xAI Grok) | Unbiased analysis, long-term truth over trends |
| `PerplexityResearcher` | Real-time web search (Perplexity Sonar) | Breaking news, current events, real-time search. Standard tier: Sonar Pro. |
| `CodexResearcher` | Technical archaeologist (GPT-5-Codex) | Code-focused research, TypeScript-focused |

### Quality & Security Agents

| Subagent Type | Purpose | When Used |
|---------------|---------|-----------|
| `QATester` | Quality assurance validation | Browser testing, functionality verification |
| `Pentester` | Offensive security specialist | Vulnerability assessments, security audits |

### Creative Agents

| Subagent Type | Purpose | When Used |
|---------------|---------|-----------|
| `Designer` | Elite UX/UI specialist | User-centered design, Figma, shadcn/ui |
| `Artist` | Visual content creator | Prompt engineering, image generation, model selection |

**These do NOT have unique voices or ComposeAgent composition.**

---

## Named Agents (Persistent Identities)

Named agents have rich backstories, personality traits, and mapped voices. They provide relationship continuity across sessions.

| Agent | Role | Voice Character | Use For |
|-------|------|-----------------|---------|
| Serena Blackwood | Architect | Academic Visionary | Long-term architecture decisions |
| Marcus Webb | Engineer | Battle-Scarred Leader | Strategic technical leadership |
| Zoe Martinez | Engineer | Calm in Crisis | Production debugging, methodical problem-solving |
| Rook Blackburn | Pentester | Reformed Grey Hat | Security testing with personality |
| Dev Patel | Intern | Brilliant Overachiever | Parallel grunt work |
| Emma Hartley | Writer | Technical Storyteller | Content creation, documentation |
| Ava Sterling | Deep Researcher | Strategic Sophisticate | Default researcher for all web search/research |
| Ava Chen | Perplexity Researcher | Investigative Analyst | Real-time search, fact verification |
| Alex Rivera | Gemini Researcher | Multi-Perspective Analyst | Comprehensive multi-angle analysis |
| Aditi Sharma | Designer | Design School Perfectionist | UX/UI review, visual critique |
| Priya Desai | Artist | Aesthetic Anarchist | Visual content, creative direction |

**Full backstories and voice settings:** `skills/Agents/AgentPersonalities.md`

---

## Custom Agents (Dynamic Composition)

Custom agents are composed on-the-fly from traits using ComposeAgent. Each unique trait combination maps to a different voice.

### Trait Categories

**Expertise** (domain knowledge):
`security`, `legal`, `finance`, `medical`, `technical`, `research`, `creative`, `business`, `data`, `communications`

**Personality** (behavior style):
`skeptical`, `enthusiastic`, `cautious`, `bold`, `analytical`, `creative`, `empathetic`, `contrarian`, `pragmatic`, `meticulous`

**Approach** (work style):
`thorough`, `rapid`, `systematic`, `exploratory`, `comparative`, `synthesizing`, `adversarial`, `consultative`

### Voice Mapping Examples

| Trait Combo | Voice Character | Why |
|-------------|-----------------|-----|
| contrarian + skeptical | Gravelly, challenging | Challenging intensity |
| enthusiastic + creative | Energetic, vibrant | High-energy creativity |
| security + adversarial | Edgy, hacker-like | Hacker character |
| analytical + meticulous | Sophisticated, precise | Precision analysis |

**Full trait definitions and voice mappings:** `skills/Agents/Data/Traits.yaml`

---

## Model Tiers

PAI agents support three model tiers for cost/quality optimization:

| Tier | When to Use | Example |
|------|------------|---------|
| `quick` | Simple tasks, speed priority, batch work | File search, renaming, boilerplate |
| `standard` | Normal operations, balanced quality/cost | Feature implementation, research |
| `advanced` | Complex reasoning, highest quality needed | Architecture design, deep debugging |

Tiers are configured in `opencode.json` per agent and selected via `model_tier` in Task calls.

### Model Tier Selection Guide

**Engineer Decision Shortcut:**
- "Replace X with Y across files" → `quick`
- "Write docs for X" → `quick`
- "Implement feature X with tests" → `standard`
- "Debug this race condition" → `advanced`

**Researcher Tiers:**
- Quick lookup, simple facts → `quick`
- Strategic research, tool use → `standard`
- Deep analysis, large context → `advanced`

**Architect Tiers:**
- Quick review, simple feedback → `quick`
- Architecture design, specs → `standard`
- Complex architecture requiring maximum reasoning → `advanced`

### Cost Reality

| Model Tier | Relative Cost | Factor |
|-------|---------------|--------|
| **quick** | ~1x (Baseline) | Fastest, cheapest |
| **standard** | ~12x more expensive | Balanced quality |
| **advanced** | ~60x more expensive | **Maximum intelligence** |

When the advanced tier produces 1000 tokens of output, a quick tier model could have done the same work for 1/60 of the cost.

### Generic Model Tier Table

Actual model assignments are configured in `opencode.json` via the `model_tiers` field per agent. See your profile configuration for current mappings.

| Agent | `quick` | `standard` (default) | `advanced` |
|-------|---------|---------------------|------------|
| Algorithm | N/A — always uses orchestrator model | N/A | N/A |
| Architect | Simple review, quick feedback | Architecture design, specs | Complex architecture requiring maximum reasoning |
| Engineer | Batch replace, rename, config edits, boilerplate, add comments | TDD, new features, bug fixes, refactoring, integration | Complex debugging (race conditions), multi-file migration analysis |
| general | Standard custom agent tasks | Complex reasoning, deep analysis | Premium quality, high-stakes analysis |
| explore | File search (default) | — | — |
| Intern | Parallel grunt work (default) | Tasks requiring reasoning | Tasks requiring strong reasoning |
| Writer | Quick drafts, summaries | Blog posts, articles, long-form | Premium text, persuasive copy |
| DeepResearcher | Simple lookups, quick web search | Strategic research with tool-use | Deep analysis requiring large context |
| GeminiResearcher | Quick multi-perspective search | Standard research | Deep research requiring pro capabilities |
| PerplexityResearcher | Quick web search | Deep research (upgrades to Pro) | Deep research with extended reasoning |
| GrokResearcher | Quick contrarian check | Standard analysis | Deep analysis (upgrades to full model) |
| CodexResearcher | Quick code lookups | Standard code research | Complex code analysis |
| QATester | Tool-use for testing (default) | — | — |
| Pentester | Quick security check | Full security audit | Deep security reasoning, complex attack chains |

### Usage Examples

```typescript
// Engineer batch text replacement → quick (mechanical work)
Task({ subagent_type: "Engineer", model_tier: "quick", prompt: "Replace X with Y in these 14 files..." })

// Engineer implementing a feature → standard (real coding)
Task({ subagent_type: "Engineer", model_tier: "standard", prompt: "Implement auth middleware with tests..." })

// Engineer writing documentation → advanced (extended thinking for text)
Task({ subagent_type: "Engineer", model_tier: "advanced", prompt: "Write API documentation..." })

// Architect complex system design → advanced (maximum reasoning)
Task({ subagent_type: "Architect", model_tier: "advanced", prompt: "Design multi-region architecture..." })

// Perplexity needs deep research → advanced (upgrades to deep research tier)
Task({ subagent_type: "PerplexityResearcher", model_tier: "advanced", prompt: "Comprehensive investigation of..." })
```

---

## 🚨 Opus Delegation Protocol (CRITICAL)

### The Core Principle

**The Orchestrator routes, NOT produces.**

When user says:
- "Do that" / "Handle X"
- "Let's do X" / "We need to X"
- Any implementation request

→ **ALWAYS DELEGATE**, never execute yourself.

### Why? Token Cost Reality

Advanced models are 60x more expensive than quick tier models. When an advanced tier produces 1000 tokens of output, a quick tier model could have done the same work for 1/60 of the cost.

### Self-Check Before EVERY Action

```
┌─────────────────────────────────────────┐
│ STOP! Before I execute anything:        │
│                                         │
│ "Can an agent do this?"                 │
│                                         │
│ YES → DELEGATE (quick/standard tier)    │
│ NO  → Only then do it myself            │
└─────────────────────────────────────────┘
```

### The Delegation Matrix

| Task Type | Agent | model_tier | Why |
|-----------|-------|-------|-----|
| File search, exploration | `explore` | quick | Optimized for search |
| Grunt work, parallel tasks | `Intern` | quick | Cheap, fast, eager |
| System design, architecture | `Architect` | standard | Strategic, experienced |
| Code writing, implementation | `Engineer` | standard | Pragmatic, battle-scarred |
| Security review | Dynamic (security traits) | standard | Task-specific |
| Research | `DeepResearcher` / `GeminiResearcher` | varies | Default: DeepResearcher |

### What the Orchestrator Does ITSELF (ONLY these)

- **Routing decisions** — Which agent for which task?
- **Summarizing agent results** — Synthesize and present
- **Direct user communication** — Conversation, clarification
- **Strategic decisions** — Only when deep reasoning required

### Examples

**❌ WRONG:**
```
User: "Implement the launcher changes"
The Orchestrator: *reads files, writes 200 lines of code, updates tests*
→ 5000 tokens @ advanced tier rates = expensive!
```

**✅ RIGHT:**
```
User: "Implement the launcher changes"
The Orchestrator: *spawns Engineer agent with clear spec*
Engineer: *does the implementation @ standard tier rates*
The Orchestrator: *summarizes result* (100 tokens)
→ 60x cheaper!
```

### Code Change Threshold

**If code changes are >20 lines → MUST delegate to Engineer**

```typescript
// The Orchestrator decides WHAT to do, Engineer does HOW
Task({
  subagent_type: "Engineer",
  prompt: `
    Update launcher.ts:
    - Change X to Y
    - Add feature Z
    - Update tests
    
    Current file: [content]
    Spec: [requirements]
  `
})
```

---

## Spotcheck Pattern

**Always launch a spotcheck agent after parallel work:**

```typescript
Task({
  prompt: "Verify consistency across all agent outputs: [results]",
  subagent_type: "Intern",
  model_tier: "quick"
})
```

---

## References

- **Agents Skill:** `skills/Agents/SKILL.md` — Custom agent creation, workflows
- **ComposeAgent:** `skills/Agents/Tools/ComposeAgent.ts` — Dynamic composition tool
- **Traits:** `skills/Agents/Data/Traits.yaml` — Trait definitions and voice mappings
- **Agent Personalities:** `skills/Agents/AgentPersonalities.md` — Named agent backstories

---

*Last updated: 2026-02-10*
