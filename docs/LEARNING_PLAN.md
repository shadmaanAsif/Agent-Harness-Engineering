# Agent Harness Engineering: Learning Plan

> Status: **REVISION 2, awaiting approval. No code written yet.**

## What changed in this revision

- **Every phase now has a plain-language explanation** (3-4 sentences: what it is, why it matters, how it builds on earlier phases).
- **Every phase has a topics checklist** and a short list of **new terms**, each defined the first time it appears.
- **Every phase is split into small numbered steps** (for example 3.4), each marked **explain**, **code** or **run**, so you always know whether you are listening, typing along, or running a command.
- **Failure-first order is built into the steps:** each phase shows the problem (a `run` step) before the fix (the next `code` step), then proves the fix.
- **New section 3** describes the code-along rhythm and the three step kinds.
- **Kept as before:** phase order, goals, failure drills, evals, checkpoint questions, hour estimates, proof commands, repo layout, assumptions.
- **Corrected earlier:** the phase hours add up to 68 h (about 12 weeks at 6 h/week), not 62 h.

## 1. Assumptions (please correct)

- **Time budget:** you left `[X] hours/week` and `[Y] weeks` blank. I assumed **~6 hours/week**, so the plan below (~68 hours) is about **12 weeks**. Each phase has its own hour estimate; tell me your real numbers and I will re-pace.
- **Environment:** Node **v26.13.0 (LTS, installed via nvm; pinned in `.nvmrc`)**, previously v20.19.5, Docker 24.0.6, empty repo (only `.claude/`). Not yet a git repo.
- **Language/stack:** TypeScript on Node. LLM provider: OpenRouter, a hosted service with an OpenAI-compatible API, called through the official `openai` npm package pointed at OpenRouter's base URL (*tradeoff*: free `:free` models cost nothing but have rate limits and varying tool-calling support; the `llm/` module is a thin wrapper, so switching to Ollama, Anthropic or another provider is a config change). Base URL, API key and model ID come from env vars (`LLM_BASE_URL`, `LLM_API_KEY`, `MODEL`), never hardcoded. **Verify:** free model names, rate limits, tool-calling support and SDK shapes change often; I will check OpenRouter's current docs before Phase 0 code.
- **You need:** an OpenRouter API key in `.env` (never committed) and a free model that supports tool calling. Evals may hit free-tier rate limits (or cost a little if you add credit); I will keep them small and add a request/cost cap (Phase 8). *Privacy:* free models may log prompts, so never send secrets.

## 2. Plain-language glossary (starter terms)

Each phase also defines its own new terms the first time they appear.

| Term | Meaning |
|---|---|
| **Agent** | A program that calls an LLM in a loop; the LLM decides which actions (tools) to take until the task is done. |
| **Harness** | Everything around the LLM: the loop, tools, context handling, memory, safety checks, logging. The LLM is the engine; the harness is the car. |
| **Tool** | A function the LLM can ask the harness to run (e.g. read a file). The LLM only emits a request; your code runs it. |
| **Context window** | The maximum text the LLM can see in one call. Everything (instructions, history, tool results) must fit. |
| **Eval** | An automated test for LLM behavior: fixed tasks + a scoring rule, run repeatedly, because outputs vary. |
| **Trace** | A recorded timeline of one agent run (each LLM call, tool call, timing, cost). |

## 3. How each lesson runs (code-along rhythm)

Every phase is a numbered list of steps. Each step has one of three kinds:

| Kind | What happens | What you do |
|---|---|---|
| **explain** | I teach one concept in plain language, with a small example. No files change. | Read, ask questions. |
| **code** | I explain the change first, then make one small diff (no unrelated edits). | Type it along with me or read the diff; ask questions. |
| **run** | I tell you the exact command and the output to expect. | Run it yourself and compare. Tell me if it differs. |

Rules for every step:

1. I **pause after each step** so you can type along, run it, and ask questions. I continue when you say so.
2. **Failure first:** the problem is shown in a `run` step before the `code` step that fixes it; then a `run` step proves the fix.
3. I advance to the next phase only when you say **"next"**.
4. I ask before installing dependencies, running destructive commands, deviating from the plan, and before any `git add`/commit/push.
5. Choices are labeled **tradeoff** (a judgment call with a real downside) or **best practice** (the usual right answer). Anything that may change soon is flagged **verify**.
6. Each phase ends with 3-5 checkpoint questions, then I update `docs/PROGRESS.md` and add short notes to `docs/NOTES.md`.

## 4. Final repo layout (target after Phase 10)

```
Agent-Harness/
├── docs/
│   ├── LEARNING_PLAN.md      # this file
│   ├── PROGRESS.md           # checklist + status log
│   └── NOTES.md              # save-worthy notes per phase
├── src/
│   ├── phase0/agent.ts       # frozen snapshot: the <150-line agent
│   ├── llm/                  # provider wrapper, retries, token/cost counting
│   ├── loop/                 # agent loop, stop conditions, step limits
│   ├── tools/                # tool registry, schemas, individual tools
│   ├── context/              # prompt assembly, truncation, summarization
│   ├── memory/               # session state, persistent memory store
│   ├── safety/               # permissions, sandboxing, input/output guards
│   ├── verify/               # verification loops, critics, test runners
│   ├── orchestration/        # planner, sub-agents, handoffs
│   ├── observability/        # structured logs, traces, metrics
│   ├── server/               # HTTP API (health, run endpoint)
│   └── cli.ts                # entry point for local use
├── tests/
│   ├── unit/                 # deterministic tests with a fake LLM
│   └── integration/
├── evals/
│   ├── tasks/                # eval cases (input + scoring rule)
│   ├── runner.ts             # runs cases, writes results
│   └── results/              # JSON outputs (gitignored except baselines)
├── drills/                   # one script per failure drill
├── deploy/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── monitoring/           # dashboards/alert definitions
├── .github/workflows/        # CI: lint, test, eval gate, deploy
├── package.json, tsconfig.json, .env.example, .gitignore
```

Phase 0 lives in one file on purpose. From Phase 1 the code is split into modules, one folder per job of the harness.

---

## 5. Phases

How the phases connect: the **loop** (0, 1) calls **tools** (2), whose results fill the **context** (3), which **memory** (4) saves, all inside **safety** limits (5), checked by **verification** (6), split across agents by **orchestration** (7), measured by **evals and tracing** (8), hardened for **production** (9) and shipped by **deployment** (10).

### Phase 0: Smallest working agent (~3 h)

**What it is.** An agent is a loop: send the model a task and a list of tools; if it asks for a tool, your code runs it, adds the result to the conversation, and asks again; when it replies with plain text, you are done. This phase builds only that, in one file under 150 lines with no frameworks, so nothing is hidden. It matters because every later phase is a better version of one piece of this loop. It builds on nothing; it is the foundation.

**Topics**
- [ ] Messages and roles
- [ ] Tool calling (the model requests, your code runs)
- [ ] The agent loop
- [ ] Max-step limit
- [ ] Working-directory sandbox for file tools
- [ ] API key kept out of git

**New terms**
- **Message:** one turn of the conversation, with a role (user or assistant) and content.
- **Tool call:** the model's structured request to run a named tool with given arguments; it is only text, nothing runs yet.
- **Tool result:** what your code sends back after running the tool.
- **Max-step limit:** a hard cap on loop rounds so a confused model cannot run forever.

**Steps**
- **0.1 explain:** what an agent is; messages; why a tool call is a request, not an action.
- **0.2 code:** scaffold `package.json`, `tsconfig.json`, `.env.example`, `.gitignore`. *Ask first* before installing `openai`, `typescript`, `tsx`, `@types/node`. *Best practice:* `.env` is gitignored.
- **0.3 run:** `node -v` and `npx tsc --version`. Expect Node v26.13.0 and a TypeScript version.
- **0.4 code:** call the model once and print the reply.
- **0.5 run:** `npx tsx src/phase0/agent.ts "Say hi"`. Expect a short greeting.
- **0.6 explain:** how a tool is described to the model (name, description, argument schema).
- **0.7 code:** add `read_file` and `list_dir`, and the loop. At this point the step cap is deliberately high (25) and file paths are unchecked, so the drill can fail.
- **0.8 run:** the proof command on a normal task. Expect logged tool calls, then a correct answer.
- **0.9 run (failure drill):** (a) a prompt that keeps the model calling tools, to show a runaway loop and its cost; (b) a request to read `../../etc/hosts`, which succeeds and shows the path escape.
- **0.10 code:** lower the cap to 10 and add a path check that rejects anything outside the working directory. *Best practice* for both.
- **0.11 run:** rerun the drills (now stopped and blocked), then `wc -l src/phase0/agent.ts` (expect under 150) and the proof command.
- **0.12 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 0.9. **Eval:** 3 manual tasks (e.g. "what is the name field in package.json?"), checked by eye; pass = correct answer in under 10 steps. Automated evals start in Phase 8.

**Checkpoint questions**
1. Who actually runs the tool, the model or your code?
2. What happens if the model never says it is done?
3. Why are tool results appended as messages?
4. What would you lose by dropping the step limit?
5. Why keep the file under 150 lines?

**Proof command:** `npx tsx src/phase0/agent.ts "What is the name field in package.json?"`

### Phase 1: The agent loop, done properly (~6 h)

**What it is.** Phase 0's loop works but cannot be tested without spending money, and it only has one way to stop. This phase turns it into a small module that always ends for a named reason and takes the model as a parameter, so tests can pass in a pretend model. It matters because you cannot improve what you cannot test. It builds directly on the Phase 0 loop and keeps its behavior.

**Topics**
- [ ] Wrapping the model call (`llm/`)
- [ ] Stop reasons
- [ ] Retries with backoff and timeouts
- [ ] Token usage and a budget
- [ ] Fake LLM for offline tests
- [ ] Unit tests with Vitest

**New terms**
- **Stop reason:** the named cause a run ended: `done`, `max_steps`, `budget`, `error`.
- **Fake LLM:** a stand-in that returns scripted replies, so tests need no network or money.
- **Dependency injection:** passing the model into the loop instead of the loop creating it.
- **Backoff:** waiting longer after each failed retry so you do not hammer a struggling API.
- **Token:** the unit the model reads and bills in, roughly three quarters of a word.

**Steps**
- **1.1 explain:** why Phase 0 is hard to test; stop reasons; dependency injection with a tiny example.
- **1.2 code:** an `LLM` interface and a real OpenRouter implementation (OpenAI-compatible) in `llm/`.
- **1.3 code:** `loop/` that takes an `LLM`, tools and limits, and returns a result with a stop reason and token usage.
- **1.4 code:** `FakeLLM` that plays back a scripted list of replies.
- **1.5 run (failure drill):** a fake that always asks for a tool (the run only stops at the step cap), and a fake that throws a 429 rate-limit error (the run crashes).
- **1.6 code:** retries with backoff and a timeout in the wrapper, plus a token `budget` stop reason. *Best practice.*
- **1.7 code:** Vitest tests, one per stop reason. *Ask first* before installing Vitest. *Tradeoff:* Vitest is fast and TypeScript-friendly; Jest also works. **Verify** current versions.
- **1.8 run:** `npm test -- loop`. Expect all tests passing in about a second, with no API calls.
- **1.9 code:** `cli.ts` entry point that runs a task from the command line.
- **1.10 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 1.5. **Eval:** unit tests assert each stop reason triggers; 5 scripted fake-LLM scenarios must match expected reasons.

**Checkpoint questions**
1. Why test with a fake LLM instead of the real one?
2. What are the stop reasons and who decides each?
3. Why retry with backoff rather than immediately?
4. What does a token budget protect you from?

**Proof command:** `npm test -- loop`

### Phase 2: Tool design (~6 h)

**What it is.** The model never sees your code, only each tool's name, description and argument shape. Those words are the tool's user interface, so writing them well, validating arguments, and returning clear errors decides whether the agent succeeds. It matters because most agent failures are tool failures. It builds on Phase 1: tools plug into the loop you already have, and errors flow back as ordinary tool results.

**Topics**
- [ ] Tool contract (name, description, schema)
- [ ] Argument validation
- [ ] Structured errors the model can act on
- [ ] Output size limits
- [ ] Allowlisting shell commands
- [ ] A first small eval and baseline

**New terms**
- **Schema:** a strict description of valid arguments: names, types, which are required.
- **Validation:** checking arguments against the schema before running anything.
- **Allowlist:** a list of what is permitted, with everything else refused.
- **Structured error:** an error returned as data the model can read and recover from.

**Steps**
- **2.1 explain:** the tool contract and why vague descriptions cause wrong calls.
- **2.2 code:** a tool registry (one place that lists tools and runs them) with deliberately loose schemas.
- **2.3 run (failure drill):** a vague description makes the model pass bad arguments; a tool returns a 5 MB file that blows the context; a tool throws a raw stack trace.
- **2.4 code:** Zod validation and a single success/error result shape. *Ask first* before installing `zod`. *Tradeoff:* one more dependency, much clearer errors.
- **2.5 code:** cap tool output size, with a note when something is cut.
- **2.6 code:** `write_file`, `run_shell` (allowlisted commands only) and `search_text`, alongside `read_file` and `list_dir`.
- **2.7 code:** 8 tool-use eval cases and a tiny runner (Phase 8 generalizes it).
- **2.8 run:** `npm test -- tools && npm run eval -- --suite tools`. Expect passing tests, then a pass rate such as 7/8 saved as the first baseline.
- **2.9 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 2.3. **Eval:** 8 cases, does the model pick the right tool with valid arguments; the pass rate is the first baseline.

**Checkpoint questions**
1. Why return errors as data instead of throwing?
2. How does a tool description affect behavior?
3. Why cap tool output size?
4. Which validation catches model mistakes vs. attacker input?

**Proof command:** `npm test -- tools && npm run eval -- --suite tools`

### Phase 3: Context management (~7 h)

**What it is.** The model can only read so much at once, and every message and tool result is re-sent on each call, so long tasks eventually overflow. Context management decides what to keep, shorten or drop, while making sure the task itself is never lost. It matters because without it long runs crash or silently forget their goal. It builds on Phase 2: tool outputs are the biggest thing that fills the context.

**Topics**
- [ ] Counting tokens
- [ ] Prompt assembly (system prompt, pinned task, history)
- [ ] Truncating old tool results
- [ ] Summarizing old turns
- [ ] Pinned content
- [ ] Prompt caching (check docs)

**New terms**
- **Context budget:** how many tokens you allow the prompt to use.
- **Truncation:** cutting old or large content short.
- **Compaction:** replacing old turns with a short summary.
- **Pinned content:** text never dropped, such as the task.
- **Prompt caching:** reusing an unchanged prompt prefix to save cost and time. **Verify** current provider behavior.

**Steps**
- **3.1 explain:** the context window, tokens and budgets, with a worked example of a run filling up.
- **3.2 code:** token counting before each call.
- **3.3 code:** prompt assembly: system prompt, pinned task, then history.
- **3.4 run (failure drill):** a 40-step task overflows the window; a second run drops early messages so the agent forgets its instructions.
- **3.5 code:** truncate old tool results first (cheapest saving).
- **3.6 code:** compaction: a model call that summarizes the oldest turns. *Tradeoff:* saves space but summaries can lose details.
- **3.7 explain:** prompt caching and any built-in context-editing features. **Verify:** these change; I check current docs and tell you what I found.
- **3.8 code:** long-task eval: a fact stated at step 1 must be used correctly at step 30, with and without compaction.
- **3.9 run:** `npm run eval -- --suite context`. Expect a table comparing correctness with and without compaction.
- **3.10 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 3.4. **Eval:** step-1 fact used correctly at step 30; pass = no drop in correctness with compaction.

**Checkpoint questions**
1. What gets dropped first and why?
2. What is the risk of summarizing?
3. Why pin the task?
4. How do you measure that compaction did not hurt quality?

**Proof command:** `npm run eval -- --suite context`

### Phase 4: Memory and state (~6 h)

**What it is.** State is where a run is right now; memory is what is kept for later. Saving state to disk lets you resume after a crash, and a small memory store lets the agent carry useful facts into a new session. It matters because real agents run long enough to fail midway. It builds on Phase 3: what you save must be small enough to reload into the context.

**Topics**
- [ ] What a run needs in order to resume
- [ ] Checkpointing after each step
- [ ] Resuming by run ID
- [ ] Avoiding repeated side effects
- [ ] Long-term memory store and memory tools
- [ ] Forgetting and correcting bad memory

**New terms**
- **Checkpoint:** a saved snapshot of a run you can resume from.
- **Short-term memory:** what is in the current conversation.
- **Long-term memory:** facts stored on disk for future sessions.
- **Idempotent:** safe to repeat without extra side effects.

**Steps**
- **4.1 explain:** state vs. memory; what belongs in a checkpoint; why repeating a side effect is dangerous.
- **4.2 code:** the run-state shape.
- **4.3 run (failure drill 1):** kill the process halfway and watch all progress vanish.
- **4.4 code:** save a checkpoint after each step and resume by run ID (JSON files on disk).
- **4.5 code:** mark side-effecting steps as done so resume does not repeat them.
- **4.6 code:** a memory store with save, search and forget tools. *Tradeoff:* files are simplest; SQLite adds querying later.
- **4.7 run (failure drill 2):** plant a wrong saved fact and watch the agent trust it.
- **4.8 code:** store source and timestamp with each memory, and add correction/forget.
- **4.9 run:** `npm run drill:crash-resume`. Expect it to kill a run, resume it, and print that both reach the same answer.
- **4.10 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 4.3 and 4.7. **Eval:** run, kill at step N, resume; pass = same result and no repeated side effects. Plus a memory-recall case across two sessions.

**Checkpoint questions**
1. What exactly is saved in a checkpoint?
2. How do you avoid re-running side-effecting tools on resume?
3. When should memory be forgotten or corrected?
4. Where can memory be poisoned?

**Proof command:** `npm run drill:crash-resume`

### Phase 5: Safety and permissions (~7 h)

**What it is.** The agent reads files and text that someone else wrote. If that text says "ignore your rules and delete everything", a naive agent may obey. Safety means the harness enforces limits in code, because asking the model nicely is not a security boundary. It matters because an agent with tools can cause real damage. It builds on Phase 2 (tools are what get restricted) and Phase 4 (memory is another place untrusted text can enter).

**Topics**
- [ ] Prompt injection
- [ ] Least privilege
- [ ] Allow / ask / deny permission levels
- [ ] Human approval prompts
- [ ] Path and command sandboxing
- [ ] Secret redaction
- [ ] Treating tool output as untrusted

**New terms**
- **Prompt injection:** instructions hidden in data the agent reads.
- **Least privilege:** give only the access a task needs.
- **Sandbox:** a restricted space where actions cannot reach the rest of the machine.
- **Human approval:** pausing so a person confirms a risky action.
- **Redaction:** removing secrets from logs and outputs.

**Steps**
- **5.1 explain:** how injection works, and why prompt wording cannot be the defense.
- **5.2 code:** injection fixtures: files that try to hijack the agent, in a throwaway temp directory.
- **5.3 run (failure drill):** the unprotected agent obeys a file telling it to run `rm` or read `.env`. Temp directory only; nothing real is touched.
- **5.4 code:** a permission policy: allow, ask or deny per tool.
- **5.5 code:** the approval prompt that pauses the run until you answer.
- **5.6 code:** path and command sandbox checks in code.
- **5.7 code:** label tool output as untrusted data, and redact secrets from logs.
- **5.8 code:** a safety eval suite of injection and escape attempts.
- **5.9 run:** `npm run eval -- --suite safety`. Expect every attack listed as blocked and 0 violations.
- **5.10 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 5.3. **Eval:** safety suite; target 0 policy violations (a hard gate in CI later).

**Checkpoint questions**
1. Why is prompt wording not a security boundary?
2. What does least privilege look like for each tool?
3. Where does untrusted text enter the context?
4. What should require human approval?

**Proof command:** `npm run eval -- --suite safety`

### Phase 6: Verification loops (~6 h)

**What it is.** Models say "done" with confidence even when they are wrong. A verification loop runs something objective, like tests or a type check, and feeds any failure back for another try, within a limit. It matters because it turns "looks right" into "was checked". It builds on Phase 1 (the loop gets an extra check step) and Phase 2 (verifiers run commands as tools).

**Topics**
- [ ] Ground truth vs. self-assessment
- [ ] Verifier interface
- [ ] Running tests as a verifier
- [ ] Schema checks
- [ ] Bounded retry with feedback
- [ ] LLM-as-critic and its limits

**New terms**
- **Verifier:** code that checks a result and returns pass or fail with a reason.
- **Ground truth:** an objective answer, such as a passing test.
- **LLM-as-critic:** a second model call that reviews the first. *Tradeoff:* cheap, but also fallible.
- **Bounded retry:** a limit on fix attempts so it cannot loop forever.

**Steps**
- **6.1 explain:** why a test result beats model confidence.
- **6.2 code:** a small set of buggy files, each with hidden tests.
- **6.3 run (failure drill):** the agent claims success but a hidden test fails.
- **6.4 code:** the verifier interface and a test-running verifier.
- **6.5 code:** a bounded retry loop that sends failure details back to the agent.
- **6.6 code:** a schema verifier; explain where an optional critic helps and where it fails.
- **6.7 run:** `npm run eval -- --suite verify`. Expect two pass rates side by side, with verification clearly higher.
- **6.8 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 6.3. **Eval:** buggy-file tasks with hidden tests; compare pass rate with and without the verify loop.

**Checkpoint questions**
1. Why is a test result better evidence than the model's confidence?
2. How many verify-retries are enough?
3. What can a critic model miss?
4. How do you stop infinite fix loops?

**Proof command:** `npm run eval -- --suite verify`

### Phase 7: Orchestration (~6 h)

**What it is.** Sometimes one agent is overloaded by a big job. An orchestrator makes a plan and hands pieces to worker agents, each with its own small context and limited tools, then combines the results. It matters because workers keep contexts small and independent pieces can run at the same time, but more agents also mean more cost and more ways to fail. It builds on Phase 3 (context isolation), Phase 5 (narrow permissions per worker) and Phase 1 (each worker is a loop).

**Topics**
- [ ] When one agent is enough
- [ ] Planner
- [ ] Workers with narrow tools and fresh context
- [ ] Handoff format
- [ ] Parallel independent subtasks
- [ ] Shared budget

**New terms**
- **Orchestrator:** the agent that plans and delegates.
- **Worker (sub-agent):** an agent that does one narrow piece.
- **Context isolation:** each worker sees only what it needs.
- **Handoff:** the format workers use to return results.

**Steps**
- **7.1 explain:** orchestrator/worker pattern, and when not to use it.
- **7.2 run (failure drill 1):** one agent on a 5-file task gets overloaded.
- **7.3 code:** the typed handoff result.
- **7.4 code:** a worker runner (a fresh loop with narrow tools).
- **7.5 code:** the planner that splits a task into subtasks.
- **7.6 code:** parallel execution and one shared budget.
- **7.7 run (failure drill 2):** a naive version loses a result and double-edits a file; then I add a rule that each worker owns its files.
- **7.8 run:** `npm run eval -- --suite orchestration`. Expect a single-vs-orchestrated comparison table.
- **7.9 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 7.2 and 7.7. **Eval:** summarize 5 files, single-agent vs. orchestrated, compared on quality, steps and cost.

**Checkpoint questions**
1. When is a single agent better?
2. What does a sub-agent get to see?
3. How do failures in one worker propagate?
4. How is total cost bounded?

**Proof command:** `npm run eval -- --suite orchestration`

### Phase 8: Evals and tracing (~7 h)

**What it is.** The same prompt can give different answers each time, so one good run proves nothing. Evals run many tasks many times and report a pass rate; traces record every step of a run so you can see exactly what happened. It matters because this is how you know a change helped, and it later lets CI block a change that makes the agent worse. It builds on the small eval runners from Phases 2 to 7, which become one general runner here.

**Topics**
- [ ] Non-determinism and trials
- [ ] Trace format and viewer
- [ ] Scoring: exact, programmatic, LLM-judge
- [ ] Baselines and regression reports
- [ ] Cost and latency per case
- [ ] Spend cap

**New terms**
- **Trial:** one run of one eval case; you run several because outputs vary.
- **Baseline:** the saved pass rate you compare against.
- **Regression:** a drop in quality compared to the baseline.
- **LLM-as-judge:** a model that scores outputs; useful but biased, so use sparingly.

**Steps**
- **8.1 explain:** non-determinism, trials, baselines.
- **8.2 code:** the trace format: one JSON line per event with a run ID.
- **8.3 code:** a trace viewer script that prints one run as a timeline.
- **8.4 code:** one general eval runner with repeated trials, replacing the small ones.
- **8.5 code:** scorers: exact, programmatic, and an LLM judge with its caveats.
- **8.6 run (failure drill):** quietly worsen a prompt; a single manual run looks fine while the eval pass rate drops.
- **8.7 code:** baseline file, regression report, cost and latency per case, and a spend cap.
- **8.8 run:** `npm run eval:all -- --trials 3`. Expect a combined report with any regressions marked.
- **8.9 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 8.6. **Eval:** the eval suite itself is the deliverable: all earlier suites run through one command with a combined report.

**Checkpoint questions**
1. Why run each case multiple times?
2. What is a regression and how is it detected?
3. When is an LLM judge acceptable?
4. What belongs in a trace?

**Proof command:** `npm run eval:all -- --trials 3`

### Phase 9: Production hardening (~6 h)

**What it is.** Real services face API outages, slow calls, floods of requests and shutdowns mid-run. Hardening makes each of those end in a defined, safe state instead of a hang or a mess. It matters because in production, failure is normal. It builds on Phase 4 (saved state makes safe shutdown possible), Phase 1 (retries and budgets) and Phase 8 (traces and metrics show what failed).

**Topics**
- [ ] HTTP service with a health check
- [ ] Config validation at startup
- [ ] Timeouts and cancellation
- [ ] Concurrency and cost limits
- [ ] Idempotency
- [ ] Graceful shutdown
- [ ] Structured logs with redaction

**New terms**
- **Timeout:** a time limit on a call.
- **Graceful shutdown:** finishing or saving current work before exiting.
- **Idempotency key:** a token so a retried request is not executed twice.
- **Health check:** an endpoint that reports whether the service is alive.
- **Chaos test:** deliberately injecting failures to see what happens.

**Steps**
- **9.1 explain:** the common failure modes and what "defined state" means.
- **9.2 code:** an HTTP server with `/healthz` and `/run`. *Tradeoff:* Node's built-in `http` module avoids a dependency; Express or Fastify add convenience.
- **9.3 run (failure drill):** simulate an API outage, slow responses and a flood of requests; watch for hangs, memory growth and double execution.
- **9.4 code:** config validation at startup, per-request timeouts and cancellation.
- **9.5 code:** concurrency and per-request/daily cost limits, retry rules for retryable errors, and idempotency keys.
- **9.6 code:** graceful shutdown on SIGTERM, and structured logging with redaction.
- **9.7 run:** `npm run drill:chaos`. Expect a summary of each run's end state, none stuck or duplicated.
- **9.8 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 9.3. **Eval:** N concurrent runs with injected faults; pass = all runs end in a defined state.

**Checkpoint questions**
1. What happens on SIGTERM mid-run?
2. Why a per-request budget?
3. Which errors are retryable?
4. What does a health check prove and not prove?

**Proof command:** `npm run drill:chaos`

### Phase 10: Deployment (~8 h)

**What it is.** Deployment packages the service in a container, tests it automatically on every change, blocks changes that lower eval quality, watches it live, and lets you roll back fast. It matters because it makes shipping repeatable and safe, and the eval gate catches regressions that normal tests miss. It builds on everything before it: Phase 8's baselines power the gate, Phase 9's service is what gets packaged.

**Topics**
- [ ] Multi-stage Docker image, non-root user
- [ ] docker-compose for local runs
- [ ] CI: lint, typecheck, tests
- [ ] Eval gate against the baseline
- [ ] Deploy target and secret store
- [ ] Metrics and alerts
- [ ] Versioned images and rollback

**New terms**
- **Container image:** a packaged, runnable copy of the app.
- **CI/CD:** automatic testing and shipping on every change.
- **Eval gate:** a CI check that fails when eval quality drops.
- **Rollback:** redeploying the previous good version.
- **Secret store:** the platform's safe place for keys.

**Steps**
- **10.1 explain:** containers, CI/CD, eval gates and rollback.
- **10.2 code:** multi-stage `Dockerfile`, small image, non-root user.
- **10.3 run:** `docker build` and run it. Expect the container to start and `/healthz` to answer.
- **10.4 code:** `docker-compose.yml` for local use.
- **10.5 code:** CI workflow: lint, typecheck, unit tests. *Ask first:* running it needs a GitHub repo and a push, which only happens when you say so.
- **10.6 code:** the eval gate that fails CI if the pass rate drops below baseline.
- **10.7 run (failure drill 1):** push a deliberately bad prompt change; CI should block it.
- **10.8 explain:** deployment targets. *Tradeoff:* because the model is hosted (OpenRouter), the harness is a small container with no GPU, so a single-container platform like Fly.io or Cloud Run is simpler; Kubernetes is more flexible. The API key is stored as a platform secret. You choose here.
- **10.9 code:** deploy config, secrets in the platform store, metrics and alerts (error rate, latency, cost), versioned image tags.
- **10.10 run (failure drill 2):** deploy a broken build to staging and roll it back.
- **10.11 explain:** checkpoint questions; update PROGRESS.md and NOTES.md.

**Failure drill:** see 10.7 and 10.10. **Eval:** full pipeline run on a branch; the green path deploys and the regression path is blocked.

**Checkpoint questions**
1. Why gate on evals, not just unit tests?
2. How do you roll back and how fast?
3. Where do secrets live?
4. Which three metrics would page you?

**Proof command:** `docker compose up --build -d && curl -s localhost:3000/healthz` locally, and a green CI run on the PR for the pipeline.

**Time total:** 3 + 6 + 6 + 7 + 6 + 7 + 6 + 6 + 7 + 6 + 8 = **68 h** (about 12 weeks at 6 h/week).

---

## 6. Progress checklist

(Mirrored and kept up to date in `docs/PROGRESS.md`.)

- [ ] Plan approved
- [ ] Phase 0: Smallest working agent
- [ ] Phase 1: Agent loop
- [ ] Phase 2: Tool design
- [ ] Phase 3: Context management
- [ ] Phase 4: Memory and state
- [ ] Phase 5: Safety and permissions
- [ ] Phase 6: Verification loops
- [ ] Phase 7: Orchestration
- [ ] Phase 8: Evals and tracing
- [ ] Phase 9: Production hardening
- [ ] Phase 10: Deployment

## 7. Open questions for you

1. Real hours/week and number of weeks?
2. OK to use OpenRouter (needs your free API key; free models are rate-limited, and a small credit balance may be needed for longer eval runs)?
3. OK to run `git init` and work on a feature branch (I will not stage or commit without asking)?
4. Deployment target preference for Phase 10 (can defer)?
5. Is any topic still unclear? Name it and I will revise before we start.
