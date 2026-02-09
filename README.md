# OpenClaw Research Skills

Custom OpenClaw skills for academic research and financial market analysis.

## Skills

### 📚 arxiv
Search and analyze scientific papers from arXiv. MCP-based skill for academic research in AI, Multi-Agent Systems, and related fields.

**Features:**
- Search papers by category, keyword, date range
- Retrieve paper metadata, abstracts, PDFs
- Support for cs.AI, cs.MA, and other arXiv categories

### 📈 seeking_alpha
Access Seeking Alpha data for stock research and market insights.

**Features:**
- Quant ratings and grades (Value, Growth, Profitability, Momentum, etc.)
- Analyst articles and summaries
- News, earnings call transcripts, press releases
- Dividend history
- Market status

**Note:** Requires premium Seeking Alpha account credentials (not included).

### 💹 yahooquery
Comprehensive access to Yahoo Finance data via Python library.

**Features:**
- Real-time pricing and historical data
- Fundamentals (income statement, balance sheet, cash flow)
- Analyst estimates and recommendations
- Options data, dividends, splits
- News and research

## Installation

1. Place these skill folders in your OpenClaw workspace:
   ```
   ~/.openclaw/workspace/skills/
   ```

2. For seeking_alpha, add your credentials:
   ```
   skills/seeking_alpha/sa_credentials.json
   ```

3. Restart OpenClaw to load the skills:
   ```bash
   openclaw gateway restart
   ```

## Requirements

- **arxiv**: `mcporter` with arXiv MCP server configured
- **seeking_alpha**: Premium Seeking Alpha account, `requests` library
- **yahooquery**: `yahooquery` Python package

## Usage

These skills are automatically loaded into OpenClaw's prompt. The agent can read and follow the instructions in each `SKILL.md` file.

## License

MIT
