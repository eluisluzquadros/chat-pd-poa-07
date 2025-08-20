# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chat PD POA is an AI-powered virtual assistant for Porto Alegre's Urban Development Plan (PDUS 2025). It uses Agentic-RAG v3 architecture with Supabase Edge Functions and PostgreSQL + pgvector for semantic search. The system processes queries about urban regulations, construction parameters, disaster risks, and zoning information.

## Key Architecture Components

### 1. Frontend (React + Vite + TypeScript)
- **Entry**: `src/App.tsx` - Main application router
- **Pages**: `src/pages/` - Route components (Auth, Chat, Admin dashboards)
- **Components**: `src/components/` - Reusable UI components
- **Services**: `src/services/` - API integrations (multiLLMService, chatService)
- **Hooks**: `src/hooks/` - Custom React hooks for state management
- **UI Library**: shadcn/ui components in `src/components/ui/`

### 2. Backend (Supabase Edge Functions)

#### Agentic-RAG v3 Pipeline (Current)
The main orchestration function with specialized agents:
- **agentic-rag-v3**: Main orchestrator that coordinates the entire query processing
- **agent-legal**: Handles legal and regulatory queries
- **agent-urban**: Processes urban planning and zoning questions
- **agent-validator**: Validates responses for accuracy
- **agent-reasoning**: Complex reasoning and multi-step queries
- **agent-evaluation**: Evaluates response quality

#### Core Processing Functions
- **query-analyzer**: Analyzes user intent and determines search strategy
- **sql-generator**: Converts natural language to SQL for structured data queries
- **enhanced-vector-search**: Performs semantic search on document embeddings
- **response-synthesizer**: Combines results into coherent Portuguese responses
- **contextual-scoring**: Scores and ranks search results

### 3. Database Structure
- **document_rows**: Tabular data (zones, neighborhoods, parameters)
- **document_sections**: Document chunks with embeddings for semantic search
- **legal_articles**: PDUS and LUOS articles with hierarchical structure
- **regime_urbanistico_consolidado**: Consolidated urban planning parameters
- **query_cache**: Performance optimization cache
- **qa_test_cases**: Quality assurance test scenarios (121 test cases)
- **llm_metrics**: Token usage and performance tracking

## Development Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build
npm run build:dev  # Development build

# Code Quality
npm run lint       # Run ESLint
npm run type-check # TypeScript type checking

# Testing
npm run test           # Run all tests
npm run test:watch     # Watch mode
npm run test:coverage  # Coverage report
npm run test:integration # Integration tests
npm run test:qa        # QA validation tests

# LLM & API Management
npm run test-llm-connections  # Test all LLM connections
npm run validate-keys         # Validate API keys
npm run deploy-env           # Deploy environment variables to Supabase

# Setup Wizard
npm run setup-wizard  # Interactive setup for new developers
```

## Supabase Edge Functions Deployment

```bash
# Deploy specific function
npx supabase functions deploy [function-name] --project-ref ngrqwmvuhvjkeohesbxs

# Deploy main Agentic-RAG v3 pipeline
npx supabase functions deploy agentic-rag-v3 --project-ref ngrqwmvuhvjkeohesbxs
npx supabase functions deploy agent-legal --project-ref ngrqwmvuhvjkeohesbxs
npx supabase functions deploy agent-urban --project-ref ngrqwmvuhvjkeohesbxs
npx supabase functions deploy agent-validator --project-ref ngrqwmvuhvjkeohesbxs

# Deploy core processing functions
npx supabase functions deploy query-analyzer --project-ref ngrqwmvuhvjkeohesbxs
npx supabase functions deploy sql-generator --project-ref ngrqwmvuhvjkeohesbxs
npx supabase functions deploy enhanced-vector-search --project-ref ngrqwmvuhvjkeohesbxs
npx supabase functions deploy response-synthesizer --project-ref ngrqwmvuhvjkeohesbxs

# Automated deployment scripts
npm run deploy-functions      # Deploy all functions via bash script
npm run deploy               # Complete deployment process
npm run deploy:verify        # Verify deployment status
npm run verify-deployment    # Alternative verification command
```

## Database Operations

```bash
# Regime Urbanístico Management
npm run regime:status      # Check current status
npm run regime:setup       # Initial setup
npm run regime:import      # Import data (with --direct flag)
npm run regime:import-force # Force reimport
npm run regime:test        # Test regime queries
npm run regime:clean       # Clean regime data
npm run regime:full-setup  # Complete setup process
npm run regime:help        # Show examples
npm run regime:monitor     # Monitor import progress
npm run regime:monitor-once # Single status check

# Knowledge Base Processing
npm run process-kb-local  # Process knowledge base locally
npm run kb:process        # Alternative KB processing
npm run kb:clean          # Clean local KB files

# Cache Management
node scripts/clear-cache-and-fix.ts  # Clear and fix cache issues

# Testing
npm run test:qa           # Run QA validation tests
npm run test-rag-altura   # Test height limit queries
```

## Environment Setup

1. Copy `.env.example` to `.env.local`
2. Required environment variables:
   ```env
   # Supabase Configuration
   NEXT_PUBLIC_SUPABASE_URL=https://ngrqwmvuhvjkeohesbxs.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
   SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
   NEXT_PUBLIC_SUPABASE_CLI_KEY=your_cli_key
   
   # LLM API Keys
   OPENAI_API_KEY=your_openai_key
   ANTHROPIC_API_KEY=your_anthropic_key
   GEMINI_API_KEY=your_gemini_key
   DEEPSEEK_API_KEY=your_deepseek_key
   ZHIPUAI_API_KEY=your_zhipuai_key
   GROQ_API_KEY=your_groq_key
   ```

3. Deploy environment variables to Supabase:
   ```bash
   npm run deploy-env
   ```

4. Run setup wizard for guided configuration:
   ```bash
   npm run setup-wizard
   ```

## Testing Strategy

### Unit Tests
- Located in `tests/` directory
- Test files: `*.test.ts`
- Coverage includes: Edge Functions, services, RAG components
- Run with `npm run test`

### Integration Tests
- Located in `scripts/test-*.mjs` files
- End-to-end pipeline testing
- Multi-model validation
- Run with `npm run test:integration`

### QA Validation System
- 121 predefined test cases
- Automated accuracy measurement
- Real-time monitoring at `/admin/quality`
- Run with `npm run test:qa`

### Test Individual Components
```bash
# Test specific Edge Functions
node scripts/test-query-analyzer.mjs
node scripts/test-sql-generator-direct.mjs
node scripts/test-response-synthesizer.mjs
node scripts/test-agentic-rag-v3.mjs

# Test with specific queries
node scripts/test-altura-maxima.mjs
node scripts/test-regime-queries.mjs
node scripts/test-bairros-queries.mjs
```

## RAG Pipeline Flow

### Agentic-RAG v3 (Current)
```
User Query → agentic-rag-v3 (orchestrator)
                ↓
         Intent Analysis & Routing
                ↓
    ┌───────────┴───────────┐
    ↓                       ↓
Specialized Agents    Core Functions
(legal, urban)        (SQL, vector search)
    ↓                       ↓
    └───────────┬───────────┘
                ↓
         agent-validator
                ↓
       response-synthesizer
         (21 LLM models)
                ↓
          Final Answer
```

### Legacy Pipeline (v1)
```
User Query → agentic-rag → query-analyzer
                              ↓
                 ┌────────────┴────────────┐
                 ↓                         ↓
           sql-generator          enhanced-vector-search
                 ↓                         ↓
                 └────────────┬────────────┘
                              ↓
                     response-synthesizer
```

## Key Data Sources

1. **Structured Data** (PostgreSQL tables):
   - `regime_urbanistico_consolidado`: 94 neighborhoods with urban parameters
   - `document_rows`: Zoning information (ZOTs)
   - `legal_articles`: PDUS/LUOS articles (340+ articles)
   - Risk areas and disaster zones

2. **Document Embeddings** (pgvector):
   - PDUS 2025 documents (217 articles)
   - LUOS regulations (123 articles)
   - Hierarchical chunks (8 levels of granularity)
   - Q&A knowledge base

## Multi-LLM Support (21 Models)

The system supports multiple LLM providers through `multiLLMService`:

### OpenAI (5 models)
- `gpt-4`, `gpt-4-turbo`, `gpt-3.5-turbo`, `gpt-4o`, `gpt-4o-mini`

### Anthropic (3 models)
- `claude-3-5-sonnet-20241022`, `claude-3-5-haiku-20241022`, `claude-3-opus-20240229`

### Google (3 models)
- `gemini-1.5-pro`, `gemini-1.5-flash`, `gemini-1.5-flash-8b`

### DeepSeek (2 models)
- `deepseek-chat`, `deepseek-coder`

### Groq (2 models)
- `llama-3.1-70b-versatile`, `mixtral-8x7b-32768`

### ZhipuAI (6 models)
- `glm-4-plus`, `glm-4-0520`, `glm-4-long`, `glm-4-air`, `glm-4-airx`, `glm-4-flash`

Model selection can be configured per query or globally in the admin dashboard.

## Debugging Tips

1. **Check Edge Function Logs**:
   ```bash
   # View logs for specific function
   supabase functions logs agentic-rag-v3 --project-ref ngrqwmvuhvjkeohesbxs
   
   # Stream logs in real-time
   supabase functions logs --tail --project-ref ngrqwmvuhvjkeohesbxs
   ```

2. **Test Individual Functions**:
   ```bash
   # Test query analyzer
   node scripts/test-query-analyzer.mjs
   
   # Test SQL generator
   node scripts/test-sql-generator-direct.mjs
   
   # Test full RAG pipeline
   node scripts/test-agentic-rag-v3.mjs
   ```

3. **Clear Cache for Fresh Results**:
   ```bash
   node scripts/clear-cache-and-fix.ts
   ```

4. **Monitor Performance**:
   - Admin Dashboard → `/admin/benchmark`
   - Token usage: `/admin/metrics`
   - Agent status: `/admin/agents`

## Common Issues and Solutions

1. **"Bairro not found" errors**: 
   - Clear cache: `node scripts/clear-cache-and-fix.ts`
   - Check if neighborhood exists: `npm run regime:test`

2. **Slow responses**: 
   - Check cache status in `/admin/cache`
   - Verify indexed columns in database
   - Monitor Edge Function cold starts

3. **Wrong LLM model responses**: 
   - Verify API keys: `npm run validate-keys`
   - Check Supabase secrets configuration
   - Test specific model: `/admin/benchmark`

4. **Deployment failures**: 
   - Always use `--project-ref ngrqwmvuhvjkeohesbxs`
   - Check Supabase CLI version: `supabase --version`
   - Verify authentication: `supabase projects list`

## Security Considerations

- All Edge Functions require JWT authentication
- Service role key should only be used server-side
- API keys are stored as Supabase secrets
- Row-level security (RLS) is enabled on sensitive tables

## Performance Optimization

1. **Query Cache**: Automatic caching of common queries
2. **Hierarchical Chunking**: Optimized document splitting
3. **Composite Indexes**: Database indexes for fast lookups
4. **Token Tracking**: Monitor and optimize LLM token usage

## Admin Features

Access admin panel at `/admin` (requires admin role):

### Dashboard Routes
- `/admin` - Main dashboard with system metrics
- `/admin/users` - User management and roles
- `/admin/quality` - QA testing (121 test cases)
- `/admin/benchmark` - Compare all 21 LLM models
- `/admin/feedback` - User feedback analysis
- `/admin/knowledge` - Knowledge base management
- `/admin/test-qa` - Manual QA test execution
- `/admin/test-cases` - Manage test cases
- `/admin/agents` - Monitor agent performance
- `/admin/cache` - Cache management interface
- `/admin/metrics` - Token usage and costs

### Key Features
- **Real-time Metrics**: Token usage, response times, error rates
- **Agent Monitoring**: Track performance of specialized agents
- **Session Viewer**: Review conversation history with context
- **Knowledge Graph Editor**: Visual graph management
- **Benchmark Tools**: Compare model accuracy and speed
- **Test Runner**: Execute and monitor QA test runs

## Architecture Decisions

### Why Agentic-RAG v3?
- **Multi-agent orchestration**: Specialized agents for different domains
- **Self-refinement**: Automatically improves responses when confidence < 70%
- **Parallel processing**: Multiple agents work simultaneously
- **Hierarchical chunking**: 8 levels of document granularity
- **Session memory**: Maintains context across conversations
- **21 LLM models**: Maximum flexibility and redundancy

### Database Indexing Strategy
- Composite indexes on frequently joined columns
- GIN indexes for full-text search on Portuguese content
- HNSW indexes for vector similarity (pgvector)
- Partial indexes for filtered queries
- B-tree indexes on foreign keys

### Caching Strategy
- Query cache: 24-hour TTL for common queries
- Session memory: 7-day retention
- Embedding cache: Permanent for processed documents
- Agent results: 1-hour cache for identical inputs
- Neighborhood data: Pre-cached for all 94 neighborhoods

## Project-Specific Conventions

### Error Handling
- All Edge Functions return standardized error responses
- Errors logged to `error_logs` table with correlation IDs
- Client-side retry logic with exponential backoff
- Graceful degradation when services unavailable

### Testing Requirements
- Unit tests required for all new Edge Functions
- Integration tests for pipeline changes
- QA test cases for new query types
- Performance benchmarks for optimization changes
- Regression tests before major deployments

### Deployment Checklist
1. Run `npm run test` locally
2. Validate API keys: `npm run validate-keys`
3. Deploy functions: `npm run deploy-functions`
4. Deploy environment: `npm run deploy-env`
5. Run integration tests: `npm run test:integration`
6. Verify deployment: `npm run verify-deployment`
7. Monitor logs: `supabase functions logs --tail --project-ref ngrqwmvuhvjkeohesbxs`
8. Check metrics dashboard: `/admin/metrics`