# Agent Harness

A TypeScript project for learning agent harness engineering: the runtime around an LLM (loop, tools, context, memory, safety, verification, observability), built one phase at a time and taken to a production deployment.

- Full plan: [docs/LEARNING_PLAN.md](docs/LEARNING_PLAN.md) (page version: [docs/LEARNING_PLAN.html](docs/LEARNING_PLAN.html))
- Progress log: [docs/PROGRESS.md](docs/PROGRESS.md)
- Notes worth keeping: [docs/NOTES.md](docs/NOTES.md)

## Branching and PR rules

Every `code` step in the plan is one branch and one pull request. Branch names start with a two-digit sequence number that counts up across the whole repo, so the branch list sorts in the order the work happened:

```
<NN>-phase-<P>-<short-slug>
```

Examples: `01-init`, `02-phase-0-scaffold-project`, `03-phase-0-call-model-once`, `04-phase-0-tools-and-loop`. The next branch always takes the next free number, whatever phase it belongs to.

1. Branch from `main`. One step per branch; never commit directly to `main`.
2. Open one PR per branch into `main`. The PR title starts with the branch number and plan step, e.g. `02 (0.2) Scaffold project`.
3. Each PR has a record file at `docs/prs/phase-<P>/<NN>-<slug>.md`, copied from [docs/prs/_TEMPLATE.md](docs/prs/_TEMPLATE.md), holding the PR link and the details of what was done.
4. Each phase has an overview at `docs/prs/phase-<P>/README.md` that lists its PRs.
5. Add each PR record to the table below when the PR is opened.
6. Nothing is staged, committed or pushed without asking first.

## Phases and pull requests

| Phase | Topic | Overview | Code steps (one PR each) | Status |
|---|---|---|---|---|
| 0 | Smallest working agent | [docs/prs/phase-0](docs/prs/phase-0/README.md) | 0.2 ([PR #2](https://github.com/shadmaanAsif/Agent-Harness-Engineering/pull/2)), 0.4, 0.7, 0.10 | in progress |
| 1 | Agent loop | _not started_ | 1.2, 1.3, 1.4, 1.6, 1.7, 1.9 | planned |
| 2 | Tool design | _not started_ | 2.2, 2.4, 2.5, 2.6, 2.7 | planned |
| 3 | Context management | _not started_ | 3.2, 3.3, 3.5, 3.6, 3.8 | planned |
| 4 | Memory and state | _not started_ | 4.2, 4.4, 4.5, 4.6, 4.8 | planned |
| 5 | Safety and permissions | _not started_ | 5.2, 5.4, 5.5, 5.6, 5.7, 5.8 | planned |
| 6 | Verification loops | _not started_ | 6.2, 6.4, 6.5, 6.6 | planned |
| 7 | Orchestration | _not started_ | 7.3, 7.4, 7.5, 7.6 | planned |
| 8 | Evals and tracing | _not started_ | 8.2, 8.3, 8.4, 8.5, 8.7 | planned |
| 9 | Production hardening | _not started_ | 9.2, 9.4, 9.5, 9.6 | planned |
| 10 | Deployment | _not started_ | 10.2, 10.4, 10.5, 10.6, 10.9 | planned |

Phase overview links and per-PR links are added to this table as each phase is built. The step lists come from `LEARNING_PLAN.md` and may gain entries (for example 1.5 or 3.7) if a step turns out to need code.
