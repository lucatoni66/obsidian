---
title: "Welcome to FastMCP"
source: "https://gofastmcp.com/getting-started/welcome"
author:
published:
created: 2026-04-30
description: "The fast, Pythonic way to build MCP servers, clients, and applications."
tags:
  - "clippings mcp"
---
<video src="https://mintcdn.com/fastmcp/-fU9AuXWlaP61Fuq/assets/brand/f-watercolor-waves-4-animated.mp4?fit=max&amp;auto=format&amp;n=-fU9AuXWlaP61Fuq&amp;q=85&amp;s=5eb68c6916a4c338185cae8b742f144d" controls=""></video><video src="https://mintcdn.com/fastmcp/-fU9AuXWlaP61Fuq/assets/brand/f-watercolor-waves-4-dark-animated.mp4?fit=max&amp;auto=format&amp;n=-fU9AuXWlaP61Fuq&amp;q=85&amp;s=aa3158596f22114e69a601d9c68aa8e4" controls=""></video>

**FastMCP is the standard framework for building MCP applications.** The [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) connects LLMs to tools and data. FastMCP gives you everything you need to go from prototype to production — build servers that expose capabilities, connect clients to any MCP service, and give your tools interactive UIs:

```python
from fastmcp import FastMCP

mcp = FastMCP("Demo 🚀")

@mcp.tool
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

if __name__ == "__main__":
    mcp.run()
```

## Move Fast and Make Things

The [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) lets you give agents access to your tools and data. But building an effective MCP application is harder than it looks.

FastMCP handles all of it. Declare a tool with a Python function, and the schema, validation, and documentation are generated automatically. Connect to a server with a URL, and transport negotiation, authentication, and protocol lifecycle are managed for you. You focus on your logic, and the MCP part just works: **with FastMCP, best practices are built in.**

**That’s why FastMCP is the standard framework for working with MCP.** FastMCP 1.0 was incorporated into the official MCP Python SDK in 2024. Today, the actively maintained standalone project is downloaded a million times a day, and some version of FastMCP powers 70% of MCP servers across all languages.

FastMCP has three pillars:

![servers-card](https://mintcdn.com/fastmcp/uaPe2cZCul164Sax/assets/images/servers-card.png?fit=max&auto=format&n=uaPe2cZCul164Sax&q=85&s=2cddc3be3355623b1b81024811a9f443)

## [Servers](https://gofastmcp.com/servers/server)

Expose tools, resources, and prompts to LLMs.

![apps-card](https://mintcdn.com/fastmcp/uaPe2cZCul164Sax/assets/images/apps-card.png?fit=max&auto=format&n=uaPe2cZCul164Sax&q=85&s=865d32af9c41cf6266a09a8a4fc03fe1)

## [Apps](https://gofastmcp.com/apps/overview)

Give your tools interactive UIs rendered directly in the conversation.

![clients-card](https://mintcdn.com/fastmcp/uaPe2cZCul164Sax/assets/images/clients-card.png?fit=max&auto=format&n=uaPe2cZCul164Sax&q=85&s=fbb306d0b3e0858afd1eef7aeacc02cf)

## [Clients](https://gofastmcp.com/clients/client)

Connect to any MCP server — local or remote, programmatic or CLI.

**[Servers](https://gofastmcp.com/servers/server)** wrap your Python functions into MCP-compliant tools, resources, and prompts. **[Clients](https://gofastmcp.com/clients/client)** connect to any server with full protocol support. And **[Apps](https://gofastmcp.com/apps/overview)** give your tools interactive UIs rendered directly in the conversation.

Ready to build? Start with the [installation guide](https://gofastmcp.com/getting-started/installation) or jump straight to the [quickstart](https://gofastmcp.com/getting-started/quickstart).

FastMCP is made with 💙 by [Prefect](https://www.prefect.io/).

## Run FastMCP in production with Horizon

FastMCP is the standard way to build MCP servers. **[Prefect Horizon](https://www.prefect.io/horizon?utm_source=gofastmcp&utm_medium=docs&utm_campaign=docs_welcome&utm_content=welcome_body)** is the enterprise MCP gateway for running them safely.

Built by the FastMCP team, Horizon packages the best practices we’ve learned shipping the world’s most popular MCP framework.

Deploy FastMCP servers from GitHub with branch previews and instant rollback. Create a private registry of every MCP your company uses. Secure access with SSO and tool-level RBAC. Get audit logs, observability, and governance across your MCP stack. Remix approved tools into purpose-built endpoints for teams and agents.

Start with FastMCP. [Scale with Horizon →](https://www.prefect.io/horizon?utm_source=gofastmcp&utm_medium=docs&utm_campaign=docs_welcome&utm_content=welcome_cta)

**This documentation reflects FastMCP’s `main` branch**, meaning it always reflects the latest development version. Features are generally marked with version badges (e.g. `New in version: 3.0.0`) to indicate when they were introduced. Note that this may include features that are not yet released.

## LLM-Friendly Docs

The FastMCP documentation is available in multiple LLM-friendly formats:

### MCP Server

The FastMCP docs are accessible via MCP! The server URL is `https://gofastmcp.com/mcp`.

In fact, you can use FastMCP to search the FastMCP docs:

```python
import asyncio
from fastmcp import Client

async def main():
    async with Client("https://gofastmcp.com/mcp") as client:
        result = await client.call_tool(
            name="SearchFastMcp",
            arguments={"query": "deploy a FastMCP server"}
        )
    print(result)

asyncio.run(main())
```

### Text Formats

The docs are also available in [llms.txt format](https://llmstxt.org/):

- [llms.txt](https://gofastmcp.com/llms.txt) - A sitemap listing all documentation pages
- [llms-full.txt](https://gofastmcp.com/llms-full.txt) - The entire documentation in one file (may exceed context windows)

Any page can be accessed as markdown by appending `.md` to the URL. For example, this page becomes `https://gofastmcp.com/getting-started/welcome.md`.

You can also copy any page as markdown by pressing “Cmd+C” (or “Ctrl+C” on Windows) on your keyboard.