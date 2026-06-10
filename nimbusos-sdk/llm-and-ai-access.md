---
description: How AI tools can read, index, and query the published NimbusOS SDK documentation.
---

# LLM and AI Access

This GitBook space is structured so humans and AI tools can read the same source of truth.

## GitBook LLM surfaces

After this space is published with GitBook, AI tools can use these generated endpoints:

| Endpoint | Purpose |
| -------- | ------- |
| `<published-site-url>/llms.txt` | Index of documentation pages with Markdown URLs. |
| `<published-site-url>/llms-full.txt` | Full documentation content in one text file. |
| `<published-site-url>/~gitbook/mcp` | GitBook MCP server for structured resource access. |
| `<page-url>.md` | Markdown version of any individual published page. |

Use `/llms.txt` for discovery. Use `/llms-full.txt` when an assistant needs the whole docs set as context. Use the MCP endpoint when the AI tool supports Model Context Protocol resources.

## Querying pages

GitBook Markdown pages can answer focused questions through the `ask` query parameter:

```http
GET <page-url>.md?ask=<specific-question>
```

Questions should be self-contained and specific. For example:

```http
GET <published-site-url>/nimbusos-sdk/python-api/api.md?ask=How do I publish a relative waypoint?
```

## Content maintenance rules

Keep these docs effective for GitBook AI Search and external LLM tools:

* Keep every user-facing page in `SUMMARY.md` so it is discoverable.
* Give each page one clear `#` heading.
* Use concise headings, short paragraphs, tables, and runnable examples.
* Prefer current SDK method names over legacy aliases.
* Verify Python and CLI examples against the current SDK source before publishing.
* Avoid hiding useful pages unless they should be omitted from human navigation.

## Current SDK ground truth for maintainers

When updating examples, verify against the current NimbusOS SDK source:

* `src/nimbusos_sdk/client.py`
* `src/nimbusos_sdk/cli.py`
* `pyproject.toml`

The docs should describe the current SDK surface, including `publish_autonomy_request(...)`, `publish_relative_waypoint(...)`, `publish_waypoint_speed(...)`, `selected_state`, `autonomy_status`, `camera_overlay`, and the installed `nimbusos-*` command-line tools.

Run the local readiness check before publishing:

```bash
python scripts/check_llm_ready.py
```

Set `NIMBUSOS_SDK_ROOT` if the SDK checkout is not in the default local location used by the script.
