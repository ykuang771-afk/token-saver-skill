---
name: token-saver
description: Orchestrate engineering implementation when the user wants the currently selected main model to plan, design, review, and advise while GPT-5.6 Luna performs the implementation; the main model takes over work Luna cannot complete.
---

# Current-model planner, Luna executor

Use the model already selected for the main thread as the controller. Do not replace it with a hard-coded planning or review model. The main thread owns repository inspection, requirements interpretation, architecture, the implementation plan, risk decisions, review feedback, integration, and final acceptance.

Use `gpt-5.6-luna` for implementation work. The main thread must not start editing merely to save delegation overhead; it may edit only after Luna is unavailable, reports that it cannot complete the bounded work, or fails the correction gate below. Pure planning, review, explanation, or diagnosis tasks need no Luna executor because they contain no implementation phase.

The user's latest instructions always take precedence over this workflow. This skill does not grant permission for deployments, destructive operations, external messages, purchases, or other actions outside the user's authorized scope.

## Workflow

1. Inspect the relevant repository state and existing user changes. Before delegation, capture a baseline for the writable scope: use version-control status and diffs plus the relevant file list when available; otherwise record a file inventory and hashes for existing writable files.
2. In the main thread, resolve routine ambiguity, choose the design, identify risks, define acceptance criteria, and write a concrete implementation plan.
3. Read [references/luna-task-packet.md](references/luna-task-packet.md), then delegate each implementation packet with `collaboration.spawn_agent` using:
   - model `gpt-5.6-luna`;
   - reasoning effort `high` as the target;
   - `fork_turns: "none"` and a self-contained prompt so the explicit model override is honored.
4. Keep delegation internal to the current request. Do not create a user-owned Codex thread unless the user separately asks for one.
5. Wait for Luna to finish, then compare the actual files with the pre-delegation baseline. In a version-controlled workspace, compare status, staged and unstaged diffs, and the relevant file list. Outside version control, regenerate the inventory and hashes and compare them with the saved values. Identify every Luna-created change, any touched pre-existing user change, and any write outside the packet. Run validation appropriate to the risk and compare the result with the acceptance criteria; do not accept the worker summary as proof.
6. If the implementation is close but defective, send Luna one focused correction packet containing the observed failure, evidence, required change, writable scope, and exact validation.
7. If Luna cannot complete the work or the correction still fails, the current main model takes over the unresolved implementation, preserves completed correct work, and verifies the integrated result.
8. Report the implementation split, review findings, corrections, any main-model takeover, validation results, and remaining limitations.

## Delegation rules

- Use `collaboration.spawn_agent` with `model: "gpt-5.6-luna"`, `reasoning_effort: "high"`, and `fork_turns: "none"`; include all necessary context in the task packet. A representative call is:

  ```text
  spawn_agent({
    task_name: "luna_implementation",
    fork_turns: "none",
    model: "gpt-5.6-luna",
    reasoning_effort: "high",
    message: "<self-contained Luna implementation packet>"
  })
  ```

  Use a unique task name when more than one worker is needed. Target `high` effort for Luna; if `high` is unavailable, keep the Luna model, use Luna's highest available effort, and disclose the effort change. If the explicit Luna model override is unavailable, rejected, or cannot be confirmed by runtime activity, do not silently substitute another worker model; use the main-model takeover path.
- A single bounded implementation packet is the default. Use multiple Luna workers only for genuinely independent work with disjoint writable files and separately testable outcomes.
- Assign each writable file to exactly one active worker. The main thread owns integration and can change those files later only after the worker has finished or been interrupted.
- Luna implements approved decisions; it does not choose the overall architecture, redefine product requirements, approve security or data-integrity tradeoffs, or declare final acceptance.
- Do not send secrets, unrelated private data, or unnecessary context to a worker.
- If the post-run comparison shows an out-of-scope write or damage to a pre-existing user change, stop further delegation and preserve the evidence. The main model must repair only the attributable worker changes with targeted edits; never use a broad reset or discard an ambiguous user change. If attribution is genuinely ambiguous, stop recovery and ask the user how to proceed.

## Main-model takeover gate

Treat Luna as unable to complete a packet when any of these is true:

- the Luna model or internal subagent tool is unavailable, rejects the requested model, or runtime activity cannot confirm that the worker is `gpt-5.6-luna`;
- Luna explicitly reports a blocker or returns without the required implementation;
- new evidence requires an architecture, security, privacy, authorization, destructive-migration, data-integrity, or breaking-compatibility decision outside the approved packet;
- required validation cannot be made available within scope;
- Luna's initial result and one evidence-based correction both fail the acceptance criteria.

Before taking over, record the concrete reason. Re-plan only the unresolved portion, then let the current main model edit and validate it. Do not restart correct Luna work from scratch.

## Truthfulness and completion

- Refer to the controller as the current main-thread model unless the runtime explicitly identifies its model.
- Claim that Luna ran only when agent activity or a tool result identifies `gpt-5.6-luna`. A configured or requested model name is not runtime proof.
- If Luna cannot be invoked, disclose that the current main model performed the implementation under the takeover rule.
- Do not claim success without inspecting the resulting artifacts and completing proportionate validation.
