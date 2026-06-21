# ConfigAtlas

**Mapping the hidden topology and static lineage of declarative configurations.**

## The Core Idea

Most tooling treats a workflow file, pipeline definition, or proxy config as a linear script to be read top to bottom and executed. This project treats it as a **topographical map** — a structural data structure with explicit and implicit relationships.

By parsing a declarative file into nodes (jobs, steps, fields) and edges (dependencies, data flow), ConfigAtlas surfaces the structural network of your configuration. This allows you to treat static YAML as queryable data to answer critical architectural questions before runtime execution:

- Which fields are shared across multiple jobs or workflows?
- If this field changes, what is the topological blast radius — what else depends on it?
- Are there fields used inconsistently across branches or environments?
- What does this file actually *contract* — independent of what the underlying code does?

## Why This Works (and Where It Doesn't)

This approach is built for **declarative, statically defined structures** — files that describe state rather than execute logic line by line. GitHub Actions workflows, GitLab CI pipelines, and proxy/gateway configs (e.g. Kong-style declarative configs) all fit this mold.

It deliberately does *not* try to understand what a job's underlying code does. That's a different, harder problem — and one suited to a separate semantic layer (see Future Direction below).

It also doesn't work well for **dynamic** configuration that's only resolved at runtime — if the values you care about don't exist until execution, you'll probably need a different capture strategy.

## Architecture

1. **Parse** — Read declarative files (YAML or similar) and extract structural elements: jobs, inputs, outputs, dependencies, proxy names/paths, or other declared structural fields, depending on the system.
2. **Store** — Persist parsed structure in a document-oriented store, decoupled from the live files. This means analysis can run on a schedule, off-hours, without needing to track every change in real time.
3. **Graph** — Layer a lightweight graph structure on top, where nodes are jobs/fields and edges are the relationships (data flow, shared fields, cross-job dependencies).
4. **Query** — Use the graph to answer structural questions: field lineage, common fields across workflows, blast radius for a given change.

Each layer does different work: the parser determines which kinds of files this approach works well on, the store decouples analysis timing from live changes, and the graph produces insights you wouldn't get by reading the file directly.

## Worked Examples

This repo demonstrates the pattern using:
- **GitHub Actions** workflows (lightweight, public, easy to follow)
- **GitLab CI** pipelines (showing the pattern generalizes across platforms)

The same approach extends naturally to any declarative system — infrastructure-as-code, API gateway configs, service mesh definitions — anywhere structure is declared rather than computed at runtime.

## Future Direction

The structural layer above can't tell you what a job's code is *doing* — only how it's connected. A natural next step is a semantic layer: summarizing job logic (potentially via an LLM) into topic labels, turning unstructured code into searchable metadata alongside the structural graph.

## Status

This is an exploratory pattern repo, not a production tool. It exists to demonstrate a way of thinking about declarative configuration that's been genuinely useful in practice — surfacing insights that aren't visible from reading the files alone.
