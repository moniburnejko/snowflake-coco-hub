# obsidian + cortex code CLI: usage patterns and examples

practical patterns for using Obsidian as a knowledge layer in Cortex Code CLI sessions.

---

## the core pattern

the most useful mental model: **context in, work done, documentation out**, all in one session.

1. read relevant context from vault (standards, specs, previous decisions)
2. do the actual work in Snowflake (SQL, DAGs, code, analysis)
3. write results and notes back to vault

no context switching, no copy-paste.

---

## how it works in practice

once Obsidian is connected via MCP, Cortex Code treats vault operations as regular tools. you don't invoke them manually. you describe what you want and the agent decides whether to read a note, search for something, or write a file.

tools are namespaced as `mcp__obsidian__*`. the agent calls them automatically. you see the tool calls in the session output as it works.

---

## pattern 1: documentation on the fly

generate notes during the session while context is fresh, not after.

```
write a summary of what we just did and save it to vault/project/task/TASK-123.md
```

```
generate a retrospective for this sprint and append it to vault/project/retro/april-2026.md
```

```
we just made an architectural decision about X — document it in vault/project/decisions/adr-012.md with context, options we considered, and what we chose
```

---

## pattern 2: vault as context for active work

your vault already has your standards, architecture notes, and decisions. instead of re-explaining or copy-pasting into the session, reference them directly:

```
read vault/project/architecture/data-layer.md and use it to write DDL for a new MY_TABLE table
```

```
check vault/project/standards/coding-standards.md and tell me if this follows our conventions
```

```
read vault/research/topic-name/ and tell me if we already have notes on X, or if that's a gap
```

```
read vault/project/requirements/new-table-spec.md and generate DDL and a DAG from it
```

this is useful when Cortex Code's session context is limited. instead of dumping documents into the conversation, pull only what's relevant on demand.

---

## pattern 3: snowflake data → vault

query Snowflake and write results directly to Obsidian without copy-pasting:

```
list all tables in MY_SCHEMA with row counts and save to vault/project/data-catalog/my-schema-overview.md
```

```
find tables in MY_DB with no inserts in the last 30 days and save the list with a recommendation to vault/project/data-quality/stale-tables.md
```

```
summarize Snowpipe error logs from the last 7 days and save to vault/project/ops/snowpipe-errors-april.md
```

```
run the data quality checks on MY_SCHEMA and update vault/project/data-quality/weekly-checks.md with today's results
```

---

## pattern 4: auto-generating object documentation

```
describe the structure of MY_SCHEMA.MY_TABLE — columns, types, short description of each — and save to vault/project/data-catalog/MY_TABLE.md
```

```
generate documentation for this DAG and save it to vault/project/airflow/dags/my-dag.md
```

```
look at all tables in MY_SCHEMA that don't have a corresponding file in vault/project/data-catalog/ and create stub pages for each
```

---

## pattern 5: standup and meeting prep

```
look at vault/project/tickets/ notes from this week and generate a summary of what i completed, suitable for a weekly sync
```

```
prepare a review agenda based on vault/project/sprint-notes/sprint-42.md
```

---

## pattern 6: research and presentation prep

```
search vault/research/ for everything about Cortex Code and generate a draft "what's new" section for the next presentation
```

```
compare vault/research/topic/march-2026.md and vault/research/topic/april-2026.md and summarize what changed in our understanding
```

```
read vault/ideas/demo-ideas.md, pick the most technically interesting one, and start building it
```

---

## pattern 7: knowledge base as input for code generation

requirements or specs written in Obsidian become direct inputs for code:

```
read vault/project/requirements/new-table-spec.md and generate DDL and a DAG from it
```

```
i have a ticket description in vault/project/tickets/TICKET-123.md — implement it
```

```
read vault/project/standards/ and generate a template for a new pipeline DAG that follows all our conventions
```

---

## pattern 8: LLM wiki ingest (karpathy pattern)

if you're running the LLM wiki pattern with Cortex Code as the engine:

```
i dropped a new article in vault/raw/articles/snowflake-cortex-update.md — process it: read it, write a source summary page, update relevant concept and entity pages, and add an entry to the log
```

```
query the wiki: what do we know about Cortex Code pricing? synthesize from existing wiki pages and cite sources
```
```
the wiki page for vault/wiki/concepts/cortex-code-cli.md is getting stale — update it based on everything in vault/raw/ from the last 30 days
```

---

## pattern 9: bidirectional sessions

less common but powerful: start with vault context, do work, update vault, continue:

```
read vault/project/architecture/pipeline-design.md, look at the actual table structures in Snowflake for MY_SCHEMA, and update the architecture note to reflect what's actually built vs what was designed
```

```
check vault/project/decisions/ for any ADRs about Snowpipe configuration, then check our actual Snowpipe setup, and flag any drift between decisions and implementation
```

---

## notes on file paths

- paths are relative to the vault root, not your filesystem
- if you're not sure about a path, ask: "list files in vault/project/tickets/" and pick from the results

---

## what doesn't work well

- **large-scale restructuring**: asking the agent to reorganize hundreds of notes in one session is slow and error-prone.
- **real-time sync**: if you edit a note manually while the agent is also editing it, you'll get conflicts. coordinate: either you're editing, or the agent is.
- **Obsidian plugins and views**: the MCP connection is to the REST API, not the Obsidian UI. the agent can't interact with graph view, canvas, or plugin-specific features.
- **Obsidian closed**: the Local REST API plugin only runs when Obsidian is open. if the app is closed, MCP calls will fail silently or with an error.

---

## references

- Cortex Code CLI extensibility docs: https://docs.snowflake.com/en/user-guide/cortex-code/extensibility
- karpathy's LLM wiki gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- mcp-obsidian: https://github.com/MarkusPfundstein/mcp-obsidian