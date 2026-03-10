---
name: motion-pharo-ast-patterns
description: Create MoTion patterns in Pharo to match FAST models. Use when a user asks to write or adapt MoTion patterns for TypeScript AST or XML AST. Route TypeScript work to MoTion.md + FASTTypeScript-MoTion.md, and XML work to MoTion.md + FASTXML-MoTion.md.
---

# MoTion Pharo AST Patterns

## Workflow

1. Identify the target AST domain from the user request.
2. Load base MoTion guidance from `references/MoTion.md`.
3. Load exactly one domain guide:
- TypeScript AST: `references/FASTTypeScript-MoTion.md`
- XML AST: `references/FASTXML-MoTion.md`
4. Build or revise the pattern in Pharo syntax.
5. Return the pattern and a short explanation of key selectors/operators used.

## Domain Routing

Use this routing consistently:

- If the request mentions TypeScript, JavaScript/TS source code, or FAST TypeScript nodes, use:
`references/MoTion.md` and `references/FASTTypeScript-MoTion.md`.

- If the request mentions XML, tags/attributes, or FAST XML nodes, use:
`references/MoTion.md` and `references/FASTXML-MoTion.md`.

If the request is ambiguous, ask whether the target is TypeScript AST or XML AST before writing the final pattern.

## Output Rules

1. Return valid Pharo/MoTion pattern code.
2. Keep the pattern minimal and composable.
3. Include assumptions when node types are inferred.
4. When asked to improve an existing pattern, preserve user intent and explain the delta briefly.

## References

Always treat these files as source of truth:
- `references/MoTion.md`
- `references/FASTTypeScript-MoTion.md`
- `references/FASTXML-MoTion.md`
