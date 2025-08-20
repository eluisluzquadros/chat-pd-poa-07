# Documentação do Schema de Banco de Dados - Sistema Agentic-RAG

## Visão Geral
Este documento detalha todas as tabelas, schemas e metadados utilizados pelo sistema Agentic-RAG v3 do Chat PD POA. O sistema utiliza PostgreSQL com a extensão pgvector para busca semântica.

## Tabelas Principais do Sistema RAG

### 1. **legal_articles**
Armazena artigos legais do PDUS, LUOS e outras legislações com embeddings para busca semântica.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT | PRIMARY KEY - Identificador único do artigo |
| article_number | INTEGER | Número do artigo na lei |
| document_type | TEXT | Tipo do documento (PDUS, LUOS, COE) |
| article_text | TEXT | Texto completo do artigo |
| full_content | TEXT | Conteúdo completo incluindo contexto |
| source | TEXT | Fonte do documento |
| embedding | vector(1536) | Embedding vetorial para busca semântica |
| metadata | JSONB | Metadados incluindo seção, capítulo, etc |
| created_at | TIMESTAMPTZ | Data de criação |
| updated_at | TIMESTAMPTZ | Data de atualização |

**Índices:**
- idx_legal_articles_metadata (GIN)
- idx_legal_articles_created_at
- idx_legal_articles_embedding (ivfflat)
- idx_legal_articles_document_type
- idx_legal_articles_article_number

**Função RPC:**
```sql
match_legal_articles(query_embedding, match_threshold, match_count)
```

---

### 2. **regime_urbanistico_consolidado**
Dados consolidados do regime urbanístico de Porto Alegre com parâmetros construtivos por zona.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| Bairro | VARCHAR | Nome do bairro em MAIÚSCULAS com acentos |
| Zona | VARCHAR | Código da zona (ex: "ZOT 07") |
| Altura_Maxima___Edificacao_Isolada | DECIMAL | Altura máxima permitida em metros |
| Coeficiente_de_Aproveitamento___Basico | DECIMAL | CA básico |
| Coeficiente_de_Aproveitamento___Maximo | DECIMAL | CA máximo |
| Área_Minima_do_Lote | INTEGER | Área mínima do lote em m² |
| Taxa_de_Permeabilidade_ate_1,500_m2 | DECIMAL | Taxa para lotes ≤ 1500m² |
| Afastamentos___Frente | TEXT | Afastamento frontal |
| Afastamentos___Laterais | TEXT | Afastamento lateral |
| Afastamentos___Fundos | TEXT | Afastamento de fundos |

**Observações:**
- Total de 385 registros
- Bairros mantêm acentuação (ex: "PETRÓPOLIS", "TRÊS FIGUEIRAS")
- Um bairro pode ter múltiplas zonas

---

### 3. **document_sections**
Chunks de documentos com embeddings para busca vetorial semântica.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY - gen_random_uuid() |
| content | TEXT | Conteúdo do chunk |
| embedding | vector(1536) | Embedding do conteúdo |
| metadata | JSONB | Metadados do documento |
| title | TEXT | Título do documento/seção |
| source | TEXT | Fonte do documento |
| created_at | TIMESTAMPTZ | Data de criação |
| updated_at | TIMESTAMPTZ | Data de atualização |

**Índices:**
- idx_document_sections_content_search (GIN - tsvector)
- idx_document_sections_metadata (GIN)
- idx_document_sections_source
- idx_document_sections_type
- idx_document_sections_article

**Função RPC:**
```sql
match_document_sections(query_embedding, match_threshold, match_count)
```

---

### 4. **query_cache**
Cache de queries para otimização de performance.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY |
| query | TEXT | Query original em lowercase |
| response | TEXT | Resposta cacheada |
| confidence | NUMERIC(3,2) | Score de confiança |
| model | TEXT | Modelo LLM usado |
| execution_time | INTEGER | Tempo de execução em ms |
| ttl_minutes | INTEGER | Tempo de vida em minutos (default: 60) |
| category | TEXT | Categoria da query |
| hit_count | INTEGER | Contador de hits |
| metadata | JSONB | Metadados adicionais |
| expires_at | TIMESTAMPTZ | Data de expiração |
| created_at | TIMESTAMPTZ | Data de criação |

**Índices:**
- idx_query_cache_query_pattern (GIN - trigram)
- idx_query_cache_expires
- idx_query_cache_category
- idx_query_cache_confidence
- idx_query_cache_hit_count

---

### 5. **chat_memory**
Memória de contexto das conversações para rastreamento de contexto.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY |
| session_id | TEXT | ID da sessão |
| user_id | TEXT | ID do usuário |
| query | TEXT | Query do usuário |
| response | TEXT | Resposta do sistema |
| context | JSONB | Contexto detectado (lei, artigos, etc) |
| confidence_score | NUMERIC(3,2) | Score de confiança |
| created_at | TIMESTAMPTZ | Data de criação |

**Uso:** Mantém contexto para identificar qual lei (PDUS/LUOS) está sendo discutida.

---

### 6. **chat_history**
Histórico completo de conversações.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY |
| session_id | TEXT | ID da sessão |
| user_id | TEXT | ID do usuário |
| message | TEXT | Mensagem do usuário |
| response | TEXT | Resposta do sistema |
| model | TEXT | Modelo LLM usado |
| confidence | NUMERIC(3,2) | Score de confiança |
| execution_time | INTEGER | Tempo de execução |
| created_at | TIMESTAMPTZ | Data de criação |

---

### 7. **legal_hierarchy**
Hierarquia de documentos legais (Títulos, Capítulos, Seções).

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| document_type | TEXT | Tipo do documento (PDUS, LUOS) |
| hierarchy_type | TEXT | Tipo (titulo, capitulo, secao) |
| hierarchy_number | TEXT | Número do elemento |
| hierarchy_name | TEXT | Nome/título do elemento |
| article_start | INTEGER | Artigo inicial |
| article_end | INTEGER | Artigo final |
| parent_id | INTEGER | ID do elemento pai |
| metadata | JSONB | Metadados adicionais |

---

### 8. **regime_urbanistico**
Tabela alternativa de regime urbanístico (estrutura simplificada).

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| bairro | VARCHAR(255) | Nome do bairro |
| zona | VARCHAR(50) | Código da zona |
| altura_maxima | DECIMAL | Altura máxima em metros |
| coef_aproveitamento_basico | DECIMAL | CA básico |
| coef_aproveitamento_maximo | DECIMAL | CA máximo |
| area_minima_lote | INTEGER | Área mínima do lote |
| taxa_permeabilidade_acima_1500 | DECIMAL | Taxa para lotes > 1500m² |
| taxa_permeabilidade_ate_1500 | DECIMAL | Taxa para lotes ≤ 1500m² |
| recuo_jardim | DECIMAL | Recuo de jardim |
| afastamento_frente | TEXT | Afastamento frontal |
| afastamento_lateral | TEXT | Afastamento lateral |
| afastamento_fundos | TEXT | Afastamento de fundos |

---

### 9. **zots_bairros**
Mapeamento entre zonas (ZOTs) e bairros.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| bairro | VARCHAR(255) | Nome do bairro |
| zona | VARCHAR(50) | Código da zona |
| caracteristicas | JSONB | Características da zona |
| restricoes | JSONB | Restrições aplicáveis |
| incentivos | JSONB | Incentivos disponíveis |
| metadata | JSONB | Metadados adicionais |
| created_at | TIMESTAMP | Data de criação |
| updated_at | TIMESTAMP | Data de atualização |

---

### 10. **bairros_risco_desastre**
Informações sobre riscos de desastres por bairro.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| bairro | VARCHAR | Nome do bairro |
| tipo_risco | VARCHAR | Tipo do risco |
| nivel_risco | VARCHAR | Nível (alto, médio, baixo) |
| risco_inundacao | BOOLEAN | Risco de inundação |
| risco_deslizamento | BOOLEAN | Risco de deslizamento |
| risco_alagamento | BOOLEAN | Risco de alagamento |
| nivel_risco_geral | INTEGER | Nível geral (1-5) |
| descricao | TEXT | Descrição detalhada |
| metadata | JSONB | Dados adicionais |

---

### 11. **qa_test_cases**
Casos de teste para validação de qualidade.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| test_id | VARCHAR(100) | ID único do teste |
| query | TEXT | Query de teste |
| expected_keywords | TEXT[] | Palavras-chave esperadas |
| category | VARCHAR(50) | Categoria do teste |
| complexity | VARCHAR(20) | Complexidade (simple, medium, high) |
| min_response_length | INTEGER | Tamanho mínimo da resposta |
| expected_response | TEXT | Resposta esperada |
| is_active | BOOLEAN | Se o teste está ativo |
| created_at | TIMESTAMPTZ | Data de criação |

---

### 12. **document_embeddings**
Embeddings de documentos com chunking hierárquico.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | SERIAL | PRIMARY KEY |
| document_id | BIGINT | FK para documents |
| content_chunk | TEXT | Chunk do conteúdo |
| embedding | vector(1536) | Embedding do chunk |
| chunk_metadata | JSONB | Metadados do chunk |
| created_at | TIMESTAMPTZ | Data de criação |

**Índices:**
- idx_document_embeddings_document_id
- idx_document_embeddings_metadata (GIN)
- idx_document_embeddings_vector_composite
- idx_document_embeddings_hierarchical
- idx_document_embeddings_altura_queries (partial)
- idx_document_embeddings_bairros_cristal (partial)
- idx_document_embeddings_bairros_petropolis (partial)

---

## Tabelas de Suporte e Métricas

### 13. **llm_metrics**
Métricas de uso dos modelos LLM.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY |
| model_id | TEXT | ID do modelo |
| model_name | TEXT | Nome do modelo |
| query | TEXT | Query processada |
| response_time_ms | INTEGER | Tempo de resposta |
| tokens_input | INTEGER | Tokens de entrada |
| tokens_output | INTEGER | Tokens de saída |
| cost_usd | NUMERIC(10,6) | Custo em USD |
| quality_score | NUMERIC(3,2) | Score de qualidade |
| confidence_score | NUMERIC(3,2) | Score de confiança |
| success | BOOLEAN | Se foi bem-sucedido |
| error_message | TEXT | Mensagem de erro |
| metadata | JSONB | Metadados |
| created_at | TIMESTAMPTZ | Data de criação |

---

### 14. **session_quality_metrics**
Métricas de qualidade por sessão.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY |
| session_id | UUID | ID da sessão |
| total_messages | INTEGER | Total de mensagens |
| positive_feedback | INTEGER | Feedbacks positivos |
| negative_feedback | INTEGER | Feedbacks negativos |
| satisfaction_rate | NUMERIC(5,2) | Taxa de satisfação |
| avg_response_time | NUMERIC(10,2) | Tempo médio de resposta |
| metadata | JSONB | Metadados |
| created_at | TIMESTAMPTZ | Data de criação |
| updated_at | TIMESTAMPTZ | Data de atualização |

---

### 15. **knowledge_gaps**
Lacunas de conhecimento identificadas.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | PRIMARY KEY |
| query | TEXT | Query que gerou a lacuna |
| confidence_score | NUMERIC(3,2) | Score de confiança |
| category | TEXT | Categoria |
| severity | TEXT | Severidade (low, medium, high, critical) |
| status | TEXT | Status (pending, in_progress, resolved, rejected) |
| resolution_content | TEXT | Conteúdo de resolução |
| resolution_metadata | JSONB | Metadados da resolução |
| detected_at | TIMESTAMPTZ | Data de detecção |
| resolved_at | TIMESTAMPTZ | Data de resolução |
| metadata | JSONB | Metadados |

---

## Funções RPC Principais

### 1. **match_legal_articles**
```sql
match_legal_articles(
    query_embedding vector(1536),
    match_threshold FLOAT DEFAULT 0.7,
    match_count INT DEFAULT 10
) RETURNS TABLE (id, content, metadata, similarity)
```

### 2. **match_document_sections**
```sql
match_document_sections(
    query_embedding vector(1536),
    match_threshold FLOAT DEFAULT 0.78,
    match_count INT DEFAULT 10
) RETURNS TABLE (id, content, metadata, similarity)
```

### 3. **match_hierarchical_documents_v2**
```sql
match_hierarchical_documents_v2(
    query_text TEXT,
    match_threshold FLOAT DEFAULT 0.78,
    match_count INT DEFAULT 10,
    metadata_filter JSONB DEFAULT NULL,
    performance_mode TEXT DEFAULT 'balanced'
) RETURNS TABLE (id, document_id, content_chunk, metadata, similarity)
```

### 4. **get_complete_hierarchy**
```sql
get_complete_hierarchy(
    doc_type TEXT,
    art_num INTEGER
) RETURNS TEXT
```

### 5. **hybrid_search**
```sql
hybrid_search(
    query_text TEXT,
    query_embedding vector(1536),
    match_threshold FLOAT DEFAULT 0.7,
    match_count INT DEFAULT 10
) RETURNS TABLE (id, content, metadata, similarity, rank)
```

---

## Fluxo de Dados no Agentic-RAG

### 1. **Recepção da Query**
- Query recebida pelo `agentic-rag`
- Verificação no `query_cache`
- Consulta `chat_memory` para contexto

### 2. **Análise e Processamento**
- `query-analyzer` identifica intenção
- Geração de embedding (OpenAI Ada-002)
- Identificação de entidades (artigos, zonas, bairros)

### 3. **Busca de Dados**

#### Para Artigos Legais:
- Busca em `legal_articles` por número/tipo
- Consulta `legal_hierarchy` para contexto
- Busca vetorial via `match_legal_articles`

#### Para Regime Urbanístico:
- Busca em `regime_urbanistico_consolidado`
- Consulta `zots_bairros` para mapeamento
- Verificação em `bairros_risco_desastre`

#### Para Documentos Gerais:
- Busca vetorial em `document_sections`
- Busca em `document_embeddings`
- Uso de `enhanced-vector-search`

### 4. **Síntese de Resposta**
- `response-synthesizer` combina resultados
- Formatação em português
- Cache da resposta em `query_cache`

### 5. **Persistência**
- Salvar em `chat_history`
- Atualizar `chat_memory` com contexto
- Registrar métricas em `llm_metrics`

---

## Considerações de Performance

### Índices Críticos
1. **Vector Search**: ivfflat com lists=100 para embeddings
2. **Text Search**: GIN indexes para JSONB e tsvector
3. **Composite Indexes**: Para queries multi-coluna frequentes
4. **Partial Indexes**: Para bairros específicos (Cristal, Petrópolis)

### Estratégias de Cache
- **query_cache**: TTL de 60 minutos por padrão
- **match_hierarchical_cache**: 30 minutos para buscas hierárquicas
- Hit count tracking para queries populares

### Normalização de Dados
- Bairros em MAIÚSCULAS com acentos preservados
- Zonas no formato "ZOT XX"
- Uso de ILIKE para buscas case-insensitive

---

## Manutenção e Monitoramento

### Funções de Limpeza
- `clean_expired_cache()`: Remove cache expirado
- `update_updated_at_column()`: Trigger para timestamps
- `update_session_metrics()`: Atualiza métricas de sessão

### Row Level Security (RLS)
- Habilitado em todas as tabelas principais
- Políticas de leitura pública para dados de regime
- Restrições em tabelas de métricas e logs

### Backup e Recovery
- Tabelas críticas: `legal_articles`, `regime_urbanistico_consolidado`
- Backup diário de `query_cache` e `chat_history`
- Retenção de 30 dias para métricas

---

## Modelos LLM Suportados

O sistema suporta 21 modelos de diferentes provedores:

### OpenAI
- gpt-4-turbo-preview
- gpt-4
- gpt-3.5-turbo

### Anthropic
- claude-3-opus
- claude-3-sonnet
- claude-3-haiku

### Google
- gemini-pro
- gemini-pro-vision
- gemini-1.5-pro
- gemini-1.5-flash

### Groq
- mixtral-8x7b
- llama-3-70b
- llama-3-8b

### DeepSeek
- deepseek-coder
- deepseek-chat

### Outros
- ZhipuAI (glm-4, glm-3-turbo)
- Cohere (command-r-plus, command-r)
- Mistral (mistral-large, mistral-medium)

---

## Conclusão

O sistema Agentic-RAG utiliza uma arquitetura robusta de banco de dados com:
- **15+ tabelas principais** para diferentes aspectos do sistema
- **Busca híbrida** combinando vetorial e textual
- **Cache multinível** para otimização
- **Suporte multi-LLM** com métricas detalhadas
- **Contexto de conversação** para melhor precisão

Esta documentação serve como referência completa para entender e manter o schema de banco de dados do sistema.