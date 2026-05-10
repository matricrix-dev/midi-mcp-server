# Product Requirements Document (PRD)

## MIDI MCP Server

**Version:** 1.0  
**Status:** Draft  
**Date:** 2026-05-10

---

## 1. Introduction

### 1.1 Purpose

This document defines the product requirements for the MIDI MCP Server, a Model Context Protocol (MCP) server designed for AI-driven MIDI composition. The server enables AI assistants and applications to generate, manipulate, and analyze MIDI music data through a standardized protocol interface.

### 1.2 Product Vision

To democratize music composition by providing AI systems with a powerful, accessible MIDI generation and analysis tool that integrates seamlessly with existing AI assistants and development frameworks.

### 1.3 Product Mission

The MIDI MCP Server bridges the gap between AI language capabilities and music composition by exposing rich MIDI manipulation tools through the Model Context Protocol, enabling AI assistants to compose, analyze, and refine musical compositions with human-understandable music theory concepts.

---

## 2. Target Users

### 2.1 Primary Users

| User Category | Description | Use Cases |
|---------------|-------------|-----------|
| AI Assistants | Large language models (Claude, GPT, etc.) | Composing music, generating MIDI files, music theory analysis |
| Music Producers | Musicians using AI for composition assistance | Quick MIDI sketch generation, chord voicing exploration |
| Developers | Software engineers building AI-music applications | Integrating MIDI generation into custom applications |
| Educators | Music theory teachers using AI tools | Demonstrating composition concepts, generating examples |

### 2.2 Secondary Users

- Audio plugins and Digital Audio Workstations (DAWs) seeking MIDI data
- Game developers generating dynamic music
- Music educational platforms

---

## 3. Product Features

### 3.1 Core Features

#### 3.1.1 MCP Tool: `create_midi`

**Description:** Generate a MIDI file from structured composition data with full chord support.

**Input Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Title of the composition |
| `composition` | object | Yes | Composition data object |

**Composition Object Schema:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bpm` | number | Yes | Tempo in beats per minute |
| `tempo` | number | No | Alternative to bpm |
| `timeSignature` | object | No | Time signature (numerator, denominator) |
| `tracks` | array | Yes | Array of track objects |

**Track Object Schema:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Track name |
| `instrument` | number | No | GM program number (0-127) |
| `notes` | array | Yes | Array of note objects |

**Note Object Schema:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pitch` | number, string, or array | No* | MIDI number, note name, or pitch array |
| `chord` | string | No* | Chord name (e.g., "Cmaj7") |
| `beat` | number | No | Beat position (1-based) |
| `startTime` | number | No | Tick offset |
| `duration` | string or number | Yes | Note duration |
| `velocity` | number | No | Velocity (0-127), default 100 |
| `channel` | number | No | MIDI channel (0-15) |

*Either `pitch` or `chord` must be specified.

**Output:**

| Field | Type | Description |
|-------|------|-------------|
| `midiBase64` | string | Base64-encoded MIDI file |
| `title` | string | Composition title |
| `bpm` | number | Tempo used |
| `trackCount` | number | Number of tracks generated |

#### 3.1.2 MCP Tool: `parse_chord`

**Description:** Parse a chord name and return component MIDI pitches and note names.

**Input Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chord` | string | Yes | Chord name (e.g., "Cmaj7", "Dm", "F#m7") |
| `octave` | number | No | Root octave (default: 4) |

**Output:**

```json
{
  "chord": "Cmaj7",
  "octave": 4,
  "midiNumbers": [60, 64, 67, 71],
  "noteNames": ["C4", "E4", "G4", "B4"]
}
```

### 3.2 Pitch Input Formats

The server supports multiple pitch input formats for flexibility:

| Format | Example | Description |
|--------|---------|-------------|
| MIDI number | `60` | Standard MIDI note number (0-127) |
| Note name | `"C4"` | Letter + optional accidental + octave |
| Pitch array | `[60, 64, 67]` | Multiple pitches played simultaneously |
| Chord field | `chord: "Cmaj7"` | Chord name expanded automatically |

### 3.3 Supported Chord Qualities

The server supports 25+ chord qualities:

| Quality | Example | Description |
|---------|---------|-------------|
| (none) / `maj` | `C`, `Cmaj` | Major |
| `m` / `min` | `Dm` | Minor |
| `dim` | `Bdim` | Diminished |
| `aug` | `Eaug` | Augmented |
| `7` | `G7` | Dominant 7th |
| `maj7` / `M7` | `Cmaj7` | Major 7th |
| `m7` / `min7` | `Am7` | Minor 7th |
| `dim7` | `Bdim7` | Diminished 7th |
| `m7b5` | `Bm7b5` | Half-diminished |
| `aug7` | `Eaug7` | Augmented 7th |
| `6` / `m6` | `C6`, `Am6` | 6th |
| `9` / `maj9` / `m9` | `G9` | 9th variants |
| `add9` | `Cadd9` | Add 9th |
| `11` / `13` | `C11` | Extended |
| `sus2` / `sus4` | `Gsus4` | Suspended |
| `7sus4` / `7sus2` | `G7sus4` | 7th suspended |
| `power` / `5` | `G5` | Power chord |

### 3.4 Duration Reference

Supported duration formats:

| Value | Description |
|-------|-------------|
| `'1'` | Whole note |
| `'2'` | Half note |
| `'4'` | Quarter note |
| `'8'` | Eighth note |
| `'16'` | Sixteenth note |
| `'32'` | Thirty-second note |
| `'d1'` `'d2'` `'d4'` `'d8'` `'d16'` | Dotted variants |
| `'dd4'` | Double-dotted quarter |
| `'T4'` `'T8'` … | Triplet variants |
| `4` (number) | Beat-based: `1`=quarter, `2`=half, `4`=whole, `0.5`=eighth |

### 3.5 Music Theory Resources

The server exposes 7 built-in reference documents as MCP resources:

| URI | Description |
|-----|-------------|
| `music-theory://harmony` | Intervals, chord types, diatonic chords, cadences, voice leading |
| `music-theory://chord-progressions` | Common progressions by mood/genre, substitutions, modulation |
| `music-theory://counterpoint` | Species counterpoint rules, consonance/dissonance, motion types |
| `music-theory://modes-scales` | Diatonic modes, minor scale variants, pentatonic/blues, genre guide |
| `music-theory://orchestration` | Instrument ranges, GM program numbers, texture types |
| `music-theory://rhythm-patterns` | Time signatures, MIDI duration reference, genre grooves |
| `music-theory://voice-leading` | Forbidden parallels, voicing strategies, non-chord tones |

### 3.6 Interactive Preview UI

The server provides an interactive MCP App UI with:

- **Piano-roll visualization**: SVG-based note display showing pitch and timing
- **Audio playback**: Dual-mode playback system
  - Primary: HD soundfont audio (MusyngKite)
  - Fallback: Web Audio oscillator synthesis
- **Playback controls**: Play, Stop, progress indicator
- **Download**: Direct MIDI file download
- **Continue composition**: Send message back to AI for extending composition
- **Chord analyzer**: Interactive chord parsing tool
- **Theory reference**: In-app music theory document viewer
- **Fullscreen mode**: Display mode control

### 3.7 Deployment Options

#### 3.7.1 Remote (Cloudflare Workers)

- Pre-deployed at: `https://midi-mcp-server.tubone24.workers.dev/mcp`
- Uses Streamable HTTP transport
- Best for cloud-based AI assistants

#### 3.7.2 Local stdio

- Runs as stdio server
- Recommended for desktop MCP clients
- Build: `npm run build`, then configure in client

#### 3.7.3 Local HTTP

- Runs as local Streamable HTTP server
- Default port: 3001
- Endpoints:
  - `POST /mcp` — MCP Streamable HTTP endpoint
  - `GET /health` — Health check

---

## 4. User Stories

### 4.1 Composition

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As an AI assistant, I want to generate a complete MIDI file from a simple chord progression so that I can provide users with downloadable music. | P0 |
| US-2 | As a music producer, I want to specify notes using familiar chord names (like "Cmaj7") instead of MIDI numbers so that composition feels more natural. | P0 |
| US-3 | As a developer, I want to preview generated music visually with a piano-roll so that I can verify the output before downloading. | P1 |
| US-4 | As an educator, I want to access music theory references within the tool so that I can provide accurate guidance. | P2 |

### 4.2 Analysis

| ID | Story | Priority |
|----|-------|----------|
| US-5 | As a user, I want to know what notes are in a specific chord so that I can understand voicings. | P1 |
| US-6 | As an AI, I want to parse any valid chord name and get its MIDI representation for further composition. | P0 |

### 4.3 Playback

| ID | Story | Priority |
|----|-------|----------|
| US-7 | As a user, I want to hear the generated composition with realistic instrument sounds so that I can evaluate the result. | P1 |
| US-8 | As a user, I want to download the generated MIDI file so that I can use it in my DAW. | P0 |

---

## 5. Non-Functional Requirements

### 5.1 Performance

| Requirement | Target |
|-------------|--------|
| MIDI generation time | < 500ms for typical compositions |
| Tool response time | < 1 second |
| Preview UI load time | < 2 seconds |

### 5.2 Compatibility

| Requirement | Description |
|-------------|-------------|
| MCP Protocol | v1.x compliant |
| Node.js | v18+ |
| Browsers | Chrome 90+, Firefox 88+, Safari 14+, Edge 90+ |
| MCP Clients | Claude.ai, Cursor, other MCP-compatible clients |

### 5.3 Reliability

| Requirement | Description |
|-------------|-------------|
| Uptime | 99.9% for hosted version |
| Error handling | Graceful degradation with clear error messages |
| Input validation | Zod schema validation for all inputs |

### 5.4 Usability

| Requirement | Description |
|-------------|-------------|
| Error messages | Clear, actionable error messages |
| Documentation | Comprehensive README with examples |
| Type safety | Full TypeScript with strict mode |

---

## 6. Success Metrics

### 6.1 Key Performance Indicators

| Metric | Target | Measurement |
|--------|--------|-------------|
| Tool usage | 1000+ calls/day (hosted) | Server logs |
| User satisfaction | > 4.5/5 stars | Feedback |
| Error rate | < 0.1% | Exception tracking |
| Average composition size | 8-16 bars | Analysis |

### 6.2 Adoption Metrics

| Metric | Target |
|--------|--------|
| GitHub stars | 500+ |
| MCP registry listings | Primary registries |
| Community contributions | 10+ PRs/year |

---

## 7. Roadmap

### 7.1 Phase 1 (v0.2.x) — Current

- [x] Core MIDI generation
- [x] Chord name support
- [x] Multiple deployment modes
- [x] Preview UI with playback
- [x] Music theory resources

### 7.2 Phase 2 (v0.3.x) — Planned

- [ ] MIDI file upload/parsing tool
- [ ] Track manipulation (add/remove tracks)
- [ ] MIDI export options (Format 0/1)
- [ ] Expanded chord voicings

### 7.3 Phase 3 (v1.0) — Future

- [ ] MIDI editing capabilities
- [ ] Pattern library
- [ ] Integration with DAWs
- [ ] Plugin system

---

## 8. Appendix

### 8.1 Glossary

| Term | Definition |
|------|------------|
| MCP | Model Context Protocol - A standardized protocol for AI tool interactions |
| MIDI | Musical Instrument Digital Interface - Standard protocol for music data |
| GM | General MIDI - Standard instrument assignments |
| BPM | Beats Per Minute - Tempo measurement |

### 8.2 References

- Model Context Protocol Specification: https://modelcontextprotocol.io/
- MIDI Specification: https://www.midi.org/
- GM Instrument List: https://www.midi.org/specifications/midi-reference-tables/gm-instrument-patch-map