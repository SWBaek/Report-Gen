# AGENTS.md

This is the repository-wide working guide for contributors and coding agents in Report-Gen.
Use it to preserve the product contract while the implementation and workspace structure evolve.

## Start Here

- The repository currently contains architecture, design, and governance documents only. The implementation layout in `docs/ARCHITECTURE.md` is planned, not an existing workspace; do not invent source paths or validation commands.
- Start with `docs/ARCHITECTURE.md` for accepted v1 boundaries and ADR requirements.
- Use `DESIGN.md` as the visual source of truth for generated reports, not as a complete desktop-application UI specification.
- Follow `.github/ISSUE_MANAGEMENT.md` before starting implementation work.

## Product Contract

- Report-Gen is a local desktop report generator for people who should not need to understand a general-purpose AI chat product or a terminal workflow.
- The primary product surface is a report-focused, chat-shaped UI. The application owns report sessions, workflow state, validation, artifacts, and rendering; a Provider does not own the product experience.
- Report-Gen accepts data expressed in natural language, evaluates and refines the request, delegates analysis to an LLM, optionally gathers evidence, and produces a consistent report from the result.
- Version 1 supports exactly two local Provider integrations: Codex App Server launched with `codex app-server`, and GitHub Copilot CLI in programmatic mode. These are internal integration and authentication boundaries, not user-facing product interfaces.
- Reuse the Provider's existing authenticated account or launch its official interactive sign-in flow. Do not accept, store, proxy, or document an API-key/BYOK flow, and never read Provider credential files directly.
- Provider-specific conversation, streaming, Skill, agent, and Hook capabilities may differ. Normalize only the product behavior that Report-Gen needs; do not pretend unsupported capabilities are equivalent.
- Version 1 emits a borderless HTML report. A4 and slide output are possible future formats, not current supported outputs.
- The LLM produces structured report data, such as JSON. It does not own final HTML, layout, or styling.
- Prompt refinement, evidence collection, composition, and review are explicit application workflow stages with validated inputs and outputs. Do not infer completion solely from conversational prose.
- A static, deterministic renderer turns the structured result into the report. Given the same valid structured input and design version, rendering must not depend on another LLM call.
- Report structure, section semantics, and visual hierarchy must remain consistent regardless of the submitted subject matter. Content may vary only within the supported structured fields.
- User-provided content must not cause clipping, overlap, hidden text, or unintended horizontal page overflow. Renderer work is incomplete until representative long-content and narrow-viewport cases have been checked.
- The visual source of truth for generated reports is the repository-root `DESIGN.md`, derived from verified LG Electronics corporate-identity sources. Keep renderer styling aligned with it. Do not silently treat report-layout rules as a complete desktop-application UI specification.

## Change Routing

Before editing, classify the change:

Follow `docs/ARCHITECTURE.md` for the accepted v1 desktop runtime, dependency direction, Provider boundary, workflow, output artifacts, and validation strategy. Record a new ADR before changing an accepted architecture decision.

1. **Desktop application** — keep privileged process execution, authentication coordination, local persistence, and file access outside the unprivileged renderer UI.
2. **Provider execution** — preserve both authenticated local Provider runtimes, prohibit API-key/BYOK product flows, and keep Provider-specific behavior from changing the report contract.
3. **Agent workflow** — change the application-owned stage contract and routing policy before changing Provider prompts, Skills, agents, or Hooks. Hooks are guardrails and lifecycle integrations, not the workflow engine or sole security boundary.
4. **Structured report contract** — preserve one Provider-neutral representation that the static renderer can consume. Treat field names, required sections, ordering rules, and validation behavior as a public contract once implemented.
5. **HTML rendering** — change the deterministic renderer or its inputs instead of asking the LLM to generate or repair presentation markup.
6. **Visual design** — follow `DESIGN.md` for generated reports. A report design change and its corresponding renderer change belong in the same change.
7. **Output formats** — keep v1 behavior HTML-only. Do not claim A4 or slide support until a separate renderer and its validation are implemented.

## Synchronization Rules

These rules apply only to implementation surfaces that exist in the repository. Do not create a missing surface solely to satisfy a synchronization list.

- When the structured report contract changes, update every existing provider prompt or adapter, validator, representative structured sample, renderer expectation, and contract-focused test in the same change.
- When provider invocation or output normalization changes, verify the shared structured contract against both Codex App Server and GitHub Copilot CLI.
- When an application workflow stage changes, update its Schema, Provider-specific projection, fixtures, persistence representation, progress events, and stage-focused tests together.
- When authentication changes, verify existing-session reuse, signed-out behavior, official interactive sign-in, cancellation, expiry, and redaction. Do not add API-key fallback behavior.
- When bundled Skills, agents, or Hooks change, keep their Provider-specific forms semantically aligned and verify that application validation still rejects invalid output independently of them.
- When layout or styling changes, update `DESIGN.md` if the design contract changed and review generated HTML with both typical and stress-case content.
- Treat rendered reports as generator output. Change the structured input, renderer, template, or design contract rather than hand-fixing a generated report.

## Validation Expectations

- Use the narrowest repository-provided check that proves the changed contract once validation commands exist.
- For desktop boundary changes, verify that the renderer UI has no direct Node.js, credential, child-process, or unrestricted filesystem access.
- For Provider changes, exercise both supported authenticated runtimes or state exactly which Provider remains unverified and why.
- For authentication changes, verify that the app never requests or persists an API key and never exposes tokens in logs, IPC, errors, or artifacts.
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
- When the first implementation establishes source paths, schemas, fixtures, tests, and runnable commands, update this guide with only the real entry points and validated commands.
- Update this guide when the supported providers, output formats, structured contract, rendering boundary, or design source of truth changes.
