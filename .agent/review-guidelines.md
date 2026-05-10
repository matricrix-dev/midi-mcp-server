# Code Review Guidelines

## Review Checklist

### TypeScript & Type Safety
- [ ] Proper type annotations on function parameters/returns
- [ ] No `any` types without justification
- [ ] Zod schemas used for input validation
- [ ] Interface definitions for data structures

### Code Quality
- [ ] No commented-out code
- [ ] Meaningful variable/function names
- [ ] Single responsibility principle
- [ ] DRY - don't repeat yourself
- [ ] Proper error handling with clear messages

### Testing
- [ ] Unit tests for utility functions (chord-utils.ts)
- [ ] Test coverage > 80%
- [ ] Tests follow Arrange-Act-Assert pattern
- [ ] Mock external dependencies

### Performance
- [ ] No unnecessary allocations
- [ ] Efficient data structures
- [ ] Lazy loading where appropriate

### Security
- [ ] Input validation on all user inputs
- [ ] No hardcoded secrets
- [ ] Proper CORS configuration for HTTP mode

---

## File-Specific Guidelines

### src/index.ts
- Entry point should be minimal
- Transport setup should be clear
- CLI argument parsing needs validation
- Health endpoint should be fast

### src/server.ts
- MCP tools registered correctly
- Zod schemas for tool input/output
- Error handling returns proper error format
- Resources loaded from local files only
- MIDI generation uses in-memory processing

### src/chord-utils.ts
- All chord intervals defined in CHORD_INTERVALS
- Note name parsing handles sharps/flats
- MIDI range validation (0-127)
- Duration normalization covers all variants
- Export all utilities for reuse

### src/mcp-app.ts
- Audio context management (lazy creation)
- Soundfont loading with caching
- Oscillator fallback for unsupported instruments
- SVG rendering handles edge cases
- Theme integration with host

---

## Common Issues to Catch

| Issue | Location | Fix |
|-------|----------|-----|
| Missing zod import | server.ts | Import from 'zod' |
| CJS require issue | server.ts | Use createRequire |
| Type errors | mcp-app.ts | Check null safety |
| Resource path | server.ts | Use import.meta.url |
| Duration mismatch | server.ts | Use durationToBeats() |
| Channel overflow | server.ts | Modulo 16 |

---

## Review Comments Format

When providing feedback:

```
File:Line - [Issue description]

[What is wrong]
[Why it's a problem]
[Suggested fix]
```

Example:
```
src/chord-utils.ts:45 - Missing null check

The parseNoteName function doesn't validate the input
can be null, causing runtime errors.

Add: if (!noteName) throw new Error('Note name required')
```

---

## Approval Criteria

PR requires approval when:
- All CI checks pass
- TypeScript compiles without errors
- Tests pass with > 80% coverage
- Code follows style guide
- No security issues
- Documentation updated if needed