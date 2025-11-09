---
name: dev-docs-fetcher
description: Fetch and analyze developer documentation from any website. Use when user requests API references, library guides, or technical documentation.
version: 1.0.0
allowed-tools:
  - fetch
---

# Developer Documentation Fetcher

## Overview

This skill enables fetching and processing developer documentation from websites using the fetch MCP server. The fetch tool retrieves web content and converts it to markdown for easier analysis and presentation.

## Core Capabilities

The fetch MCP server provides a `fetch` tool with these parameters:
- `url` (required): The URL to fetch
- `max_length` (optional): Maximum characters to return (default: 5000)
- `start_index` (optional): Starting character position for chunked reading (default: 0)
- `raw` (optional): Get raw HTML instead of markdown conversion (default: false)

## When to Use This Skill

Use this skill when the user asks for:
- Current API documentation (e.g., "Get the latest React hooks documentation")
- Library references (e.g., "Show me Python's requests library docs")
- Framework guides (e.g., "Fetch Next.js routing documentation")
- Package documentation from npm, PyPI, or other registries
- Technical specifications or RFCs
- Up-to-date SDK documentation

## Workflow

### 1. Identify Documentation URL

When the user requests documentation:

**If URL is provided:** Use it directly

**If library/framework name is provided:** Determine the official documentation URL:
- Python libraries: `https://docs.python.org/3/library/{module}.html`
- PyPI packages: `https://pypi.org/project/{package}/` or check package homepage
- React: `https://react.dev/reference/react/{topic}`
- Node.js: `https://nodejs.org/api/{module}.html`
- MDN Web Docs: `https://developer.mozilla.org/en-US/docs/Web/{topic}`
- GitHub repos: Check README or docs folder

**If unsure:** Ask the user for the specific documentation URL

### 2. Fetch the Documentation

Use the `fetch` tool to retrieve the content:

```
fetch(url="https://example.com/docs/api")
```

**Key considerations:**
- Start with default parameters (no max_length specified)
- The tool will return markdown-converted content
- Content is automatically truncated at 5000 characters by default

### 3. Handle Large Documentation Pages

If the response is truncated (reaches max_length), use chunking to read more:

**To read the next chunk:**
```
fetch(url="https://example.com/docs/api", start_index=5000)
```

**Continue reading in chunks:**
- Increment start_index by the chunk size (e.g., 5000, 10000, 15000)
- Stop when you've found the needed information
- Inform the user if you're reading a large page in sections

**Example chunking workflow:**
1. First call: `fetch(url=doc_url)` → reads characters 0-5000
2. If more needed: `fetch(url=doc_url, start_index=5000)` → reads 5000-10000
3. Continue until information is found or page is complete

### 4. Extract and Format Key Information

Focus on extracting:

**For API Documentation:**
- Function/method signatures
- Parameters and their types
- Return values
- Code examples
- Common use cases
- Error handling

**For Library Documentation:**
- Installation instructions
- Basic usage examples
- Core concepts
- Configuration options
- Best practices

**For Framework Documentation:**
- Getting started guide
- Key features
- Code patterns
- Integration examples

### 5. Present the Information

Format the response clearly:
- Use code blocks for examples
- Highlight important parameters or methods
- Include direct links to specific sections
- Summarize complex information
- Provide context for how to use the APIs

## Tips for Effective Documentation Fetching

### Multiple Related Pages

For comprehensive documentation spanning multiple pages:
1. Start with the main/overview page
2. Ask user which specific topics they want to explore
3. Fetch additional pages as needed
4. Synthesize information across pages

### Dealing with Navigation/Headers

The markdown conversion may include navigation menus and headers:
- Focus on the main content area
- Skip repetitive navigation elements
- Extract code examples and API signatures
- Summarize prose sections

### Version-Specific Documentation

When fetching versioned documentation:
- Confirm the version with the user if not specified
- Include version number in URLs (e.g., `/v2/`, `/3.11/`)
- Note the version in your response
- Mention if newer versions exist

### Raw HTML vs Markdown

Use `raw=true` only when:
- The markdown conversion is losing important structure
- You need to parse specific HTML elements
- The page has complex formatting that markdown doesn't preserve

Default to markdown (raw=false) for easier reading and analysis.

## Example Usage Scenarios

### Scenario 1: Quick API Lookup
```
User: "How do I use Python's pathlib.Path?"
1. fetch(url="https://docs.python.org/3/library/pathlib.html")
2. Extract Path class documentation
3. Show key methods and examples
```

### Scenario 2: Framework Feature Documentation
```
User: "Get me Next.js App Router documentation"
1. fetch(url="https://nextjs.org/docs/app/building-your-application/routing")
2. If truncated, read additional chunks
3. Extract routing patterns and examples
4. Summarize with code snippets
```

### Scenario 3: Package Research
```
User: "What can the requests library do?"
1. fetch(url="https://requests.readthedocs.io/")
2. Extract feature overview
3. Show common usage patterns
4. Highlight key capabilities
```

## Common Documentation Sources

- **Python**: docs.python.org, readthedocs.io
- **JavaScript/Node**: nodejs.org, developer.mozilla.org
- **React/Next.js**: react.dev, nextjs.org
- **Package Registries**: pypi.org, npmjs.com
- **API References**: Often at `/docs`, `/api`, `/reference` paths
- **GitHub**: Raw README.md or docs/ folder content

## Error Handling

If fetch fails:
- Check if the URL is accessible
- Try alternative documentation sources
- Suggest official documentation links
- Offer to search for the documentation URL

If content is empty or malformed:
- Try with `raw=true` to see HTML structure
- Try alternative URL patterns (with/without trailing slash, www vs non-www)
- Check if the page requires JavaScript rendering (fetch won't execute JS)
