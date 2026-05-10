# Agent Development Guide

This project includes a Senior Node.js/TypeScript Developer agent configuration to assist with development tasks.

## Agent Configuration

The agent configuration is located in `.agent/`:
- `.agent/prompts.md` — Role definition and project context
- `.agent/tasks.md` — Task templates for common operations
- `.agent/review-guidelines.md` — Code review checklist

## Using the Agent

Invoke the agent using the Task tool with `subagent_type: "explore"` (for research/analysis) or `"general"` (for implementation/writing).

### Example: Code Review

```
Use Task tool with:
- description: "Code review"
- prompt: "Review src/server.ts for code quality, TypeScript best practices, and potential bugs. Focus on the generateMidiBase64 function and MCP tool registration."
- subagent_type: "explore"
```

### Example: Bug Investigation

```
Use Task tool with:
- description: "Debug build error"
- prompt: "The build is failing with: [error]. Run npm run build and analyze the output. Identify the root cause and suggest a fix."
- subagent_type: "explore"
```

## Quick Commands

| Task | Command |
|------|---------|
| Full CI locally | `npm run ci` |
| Type check | `npm run typecheck` |
| Lint | `npm run lint` |
| Tests | `npm test` |
| Coverage | `npm run test:coverage` |
| Full build | `npm run build` |

## Agent Tasks

See `.agent/tasks.md` for pre-defined task templates including:
- Code review
- Bug investigation
- Feature implementation
- Test coverage analysis
- Build debugging
- Refactoring

## Review Guidelines

Before submitting code for review, ensure:
- [ ] `npm run ci` passes (lint + format + test + build)
- [ ] No TypeScript errors
- [ ] Tests pass with > 80% coverage
- [ ] New features have tests
- [ ] No hardcoded secrets

See `.agent/review-guidelines.md` for detailed checklist.