# Agent Skills

[![skills.sh](https://skills.sh/b/amaranthic/skills)](https://skills.sh/amaranthic/skills)

A curated collection of modular, plug-and-play skills for autonomous AI agents, terminal assistants, and LLM workflows following the [Agent Skills](https://agentskills.io) specification.

---

## Skills Catalog

| Skill | Description | Location |
| :--- | :--- | :--- |
| **`agent-context`** | Manages persistent agent session state, user invariants, and error ledgers across sessions without token bloat. | [`skills/agent-context`](./skills/agent-context) |
| **`agent-behavior`** | Governs collaborative agent execution: shared mental models, proactive alignment, cognitive transparency, and self-verification. | [`skills/agent-behavior`](./skills/agent-behavior) |

---

## Getting Started

Each skill is self-contained inside its respective directory, defined by a standard `SKILL.md` file with YAML frontmatter for discovery and execution.

### Installation

Install skills directly using the [Skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add amaranthic/skills
```

To install a specific skill, use:

```bash
npx skills add amaranthic/skills --skill agent-context
```

### Manual Installation

Alternatively, clone the repository and copy the desired skill into your project's agent configuration directory.

```bash
# Clone the repository
git clone https://github.com/amaranthic/skills.git

# Copy a skill into your target project
cp -r skills/skills/agent-context /path/to/your/project/.skills/
```

Replace `/path/to/your/project` with your project's actual path.
