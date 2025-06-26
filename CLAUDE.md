# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Crawl4AI RAG is a Model Context Protocol server that provides web crawling and RAG (Retrieval Augmented Generation) capabilities. It combines Crawl4AI for intelligent web scraping with Supabase for vector storage and various advanced RAG strategies.

## Development Commands

```bash
# Install dependencies and setup
uv pip install -e .
crawl4ai-setup

# Run the server locally
uv run src/crawl4ai_mcp.py

# Build Docker image with custom port
docker build -t mcp/crawl4ai-rag --build-arg PORT=8051 .

# Run Docker container
docker run --env-file .env -p 8051:8051 mcp/crawl4ai-rag

# Test hallucination detection (if USE_KNOWLEDGE_GRAPH=true)
python knowledge_graphs/ai_hallucination_detector.py <script_path>
```

## Architecture & Key Components

### Main Entry Point
- `src/crawl4ai_mcp.py` - FastMCP server implementation with tool registration and lifecycle management

### Core Functions
- **Crawling**: `crawl_single_page()` and `smart_crawl_url()` handle web scraping with intelligent URL type detection
- **RAG Query**: `perform_rag_query()` implements semantic search with optional filtering and reranking
- **Knowledge Graph**: Optional features in `knowledge_graphs/` for GitHub parsing and hallucination detection

### Tool Availability
Tools are conditionally registered based on environment variables:
- Always available: crawling and basic RAG query tools
- `USE_AGENTIC_RAG=true`: Enables `search_code_examples` 
- `USE_KNOWLEDGE_GRAPH=true`: Enables GitHub parsing and hallucination detection

### Async Pattern
The codebase uses asyncio throughout. Key patterns:
- `async with crawl4ai_lifespan()` manages resource lifecycle
- Batch operations use `asyncio.gather()` with semaphores for concurrency control
- ThreadPoolExecutor for CPU-bound operations like embeddings

## Environment Configuration

Required environment variables in `.env`:
```
OPENAI_API_KEY=<your-key>
SUPABASE_URL=<your-url>
SUPABASE_SERVICE_KEY=<your-key>
MODEL_CHOICE=gpt-4.1-nano
```

Optional RAG strategies (all default to false):
```
USE_CONTEXTUAL_EMBEDDINGS=true  # Enhanced embeddings with LLM context
USE_HYBRID_SEARCH=true          # Combine vector + keyword search
USE_AGENTIC_RAG=true           # Extract and search code examples
USE_RERANKING=true             # Cross-encoder result reranking
USE_KNOWLEDGE_GRAPH=true       # GitHub parsing & hallucination detection
```

## Database Schema

PostgreSQL with pgvector extension. Key tables:
- `sources`: Domain summaries with statistics
- `crawled_pages`: Document chunks with vector embeddings
- `code_examples`: Extracted code snippets (if agentic RAG enabled)

Uses ivfflat indexing for efficient vector similarity search.

## Error Handling & Retry Logic

- All database operations have exponential backoff retry (max 3 attempts)
- Graceful fallbacks for optional features
- Comprehensive error logging with context

## Transport Modes

Supports both SSE (for remote/Docker) and stdio (for local MCP clients). Automatically detected based on MCP_TRANSPORT environment variable.

## Code Style Guidelines

- Use type hints for all function parameters and returns
- Follow async/await patterns consistently
- Preserve existing error handling and retry logic
- Batch operations where possible for performance
- Use the existing logging pattern for debugging