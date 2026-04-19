# Changelog

All notable changes to `bx-AISentinel-demo` are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.0.0] - 2026-04-19

First non-pre release. The demo now installs cleanly from a fresh clone, walks the user through Tier 0 → Tier 1 → failure-mode validation per `End-User-Testing-Path.md`, and renders the v1.0.0 surfaces of both `bx-AISentinel` (core middleware) and `bx-AISentinel-ONNX` (Tier 1 NER sibling).

### Added

- Help page at `/help/tier1-ner` walks new users through the four-step Tier 1 install (module install → 3 JARs → Piiranha model → boxlang.json wiring). Mirrors the testing-path doc verbatim.
- Sidebar links: Dashboard, Chat, Run tests, Tier 1 NER setup, BoxLang AI Docs, OpenRouter Models, bx-AISentinel Repo.
- Chat toolbar Tier 1 indicator (loaded / degraded / not-installed states) computed from the sentinel's `getLoadReport()` on mount.
- Dashboard "Tier 1 NER is active" callout when the detector has fired at least once in the session.
- Inspector panel showing the exact tokenized payload sent to the LLM vs the restored reply.
- Per-reply timing badge (Sentinel overhead vs LLM round-trip, color-coded).
- Master Sentinel toggle (live, no agent rebuild required) + token-coaching toggle.
- Seeded prompts including the "Free-form PII scenario" that demonstrates the Tier 1 gap on Tier 0 alone, then the Tier 1 fill once the ONNX module is installed.

### Fixed

- Send button now enables when the textarea has content. Prior version rendered a server-side `disabled` attribute based on `data.draft` being empty; `wire:model="draft"` updated server state without triggering a re-render, so the disabled attribute stayed in the DOM and only the Enter key worked. Added an `oninput` handler that updates the button's disabled state in real time.

### Known model behavior (Tier 1 / Piiranha)

Documented in `End-User-Testing-Path.md` "Open items" and the help page:

- First names leak; surnames get caught (Piiranha I-SURNAME accuracy is high; I-GIVENNAME is weak)
- MRN-style identifiers leak (not in Piiranha's label schema)
- Email addresses fragment across I-EMAIL + I-USERNAME labels (Tier 0 regex covers full emails reliably)
