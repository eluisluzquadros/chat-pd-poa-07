# Chat PD POA - Sistema de Consulta Urbanística de Porto Alegre

[![Status](https://img.shields.io/badge/status-beta-yellow)]()
[![Accuracy](https://img.shields.io/badge/accuracy-86.7%25-orange)]()
[![Target](https://img.shields.io/badge/target-95%25-green)]()
[![License](https://img.shields.io/badge/license-MIT-blue)]()

## 📋 Sobre o Projeto

O **Chat PD POA** é um sistema de inteligência artificial desenvolvido para auxiliar cidadãos, arquitetos, engenheiros e urbanistas a compreender e consultar a legislação urbanística de Porto Alegre (PDPOA 2025). 

O sistema utiliza técnicas avançadas de RAG (Retrieval-Augmented Generation) para fornecer respostas precisas sobre:
- 📐 Parâmetros construtivos por zona
- 🏢 Alturas máximas permitidas
- 📊 Coeficientes de aproveitamento
- 🌳 Taxas de permeabilidade
- 🏘️ Zoneamento urbano
- 📜 Artigos da LUOS e PDUS

## 🚀 Funcionalidades Principais

### Para Usuários
- **Chat Inteligente**: Perguntas em linguagem natural sobre legislação urbanística
- **Consulta por Bairro**: Informações específicas dos 94 bairros de Porto Alegre
- **Citações de Fontes**: Referências diretas aos artigos de lei
- **Histórico de Conversas**: Acesso a consultas anteriores

### Para Administradores
- **Dashboard de Qualidade**: Monitoramento de acurácia em tempo real
- **Sistema de Validação**: 125 casos de teste automatizados
- **Análise de Gaps**: Identificação de lacunas no conhecimento
- **Métricas de Performance**: Tempo de resposta, uso de tokens, cache hits

## 🛠️ Tecnologias

- **Frontend**: React + TypeScript + Vite + Tailwind CSS
- **Backend**: Supabase (PostgreSQL + Edge Functions)
- **IA/LLM**: OpenAI GPT-4, Claude 3, Gemini Pro, Groq
- **Busca**: pgvector (embeddings) + Full-text search
- **Deploy**: Supabase Cloud + Vercel

## 📦 Instalação

### Pré-requisitos
- Node.js 18+ 
- npm ou yarn
- Conta no Supabase
- API Keys (OpenAI obrigatória, outras opcionais)

### Passo a Passo

1. **Clone o repositório**
```bash
git clone https://github.com/usuario/chat-pd-poa-07.git
cd chat-pd-poa-07
```

2. **Instale as dependências**
```bash
npm run install:all
```

3. **Configure as variáveis de ambiente**

Crie um arquivo `.env` na raiz:
```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=sua_url_aqui
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua_chave_aqui
SUPABASE_SERVICE_ROLE_KEY=sua_service_key_aqui

# LLM APIs (OpenAI obrigatória)
OPENAI_API_KEY=sua_openai_key_aqui
ANTHROPIC_API_KEY=opcional
GOOGLE_GENERATIVE_AI_API_KEY=opcional
GROQ_API_KEY=opcional
```

4. **Configure o banco de dados**
```bash
# Execute as migrations
cd backend/supabase
npx supabase db push

# Importe os dados base
cd ../../frontend
npm run regime:setup
npm run regime:import
npm run kb:import-full
```

5. **Inicie o desenvolvimento**
```bash
npm run dev
```

Acesse http://localhost:5173

## 📊 Status do Sistema

### Performance Atual
- **Acurácia**: 86.7% em 125 casos de teste
- **Tempo de Resposta**: 3-5 segundos
- **Cache Hit Rate**: ~30%
- **Bases de Conhecimento**: 100% carregadas, mas apenas 56% sendo consultadas

### ⚠️ DESCOBERTA CRÍTICA: Sistema está ignorando 44% dos dados!

| Base | Status no BD | Status no RAG | Registros | % do Total |
|------|--------------|---------------|-----------|------------|
| LUOS | ✅ Presente | ✅ Consultado | 398 | 19.9% |
| PDUS | ✅ Presente | ✅ Consultado | 720 | 36.0% |
| REGIME_FALLBACK | ✅ Presente | ❌ IGNORADO | 864 | 43.2% |
| QA_CATEGORY | ✅ Presente | ❌ IGNORADO | 16 | 0.8% |
| **TOTAL** | **1,998** | **1,118 (56%)** | **1,998** | **100%** |

**Problema**: O agentic-rag só busca `document_type IN ('LUOS', 'PDUS')`, ignorando 880 registros!

## 🎯 Roadmap

### Fase 1 - FIX URGENTE (5 minutos para 95% acurácia!)
- [ ] Corrigir query no agentic-rag para incluir TODOS os document_types
- [ ] Mudar campo de `content` para `full_content`
- [ ] Testar com os 125 casos

### Fase 2 - Otimizações (Após o fix)

### Fase 2 - Otimizações
- [ ] Melhorar estratégia de cache
- [ ] Implementar reranking de resultados
- [ ] Adicionar índices compostos
- [ ] Otimizar embeddings

### Fase 3 - Novas Features
- [ ] Visualização de mapas
- [ ] Export de relatórios
- [ ] API pública
- [ ] Mobile app

## 🧪 Testes

### Executar testes unitários
```bash
cd frontend
npm test
```

### Executar validação de QA
```bash
npm run test:qa
```

### Testar conexões com LLMs
```bash
npm run test-llm-connections
```

## 📝 Documentação

- [CLAUDE.md](./CLAUDE.md) - Guia para desenvolvimento com Claude Code
- [PRD.md](./PRD.md) - Documento de requisitos do produto
- [PLANO_ACAO.md](./docs/PLANO_ACAO_MELHORIAS_2025.md) - Plano de implementação de melhorias

## 🤝 Contribuindo

1. Fork o projeto
2. Crie uma branch (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👥 Equipe

- **Desenvolvimento**: Aurora
- **Arquitetura**: Equipe Técnica PUCRS
- **Conteúdo Legal**: Prefeitura de Porto Alegre

## 📞 Suporte

- **Issues**: [GitHub Issues](https://github.com/usuario/chat-pd-poa-07/issues)
- **Email**: suporte@chatpdpoa.com.br
- **Documentação**: [Wiki do Projeto](https://github.com/usuario/chat-pd-poa-07/wiki)

## 🙏 Agradecimentos

- Prefeitura de Porto Alegre pela disponibilização dos dados
- Comunidade open source pelos componentes utilizados
- Beta testers pelo feedback valioso

---

**Nota**: Este sistema está em fase beta. As respostas devem ser validadas com a legislação oficial antes de uso em projetos reais.