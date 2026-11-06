# Beads Issue Tracker

Apply this guidance in repositories that use Beads (`bd`). Follow explicit user, repository, and orchestrator instructions when they specify a different tracker or workflow. Do not introduce Beads solely because this global resource exists.

Read the repository's `.agents/skills/beads/SKILL.md` when present, otherwise use [the global Beads skill](/Users/jo3/.agents/skills/beads/SKILL.md). Run `rtk bd prime` when workflow context is missing or stale; hooks may already provide it. If it prints nothing, use `rtk bd where` to check for an active workspace.

## Workflow

1. Find unblocked work with `rtk bd ready --json`; inspect it with `rtk bd show <id> --json`.
2. Claim work atomically with `rtk bd update <id> --claim --json`.
3. Use Beads as the durable source of truth for shared project tasks, blockers, dependencies, and handoff state. Do not create competing Markdown TODO files or external trackers. Agent-local checklists are temporary execution aids.
4. Create discovered work with a `discovered-from:<parent-id>` dependency.
5. Close completed work with `rtk bd close <id> --reason="Completed" --json`.

```bash
rtk bd create "Issue title" --description="Context and required outcome" --type=task --priority=2 --json
rtk bd create "Discovered work" --description="Finding and required outcome" --type=bug --priority=1 --deps discovered-from:<parent-id> --json
rtk bd update <id> --priority 1 --json
```

- Types: `bug`, `feature`, `task`, `epic`, `chore`.
- Priorities: `0` critical, `1` high, `2` medium/default, `3` low, `4` backlog.
- Use `--acceptance` and `--design` to capture requirements; use `--validate` to check description completeness.
- Prefer `--json` for programmatic use. Use `bd update` flags instead of interactive `bd edit`.
- Use `bd remember` for persistent Beads project knowledge; do not create a competing project memory system. Follow the active harness's separate memory controls.
- Check `bd ready` before asking what to work on.

## Lifecycle and Sync

- `bd defer` and `bd supersede` manage deferred or replaced work.
- `bd stale`, `bd orphans`, and `bd lint` help find tracking problems.
- `bd human` flags decisions for a person; `bd formula list` and `bd mol pour` support structured workflows.
- Issue history lives in a local Dolt database. Writes auto-commit to Dolt history; remote sync uses `refs/dolt/data`. `.beads/issues.jsonl` is a passive export, not the sync protocol.
- See the [Beads sync model](https://github.com/gastownhall/beads/blob/main/docs/core-concepts/sync-concepts.md) and [quickstart](https://github.com/gastownhall/beads/blob/main/docs/getting-started/quickstart.md).

## Profiles and Completion

- **Conservative/default:** do not Git commit, Git push, or sync Dolt remotes without explicit authorization. Report changed files, validation, issue state, and proposed next commands.
- **Minimal:** use `bd prime` for detailed workflow context; retain the conservative Git policy.
- **Team-maintainer:** only when explicitly enabled by the repository, closing work may include quality gates, commits, and pushes. Current user instructions still take precedence.

At the end of a Beads implementation workflow, record remaining work, run checks appropriate to code changes, update issue status, and perform only the Git/sync operations authorized by the active profile or user. Report any blocked required operation with its exact command and error, then hand off the resulting state.
