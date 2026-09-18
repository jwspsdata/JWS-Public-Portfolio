# MCP Server — Headless Tool API for the Marketing & Forecasting Agent Teams

## What This Demonstrates

This project exposes the same LangGraph agent teams used in the **Marketing Analytics Team** and **Tool Calling** projects as a **Model Context Protocol (MCP) server** — a headless API endpoint that any MCP-compatible client (Claude Desktop, Claude Code, or a custom agent) can call as tools, with no browser or Streamlit UI required.

MCP is Anthropic's open protocol for cross-process tool communication. It standardizes how AI models discover and invoke external capabilities, so one tool server can be reused across different clients instead of being rebuilt for each one.

*Not published as its own Streamlit app — the underlying agent logic is already demonstrated live in the Marketing Analytics Team and Tool Calling apps. This project shows the same capabilities exposed a second, complementary way.*

---

## Architecture

```text
MCP Client (Claude Desktop / Claude Code / custom agent)
   │
   │  SSE or stdio transport
   ▼
AWS EC2 (us-east-1)
   │
   ├── nginx (port 80/443)
   │      │  SSL termination via Let's Encrypt
   │      │  Reverse proxy → mcp-server:8000
   │
   └── mcp-server (port 8000, internal only)
          │  FastMCP server — SSE transport
          │  Registers marketing + forecasting toolsets
          │
          ├── Marketing Analytics toolset (from the Marketing Analytics Team)
          │      product_expert, bi_sql, email_writer, segment_analysis
          │
          └── Forecasting toolset (from the Tool Calling project)
                 forecasting_run_workflow
```

---

## MCP Tools Registered

### Marketing Analytics

| Tool | Description |
| --- | --- |
| `marketing_run_workflow` | Full supervisor-orchestrated marketing analytics workflow |
| `product_expert` | Product knowledge retrieval via ChromaDB RAG |
| `bi_sql` | SQL query generation and execution against the leads database |
| `marketing_email_writer` | Draft targeted email campaigns |
| `segment_analysis` | Customer segment analysis from the leads-scored database |

### Forecasting

| Tool | Description |
| --- | --- |
| `forecasting_run_workflow` | Full forecasting pipeline — SQL aggregation, XGBoost forecast, Plotly chart JSON |

---

## Also Included: An MCP Client / Teaching Console

A companion Streamlit app acts as an MCP **client**, not just a server demo. It connects to the running MCP server, discovers its tool catalog live, lets you inspect each tool's JSON schema, fire manual tool calls, inspect raw request/response JSON, and optionally route natural-language questions through an LLM that dynamically converts each MCP tool's schema into a callable LangChain tool and decides which one to invoke.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| MCP Framework | FastMCP (SSE + stdio transports) |
| Agent Runtime | LangGraph, LangChain, OpenAI |
| Marketing RAG | ChromaDB (local vector store) |
| Forecasting Model | XGBoost (`XGBRegressor`) with conformal confidence intervals |
| Cloud | AWS EC2 (containerized via Docker Compose) |
| Reverse Proxy | Nginx with SSL termination |
| SSL | Let's Encrypt (Certbot) |

---

## Docker Compose Stack

Four containers managed together:

```yaml
services:
  mcp-server:      # FastMCP server on port 8000 (internal only)
  nginx:           # Reverse proxy on ports 80/443 with SSL
  certbot:         # Let's Encrypt initial certificate
  certbot-renew:   # Automatic certificate renewal every 12h
```

## Server Entry Point

```bash
# SSE transport (for AWS / network clients)
python -m mcp_server.server --transport sse --host 0.0.0.0 --port 8000

# stdio transport (for Claude Desktop local use)
python -m mcp_server.server --transport stdio
```

---

## Notes

- The EC2 instance is stopped when not in active use to avoid ongoing compute charges, consistent with how other AWS-deployed projects in this portfolio are managed.
- Transport security is handled via TLS (Nginx + Let's Encrypt); the MCP server itself does not implement its own authentication layer — an intentional simplification for a teaching/demo deployment, not a production configuration.

---

## Why This Exists Separately From the Live Apps

The Marketing Analytics Team and Tool Calling projects already demonstrate this agent logic through a chat UI. This project demonstrates the same logic exposed a second way: as a set of callable tools any MCP-compatible agent can discover and use directly — the interface changes, the underlying reasoning doesn't.
