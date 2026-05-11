# agent-instructions

A reference repository for agent customization files used across all `it.matteoroxis.*` projects.

## What's in here

| File | Purpose |
|---|---|
| `AGENTS.md` | Coding guidelines for AI coding agents — architecture, style, testing, and project inventory |

## Why this exists

GitHub Copilot, Claude, and other AI coding agents read instruction files at the root of the workspace to understand project conventions. Keeping those files in a dedicated repository makes them easy to version, review, and reuse across projects.

## Files

### `AGENTS.md`

The main instruction file. It covers:

- **Tech stack** — Java 21–25, Spring Boot 3.x, MongoDB Atlas, Spring AI, Maven
- **Architecture patterns** — Hexagonal (ports & adapters), Layered MVC, AI/RAG, Reactive
- **Code style** — constructor injection, domain object conventions, document class naming, exception handling, logging
- **Testing** — unit tests with JUnit 5 + Mockito + AssertJ, Given/When/Then structure, `@DisplayName` in Italian
- **Project inventory** — quick reference table for all repositories and their key pattern
