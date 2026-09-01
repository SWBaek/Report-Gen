# AGENTS.md

This is the repository-wide working guide for contributors and coding agents in Report-Gen.
Use it to preserve the product contract while the implementation and workspace structure evolve.

## Product Contract

- Report-Gen accepts data expressed in natural language, delegates analysis to an LLM, and produces a consistent report from the result.
- Version 1 supports exactly two authenticated CLI providers: Codex CLI and GitHub Copilot CLI. Use each CLI's existing authenticated session; do not introduce a separate API-key flow or silently reduce support to one provider.
- Version 1 emits a borderless HTML report. A4 and slide output are possible future formats, not current supported outputs.
- The LLM produces structured report data, such as JSON. It does not own final HTML, layout, or styling.
- A static, deterministic renderer turns the structured result into the report. Given the same valid structured input and design version, rendering must not depend on another LLM call.
- Report structure, section semantics, and visual hierarchy must remain consistent regardless of the submitted subject matter. Content may vary only within the supported structured fields.
- User-provided content must not cause clipping, overlap, hidden text, or unintended horizontal page overflow. Renderer work is incomplete until representative long-content and narrow-viewport cases have been checked.
- The visual source of truth must be a repository-root `DESIGN.md` derived from verified LG Electronics corporate-identity sources. Create and approve that document before treating report UI decisions as canonical; once it exists, keep renderer styling aligned with it.

## Change Routing

Before editing, classify the change:

Follow `docs/ARCHITECTURE.md` for the accepted v1 runtime, dependency direction, Provider boundary, output artifacts, and validation strategy. Record a new ADR before changing an accepted architecture decision.

1. **Provider execution** — preserve both authenticated CLI providers and keep provider-specific behavior from changing the report contract.
2. **Structured report contract** — preserve one provider-neutral representation that the static renderer can consume. Treat field names, required sections, ordering rules, and validation behavior as a public contract once implemented.
3. **HTML rendering** — change the deterministic renderer or its inputs instead of asking the LLM to generate or repair presentation markup.
4. **Visual design** — follow `DESIGN.md` once it exists. A design change and its corresponding renderer change belong in the same change.
5. **Output formats** — keep v1 behavior HTML-only. Do not claim A4 or slide support until a separate renderer and its validation are implemented.

## Synchronization Rules

- When the structured report contract changes, update every existing provider prompt or adapter, validator, representative structured sample, renderer expectation, and contract-focused test in the same change.
- When provider invocation or output normalization changes, verify the shared structured contract against both Codex CLI and GitHub Copilot CLI.
- When layout or styling changes, update `DESIGN.md` if the design contract changed and review generated HTML with both typical and stress-case content.
- Treat rendered reports as generator output. Change the structured input, renderer, template, or design contract rather than hand-fixing a generated report.
- When the first implementation establishes source paths, schemas, fixtures, tests, and runnable commands, update this guide with only the real entry points and validated commands.

## Validation Expectations

- Use the narrowest repository-provided check that proves the changed contract once validation commands exist.
- For provider changes, exercise both supported CLIs or state exactly which provider remains unverified and why.
- For structured-output changes, check accepted data as well as malformed, missing, or oversized field cases supported by the implemented validator.
- For renderer changes, inspect representative short, long, multilingual, and unbroken content at the supported viewport sizes. Confirm that content remains readable without overlap or unintended page-level horizontal scrolling.
- Do not report a command or visual check as completed unless it was actually run. Record any unvalidated surface in the handoff.

## Work Tracking

- Track ideas, plans, features, and bugs through GitHub Issues and the repository-linked `Report-Gen` Project.
- Follow `.github/ISSUE_MANAGEMENT.md` for triage, fields, priorities, and status transitions.
- Before implementation, ensure the Issue has a verifiable outcome. Link pull requests with `Closes #<issue-number>` when the change should close the Issue.

## Guide Maintenance

- Keep this root guide repository-wide and concise. Add nested `AGENTS.md` files only after a real subtree needs different ownership, generation, or validation instructions.
- Do not add tools, dependencies, frameworks, package managers, or commands to this guide until the repository or an explicit project decision establishes them.
- Update this guide when the supported providers, output formats, structured contract, rendering boundary, or design source of truth changes.
