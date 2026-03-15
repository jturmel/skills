---
name: manage-project-context-features
description: Use to keep a project's context's features up-to-date as they're built out and completed.
---

# Product Features Knowledge Base

This folder is the canonical business-feature map for AI agents and stakeholders working in this repository.

These docs are optimized for retrieval, navigation, and product understanding. They describe what the product does, who it serves, and the important business rules and constraints. They do not describe implementation details.

## Purpose

- Keep high-level, business-oriented feature knowledge current.
- Help LLMs quickly locate the right product capability feature spec.
- Keep docs useful for non-technical stakeholders.

## Audience

- Primary: LLMs/AI agents.
- Secondary: product, operations, and stakeholder humans.
- Not intended for engineering implementation details.

## How To Navigate

- Start at `.context/features/README.md`.
- Read `overview.md` files first, then follow child links to leaf specs.
- Use `Parent`, `Children`, and `Related Feature Specs` links to traverse the graph.

## What Belongs In Specs

- User-facing capabilities and workflows.
- Workflows from a business perspective.
- Business rules, constraints, permissions, and outcomes.
- High-level feature boundaries and relationships.

## What Does Not Belong

- Models, endpoints, classes, migrations, and schema details.
- Implementation steps, test plans, or engineering notes.
- Temporary brainstorming notes.

## Writing Rules

- Keep documents concise, skimmable, and durable.
- Use bullets; nested bullets are allowed when they improve clarity.
- Keep content business/end-user oriented, not technical.
- Prefer one focused capability per leaf spec.
- Update `overview.md` files when child specs are added or removed.
- The features should be written once all the work is finalized
  and has finished receiving code review.

## Standard Metadata Block

Each spec should include:

- `Purpose`
- `Audience`
- `Scope`
- `Parent`
- `Children` (if applicable)
- `Related Feature Specs`
- `Actors`
- `Aliases`
- `Keywords`
- `Status`

## AI Retrieval Guidance

- Treat `Aliases` as natural-language synonyms users might type in prompts.
- Treat `Keywords` as short lookup terms for fast feature routing.
- Add both product names and plain-language terms when useful.
- Prefer retrieval phrases such as "sign in", "on behalf", and "training stages".

## Naming Conventions

- Use kebab-case filenames.
- Name files after business capabilities.
- Avoid generic names such as `admin.md` or `profile.md`.

## Role Vocabulary

Use consistent role names across feature specs.

## Update Rules

- Update a feature spec when product behavior changes for users.
- Prefer updating existing canonical feature specs over creating near-duplicates.
- Split a feature spec when it starts covering unrelated capabilities.
- Nest feature slices under their parent domain.

