# dotnet-orchestration

This repository defines a small multi-agent orchestration setup for GitHub Copilot in VS Code.

The main agent is an Orchestrator that does not implement features itself. Its job is to take a user request, break it into phases, delegate the work to specialist agents, coordinate execution order, and verify that the final result hangs together.

## What The Orchestrator Does

The Orchestrator is responsible for:

- turning a high-level request into an execution plan
- delegating planning work to a Planner agent
- splitting work into phases based on file overlap and dependencies
- running independent tasks in parallel when they do not touch the same files
- routing implementation work to the right specialist
- validating that the combined output is coherent before reporting back

Just as importantly, it has a hard constraint: it should not do the implementation itself.

## Execution Model

The workflow is intentionally strict:

1. The Orchestrator sends the user request to the Planner.
2. The Planner returns an implementation plan with steps and file assignments.
3. The Orchestrator groups those steps into phases.
4. Tasks with no overlapping files can run in parallel.
5. Tasks that depend on each other, or touch the same files, run sequentially.
6. The Orchestrator verifies the combined result and reports completion.

This makes the setup useful for larger changes where a single agent would otherwise mix planning, coding, and design work in one pass.

## Specialist Agents

### Orchestrator

The top-level coordinator.

- plans the execution flow
- delegates work to subagents
- manages sequencing and parallelism
- prevents file conflicts
- validates the final output

### Planner

The research and planning agent.

- explores the codebase
- checks relevant documentation when needed
- identifies edge cases and uncertainties
- returns ordered implementation steps

### DotNet Coder

The implementation agent for .NET work.

- writes and updates production-oriented .NET code
- favors simple, maintainable solutions
- follows clean architecture and strong separation of concerns

### Designer

The design-focused agent.

- handles UI and UX direction
- focuses on usability, accessibility, and visual quality

## Why This Repo Exists

This setup is useful when you want a more disciplined agent workflow than a single general-purpose coding agent can provide.

It separates responsibilities clearly:

- planning happens in one place
- implementation happens in a specialist
- design work is isolated when needed
- orchestration logic controls coordination and conflict avoidance

That separation makes it easier to scale multi-step tasks, especially when different parts of the work should happen in parallel or require distinct expertise.

## Repository Layout

```text
.github/
  agents/
    orchestrator.agent.md
    planner.agent.md
    dotnet-coder.agent.md
    designer.agent.md
  skills/
    dotnet-logging/
      SKILL.md
```

## Included Skill

The repository also includes a `dotnet-logging` skill.

That skill provides guidance for .NET logging work, including:

- structured logging with `ILogger`
- log level selection
- exception logging
- correlation and scope handling
- avoiding noisy or sensitive logs

It supports the implementation agent when a task involves observability or logging changes.

## In Practice

For a complex request, the intended flow looks like this:

1. A user asks for a feature or cross-cutting change.
2. The Orchestrator asks the Planner for a structured plan.
3. The Orchestrator determines which tasks can run in parallel.
4. Work is delegated to the DotNet Coder and, when relevant, the Designer.
5. The Orchestrator reviews the combined outcome and returns a final status.

The result is a controlled multi-agent workflow focused on planning first, specialist execution second, and coordination throughout.