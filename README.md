# Trend Analyzer — Análise de Tendências com IA

Este repositório contém a **primeira parte de um projeto maior e mais complexo de análise de tendências digitais**. O objetivo desta etapa é validar o fluxo completo de coleta, normalização, cálculo de métricas, agrupamento por assunto e análise com inteligência artificial antes da inclusão de novas redes sociais, relatórios avançados e recursos de produto.

Atualmente, a aplicação coleta vídeos populares do YouTube, transforma os dados em um formato comum, calcula métricas de tendência, agrupa conteúdos relacionados e permite gerar uma análise com IA. As análises concluídas são armazenadas em PostgreSQL para que possam ser consultadas novamente sem consumir tokens desnecessariamente.

![Fluxo atual do projeto](./mermaid-diagram.png)

## Fluxo atual

```text
YouTube
   ↓
Normalização dos dados
   ↓
Cálculo de métricas
   ↓
Agrupamento por assunto
   ↓
Análise com IA
   ↓
Persistência no PostgreSQL
   ↓
Exibição das tendências e oportunidades de conteúdo
```

## Funcionalidades disponíveis

- Coleta dos vídeos populares do YouTube por região.
- Normalização dos dados recebidos da API.
- Cálculo de visualizações por hora, engajamento, recência e score de tendência.
- Ordenação e exibição das tendências em uma tabela com rolagem.
- Agrupamento de conteúdos por assunto.
- Análise dos principais grupos com a API da OpenAI.
- Cache das análises para evitar chamadas repetidas e consumo desnecessário de tokens.
- Persistência das análises em PostgreSQL com volume Docker.
- Consulta da última análise armazenada sem chamar novamente o YouTube ou a OpenAI.

## Tecnologias

### Backend

- Node.js e TypeScript
- NestJS
- OpenAI SDK
- Zod para validação das respostas estruturadas
- PostgreSQL
- Jest

### Frontend

- React
- Vite
- Tailwind CSS
- Heroicons

### Infraestrutura

- Docker e Docker Compose
- PostgreSQL 17

## Estrutura do projeto

```text
ProjetoAnaliseDeTendencia/
├── api/                 # API NestJS, regras de negócio e integração com IA
├── frontend/            # Interface React/Vite
├── compose.yaml         # PostgreSQL para desenvolvimento local
├── mermaid-diagram.png  # Diagrama do fluxo da aplicação
└── README.md
```

## Pré-requisitos

Antes de iniciar, instale:

- Node.js 20 ou superior;
- npm;
- Docker Desktop;
- uma chave da YouTube Data API v3;
- uma chave da API da OpenAI com créditos disponíveis.

> A assinatura do ChatGPT não inclui créditos da API da OpenAI. O faturamento da API é separado.

## Configuração

### 1. Banco de dados

Na raiz do projeto, inicie o PostgreSQL:

```powershell
docker compose up -d postgres
```

Confirme se o contêiner está saudável:

```powershell
docker compose ps
```

O Compose cria o banco `trend_analysis` e mantém os dados no volume `trend_postgres_data`.

### 2. Backend

Entre na pasta da API e instale as dependências:

```powershell
cd api
npm install
```

Crie o arquivo de ambiente a partir do exemplo:

```powershell
Copy-Item .env.example .env
```

Preencha pelo menos estas variáveis em `api/.env`:

```env
FRONTEND_URL=http://localhost:5173
YOUTUBE_V3_API_KEY=sua_chave_do_youtube
OPENAI_API_KEY=sua_chave_da_openai
OPENAI_MODEL=gpt-5.6-luna
OPENAI_AI_ANALYSIS_LIMIT=10
OPENAI_TIMEOUT_MS=120000
AI_ANALYSIS_CACHE_TTL_SECONDS=1800
DATABASE_URL=postgresql://trend_app:trend_dev_change_me@127.0.0.1:5432/trend_analysis
```

O token do Reddit é opcional nesta fase:

```env
REDDIT_ACCESS_TOKEN=
```

Nunca envie o arquivo `.env` ao Git e nunca coloque chaves privadas no frontend.

Inicie a API:

```powershell
npm run start:dev
```

A API ficará disponível em `http://localhost:3000`.

Para verificar o funcionamento:

```text
http://localhost:3000/status
```

### 3. Frontend

Em outro terminal, entre na pasta do frontend e instale as dependências:

```powershell
cd frontend
npm install
```

Crie `frontend/.env.local` com apenas o endereço da API:

```env
VITE_API_URL=http://localhost:3000
```

Inicie o frontend:

```powershell
npm run dev
```

A interface ficará disponível em `http://localhost:5173`.

## Ordem recomendada para iniciar

Sempre inicie os serviços nesta ordem:

1. Docker Desktop;
2. PostgreSQL com `docker compose up -d postgres`;
3. backend com `npm run start:dev` dentro de `api`;
4. frontend com `npm run dev` dentro de `frontend`.

## Como gerar uma análise

1. Abra `http://localhost:5173`.
2. Aguarde o carregamento das tendências do YouTube.
3. Localize a seção **Análise de tendências com IA**.
4. Clique em **Gerar análise com IA**.
5. Aguarde a resposta; a geração pode levar até 120 segundos.

Uma análise bem-sucedida é salva na tabela `public.ai_analyses`. Enquanto o fingerprint, o modelo e o prazo do cache forem válidos, a aplicação reutiliza o resultado armazenado. A opção de gerar uma nova análise pode fazer outra chamada paga à OpenAI.

## Principais endpoints

| Método | Endpoint | Descrição |
| --- | --- | --- |
| `GET` | `/status` | Verifica se a API está funcionando |
| `GET` | `/youtube/popular?regionCode=BR` | Retorna os dados populares brutos do YouTube |
| `GET` | `/trends/youtube?regionCode=BR` | Retorna vídeos normalizados com métricas calculadas |
| `GET` | `/trends/youtube/grouped?regionCode=BR` | Retorna tendências agrupadas por assunto |
| `POST` | `/trends/youtube/ai-analysis?regionCode=BR` | Gera ou reutiliza uma análise com IA |
| `GET` | `/trends/youtube/ai-analysis/latest?regionCode=BR` | Consulta a última análise armazenada |

## Acesso ao PostgreSQL

Pelo terminal:

```powershell
docker compose exec postgres psql -U trend_app -d trend_analysis
```

Consulta básica:

```sql
SELECT
    region_code,
    model,
    generated_at,
    expires_at,
    result
FROM public.ai_analyses
ORDER BY generated_at DESC;
```

Para HeidiSQL, DBeaver ou pgAdmin, use:

```text
Servidor: 127.0.0.1
Porta: 5432
Banco: trend_analysis
Usuário: trend_app
Senha: trend_dev_change_me
```

Essas credenciais são destinadas somente ao desenvolvimento local. Use segredos fortes e configuração própria em produção.

## Testes e validações

No backend:

```powershell
cd api
npm run build
npm run test
```

No frontend:

```powershell
cd frontend
npm run lint
npm run build
```

## Estado atual e próximos passos

Esta versão é uma fundação funcional, não o produto final. O projeto ainda está concentrado no YouTube e utiliza os dados disponíveis no momento da coleta. Entre as evoluções planejadas estão:

- integração com Reddit, Google Trends e outras fontes;
- armazenamento histórico das coletas;
- cálculo de crescimento real entre períodos;
- relatórios completos de tendências;
- geração de ideias de conteúdo;
- filtros e ordenações interativas;
- autenticação e perfis de usuário;
- alertas e acompanhamento contínuo de assuntos;
- dashboards por plataforma, categoria e região;
- execução programada das coletas e análises;
- preparação para implantação em produção.

O propósito desta primeira etapa é estabelecer uma arquitetura confiável para que essas funcionalidades possam ser adicionadas de forma gradual, testável e sustentável.

## Segurança

- Não versione `.env` ou `.env.local`.
- Não exponha `OPENAI_API_KEY` nem `YOUTUBE_V3_API_KEY` no código ou no frontend.
- Troque imediatamente qualquer chave publicada por engano.
- Altere a senha padrão do PostgreSQL antes de disponibilizar o sistema fora da máquina local.

## Licença

Consulte o arquivo [LICENSE](./LICENSE) deste repositório.
