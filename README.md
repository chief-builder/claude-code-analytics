# Claude Code Analytics Dashboard

A local web dashboard for analyzing Claude Code usage data from your `~/.claude` directory.

**[View Screenshots & Demo](https://chief-builder.github.io/claude-code-analytics/)**

## Overview

This single-file HTML dashboard provides comprehensive analytics for your Claude Code usage, including session tracking, token consumption, cost estimation, tool usage patterns, and project breakdowns. All data stays local - nothing is sent to any server.

## Features

### Overview Tab
- **Summary Cards**: Total sessions, messages, API calls, tool calls, and cost
- **Daily Activity Chart**: Messages, sessions, and tool calls over time
- **Hourly Distribution**: Activity patterns throughout the day

### Sessions Tab
- **Session List**: All sessions with duration, message count, API calls, tokens, and tools used
- **Session Details**: Start/end time, models used, project association
- **Sortable Table**: Click headers to sort by any metric

### Tokens Tab
- **Token Usage by Model**: Input, output, cache read, and cache creation tokens
- **Model Breakdown**: Per-model statistics with costs
- **Cache Efficiency**: Cache hit rates and savings analysis

### Tools Tab
- **Tool Frequency Chart**: Most-used tools ranked by call count
- **Bash Command Details**: Actual commands executed with timestamps
- **Tool Distribution**: Visual breakdown of tool usage

### Projects Tab
- **Project List**: All projects with session counts and activity
- **Project Coverage**: Percentage of sessions across projects
- **Date-Filtered View**: Shows only projects active in selected date range

### Network Tab
- **API Call Statistics**: Total calls, calls by model, peak usage
- **Data Transfer Estimates**: Bytes sent/received (derived from token counts)
- **Performance Metrics**: Average session duration, message frequency

### Prompts Tab
- **Agent Loop Insights**: Understand how Claude Code works under the hood
- **Summary Cards**: User prompts count, avg tools/turn, thinking rate, model mix
- **Model Distribution Chart**: Visualize Opus vs Sonnet vs Haiku usage
- **Turn Type Distribution**: See text-only vs tool-using vs thinking turns
- **Conversation History**: Searchable list of all prompts with timestamps
- **Prompt Details**: Click any prompt to see full exchange with tool calls and token usage
- **Source Filter**: Filter prompts by source type (User, System, or Tool)

#### Source Types
| Source | Badge Color | Description |
|--------|-------------|-------------|
| User | Green | Prompts you actually typed |
| System | Purple | System-generated messages (`isMeta: true`) |
| Tool | Amber | Tool result messages returned to Claude |

#### Agent Loop Data Available
| Metric | Description |
|--------|-------------|
| Avg Tools/Turn | Average tool calls per assistant response |
| Thinking Rate | Percentage of responses using extended thinking |
| Model Mix | Distribution across Opus, Sonnet, and Haiku |
| Tool Details | Each tool called with its input (command, file path, etc.) |
| Token Breakdown | Input, output, cache read, and cache create per response |

### Custom Metrics Tab
- **User-Defined Formulas**: Create custom metrics using available data
- **Saved Configurations**: Metrics persist in localStorage
- **Available Variables**: totalMessages, totalSessions, totalTokens, totalCost, etc.

## Data Sources

The dashboard reads from files in your `~/.claude` directory:

| File | Purpose |
|------|---------|
| `stats-cache.json` | Pre-aggregated daily metrics (fast load) |
| `history.jsonl` | Session metadata and project mapping |
| `projects/*/[session-id].jsonl` | Detailed session logs with tool usage |

### Session Log Entry Structure

Each entry in session logs contains fields that help understand the agent loop:

| Field | Description |
|-------|-------------|
| `type` | Entry type: `user`, `assistant`, `file-history-snapshot` |
| `isMeta` | `true` = system-generated message, `false`/absent = user-typed prompt |
| `isSidechain` | `true` = subagent/parallel execution, `false` = main conversation |
| `message.model` | Model used: `claude-opus-4-5-*`, `claude-sonnet-*`, `claude-haiku-*` |
| `message.content[]` | Array of content blocks: `text`, `tool_use`, `tool_result`, `thinking` |
| `message.usage` | Token counts: `input_tokens`, `output_tokens`, `cache_read_input_tokens` |
| `uuid` / `parentUuid` | Message threading for conversation flow |
| `toolUseResult` | Tool execution output (for tool result entries) |

**Distinguishing User Prompts:**
- **User-typed prompts**: `type: "user"` with `isMeta: false` or absent
- **System messages**: `type: "user"` with `isMeta: true` (caveats, metadata)
- **Tool results**: `type: "user"` with `message.content` as array containing `tool_result`

## Cost Calculation

Costs are calculated from **real token data** in your logs using Anthropic's pricing:

```
Total Cost = Sum for each model of:
  (inputTokens * input_price / 1,000,000) +
  (outputTokens * output_price / 1,000,000) +
  (cacheReadTokens * cacheRead_price / 1,000,000) +
  (cacheCreateTokens * cacheCreate_price / 1,000,000)
```

### Pricing (per 1M tokens)

| Model | Input | Output | Cache Read | Cache Create |
|-------|-------|--------|------------|--------------|
| Claude Opus 4.5 | $15.00 | $75.00 | $1.50 | $18.75 |
| Claude Sonnet 4.5 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Sonnet 4 | $3.00 | $15.00 | $0.30 | $3.75 |

**Cache Pricing Logic**:
- **Cache Read**: 10% of input price (90% discount when reading from cache)
- **Cache Create**: 125% of input price (25% premium to populate cache, but saves on future reads)

## Usage

1. **Open the Dashboard**: Open `claude-code-analytics.html` in any modern browser
2. **Load Data**: Click "Load .claude Directory" and select your `~/.claude` folder
3. **Explore Tabs**: Navigate between tabs to view different analytics
4. **Filter by Date**: Use the date pickers to analyze specific time periods
5. **Export Data**: Use the export buttons to download CSV or JSON

### Date Filtering

All tabs respond to date filtering:
- Select "From" and "To" dates
- Click "Apply Filter" to update all views
- Click "Reset" to show all data

### Serving Locally (Optional)

For development or if you encounter CORS issues:

```bash
cd /path/to/CC_Usage
python3 -m http.server 8888
# Open http://localhost:8888/claude-code-analytics.html
```

## Technical Details

- **Single HTML File**: No build process, no dependencies to install
- **Chart.js 4.x**: Loaded from CDN for visualizations
- **Vanilla JavaScript**: No framework dependencies
- **CSS Grid**: Modern responsive layout
- **File API**: Browser-based file loading (no server required)
- **LocalStorage**: Custom metrics persist between sessions

## File Structure

```
CC_Usage/
  claude-code-analytics.html   # Main dashboard (single file)
  README.md                    # This file
  docs/
    IMPLEMENTATION_PLAN.md     # Technical implementation notes
```

## Privacy

- All data processing happens locally in your browser
- No data is transmitted to any external server
- Your Claude Code usage data never leaves your machine

## Browser Compatibility

Tested with:
- Chrome 90+
- Firefox 90+
- Safari 14+
- Edge 90+

Requires JavaScript enabled and support for the File System Access API or `webkitdirectory` attribute.

## License

MIT License - Feel free to modify and distribute.
