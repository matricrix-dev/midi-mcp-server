# Roles
- Use **Senior Node.js/TypeScript Developer** agent instructions for development tasks.

# Tasks
- [x] midi-mcp (custom): Investigate "undefined" error - Finding: The `midi-mcp__custom__create_midi` tool is not actually connected to this MCP server implementation. The server code works correctly (all 11 tests pass), but the tool invocation mechanism in the opencode environment isn't connecting to the actual server. No code fix needed - this is an environment/integration issue.

## Fix: Tool name mismatch between opencode and MCP server

**Root cause**: Server registers tool as `create_midi` but opencode calls `midi-mcp__custom__create_midi`

- [x] 1. Investigate how opencode maps custom tool names to MCP server tools
- [x] 2. Add tool alias or rename in server.ts to match opencode naming convention
- [x] 3. Verify the connection works by running the MCP server and calling the tool
- [x] 4. Test MIDI generation with sample composition

**Status**: MIDI generation works! The tool now generates MIDI files successfully. The saveToFile feature may have environment limitations (filesystem access in opencode sandbox).
