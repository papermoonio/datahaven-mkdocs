# Agent Tasks Config Generator — Skill File

## Purpose

This skill guides an AI agent through the process of analyzing a documentation project and its associated reference repositories to produce an `agent_tasks_config.json` file. That config is then consumed by the `agent_tasks` build plugin to generate self-contained, agent-optimized task files at build time.

This is a **one-time onboarding process** (per project or per major update). The output is a committed config file that the build pipeline uses on every subsequent build. The agent running this skill is *not* the same agent that will later consume the task files — this agent is a collaborator helping a documentation engineer set up the config.

---

## Context: LLM Ready vs Agent Ready

Documentation projects using the `resolve_md` plugin already produce **LLM Ready** artifacts — clean, resolved markdown pages optimized for context-window use cases where an LLM answers questions for a user.

**Agent Ready** is a separate tier. Agents completing tasks independently need different things than LLMs answering questions:

| LLM Ready | Agent Ready |
|---|---|
| Clean prose explaining concepts | Sequential steps with concrete commands |
| Resolved snippets inline | Fetchable URLs to complete, working code files |
| Category bundles for broad context | Single task files scoped to one workflow |
| Optimized for comprehension | Optimized for execution |

The `agent_tasks_config.json` bridges the gap by defining task workflows that the build plugin assembles into agent-consumable files.

---

## What You Will Produce

A complete `agent_tasks_config.json` file with this structure:

```json
{
    "schema_version": "0.1",
    "project": { ... },
    "reference_repos": { ... },
    "tasks": [ ... ],
    "outputs": { ... }
}
```

This file lives in the project root alongside `llms_config.json`.

---

## Prerequisites

Before starting, confirm you have access to:

1. **The documentation project** — an MkDocs site with `resolve_md` already configured and an existing `llms_config.json`
2. **At least one reference repository** — a repo containing working, tested code that mirrors what the documentation teaches (e.g., example scripts, sample projects, SDK usage examples)
3. **The documentation engineer** — a human collaborator who can answer questions about task scope, prerequisites, and known error patterns

---

## Process

### Phase 1: Inventory the Documentation

**Goal:** Understand what the docs cover and how they're organized.

1. Read `llms_config.json` to understand:
   - Project identity (`project.id`, `project.name`, `docs_base_url`)
   - Content categories (`content.categories_info`) — these tell you the domain areas the docs cover
   - Output paths (`outputs`) — where resolved pages will be served from

2. Scan the docs directory structure:
   - Read `.nav.yml` files to understand the navigation hierarchy and logical groupings
   - Identify tutorial/guide pages vs reference pages vs conceptual pages
   - Note which pages walk users through multi-step workflows — these are your task candidates

3. Read frontmatter on candidate pages:
   - `title` and `description` for understanding scope
   - `categories` for understanding which domain areas a page covers

4. Identify task-shaped content:
   - Pages or page sequences that walk through a complete workflow from start to finish
   - Content that has a clear objective (e.g., "by the end of this guide, you will have...")
   - Multi-page flows connected by navigation ordering

**Output of this phase:** A list of candidate tasks with:
- A working title for each task
- The doc pages involved
- The approximate scope (what does the user accomplish?)

**Checkpoint:** Present the candidate task list to the documentation engineer for validation before proceeding.

---

### Phase 2: Analyze the Reference Repository

**Goal:** Map the reference repo's working code to the candidate tasks.

1. Examine the reference repo structure:
   - Clone or browse the repo to understand its directory layout
   - Identify the main code files and their purposes
   - Note the language, runtime, and dependency requirements

2. For each candidate task, identify:
   - Which files from the reference repo are needed
   - The logical order in which an agent should create/use those files
   - The base path within the repo (e.g., `node/scripts`, `examples/python`)

3. Construct the reference repo entry:
   - `url`: The GitHub URL of the repo
   - `default_branch`: The branch to reference (usually `main`)
   - `raw_base_url`: Constructed as `https://raw.githubusercontent.com/{org}/{repo}/{branch}`
   - `description`: What this repo provides

4. For each file, determine:
   - `path`: Relative path from the base path
   - `description`: One-line summary of what this file does

**Output of this phase:** A complete `reference_repos` block and a `reference_code` block for each task.

---

### Phase 3: Define Task Steps

**Goal:** Create the sequential execution plan for each task.

For each candidate task:

1. **Write the objective** — one sentence describing what the agent will accomplish. Be specific and outcome-oriented.

2. **Define prerequisites** — group by type:
   - `runtime`: Language versions, package managers, build tools
   - `network`: Network access, API endpoints, chain connections
   - `tokens`: Testnet tokens, API keys, credentials needed
   - Add other groups as needed for the project

3. **Define environment variables** — for each:
   - `name`: The variable name exactly as it appears in code
   - `description`: What it is and where to get it
   - `required`: Whether the task cannot proceed without it

4. **Define steps** — each step should have:
   - `order`: Sequential number
   - `action`: Short imperative description of what to do
   - `description` (optional): More detail if the action isn't self-explanatory
   - `commands` (optional): Literal shell commands to run — use these for setup steps like project init, dependency installation, and execution
   - `reference_file` (optional): Points to a file in `reference_code` — use these for steps where the agent needs to create or understand a code file
   - `expected_output` (optional): What success looks like — use for the final execution step or key milestones

   **Guideline:** A step should either have `commands` (do this) or `reference_file` (create/use this code), rarely both. Keep steps atomic — one action per step.

5. **Define error patterns** — for each known failure mode:
   - `pattern`: The error message or symptom the agent might encounter
   - `cause`: Why this happens
   - `resolution`: How to fix it

   **Sources for error patterns:** The docs' troubleshooting pages, the reference repo's issues, and the documentation engineer's experience. Ask the engineer explicitly: "What are the most common things that go wrong when someone follows this workflow?"

6. **Define supplementary context** — identify resolved doc pages that provide deeper explanation:
   - `slug`: The page slug matching the resolve_md output filename
   - `url`: The full URL to the resolved page (constructed from `docs_base_url` + `outputs.public_root` + `outputs.files.pages_dir` + `/{slug}.md` in `llms_config.json`)
   - `relevance`: Why an agent might need this page

   **Guideline:** These are fallback resources. The agent's primary path is the steps + reference code. Supplementary context is for when the agent needs to understand *why* something works, not *how* to do it.

**Checkpoint:** Present the complete task definition to the documentation engineer for review before finalizing.

---

### Phase 4: Assemble the Config

**Goal:** Produce the final `agent_tasks_config.json`.

1. **Set the project block** — mirror the relevant fields from `llms_config.json`:
   ```json
   "project": {
       "id": "<from llms_config>",
       "name": "<from llms_config>",
       "docs_base_url": "<from llms_config>"
   }
   ```

2. **Set the reference_repos block** — one entry per reference repository:
   ```json
   "reference_repos": {
       "<repo-id>": {
           "url": "https://github.com/<org>/<repo>",
           "default_branch": "main",
           "raw_base_url": "https://raw.githubusercontent.com/<org>/<repo>/main",
           "description": "<what this repo provides>"
       }
   }
   ```

3. **Set the tasks array** — one entry per task, using all the data from Phases 1-3.

4. **Set the outputs block**:
   ```json
   "outputs": {
       "public_root": "/ai/",
       "tasks_dir": "tasks"
   }
   ```
   The `public_root` should match the value in `llms_config.json`. The `tasks_dir` is the subdirectory within the AI output where task files will be written.

5. **Set the schema_version** to `"0.1"`.

---

### Phase 5: Validate

Before delivering the config:

1. **Structural check** — verify the JSON is valid and all required fields are present
2. **Reference check** — verify that every `reference_file` in a step has a corresponding entry in the task's `reference_code.files` array
3. **URL check** — verify that `raw_base_url` + `base_path` + `path` constructs a valid GitHub raw URL for each reference file
4. **Supplementary page check** — verify that referenced page slugs correspond to actual pages in the docs
5. **Step ordering check** — verify steps are sequential and logically ordered (setup before implementation, implementation before execution)
6. **Completeness check** — could an agent with no prior knowledge of this project follow the steps from start to finish using only the task file and fetchable reference code?

---

## Schema Reference

### Top Level

| Field | Type | Required | Description |
|---|---|---|---|
| `schema_version` | string | yes | Schema version, currently `"0.1"` |
| `project` | object | yes | Project identity |
| `reference_repos` | object | yes | Map of repository ID to repo metadata |
| `tasks` | array | yes | Array of task definitions |
| `outputs` | object | yes | Output path configuration |

### project

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Project identifier (match `llms_config.json`) |
| `name` | string | yes | Human-readable project name |
| `docs_base_url` | string | yes | Base URL of the published docs site |

### reference_repos[id]

| Field | Type | Required | Description |
|---|---|---|---|
| `url` | string | yes | GitHub repository URL |
| `default_branch` | string | yes | Branch to reference for raw URLs |
| `raw_base_url` | string | yes | Base URL for raw file access |
| `description` | string | yes | What this repo provides |

### tasks[]

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | URL-safe task identifier |
| `title` | string | yes | Human-readable task title |
| `objective` | string | yes | One sentence describing what the agent accomplishes |
| `prerequisites` | object | yes | Grouped prerequisite lists |
| `env_vars` | array | yes | Environment variables needed (can be empty array) |
| `steps` | array | yes | Sequential execution steps |
| `reference_code` | object | yes | Index of fetchable code files |
| `error_patterns` | array | yes | Known failure modes and fixes (can be empty array) |
| `supplementary_context` | object | yes | Links to resolved doc pages for deeper context |

### tasks[].steps[]

| Field | Type | Required | Description |
|---|---|---|---|
| `order` | integer | yes | Sequential step number |
| `action` | string | yes | Short imperative description |
| `description` | string | no | Additional detail |
| `commands` | array | no | Literal shell commands |
| `reference_file` | string | no | Path to file in `reference_code` |
| `expected_output` | string | no | What success looks like |

### tasks[].reference_code

| Field | Type | Required | Description |
|---|---|---|---|
| `repo` | string | yes | Key into `reference_repos` |
| `base_path` | string | yes | Directory within the repo containing the files |
| `files` | array | yes | Array of file entries |

### tasks[].reference_code.files[]

| Field | Type | Required | Description |
|---|---|---|---|
| `path` | string | yes | File path relative to `base_path` |
| `description` | string | yes | One-line summary of the file's purpose |

### tasks[].env_vars[]

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Variable name as used in code |
| `description` | string | yes | What it is and where to get it |
| `required` | boolean | yes | Whether the task fails without it |

### tasks[].error_patterns[]

| Field | Type | Required | Description |
|---|---|---|---|
| `pattern` | string | yes | Error message or symptom |
| `cause` | string | yes | Why this happens |
| `resolution` | string | yes | How to fix it |

### tasks[].supplementary_context

| Field | Type | Required | Description |
|---|---|---|---|
| `description` | string | yes | Guidance for the consuming agent on when to use these |
| `pages` | array | yes | Array of page references |

### tasks[].supplementary_context.pages[]

| Field | Type | Required | Description |
|---|---|---|---|
| `slug` | string | yes | Page slug matching resolve_md output |
| `url` | string | yes | Full URL to the resolved page |
| `relevance` | string | yes | Why an agent might need this page |

### outputs

| Field | Type | Required | Description |
|---|---|---|---|
| `public_root` | string | yes | Root path for AI artifacts (match `llms_config.json`) |
| `tasks_dir` | string | yes | Subdirectory for task file output |
