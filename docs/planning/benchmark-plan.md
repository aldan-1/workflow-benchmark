# Benchmark planning

Status: draft for discussion. No test suite, budgets, or routing policy has been
finalized.

## Objective

Determine whether the proposed Hermes workflows improve independently verified
task completion enough to justify their execution cost and time.

## Configurations

| ID | Workflow | Diagram |
| --- | --- | --- |
| A | Single agent, Sol medium | [A](../workflows/workflow_A_single_agent_medium.txt) |
| B | Single agent, Sol high | [B](../workflows/workflow_B_single_agent_high.txt) |
| C | Sol medium with delegation on demand | [C](../workflows/workflow_C_multi_agent.txt) |
| D | C with Sol high/xhigh escalation | [D](../workflows/workflow_D_multi_agent_escalation.txt) |
| E | Full GPT framework with Astra orchestration and bounded Sol 6.1 escalation | [E](../workflows/workflow_E_full_gpt.txt) |

### Workflow E: experimental full GPT framework

- GPT-6 Astra: main planner and orchestrator, selecting workers on demand.
- GPT-6 Sol: light research and code scaffolding.
- GPT-6 Luna at medium effort: execution by plan.
- GPT-6 Luna at xhigh effort: data analysis.
- GPT-6.1 Sol: integration and verification, starting at medium effort.

Run dependent work sequentially: Luna must receive Sol's scaffold before executing
when its task depends on that output. Define worker inputs, deliverables, and
acceptance checks explicitly to make handoffs inspectable.

When the escalation trigger is met, Sol 6.1 repairs and verifies at the next
effort level: medium -> high -> xhigh -> max. Increase effort by one level after
each failed repair and verification. Stop at success or the predefined attempt,
time, or cost limit; exhaustion produces an explicit failed or blocked result.
The exact triggers and limits remain to be specified before any run.

Treat E as a complete-system comparison against A-D. It changes the planner,
worker models, role allocation, and escalation policy simultaneously, so results
cannot attribute an improvement to any single change without controlled variants.
Include planning, handoffs, workers, and repair attempts in total cost and runtime.

Verify provider model identifiers and supported effort levels before implementing
E, including Luna medium and xhigh support. Its executor and analysis roles are hypotheses
to evaluate using independent acceptance checks.

## Decisions to discuss first

1. Intended workloads: coding, research, writing, or a defined mixture.
2. Task sources: candidate repositories, historical issues, or purpose-built tasks.
3. Success criteria: required behavior, regression checks, and qualitative rubrics
   where deterministic checks cannot capture the requirement.
4. Comparison constraints: tool access, environment, context, time and cost caps,
   and whether to compare under equal budgets, operational defaults, or both.
5. Routing and escalation: allowed workers, concurrency, retry limits, triggers,
   and what evidence permits a workflow to stop.
6. Pilot size, repetitions, spending ceiling, and expansion criteria.

## Proposed design principles

- Run all configurations against the same versioned task inputs and starting state.
- Keep the evaluator separate from the agent's own verification.
- Freeze configurations before a comparison; record changes as new versions.
- Reset repository and session state between runs. Define a policy for memory,
  caches, network access, and dependencies to prevent unintended carryover.
- Keep hidden acceptance checks inaccessible to the agents.
- Define retries and escalation as part of the workflow. Include all attempts
  in cost and runtime totals, including failed and timed-out runs.
- Record setup/infrastructure failures separately from task failures.
- Distinguish provider model identifiers from descriptive diagram labels.
- Report uncertainty and task-level differences alongside aggregate results.

## Task definition to design

Each task will need an identifier, category, request, pinned starting state,
environment requirements, allowed tools, acceptance checks, and explicit limits.
Define expected behavior independently of a particular reference patch.

## Run record to design

Record task and configuration versions, repetition, model settings, final status,
evaluator results, claimed completion, time, token usage, pricing assumptions,
total cost, calls, delegations, escalation, and human intervention. Decide how
to redact and retain trajectories before collecting them.

## Implementation sequence

1. Agree on workload scope and success definitions.
2. Verify Hermes integration capabilities and provider configuration support.
3. Specify a small pilot task set and independent evaluators.
4. Implement configuration A and the runner end to end.
5. Add B, C, D, and experimental E with consistent logging and limits.
6. Run the pilot, investigate evaluator and infrastructure defects, then freeze
   a comparison suite and execute repeated trials.
7. Analyze results and decide whether to expand the suite or revise the workflow.
