# CLAUDE.md

## Think Before You Code
1. **Understand** — What is the actual problem? What are the requirements?
2. **Explore** — Read relevant existing code first
3. **Plan** — Outline your approach before writing any code
4. **Validate** — Does the plan fit the existing architecture?
5. **Then code** — Only after steps 1-4 are clear

## Code Style
- Follow OOP principles: encapsulation, single responsibility, composition over inheritance
- Prefer small, focused classes and methods
- Use meaningful names that reveal intent

## Syntax Checking
After ANY code change:
1. Compile ALL affected assemblies before reporting completion
2. Fix all compilation errors before moving on
3. Run `dotnet build` (or equivalent) to verify

## Workflow
- Read existing code before modifying—match the project's patterns
- Make minimal, targeted changes
- One logical change per commit

## Before Marking Complete
- [ ] Code compiles without errors
- [ ] No obvious regressions introduced
- [ ] Changes are tested if tests exist

## Communication
- State what you're about to do before doing it
- If uncertain, ask rather than assume
- Report blockers immediately

## CSS: Global vs Local
**Global styles** (design system level):
- Typography, colors, spacing scales
- CSS variables/tokens
- Reset/normalize styles
- Shared utility classes

**Local styles** (component level):
- Scoped to individual components
- Use CSS modules, BEM, or scoped styles
- Never leak styles to parent or sibling components
- Component-specific overrides stay with the component

**Principle**: Global for consistency, local for isolation.


**Code Quality Requirements**

When writing or modifying code, adhere to these standards:

**Style & Idiom** — Write code that feels native to the language. Follow the project's established conventions. Prefer idiomatic patterns over clever or verbose alternatives. When no project style exists, follow the language community's accepted standards.

**Simplicity** — Choose the simplest solution that meets requirements. Avoid over-engineered abstractions, unnecessary indirection, premature optimization, and convoluted logic. If a straightforward approach works, use it.

**Efficiency** — Select appropriate algorithms for the problem scale. Question O(n²) when O(n) exists, redundant iterations, repeated computations, and wasteful allocations. Balance performance against readability—optimize only where it matters.

**Robustness** — Handle failure modes gracefully. Validate inputs at boundaries, catch exceptions meaningfully, provide actionable error messages, and fail fast when appropriate. Never silently swallow errors.

**Cleanliness** — Deliver production-ready code. Remove all scaffolding: debug prints, commented-out code, TODO placeholders, and temporary workarounds. Every line should earn its place.

**Readability** — Use consistent formatting, clear naming, and logical organization. Comments explain *why*, not *what*. Aim for self-documenting code; add documentation where intent isn't obvious.

**Housekeeping** — Clean up after yourself. Remove temp files, scratch scripts, debug code paths, and log statements. Delete or merge temporary branches. Ensure all untracked files are either added to version control or removed. The AI may have created the mess, but it's your repository now.

**Documentation** — Keep documentation and project artifacts current. Update READMEs, changelogs, API docs, and configuration examples when code changes affect them. These are easily overlooked in the momentum of development—don't let them drift.

**Summary:** Code should be compact, readable, documented, idiomatic, robust, and efficient—with nothing extraneous, no loose ends, and all artifacts in sync.