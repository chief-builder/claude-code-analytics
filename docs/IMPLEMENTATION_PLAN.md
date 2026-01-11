# Claude Code Usage Analytics Dashboard

A local web dashboard for analyzing Claude Code usage from `~/.claude` data.

## Overview

Single-file HTML dashboard with Chart.js visualizations showing:
- Session activity over time
- Token usage by model with cost estimates
- Tool usage frequency
- Project breakdown
- Hourly activity patterns
- Custom user-defined metrics
- Network metrics (estimated bytes, timing, streaming analysis)

## Data Sources

| File | Purpose |
|------|---------|
| `stats-cache.json` | Pre-aggregated metrics (fast load) |
| `history.jsonl` | Session metadata + project mapping |
| `projects/*/` | Detailed session logs with tool usage |

## Implementation Plan

### Step 1: Create Single HTML Dashboard File
Create `claude-code-analytics.html` in the project directory with:
- Embedded CSS (dark theme, card layout)
- Chart.js loaded from CDN
- File input for loading `.claude` directory

### Step 2: Implement Data Loader
JavaScript class to parse:
- `stats-cache.json` - main metrics source
- `history.jsonl` - session/project linking
- Support for File System Access API or `<input webkitdirectory>`

### Step 3: Build Dashboard Views

**Overview Tab:**
- Summary cards: Total sessions (230), messages (19,502), tokens, estimated cost
- Daily activity bar chart (messages/sessions/tools)
- Hourly distribution bar chart (24 hours)

**Sessions Tab:**
- Session list with duration, message count
- Messages per session trend

**Tokens Tab:**
- Token usage by model (stacked bar)
- Cache efficiency (cache_read vs cache_creation)
- Cost estimate using pricing:
  - Opus: $15 input, $75 output per MTok
  - Sonnet: $3 input, $15 output per MTok

**Tools Tab:**
- Tool frequency bar chart
- Tool calls per session trend

**Projects Tab:**
- Project breakdown pie chart
- 12 tracked projects with usage comparison

**Custom Metrics Tab:**
- Config-driven custom metrics
- Formula-based calculations (e.g., productivity score)
- Stored in localStorage

**Network Tab:**
- **Estimated Data Transfer:**
  - Bytes sent: `inputTokens × 4` (approximate)
  - Bytes received: `outputTokens × 4` (approximate)
  - Total data: combined with cache tokens
- **Raw Token Metrics:**
  - Input/output token breakdown by model
  - Cache read vs cache creation tokens
  - Daily token consumption bar chart
- **Timing Analysis:**
  - Session duration distribution
  - Average session length trend
  - Message frequency over time
  - Peak activity hours
- **Streaming Indicators:**
  - Cache hit rate: `cacheRead / (cacheRead + input)`
  - Cache efficiency trends over time
  - Streaming savings estimate (tokens served from cache)

### Step 4: Add Export Functionality
- Export data as CSV/JSON
- Date range filtering

## File Structure

```
CC_Usage/
  claude-code-analytics.html   # Main dashboard (single file)
  custom-metrics.json          # Optional: custom metric definitions
  docs/
    IMPLEMENTATION_PLAN.md     # This file
```

## Technical Choices

- **Chart.js 4.x** - CDN loaded, no build needed
- **Vanilla JS** - No framework dependencies
- **CSS Grid** - Modern responsive layout
- **File API** - Browser-based file loading (no server required)

## Key Calculations

**Cost Estimation:**
```javascript
inputTokens * 0.015 + outputTokens * 0.075 +
cacheRead * 0.0015 + cacheCreate * 0.01875  // Opus pricing per 1K tokens
```

**Productivity Score:**
```javascript
totalMessages / activeHours
```

**Cache Efficiency:**
```javascript
cacheReadTokens / (cacheReadTokens + inputTokens)
```

**Estimated Bytes Sent:**
```javascript
(inputTokens + cacheCreationTokens) * 4  // ~4 bytes per token
```

**Estimated Bytes Received:**
```javascript
outputTokens * 4  // ~4 bytes per token
```

**Session Duration (from timestamps):**
```javascript
lastMessageTimestamp - firstMessageTimestamp  // per session
```

**Streaming/Cache Savings:**
```javascript
cacheReadTokens * tokenPrice * 0.9  // 90% savings on cached tokens
```

## Verification

1. Open `claude-code-analytics.html` in browser
2. Click "Load Data" and select `~/.claude` directory
3. Verify all charts render with expected data:
   - 230 sessions, 19,502 messages
   - Daily activity matches `dailyActivity` array
   - Hourly distribution shows peak at hours 23, 9, 10
4. Test date range filtering
5. Test CSV export
6. Verify Network tab shows:
   - Estimated bytes (~2.3 MB sent, ~5.5 MB received based on token counts)
   - Cache hit rate visualization
   - Session timing analysis

## Files to Modify/Create

1. **CREATE:** `/Users/chiefbuilder/Documents/Projects/CC_Usage/claude-code-analytics.html`
   - Single-file dashboard with embedded CSS/JS
   - ~1000 lines total (expanded for network metrics)
