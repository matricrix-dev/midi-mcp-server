# Technical Requirements Document (TRD)

## MIDI MCP Server

**Version:** 1.0  
**Status:** Draft  
**Date:** 2026-05-10

---

## 1. Technical Overview

### 1.1 System Architecture

The MIDI MCP Server is built on a modular architecture consisting of three primary layers:

1. **Transport Layer**: Handles MCP protocol communication via stdio, HTTP, or Cloudflare Workers
2. **Application Layer**: Implements MCP tools, resources, and business logic
3. **UI Layer**: Provides interactive preview and playback via MCP Apps extension

### 1.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     MCP Client (AI Assistant)               │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────▼────┐    ┌─────▼────┐    ┌────▼────┐
    │ stdio   │    │  HTTP    │    │Worker   │
    └────┬────┘    └─────┬────┘    └────┬────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
                ┌────────▼────────┐
                │   MCP Server    │
                │  (server.ts)    │
                └────────┬────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────▼────┐    ┌─────▼────┐    ┌────▼────┐
    │  Tools  │    │Resources │    │  UI App │
    │create_  │    │Music     │    │Preview  │
    │ midi    │    │Theory    │    │         │
    │parse_   │    │Docs      │    │(mcp-    │
    │ chord   │    │          │    │ app.ts) │
    └─────────┘    └──────────┘    └─────────┘
```

---

## 2. Technology Stack

### 2.1 Runtime Environment

| Component | Technology | Version |
|-----------|------------|---------|
| Runtime | Node.js | 18+ |
| Language | TypeScript | 5.0+ |
| Module System | ES Modules | native |

### 2.2 Core Dependencies

| Package | Purpose | Version |
|---------|---------|---------|
| `@modelcontextprotocol/sdk` | MCP server implementation | ^1.12.0 |
| `@modelcontextprotocol/ext-apps` | MCP Apps extension for interactive UI | ^1.3.2 |
| `@tonejs/midi` | MIDI parsing and generation | ^2.0.28 |
| `soundfont-player` | HD audio playback | ^0.12.0 |
| `zod` | Input schema validation | ^3.23.0 |

### 2.3 Development Dependencies

| Package | Purpose | Version |
|---------|--------|---------|
| `typescript` | Type checking | ^5.0.0 |
| `vite` | UI bundling | ^8.0.3 |
| `vitest` | Testing | ^4.1.5 |
| `eslint` | Linting | ^8.56.0 |
| `prettier` | Code formatting | ^3.2.5 |
| `wrangler` | Cloudflare Workers deployment | ^4.90.0 |

### 2.4 Build Tools

The build process consists of two parallel pipelines:

```
npm run build
    │
    ├─► npm run clean
    │       └─ Removes build/ and dist/ directories
    │
    ├─► npm run build:ui (Vite)
    │       ├─ Input: src/mcp-app.ts, src/mcp-app.html, src/mcp-app.css
    │       └─ Output: dist/src/mcp-app.html
    │
    └─► npm run build:server (tsc)
            ├─ Input: src/*.ts (excluding worker, mcp-app)
            └─ Output: build/ with compiled JS + resources/
```

---

## 3. Component Specifications

### 3.1 Entry Point (index.ts)

The main entry point handles transport mode selection and server initialization.

**Responsibilities:**
- Parse command-line arguments (`--http`, `--port=`, `--stdio`)
- Initialize appropriate transport (StdioServerTransport or StreamableHTTPServerTransport)
- Create MCP server instance
- Provide MIDI file saving utility for CLI usage
- Health check endpoint for HTTP mode

**CLI Arguments:**

| Argument | Description | Default |
|----------|-------------|---------|
| `--http` | Run in HTTP mode | stdio mode |
| `--port=N` | HTTP server port | 3001 |

**HTTP Endpoints:**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/mcp` | POST | MCP Streamable HTTP endpoint |
| `/health` | GET | Health check (`{"status":"ok","version":"0.2.0"}`) |

### 3.2 Server Core (server.ts)

The server module implements MCP protocol handlers and core business logic.

**Key Functions:**

| Function | Description |
|----------|-------------|
| `createServer()` | Factory function returning configured McpServer instance |
| `generateMidiBase64()` | In-memory MIDI generation without filesystem |
| `preprocessComposition()` | Normalize and validate composition input |
| `durationToBeats()` | Convert duration strings to beat counts |

**MIDI Generation Pipeline:**

1. Create Midi instance with header (tempo, time signature)
2. Iterate through tracks, adding notes with pitch, timing, velocity
3. Handle chord expansion via `resolvePitches()`
4. Convert to base64 for output

**Time Calculation:**
- 1 beat = 60/bpm seconds
- beat parameter is 1-based (converted to 0-based internally)
- Duration in beats converted to seconds for @tonejs/midi

### 3.3 Chord Utilities (chord-utils.ts)

Implements chord name parsing and pitch resolution.

**Core Functions:**

| Function | Signature | Description |
|----------|-----------|-------------|
| `parseNoteName` | `(noteName: string) => number` | Convert "C4" to MIDI 60 |
| `midiNumberToNoteName` | `(midi: number) => string` | Convert 60 to "C4" |
| `parseChordName` | `(chord: string, octave?: number) => number[]` | Expand chord to MIDI numbers |
| `resolvePitches` | `(pitch: any, chord?: string) => number[]` | Unified pitch resolution |
| `normalizeDuration` | `(duration: string \| number) => string` | Standardize duration format |

**Chord Interval Mapping:**

The CHORD_INTERVALS constant maps quality strings to semitone intervals:

```typescript
const CHORD_INTERVALS: Record<string, number[]> = {
  '': [0, 4, 7],      // major
  m: [0, 3, 7],       // minor
  dim: [0, 3, 6],     // diminished
  // ... 20+ qualities
};
```

### 3.4 MCP App UI (mcp-app.ts)

Client-side application for interactive preview and playback.

**Features:**

| Feature | Implementation |
|---------|----------------|
| Piano-roll rendering | SVG-based visualization |
| Audio playback | Web Audio API with dual-mode |
| Soundfont loading | soundfont-player with MusyngKite |
| Oscillator fallback | Custom synthesis per GM family |
| Theme integration | @modelcontextprotocol/ext-apps APIs |

**Audio Architecture:**

```
┌────────────────────────────────────────────┐
│            MidiPlayer Class                │
├────────────────────────────────────────────┤
│  ┌─────────────┐   ┌─────────────────┐   │
│  │ Soundfont   │   │  Oscillator     │   │
│  │ Player      │   │  Fallback       │   │
│  │ (Primary)   │   │  (Secondary)    │   │
│  └─────────────┘   └─────────────────┘   │
└────────┬──────────────────────┬──────────┘
         │                      │
    ┌────▼────┐            ┌───▼────┐
    │ HD Audio│            │ Web    │
    │(Musyng) │            │ Synth  │
    └─────────┘            └────────┘
```

**Instrument Mapping:**

GM program numbers mapped to soundfont instrument names:
- Programs 0-7: Piano family
- Programs 8-15: Chromatic percussion
- Programs 16-23: Organ
- Programs 24-31: Guitar
- Programs 32-39: Bass
- Programs 40-47: Strings
- Programs 48-55: Ensemble
- Programs 56-63: Brass
- Programs 64-71: Reed
- Programs 72-79: Pipe
- Programs 80-95: Synth
- Programs 96-127: Other

### 3.5 Music Theory Resources

Seven markdown documents served as MCP resources:

| Resource | File | Content |
|----------|------|---------|
| Harmony | `harmony.md` | Intervals, chord types, diatonic chords, cadences |
| Chord Progressions | `chord-progressions.md` | Common progressions, substitutions, modulation |
| Counterpoint | `counterpoint.md` | Species rules, consonance, motion types |
| Modes & Scales | `modes-scales.md` | Modes, minor scales, pentatonic |
| Orchestration | `orchestration.md` | Instrument ranges, GM numbers |
| Rhythm Patterns | `rhythm-patterns.md` | Time signatures, duration reference |
| Voice Leading | `voice-leading.md` | Parallel prevention, voicing strategies |

---

## 4. API Specifications

### 4.1 MCP Protocol Version

The server implements MCP protocol v1.x with the following transport modes:

| Mode | Protocol | Use Case |
|------|----------|----------|
| stdio | JSON-RPC over stdio | Desktop MCP clients |
| HTTP | Streamable HTTP | Web-based clients |
| Worker | Cloudflare Worker | Remote deployment |

### 4.2 Tool Definitions

#### 4.2.1 create_midi

```typescript
{
  name: 'create_midi',
  description: 'Generate a MIDI file from structured composition data...',
  inputSchema: {
    title: z.string(),
    composition: z.any()
  },
  outputSchema: {
    midiBase64: z.string(),
    title: z.string(),
    bpm: z.number(),
    trackCount: z.number()
  },
  _meta: {
    ui: { resourceUri: 'ui://midi-preview/app.html' }
  }
}
```

#### 4.2.2 parse_chord

```typescript
{
  name: 'parse_chord',
  description: 'Parse a chord name and return its component MIDI pitches...',
  inputSchema: {
    chord: z.string(),
    octave: z.number().optional()
  }
}
```

### 4.3 Resource Definitions

#### 4.3.1 Music Theory Resources

Each resource registered with:
- Name: Human-readable title
- URI: `music-theory://{topic}` scheme
- MIME type: `text/markdown`

#### 4.3.2 App Resource

```typescript
{
  name: 'MIDI Preview',
  uri: 'ui://midi-preview/app.html',
  mimeType: 'text/html'
}
```

---

## 5. Data Models

### 5.1 TypeScript Interfaces

```typescript
interface MidiNote {
  pitch: number | string | (number | string)[];
  chord?: string;
  beat?: number;
  startTime?: number;
  time?: number;
  duration: string | number;
  velocity?: number;
  channel?: number;
}

interface MidiTrack {
  name?: string;
  instrument?: number;
  notes: MidiNote[];
}

interface TimeSignature {
  numerator: number;
  denominator: number;
}

interface MidiComposition {
  bpm: number;
  tempo?: number;
  timeSignature?: TimeSignature;
  tracks: MidiTrack[];
}
```

### 5.2 Input Validation

All tool inputs validated via Zod schemas:

| Tool | Validation |
|------|------------|
| create_midi | title required, composition must be object/JSON string |
| parse_chord | chord required, must match [A-G][#b]?[0-9]?.* pattern |

### 5.3 Error Handling

| Error Type | Response |
|------------|----------|
| Invalid chord | `{isError: true, content: [{type: 'text', text: 'Error parsing chord: ...'}]}` |
| Invalid note name | `{isError: true, content: [{type: 'text', text: 'Error generating MIDI: ...'}]}` |
| Out of MIDI range | Error with specific note number |

---

## 6. Deployment Architecture

### 6.1 Local Deployment

**Requirements:**
- Node.js 18+
- npm 9+

**Setup:**
```bash
npm install
npm run build
```

**Run Modes:**

```bash
# stdio mode (default)
node build/index.js

# HTTP mode
node build/index.js --http

# HTTP with custom port
node build/index.js --http --port=8080
```

### 6.2 Remote Deployment (Cloudflare Workers)

**Configuration (wrangler.toml):**
- Name: midi-mcp-server
- Compatibility date: 2024-XX-XX
- Routes: Workers.dev domain

**Deployment:**
```bash
npm run deploy
```

**Endpoint:**
```
https://midi-mcp-server.tubone24.workers.dev/mcp
```

### 6.3 MCP Client Configuration

#### stdio Configuration:
```json
{
  "mcpServers": {
    "midi": {
      "command": "node",
      "args": ["/path/to/midi-mcp-server/build/index.js"]
    }
  }
}
```

#### HTTP Configuration:
```json
{
  "mcpServers": {
    "midi": {
      "type": "http",
      "url": "http://localhost:3001/mcp"
    }
  }
}
```

---

## 7. Security Considerations

### 7.1 Input Validation

- All user inputs validated via Zod schemas
- MIDI note numbers bounded to 0-127
- Chord names validated against known qualities

### 7.2 CORS Configuration

HTTP mode includes CORS headers:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, mcp-session-id, mcp-protocol-version
```

### 7.3 Resource Access

- Music theory resources loaded from local filesystem only
- No external resource loading in server mode

---

## 8. Performance Requirements

### 8.1 Latency Targets

| Operation | Target | Maximum |
|-----------|--------|---------|
| MIDI generation (16 bars) | < 200ms | 500ms |
| Tool response (total) | < 500ms | 1s |
| HTTP health check | < 50ms | 100ms |

### 8.2 Resource Usage

| Resource | Limit |
|----------|-------|
| Memory (server) | < 100MB |
| MIDI file size | < 1MB typical |
| Concurrent sessions (HTTP) | Unlimited (stateless) |

### 8.3 Audio Performance

| Metric | Target |
|--------|--------|
| Soundfont preload | < 3 seconds |
| Oscillator fallback load | < 100ms |
| Playback latency | < 50ms |

---

## 9. Testing Requirements

### 9.1 Test Framework

- Framework: Vitest
- Configuration: vitest.config.ts

### 9.2 Test Coverage Areas

| Area | Files | Coverage Target |
|------|-------|-----------------|
| Chord utilities | chord-utils.ts | 90%+ |
| Server logic | server.ts | 80%+ |
| Integration | index.test.ts | Key flows |

### 9.3 Test Commands

```bash
npm test              # Run all tests
npm run test:watch   # Watch mode
npm run test:coverage # With coverage report
```

---

## 10. Development Setup

### 10.1 Prerequisites

| Requirement | Version |
|------------|---------|
| Node.js | 18+ |
| npm | 9+ |
| OS | Windows/macOS/Linux |

### 10.2 Local Development

```bash
# Install dependencies
npm install

# Run tests
npm test

# Build
npm run build

# Lint
npm run lint

# Format check
npm run format:check
```

### 10.3 Code Style

- TypeScript strict mode enabled
- ESLint with TypeScript parser
- Prettier for formatting
- No commented code unless explanatory

### 10.4 Git Workflow

1. Create feature branch
2. Implement changes with tests
3. Run lint, format, tests
4. Submit PR for review
5. Squash merge to main

---

## 11. Appendix

### 11.1 File Structure

```
midi-mcp-server/
├── src/
│   ├── index.ts           # Entry point, transport handling
│   ├── server.ts          # MCP server, tools, resources
│   ├── chord-utils.ts     # Chord parsing, pitch resolution
│   ├── mcp-app.ts         # UI client, playback, rendering
│   ├── mcp-app.html       # UI HTML template
│   ├── mcp-app.css        # UI styles
│   ├── worker.ts          # Cloudflare Worker entry
│   ├── resources/         # Music theory markdown docs
│   └── __tests__/        # Test files
├── build/                 # Compiled server output
├── dist/                  # UI build output
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── vite.config.mcp-app.ts
└── wrangler.toml
```

### 11.2 Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| HOME or USERPROFILE | For MIDI file saving (Windows) | No |
| TMPDIR | Fallback for file saving | No |

### 11.3 Known Limitations

- @tonejs/midi is CJS-only, loaded via createRequire
- Channel selection via track index modulo 16
- No MIDI file reading capability in v0.2.x
- Single file output (no batch generation)

### 11.4 Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1.0 | 2024-XX | Initial release |
| 0.2.0 | 2025-XX | MCP Apps UI, HTTP mode, resources |
| 1.0.0 | TBD | Stable release |