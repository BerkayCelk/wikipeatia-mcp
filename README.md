# WikiPeatia MCP Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-8A2BE2)](https://modelcontextprotocol.io)

An [MCP](https://modelcontextprotocol.io) server that gives AI assistants access to
**[WikiPeatia](https://wikipeatia.org)** -- a community wiki and full-text corpus of
**Ray Peat**'s work on bioenergetic health -- plus Ray Peat's original articles on
[raypeat.com](https://raypeat.com).

**32 tools · no API keys · read-only · works with any MCP client** (Claude Desktop, Claude Code, Cursor, Windsurf, Hermes, ...).

## What it gives your assistant

- **Search the entire Ray Peat corpus** (~7,000 documents / ~220,000 text chunks): articles, books, interviews (audio transcripts), newsletters, forum posts, email Q&A and wiki pages.
- Hybrid retrieval: **full-text search**, **semantic (embedding) search**, and **LLM-powered Q&A** -- a quick grounded answer mode plus a deep multi-source synthesis mode with citations.
- **Read & explore the WikiPeatia wiki**: pages, sections, categories, the Library namespace (Ray Peat's books / newsletters), backlinks, recent changes, random pages.
- **Read Ray Peat's original articles** from raypeat.com.

## Quick start

### Option A -- run with uvx (no install)

```bash
uvx --from git+https://github.com/BerkayCelk/wikipeatia-mcp wikipeatia-mcp
```

([uv](https://docs.astral.sh/uv/) will fetch and run the server; it speaks MCP over stdio.)

### Option B -- clone and run

```bash
git clone https://github.com/BerkayCelk/wikipeatia-mcp
cd wikipeatia-mcp
pip install -r requirements.txt
python server.py
```

## Client setup

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "wikipeatia": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/BerkayCelk/wikipeatia-mcp", "wikipeatia-mcp"]
    }
  }
}
```

**Claude Code**:

```bash
claude mcp add wikipeatia -- uvx --from git+https://github.com/BerkayCelk/wikipeatia-mcp wikipeatia-mcp
```

**Cursor** (`~/.cursor/mcp.json`) and **Windsurf**: same JSON shape as Claude Desktop.
**Hermes**: add to your `config.yaml` under `mcp_servers` (see `examples/hermes_config.yaml`).

Ready-made config files live in [`examples/`](examples/).

## Tools (32)

### Wiki (17)

| Tool | Description |
| --- | --- |
| `wiki_search` | Search wiki pages (title/content) |
| `wiki_get_page` | Raw page content (wikitext or HTML) |
| `wiki_get_page_summary` | First-paragraph summary |
| `wiki_read` | Read a page as plain text (whole page or one section) |
| `wiki_suggest` | Title autocomplete |
| `wiki_list_categories` | All categories |
| `wiki_get_category_members` | Pages in a category |
| `wiki_list_all_pages` | All pages, optional prefix filter |
| `wiki_list_library` | Library namespace (books/newsletters) |
| `wiki_get_page_categories` | Categories of a page |
| `wiki_get_page_links` | Outgoing links |
| `wiki_get_backlinks` | Pages linking here |
| `wiki_get_page_sections` | Section headings (ToC) |
| `wiki_get_page_info` | Page metadata |
| `wiki_get_random` | Random page(s) |
| `wiki_get_recent_changes` | Recent edits |
| `wiki_get_site_stats` | Wiki statistics |

### Corpus (12)

| Tool | Description |
| --- | --- |
| `corpus_search` | Full-text search (filters: `doc_type`, `author`, `series`, `ray_only`) |
| `corpus_semantic_search` | Semantic/embedding search + glossary terms (same filters) |
| `corpus_ask` | Quick LLM answer (~2-10 s) |
| `corpus_deep_ask` | Deep LLM synthesis with [S1][S2] citations (~30-120 s) |
| `corpus_suggest` | Autocomplete (terms + doc_ids) |
| `corpus_get_doc` | Full document content (chunks; speaker/timestamps for interviews) |
| `corpus_list_doc_types` | Document types + counts |
| `corpus_list_authors` | Authors + counts |
| `corpus_list_series` | Series/collections |
| `corpus_library` | Browse library metadata (filter by series) |
| `corpus_topics` | Topic tags |
| `corpus_health` | Corpus status (docs, chunks, semantic online?) |

### Ray Peat articles (3)

| Tool | Description |
| --- | --- |
| `raypeat_list_articles` | List all raypeat.com articles |
| `raypeat_read_article` | Read one article (full text) |
| `raypeat_search_articles` | Search article titles |

## Example prompts

- "Search the corpus for what Ray Peat wrote about PUFA and insulin resistance."
  → `corpus_search(query="PUFA insulin resistance", limit=10)`
- "Only interviews from the Ask the Herb Doctor series."
  → `corpus_search(query="thyroid", series="Ask the Herb Doctor", doc_type="interview")`
- "What has Georgi Dinkov said about serotonin?" (non-Peat author)
  → `corpus_search(query="serotonin", author="Georgi Dinkov (Haidut)", ray_only=false)`
- "Explain the Randle cycle in Ray Peat's framework." (deep synthesis)
  → `corpus_deep_ask(question="Explain the Randle cycle...")`
- "Open the Thyroid entry in the Library namespace."
  → `wiki_read(title="Library:Thyroid")`

## Environment variables

| Variable | Default | Meaning |
| --- | --- | --- |
| `WIKIPEATIA_TIMEOUT` | `30` | HTTP timeout (seconds) for regular calls |
| `WIKIPEATIA_DEEP_TIMEOUT` | `120` | HTTP timeout for semantic / ask endpoints |

## Notes

- **No API keys, read-only.** The server calls public endpoints of `wikipeatia.org` and `search.wikipeatia.org`; be considerate (avoid tight loops -- the upstream API rate-limits bursts).
- `corpus_deep_ask` can take 30-120 s. Prefer `corpus_ask` when you just need a fast answer.
- The corpus includes third-party authors (Georgi Dinkov, Danny Roddy, forum members, ...). Use `ray_only` / `author` / `series` filters to scope a search.
- This is an **unofficial community project**, not affiliated with WikiPeatia or raypeat.com. Content rights belong to the respective sources.

## Development

```bash
uv run --with fastmcp --with httpx python smoke_test.py
```

The smoke test spawns the server over stdio and exercises a handful of live calls.

## License

MIT -- see [LICENSE](LICENSE).

## Credits

- [WikiPeatia](https://wikipeatia.org) -- wiki + corpus
- [raypeat.com](https://raypeat.com) -- Ray Peat's original articles
- Built with [FastMCP](https://github.com/jlowin/fastmcp)
