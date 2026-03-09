---
name: code-auditor
description: Audit codebases for duplicate code, unused code, DRY violations, dependency bloat, and refactoring opportunities. Use when the user asks for a code audit, technical debt review, dead-code cleanup, dependency audit, consolidation plan, or maintainability review after major feature work or before refactoring.
---

# Code Auditor

Use this skill to analyze a codebase for duplication, dead code, dependency issues, and consolidation opportunities. Prefer conservative findings and make the next cleanup steps obvious.

## Core Responsibilities

1. **Detect duplicate code**
   - Scan for repeated logic patterns across files such as color mappings, formatters, validators, and API helpers.
   - Identify copy-pasted functions with minor variations.
   - Find redundant utility implementations that should be consolidated.
   - Detect repeated inline constants, magic numbers, and configuration values.
   - Look for similar component patterns that could be abstracted.

2. **Identify unused code**
   - Find exported functions or components that are never imported.
   - Detect dead code branches and unreachable logic.
   - Identify commented-out code blocks that should be removed.
   - Locate orphaned files with no imports.
   - Find unused variables, parameters, and type definitions.

3. **Audit dependencies**
   - Cross-reference dependencies against actual imports and runtime usage.
   - Identify packages that are installed but never used.
   - Find duplicate packages serving the same purpose.
   - Detect dependencies that belong in `dependencies` versus `devDependencies`.
   - Flag deprecated or unmaintained packages when evidence is available.

## Analysis Workflow

1. **Discovery**
   - Read the project structure to understand the codebase organization.
   - Identify the tech stack to contextualize findings.
   - Locate utility, helper, and shared-module directories.
   - Build a rough mental map of the import graph.

2. **Deep scan**
   - Search for common duplication patterns:
     - Color and theme mappings
     - Date and time formatting functions
     - Number and currency formatters
     - Validation logic and regex patterns
     - API response transformers
     - Error-handling utilities
     - String helpers
     - Type guards and type utilities
   - Search for similar function signatures and repeated inline implementations.
   - Compare local implementations against existing shared utilities before calling for abstraction.

3. **Usage analysis**
   - Trace each utility or helper across the codebase.
   - Identify exports with zero real consumers.
   - Find local functions that duplicate existing shared helpers.
   - Check for circular, redundant, or layered-through-barrel imports.

4. **Dependency review**
   - Read dependency manifests such as `package.json`, `pyproject.toml`, or equivalents.
   - Search the codebase for actual import, require, and dynamic loading patterns.
   - Cross-reference declared dependencies with observed usage.
   - Identify overlapping packages and likely removal candidates.

## Output Format

Present findings in this structure.

### Duplicate Code Found

For each duplication:
- **Pattern**: What is duplicated
- **Locations**: Files or code areas involved
- **Recommendation**: Proposed consolidation approach
- **Priority**: High, Medium, or Low based on maintenance risk

### Unused Code

For each unused item:
- **Type**: Function, Component, Variable, Type, or File
- **Location**: File path and relevant symbol or area
- **Confidence**: High or Medium based on certainty
- **Action**: Delete or investigate further

### Unused Dependencies

For each dependency:
- **Package**: Name and version when available
- **Type**: dependency or devDependency
- **Action**: Remove or verify usage

### Consolidation Opportunities

Close with a practical cleanup plan:
1. Create or strengthen shared utilities
2. Migrate duplicated call sites incrementally
3. Delete unused code in a safe order
4. Remove unused packages with explicit commands when possible

## Guidelines

- **Be thorough**: Read enough surrounding code to understand whether behavior is truly duplicated or unused.
- **Be conservative**: Only flag code as unused when the evidence is strong.
- **Consider edge cases**: Account for dynamic imports, conditional requires, reflection, and framework conventions.
- **Respect existing patterns**: Recommend refactors that fit the codebase style.
- **Prioritize safety**: Favor incremental cleanup steps over big-bang rewrites.
- **Check tests**: Test files may be the only consumers of some utilities.
- **Check tooling**: Build tools and config files may reference code that app code does not.
- **Call out uncertainty**: Mark likely false positives for manual review instead of overstating confidence.

## Common Pitfalls

- Do not flag framework-required exports such as page components or framework entrypoints.
- Do not assume barrel exports are unused just because the barrel itself is not imported directly.
- Do not recommend removing types that exist for tooling, contracts, or documentation without checking their consumers.
- Do not miss dynamic imports such as `import()`, `require()`, or string-based module loading.
- Do not overlook CSS or style imports that affect runtime behavior.
- Do not flag `proxy.ts` as dead code in modern Next.js projects simply because `middleware.ts` used to serve that role.

## When Uncertain

If an item cannot be verified confidently:

- State what you verified and what remains uncertain.
- Recommend manual verification steps before deletion.
- Suggest tooling that can improve confidence, such as `knip`, `depcheck`, or `ts-prune`.
- If the uncertainty is ecosystem-specific, check the relevant documentation before making a removal recommendation.
