# Workflow benchmarking

Plan and evaluate four agent workflows implemented with Hermes, comparing task
completion, reliability, cost, and execution time.

Development and planning take place on `dev`. The initial diagrams are preserved
in Git history on `master`.

## Project layout

```text
docs/
  workflows/           ASCII diagrams for configurations A, B, C, and D
  planning/            Benchmark design, decisions, and implementation plan
configs/
  workflows/           Versioned Hermes workflow configurations
benchmarks/
  tasks/               Task definitions, fixtures, and acceptance criteria
  evaluators/          Independent scoring and correctness checks
src/                   Benchmark runner and Hermes integration
tests/                 Tests for the runner and evaluation infrastructure
results/               Local generated run outputs (ignored by Git)
```

Start with [the benchmark plan](docs/planning/benchmark-plan.md) and
[the decision log](docs/planning/decisions.md).

This repository currently contains diagrams and a planning scaffold. No benchmark
runner is implemented, no experiments have been run, and model names in the
diagrams remain proposed configuration labels pending provider verification.

Keep credentials in local environment variables or an ignored `.env` file.
Never commit API keys or raw runs containing sensitive information.
