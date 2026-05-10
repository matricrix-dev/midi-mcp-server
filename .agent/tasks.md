# Agent Tasks

This document defines common development tasks for the Senior Developer Agent.

## Usage

Invoke the agent using the Task tool:

```javascript
task({
  description: "task name",
  prompt: "detailed instructions",
  subagent_type: "explore" // or "general"
})
```

---

## Task Templates

### 1. Code Review

```javascript
task({
  description: "Review code changes",
  prompt: `
Review the recent changes in this project for:
- Code quality and best practices
- TypeScript type safety
- Potential bugs or issues
- Performance concerns

Focus on: src/server.ts, src/chord-utils.ts

Provide specific feedback with file:line references.
  `,
  subagent_type: "explore"
})
```

### 2. Bug Investigation

```javascript
task({
  description: "Investigate bug",
  prompt: `
Investigate the issue: [describe bug]

Steps to reproduce:
1. [step 1]
2. [step 2]

Expected behavior: [what should happen]
Actual behavior: [what actually happens]

Debug by:
- Checking relevant source files
- Running tests
- Examining error messages

Provide root cause analysis and proposed fix.
  `,
  subagent_type: "explore"
})
```

### 3. Feature Implementation

```javascript
task({
  description: "Implement feature",
  prompt: `
Implement a new [feature name] for the MIDI MCP Server.

Requirements:
- [requirement 1]
- [requirement 2]

Existing patterns to follow:
- Look at create_midi in src/server.ts for MCP tool pattern
- Use zod for input validation
- Add tests in src/__tests__/

Output: Complete implementation with tests.
  `,
  subagent_type: "general"
})
```

### 4. Test Coverage Analysis

```javascript
task({
  description: "Analyze test coverage",
  prompt: `
Analyze the test coverage for this project:

1. Run: npm run test:coverage
2. Review coverage reports
3. Identify low-coverage areas in:
   - src/server.ts
   - src/chord-utils.ts
   - src/index.ts

Recommend specific areas needing more tests.
  `,
  subagent_type: "explore"
})
```

### 5. Build Debugging

```javascript
task({
  description: "Debug build errors",
  prompt: `
Debug the build error:
[insert error message]

Debug steps:
1. Run: npm run build
2. Examine error details
3. Check tsconfig.json and vite.config.mcp-app.ts
4. Identify root cause

Provide fix recommendations.
  `,
  subagent_type: "explore"
})
```

### 6. Refactoring

```javascript
task({
  description: "Refactor code",
  prompt: `
Refactor [specific code area] in src/[file].ts:

Goals:
- Improve readability
- Extract common utilities
- Reduce duplication

Constraints:
- Maintain all existing tests
- Keep API compatibility
- Follow project style guide

Output: Refactored code with explanation.
  `,
  subagent_type: "general"
})
```

---

## Quick Reference

| Task Type | Subagent | Description |
|-----------|----------|-------------|
| Code review | explore | Analyze code, find issues |
| Bug fix | explore | Debug and fix problems |
| Implementation | general | Build new features |
| Research | explore | Investigate, gather info |
| Writing | general | Docs, tests, refactoring |

## Project-Specific Prompts

### Check for TypeScript errors
```bash
npm run build:server 2>&1 | head -50
```

### Run linting
```bash
npm run lint
```

### Run tests
```bash
npm test
```

### Full CI check locally
```bash
npm run lint && npm run format:check && npm test && npm run build
```