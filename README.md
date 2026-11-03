# Ardour MCP 🎵

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![MCP](https://img.shields.io/badge/MCP-Server-green.svg)](https://modelcontextprotocol.io/)
[![Tests](https://img.shields.io/badge/tests-581%20passing-brightgreen.svg)](https://github.com/raibid-labs/ardour-mcp/actions)
[![Coverage](https://img.shields.io/badge/coverage-86%25-yellowgreen.svg)](docs/COVERAGE_REPORT.md)
[![Code of Conduct](https://img.shields.io/badge/Code%20of%20Conduct-Contributor%20Covenant-purple.svg)](CODE_OF_CONDUCT.md)
[![Security](https://img.shields.io/badge/Security-Policy-red.svg)](SECURITY.md)

**Model Context Protocol server for Ardour DAW - Control Ardour through AI assistants**

The first MCP integration for a major open-source Digital Audio Workstation. Ardour MCP enables natural language control of [Ardour](https://ardour.org/) through AI assistants like Claude, using the [Model Context Protocol](https://modelcontextprotocol.io/).

## 🎯 What This Does

Control Ardour using natural language through Claude Desktop or any MCP-compatible client:

- **"Start recording on tracks 1 and 2, then set up punch recording from bar 5 to bar 9"**
- **"Create 4 audio tracks for drums: Kick, Snare, Hi-Hat, Overhead"**
- **"Set track 1 volume to -6dB, pan it 30% left, and mute track 4"**
- **"Send the vocal track to reverb bus at -12dB"**
- **"Set gain automation to write mode on track 3"**
- **"Check for phase issues and clipping in my mix"**
- **"What's the LUFS loudness of the master output?"**

No commands to memorize - just describe what you want!

## 🎹 MIDI Generation Integration

**NEW**: Combine Ardour MCP with [MIDI MCP Server](https://github.com/tubone24/midi-mcp-server) for complete AI-assisted music production:

- **"Generate a jazz piano backing track and import it into Ardour"**
- **"Create a 4-bar drum pattern, bass line, and chord progression at 95 BPM"**
- **"Make an orchestral arrangement with strings and brass, then mix it"**

See [MIDI MCP Server Integration Guide](docs/integrations/MIDI_MCP_SERVER.md) for setup and [Complete Workflow Guide](docs/integrations/MIDI_ARDOUR_WORKFLOW.md) for examples.

## ✨ Current Status

**Latest Release**: [v0.3.0](https://github.com/raibid-labs/ardour-mcp/releases/tag/v0.3.0) (January 2025)

### 🎉 Phase 3 Complete - Professional Mixing & Mastering

**111 Total MCP Tools** across 9 categories:
- 🚀 **Transport Control** (11 tools): Play, stop, record, navigate
- 🎵 **Track Management** (5 tools): Create, rename, select tracks
- 📝 **Session Management** (9 tools): Tempo, time signature, session info
- 🎚️ **Basic Mixer** (14 tools): Volume, pan, mute, solo
- 🎛️ **Advanced Mixer** (15 tools): Sends, plugins, bus routing
- 📍 **Navigation** (17 tools): Markers, loops, timecode
- 🎙️ **Recording** (13 tools): Recording control, punch in/out, monitoring
- 🤖 **Automation** (13 tools): Automation modes, recording, editing
- 📊 **Metering** (12 tools): Levels, phase, loudness, clipping detection

**Quality Metrics**:
- ✅ **581 tests** passing with **86% code coverage**
- ✅ Production-ready with comprehensive error handling
- ✅ Bidirectional OSC communication with real-time feedback
- ✅ Complete documentation with usage examples

### Feature Timeline

| Phase | Status | Tools Added | Highlights |
|-------|--------|-------------|------------|
| **Phase 1** | ✅ Complete | 27 | Transport, tracks, session, basic mixer |
| **Phase 2** | ✅ Complete | 59 | Recording, navigation, enhanced mixer |
| **Phase 3** | ✅ Complete | 40 | Advanced mixer, automation, metering |
| **Phase 4** | 📋 Planned | TBD | Plugin editing, region manipulation, MIDI |

See [ROADMAP.md](docs/ROADMAP.md) for complete feature timeline.

## 🚀 Quick Start

**Complete setup in under 5 minutes!**

### Prerequisites

- **Ardour 8.x or 9.x** installed
- **Python 3.10+** (Python 3.10, 3.11, 3.12 supported)
- **Claude Desktop or Claude Code** (or any MCP-compatible client)

### Installation

```bash
# Install uv package manager
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone and install
git clone https://github.com/raibid-labs/ardour-mcp.git
cd ardour-mcp
uv sync --all-extras
```

### Configure Ardour OSC

**This is the most important step!**

1. Launch Ardour and open/create a session
2. Go to **Edit → Preferences → Control Surfaces**
3. Enable **"Open Sound Control (OSC)"**
4. Click **"Settings"**
5. Configure:
   - **OSC Server Port**: `3819` (default)
   - **Feedback / Reply port**: **Manual, `3820`** — the server listens for
     feedback on this port. Other modes (reply-to-sender or a different
     manual port) will leave the server deaf to state updates.
   - **Feedback**: Enable **all feedback options**
   - Click **OK**

   If Ardour was already running when you changed the port, toggle the OSC
   control surface off and on again in the Preferences list so the running
   surface picks up the new reply port.

### Configure MCP Client

Choose your client:

<details>

<summary><b>OpenCode v2</b></summary>

**1. Register the MCP server — project-scoped, not global.**

Add to `opencode.jsonc` in the checkout root:

```json
{
  "mcp": {
    "servers": {
      "ardour": {
        "type": "local",
        "command": ["uv", "run", "ardour-mcp"],
        "cwd": "/absolute/path/to/ardour-mcp"
      }
    }
  }
}
```

Notes:

- `cwd` must be an **absolute** path to the checkout; relative paths
  resolve against the workspace, not the checkout.
- **Why project-scoped:** the server owns a fixed UDP feedback port
  (3820), so exactly one instance can run at a time. Registered
  globally, OpenCode eagerly connects it for *every* session at
  startup, and the first session to spawn wins the port while every
  other session's instance dies with `Address already in use`.
  Project-scoped registration only spawns it for sessions inside this
  tree. Two concurrent sessions inside the tree can still race — start
  them one after another.

**2. (Optional) allow hands-off tool use.**

The assistant is most useful when it can run shell commands (probe
scripts, `uv run pytest`) and edit files without confirming every step.
In `opencode.jsonc`:

```json
{
  "permission": {
    "edit": "allow",
    "bash": "allow"
  }
}
```

Use `"ask"` instead of `"allow"` for any category you want to gate.
See OpenCode's documentation for the full permission schema.

**3. Restart OpenCode so the config takes effect.**

OpenCode spawns MCP servers lazily per session and pins the process for
the session's lifetime:

- Config changes (`mcp`, `permission`, …) need a restart — `/restart`
  or a fresh `opencode` start.
- The tool catalog is snapshotted at spawn: a tool added in new server
  code only appears after the server process is respawned.
- If a server process is killed externally, OpenCode does **not**
  respawn it — restart OpenCode.

**4. Verify.**

Start `opencode` inside the checkout and ask the assistant to call the
`get_session_info` tool — it should answer with the session name,
sample rate and transport state. If the `ardour` tools are missing
from the assistant's tool list, the server did not spawn: re-check the
JSON syntax (`opencode.jsonc` allows comments), then restart.

**5. Troubleshooting.**

| Symptom | Cause and fix |
|---------|---------------|
| Tools answer but levels/position are stale | The reply port isn't Manual `3820` (see step 5 of *Configure Ardour OSC*), or Ardour was restarted — re-arm by restarting OpenCode, or toggle Ardour's OSC surface off/on. |
| Every tool times out | Ardour isn't running or OSC is disabled on port 3819. |

</details>

<details>

<summary><b>Claude Code</b> (Recommended - One command!)</summary>

**Manual configuration** - Add to `.mcp.json` in your project or `~/.claude.json`:
```json
{
  "mcpServers": {
    "ardour": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/ardour-mcp",
        "run",
        "ardour-mcp"
      ]
    }
  }
}
```

**Verify installation**:
```bash
claude mcp list
claude mcp get ardour
```

The server should show as "✓ Connected". That's it - ardour-mcp is now available in Claude Code!

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Add to your `claude_desktop_config.json`:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
**Linux**: `~/.config/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "ardour": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/ardour-mcp",
        "run",
        "ardour-mcp"
      ]
    }
  }
}
```

Restart Claude Desktop and the server will be available.

</details>

**Detailed setup instructions**: [docs/QUICK_START.md](docs/QUICK_START.md)

## 📖 Documentation

### 🎯 Getting Started

| Document | Description |
|----------|-------------|
| **[Quick Start Guide](docs/QUICK_START.md)** | 5-minute setup guide with troubleshooting |
| **[Example Conversations](docs/EXAMPLE_CONVERSATIONS.md)** | Real dialogue examples showing workflows |
| **[Usage Examples](docs/USAGE_EXAMPLES.md)** | Comprehensive feature reference by category |

### 📚 Feature Guides

Complete usage guides for specific features:

| Guide | Tools | Description |
|-------|-------|-------------|
| **[Recording Guide](docs/guides/RECORDING_EXAMPLE_USAGE.md)** | 13 tools | Recording control, punch in/out, monitoring |
| **[Mixer Guide](docs/guides/MIXER_EXAMPLE_USAGE.md)** | 14 tools | Volume, pan, mute, solo, batch operations |
| **[Advanced Mixer](docs/guides/ADVANCED_MIXER_USAGE.md)** | 15 tools | Sends, plugins, bus routing, effects chains |
| **[Navigation Guide](docs/guides/NAVIGATION_EXAMPLE_USAGE.md)** | 17 tools | Markers, loops, tempo, timecode |
| **[Automation Guide](docs/guides/AUTOMATION_USAGE.md)** | 13 tools | Automation modes, recording, editing |
| **[Metering Guide](docs/guides/METERING_USAGE.md)** | 12 tools | Levels, phase analysis, loudness, clipping |

### 🎹 Integration Guides

Extend Ardour MCP with complementary MCP servers:

| Guide | Description |
|-------|-------------|
| **[MIDI MCP Server](docs/integrations/MIDI_MCP_SERVER.md)** | Generate MIDI compositions from natural language |
| **[Complete Workflow](docs/integrations/MIDI_ARDOUR_WORKFLOW.md)** | End-to-end music production with MIDI + Ardour |

### 🏗️ Technical Documentation

Architecture, development, and reference materials:

| Document | Description |
|----------|-------------|
| **[Architecture Overview](docs/ARCHITECTURE.md)** | System design and component interaction |
| **[Development Guide](docs/DEVELOPMENT.md)** | Setup, testing, and contribution workflow |
| **[OSC API Reference](docs/OSC_API.md)** | Complete Ardour OSC command reference |
| **[Coverage Report](docs/COVERAGE_REPORT.md)** | Test coverage analysis (86%) |

### 🚀 Project Management

Releases, roadmap, and publishing:

| Document | Description |
|----------|-------------|
| **[Roadmap](docs/ROADMAP.md)** | Feature timeline and milestones |
| **[Releasing](docs/RELEASING.md)** | Version management and release process |
| **[Pre-Release Checklist](docs/PRE_RELEASE_CHECKLIST.md)** | Comprehensive checklist before releasing |
| **[Release Workflows](docs/RELEASE-WORKFLOWS.md)** | Comparison of automation options |
| **[PyPI Setup](docs/PYPI_SETUP.md)** | Configure PyPI trusted publishing |
| **[Registry Publishing](docs/REGISTRY_PUBLISHING.md)** | Publishing to MCP registries |

### 🤝 Contributing

| Document | Description |
|----------|-------------|
| **[Contributing Guide](CONTRIBUTING.md)** | How to contribute code and documentation |
| **[Changelog](CHANGELOG.md)** | Version history and release notes |

## 🎬 Example Workflows

### Recording Session Setup

```
You: "I want to record a band. Set up 4 audio tracks called 'Kick', 'Snare',
     'Guitar', and 'Vocals'. Arm the kick and snare for recording and enable
     input monitoring on all of them."

Claude: ✓ Created audio tracks
        ✓ Armed tracks for recording
        ✓ Enabled input monitoring
        Your session is ready to record!

You: "Start recording!"

Claude: ✓ Recording started with 2 armed tracks
```

### Professional Mixing

```
You: "Set up parallel compression: create a bus called 'Parallel Comp',
     send all drum tracks to it at -10dB, and keep the original tracks active"

Claude: ✓ Created bus 'Parallel Comp'
        ✓ Routed drum tracks with sends at -10dB
        ✓ Original tracks remain active
        Your parallel compression chain is ready!

You: "Check my mix for phase issues and clipping"

Claude: Master: -8.2dB peak, -14.5 LUFS, no clipping ✓
        Phase correlation: +0.85 (good stereo image) ✓
        ⚠ Track 1 (Drums): Phase correlation -0.15 (check phase)
```

### Automation Recording

```
You: "Set track 3 to touch mode for gain automation, go to bar 8,
     and start playback so I can ride the fader"

Claude: ✓ Track 3: Gain automation set to TOUCH mode
        ✓ Positioned at bar 8
        ✓ Playback started
        Move the fader - automation is recording!
```

**More examples**: [docs/EXAMPLE_CONVERSATIONS.md](docs/EXAMPLE_CONVERSATIONS.md)

## 🏗️ Architecture

Ardour MCP uses a three-layer architecture for bidirectional communication:

```
┌─────────────────┐
│   AI Assistant  │  Claude Desktop, etc.
└────────┬────────┘
         │ MCP Protocol (stdio)
┌────────▼────────┐
│   MCP Server    │  ardour_mcp
│  ┌──────────┐   │
│  │  Tools   │   │  111 registered tools
│  └────┬─────┘   │
│  ┌────▼─────┐   │
│  │  State   │   │  Cached Ardour state with auto-updates
│  └────┬─────┘   │
│  ┌────▼─────┐   │
│  │OSC Bridge│   │  Bidirectional OSC communication
│  └────┬─────┘   │
└───────┼─────────┘
        │ OSC Protocol (UDP)
┌───────▼─────────┐
│     Ardour      │  Digital Audio Workstation
└─────────────────┘
```

**Key Features**:
- **Bidirectional OSC**: Send commands and receive real-time feedback
- **State Caching**: Fast queries without OSC round-trips
- **Automatic Updates**: State synchronizes with Ardour changes
- **Error Handling**: Comprehensive validation and error messages

See [ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed design.

## 🧪 Testing

**581 comprehensive tests** with **86% code coverage**:

```bash
# Run full test suite
uv run pytest

# Run with coverage report
uv run pytest --cov=src/ardour_mcp --cov-report=term-missing

# Run specific test file
uv run pytest tests/test_automation.py
```

**Test Coverage by Module**:
- Automation: 98%
- Metering: 96%
- Advanced Mixer: 100%
- State Management: 100%
- Integration Tests: Complete

See [COVERAGE_REPORT.md](docs/COVERAGE_REPORT.md) for detailed analysis.

## 🤝 Contributing

We welcome contributions from developers, musicians, and audio engineers!

**Ways to Contribute**:
- 🐛 Report bugs and request features via [GitHub Issues](https://github.com/raibid-labs/ardour-mcp/issues)
- 💻 Submit pull requests (see [CONTRIBUTING.md](CONTRIBUTING.md))
- 📖 Improve documentation
- 🎵 Share usage examples and workflows
- ✨ Test and provide feedback

**Good First Issues**: Look for issues labeled [`good first issue`](https://github.com/raibid-labs/ardour-mcp/labels/good%20first%20issue)

## 📦 Publishing

Ardour MCP can be published to multiple MCP registries:

- **PyPI**: Python package (already configured)
- **MCP Community Registry**: Official MCP server catalog
- **GitHub MCP Registry**: Curated integration with VS Code/Claude Desktop
- **Docker MCP Registry**: Containerized deployments

**Publication is automated** via GitHub Actions on release tags.

See [REGISTRY_PUBLISHING.md](docs/REGISTRY_PUBLISHING.md) for complete automation plans.

## 📋 Releases

**Quick Release**:
```bash
# Check what would be released
just release-status

# Create and push patch release
just release-auto-patch

# Create and push minor release
just release-auto-minor
```

**Release Workflows**:
1. **Manual** - Full control with `just release-patch`
2. **Semi-Automated** - One command with `just release-auto-patch`
3. **Fully Automated** - PR-based with Release Please

**Version Scheme**: [Semantic Versioning 2.0.0](https://semver.org/) (MAJOR.MINOR.PATCH)

See [RELEASING.md](docs/RELEASING.md) for complete release documentation.

## 🎓 Resources

### Ardour Resources
- [Ardour Official Site](https://ardour.org/)
- [Ardour Manual](https://manual.ardour.org/)
- [Ardour OSC Documentation](https://manual.ardour.org/using-control-surfaces/controlling-ardour-with-osc/)
- [Ardour Forums](https://discourse.ardour.org/)

### MCP Resources
- [MCP Specification](https://modelcontextprotocol.io/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Community Registry](https://registry.modelcontextprotocol.io/)
- [Claude Desktop](https://claude.ai/desktop)

### Project Links
- [GitHub Repository](https://github.com/raibid-labs/ardour-mcp)
- [Issue Tracker](https://github.com/raibid-labs/ardour-mcp/issues)
- [Discussions](https://github.com/raibid-labs/ardour-mcp/discussions)
- [Releases](https://github.com/raibid-labs/ardour-mcp/releases)

## 🎯 Roadmap

### ✅ Completed Phases

**Phase 1 (MVP)** - November 2024
- Core transport, session, track, and mixer functionality
- 27 MCP tools, 98 tests

**Phase 2 (Essential Features)** - November 2024
- Recording control, navigation, enhanced mixer
- 86 total tools, 293 tests

**Phase 3 (Advanced Mixing)** - January 2025
- Advanced mixer, automation, professional metering
- 111 total tools, 581 tests, 86% coverage

### 📋 Planned Phases

**Phase 4 (Production)** - Q1 2025
- Plugin parameter automation
- Region editing and manipulation
- Snapshot and template management
- Enhanced session management

**Phase 5 (Advanced Features)** - Q2 2025
- MIDI note editing
- Time signature and tempo automation
- Advanced routing (sidechain, matrix)
- Video timeline synchronization

See [ROADMAP.md](docs/ROADMAP.md) for detailed feature breakdown.

## 📊 Project Stats

| Metric | Value |
|--------|-------|
| **Total MCP Tools** | 111 |
| **Tool Methods** | 111 across 9 categories |
| **Test Suite** | 581 tests passing |
| **Code Coverage** | 86% |
| **Python Version** | 3.11+ (3.10 supported) |
| **Ardour Versions** | 8.x, 9.x |
| **Latest Release** | v0.3.0 |
| **License** | MIT |

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **[Ardour](https://ardour.org/)** - Professional open-source DAW
- **[Anthropic](https://www.anthropic.com/)** - Model Context Protocol
- **MCP Community** - Protocol specification and tooling
- **[tubone24](https://github.com/tubone24)** - MIDI MCP Server integration
- **Contributors** - Everyone who has contributed code, documentation, and feedback
- **Testers** - Community members providing valuable feedback

## 📧 Contact

- **Issues**: [GitHub Issue Tracker](https://github.com/raibid-labs/ardour-mcp/issues)
- **Discussions**: [GitHub Discussions](https://github.com/raibid-labs/ardour-mcp/discussions)
- **Email**: contact@raibid-labs.com
- **Maintainer**: Raibid Labs

---

**Built with ❤️ for the open-source audio and AI communities**

*First MCP server for professional audio production • Bridging creativity and AI assistance*
