# PR 02 (0.2): Scaffold project

- **Phase:** 0 (Smallest working agent)
- **Step:** 0.2 code, checked by 0.3 run
- **Branch:** `02-phase-0-scaffold-project`
- **PR:** https://github.com/shadmaanAsif/Agent-Harness-Engineering/pull/2
- **Status:** open

## Goal
Create the empty TypeScript project so later steps have somewhere to put code, and keep the API key out of git.

## What changed
- `package.json`: ES module project; scripts `typecheck` and `phase0` (`phase0` loads `.env` through Node's `--env-file`, so no dotenv package); dependencies declared so `npm install` installs everything.
- `tsconfig.json`: strict TypeScript, `NodeNext` modules, compiles only `src/`.
- `.env.example`: `LLM_BASE_URL` (OpenRouter), `LLM_API_KEY`, `MODEL`. The real `.env` is gitignored.
- `package-lock.json`: pins exact versions so installs are repeatable.
- `README.md`, `docs/prs/_TEMPLATE.md`: branch naming rule `<NN>-phase-<P>-<slug>`.

## How the pieces connect
Step 0.4 adds `src/phase0/agent.ts`, which `npm run phase0` executes with values from `.env`. The `openai` package talks to OpenRouter because `LLM_BASE_URL` points there.

## Failure drill (before the fix)
None for this step; the drills start at 0.9.

## Proof (after the fix)
```bash
node -v          # v26.13.0
npx tsc --version  # Version 7.0.2
npm ls --depth=0   # openai 7.54.0, typescript 7.0.2, tsx 4.23.15, @types/node 26.6.6
```
All three matched.

## Decisions
- **tradeoff:** `openai` package (types, less boilerplate) over plain `fetch` (zero dependencies, raw JSON visible). Revisit in Phase 1.
- **best practice:** secrets in `.env`, gitignored; `.env.example` committed with no values.
- **best practice:** commit `package-lock.json`.

## Things to verify later
- TypeScript 7 and `openai` 7 are new major versions; check for type errors when real code arrives.
- Free OpenRouter model names and limits change often.

## Checkpoint questions
Supports Phase 0 question 5 (why keep the file small) only indirectly; the main learning is setup.
