# Implementation Plan

## Stack

### Backend API

Use Python with FastAPI for the core trading API. It should own broker integrations, AI orchestration, risk checks, async workflows, and the API surface used by the frontend.

### Database, Auth, and RAG

Use Supabase as the application backend foundation:

- Postgres for users, portfolios, trades, orders, approvals, audit logs, risk settings, and strategy configuration.
- Supabase Auth for login, user identity, and account ownership.
- pgvector for RAG embeddings without introducing a separate vector database early.
- Supabase Storage for uploaded documents, research files, statements, and strategy notes.
- Supabase Realtime for live trade approvals, portfolio updates, and order status changes.

### Trading Engine

Keep execution-sensitive trading logic in the FastAPI backend, not in frontend serverless functions.

The AI layer generates trade proposals. A deterministic risk engine validates every proposal before an order can reach a broker API.

- Generate trade proposals from strategy, market context, RAG, and web search.
- Validate proposals against user risk settings.
- Require human approval when configured.
- Place orders through broker APIs only after validation.
- Record complete audit logs for proposals, approvals, rejections, and executed trades.
- Reconcile broker state with local database records.

### Broker Integration

Start with Alpaca because it supports paper trading and has straightforward APIs for automated trading workflows. Move to IBKR once the product and risk controls mature.

### AI and Research

Use OpenAI or Anthropic for market summaries, trade proposal explanations, portfolio reasoning, and RAG over user strategy notes, research, and historical decisions.

For current context, create a web search MCP tool or specialized research agent. Alternatively, use Tavily, Exa, Brave Search API, or SerpAPI for market, company, earnings, news, and macroeconomic signals.

### Background Jobs

Use a worker service for long-running and scheduled tasks:

- Broker account syncs.
- Portfolio reconciliation.
- Web search research runs.
- Embedding generation and refreshes.
- Strategy scans.
- Trade proposal generation.
- Order status polling and cleanup.

For the MVP, this can run as a separate worker process next to the FastAPI backend. If job complexity grows, add Redis with Celery or Dramatiq.

### Frontend

Use Next.js with TypeScript for the product UI:

- Dashboard views.
- Portfolio and position monitoring.
- Risk profile configuration.
- Broker connection status.
- Trade proposal review and approval.
- Audit history and model reasoning views.

## Deployment

Use this setup for the MVP:

- Vercel for frontend hosting.
- Supabase for Postgres, Auth, Storage, Realtime, and pgvector.
- Render for the FastAPI backend and worker service.
- Alpaca for paper trading and eventual live trading.
- OpenAI or Anthropic for model reasoning.
- Web search MCP/specialized agent, or Tavily, Exa, Brave Search API, or SerpAPI.

### Architecture

```text
Next.js frontend on Vercel
        |
        v
FastAPI trading API on Render/Fly/Railway
        |
        +--> Supabase Postgres/Auth/Storage/Realtime/pgvector
        +--> Alpaca or other broker APIs
        +--> OpenAI/Anthropic model APIs
        +--> Web search MCP, agent, or provider
        +--> Background worker
```

## Safety Principles

- Default to paper trading.
- Treat AI output as a proposal, not an executable command.
- Validate every proposed trade with deterministic risk checks.
- Require human approval based on user risk settings.
- Store full audit logs for all proposals, decisions, and broker actions.
- Keep broker credentials and model API keys in backend environment variables only.
- Never expose execution credentials to the frontend.
