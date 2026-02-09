---
name: arxiv
description: Search, retrieve, and analyze scientific papers from arXiv. Use when the user asks for academic research, specific paper IDs, summaries of latest research on a topic, or needs to find related work.
---

# arXiv MCP Skill

This skill enables deep research into academic papers hosted on arXiv.

## Quick Start

1. **Search for papers**: Use keywords to find relevant research.
2. **Get paper details**: Fetch metadata, abstracts, and categories for a specific arXiv ID.
3. **Analyze research**: Summarize or extract insights from paper abstracts.

## Workflow

### 1. Searching for Research
When a user asks for "latest papers on X" or "research about Y":
- Use `mcporter call arxiv.search` or equivalent.
- **Category Search**: You can filter by category using `cat:category_name`. 
    - Example: `cat:cs.AI` for Artificial Intelligence.
    - Example: `cat:cs.LG` for Machine Learning.
- Format the output as a list with titles, authors, and dates.

### 2. Deep Dive into a Paper
When a user provides an arXiv ID (e.g., `1706.03762`):
- Use `mcporter call arxiv.get_details` or equivalent.
- Present the abstract and key info.

### 3. Summarization
If the user wants a summary:
- Fetch the abstract and provide a high-level overview.

## MCP Configuration
This skill relies on the `arxiv` MCP server. Ensure it is configured in `mcporter`:
```json
{
  "command": "npx -y arxiv-mcp-server"
}
```

## Related Skills
- **summarize**: Use for long-form analysis if the paper text is available via URL.
- **browser**: Use to download PDFs or view papers on the arXiv website.
