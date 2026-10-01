# Workflow operation

This document describes how the five proposed Hermes configurations operate.
See [the benchmark plan](../planning/benchmark-plan.md) for experiment design,
task selection, evaluation, and implementation planning.

The configurations are design proposals. Provider model identifiers, supported
reasoning efforts, and Hermes integration capabilities still need verification.

## Shared behavior

Each workflow receives a task, starting environment, allowed tools, and execution
limits. Agents investigate, produce the requested deliverable, and verify their
work using available tools. They return a deliverable and completion status;
unresolved requirements must be reported explicitly.

For delegated work, the orchestrator provides a bounded request, relevant context,
expected output, dependencies, and acceptance checks. Workers return their work,
supporting evidence, checks performed, and unresolved issues. Independent work
may run in parallel; dependent work runs sequentially. Delegation is on demand,
so a task need not activate every worker.

The benchmark evaluator checks the final result independently, outside the
workflow. Agent verification and claimed completion do not determine its score.
Attempt, time, and cost limits apply to the entire run, including workers and
repairs. Exact limits and escalation triggers remain to be specified before runs.

## A: single-agent baseline, Sol medium

[ASCII diagram](workflow_A_single_agent_medium.txt)

GPT-6.1 Sol at medium effort handles the entire task.

1. Understand the request and inspect relevant inputs.
2. Plan and execute the work using tools.
3. Verify the deliverable and make repairs within the configured limits.
4. Return the deliverable and status.

There are no worker handoffs or changes in reasoning effort. A establishes the
baseline for direct execution by one agent.

## B: strong single-agent baseline, Sol high

[ASCII diagram](workflow_B_single_agent_high.txt)

GPT-6.1 Sol at high effort follows the same execution sequence as A, handling
investigation, implementation, repair, and verification itself.

High effort is fixed for this configuration; it does not increase during the run.
An xhigh baseline would be a separately versioned variant. B compares a stronger
fixed reasoning setting with A's medium setting without introducing delegation.

## C: multi-agent workflow with delegation on demand

[ASCII diagram](workflow_C_multi_agent.txt)

GPT-6.1 Sol at medium effort orchestrates and integrates the work.

| Role | Proposed model and effort | Responsibility |
| --- | --- | --- |
| Orchestrator and verifier | GPT-6.1 Sol, medium | Route, execute directly when useful, integrate, and verify |
| Research worker | DeepSeek V4.1 Flash, thinking | Bounded research with evidence and sources |
| Coding worker | DeepSeek V4.1 Flash, thinking | Implementation, debugging, and tests |
| Lightweight worker | GPT-6 Luna, low | Writing, summaries, and mechanical tasks |

1. Sol inspects the task and decides whether delegation is useful.
2. Sol executes directly or selects the workers needed for bounded subtasks.
3. Workers execute with explicit inputs and return outputs and verification evidence.
4. Sol integrates the outputs, checks the combined result, and repairs within limits.
5. Sol returns the deliverable and status.

Sol remains at medium effort throughout. C introduces delegation without an
effort escalation ladder.

## D: multi-agent workflow with Sol escalation

[ASCII diagram](workflow_D_multi_agent_escalation.txt)

D uses C's routing, workers, and medium-effort integration path, then adds a
bounded repair path when the escalation trigger is met.

1. Execute and integrate using C's sequence.
2. Verify at medium effort and assess the configured escalation trigger.
3. If triggered, Sol at high effort diagnoses the problem, repairs, and verifies again.
4. If still unresolved, Sol at xhigh effort performs the final repair and verification attempt.
5. Return success when checks are satisfied; otherwise return failure or blocked
   status when the ladder or overall limits are exhausted.

D's proposed ladder is medium -> high -> xhigh. Trigger definitions must specify
when unresolved checks warrant escalation rather than ordinary repair or an
immediate blocked result.

## E: experimental full GPT framework

[ASCII diagram](workflow_E_full_gpt.txt)

E assigns planning, execution, and final integration to distinct GPT roles.

| Role | Proposed model and effort | Responsibility |
| --- | --- | --- |
| Main planner and orchestrator | GPT-6 Astra, effort to be specified | Plan and select workers on demand |
| Research and scaffold worker | GPT-6 Sol, effort to be specified | Light research and code scaffolding |
| Plan executor | GPT-6 Luna, medium | Execute a bounded plan |
| Data analysis worker | GPT-6 Luna, xhigh | Perform bounded data analysis |
| Integrator and verifier | GPT-6.1 Sol, initially medium | Integrate, verify, and perform escalated repairs |

1. Astra inspects the task, creates a plan, and selects only the needed workers.
2. Sol provides research or scaffolding when requested.
3. Luna executes by plan at medium effort or performs data analysis at xhigh,
   according to the assigned role. These are distinct effort settings for distinct
   task types, rather than a single fixed Luna setting.
4. Sol 6.1 receives the outputs and relevant planning context, integrates the work,
   and verifies at medium effort.
5. If the escalation trigger is met, Sol 6.1 repairs and verifies at high effort.
   Each failed repair and verification advances one level: high -> xhigh -> max.
6. Stop at verified completion or the attempt, time, or cost limit. If verification
   still fails at max, report failure or blocked status rather than retry indefinitely.

If Luna needs Sol's scaffold, Sol must finish that output before Luna executes.
The two Luna roles need not both run for a task. Each handoff must carry the
requirements and evidence needed by the receiving agent.

Luna's executor and analysis roles are hypotheses to evaluate. Verify medium and
xhigh support before creating executable configurations. Astra and Sol effort
settings, escalation triggers, and attempt budgets remain open design choices.

## Configuration differences

| Comparison | Main change |
| --- | --- |
| A vs B | Medium versus fixed high effort for a single agent |
| A vs C | On-demand delegation under a medium-effort Sol orchestrator |
| C vs D | Addition of high/xhigh repair escalation |
| A-D vs E | Different planner, worker models, role allocation, and escalation ladder |

E is a complete-system alternative. Attributing its performance to a particular
role or model requires controlled variants, as described in the benchmark plan.
