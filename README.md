# AI Job Analyzer

Plataforma local para coletar, normalizar e analisar vagas de emprego com ajuda de IA. Ela mostra quais habilidades, senioridades e áreas aparecem nas vagas coletadas e também analisa uma vaga avulsa colada pelo usuário.

> Status: em desenvolvimento. MVP local, sem Docker e sem AWS.

## Por que este projeto existe

Em uma manhã de quinta feira tive a ideia de criar esse projeto para me ajudar a analisar o que vagas de algumas áreas pedem.

## Funcionalidades do MVP

O MVP faz cinco coisas. Ele lê vagas de JSON (saída do extrator) e de PDFs digitais, converte tudo para um formato comum e evita duplicatas. Classifica as vagas coletadas com a API da OpenAI (habilidades, senioridade, área e requisitos), validando a resposta antes de salvar. Guarda tudo em PostgreSQL com schema versionado. Permite buscar e filtrar vagas e ver gráficos de habilidades. E permite colar o texto de uma vaga e receber uma análise sob demanda.

## Arquitetura

```
Fontes (JSON, PDF) → Ingestão Python → API Spring Boot → PostgreSQL → React
                                              ↑
                              Análise de vaga avulsa (OpenAI)
```

O Python coleta, normaliza e classifica em lote. A API Java valida e grava, e também faz a análise sob demanda de uma vaga avulsa. O PostgreSQL guarda os dados. O React apresenta buscas, métricas e a tela de análise.

## Estrutura do repositório

```
ai-job-analyzer/
  ingestion/   extrator, adaptadores (JSON, PDF), normalização e pipeline Python
  backend/     API Spring Boot
  frontend/    aplicação React + TypeScript + Vite
  data/        entradas locais, ignorado pelo Git
  docs/        arquitetura, decisões, formato do extrator, dados de demonstração
```

## Tecnologias

Python 3.12+, Spring Boot (Java 21, Maven), PostgreSQL, Flyway, React com TypeScript e Vite, e a API da OpenAI.

## Pré-requisitos

Git, Python 3.12 ou superior, JDK 21 (não apenas o JRE), Node.js LTS com npm, PostgreSQL instalado e iniciado como serviço, e uma chave da API da OpenAI.

## Configuração

### Banco de dados

Crie o banco e um usuário local no psql ou no pgAdmin:

```sql
CREATE DATABASE ai_job_analyzer;
CREATE USER ai_job_user WITH PASSWORD 'defina-uma-senha-local';
GRANT ALL PRIVILEGES ON DATABASE ai_job_analyzer TO ai_job_user;
```

Dependendo da versão do PostgreSQL, pode ser necessário também conceder permissões no schema `public`. As tabelas são criadas pelas migrações Flyway quando o backend sobe pela primeira vez.

### Variáveis de ambiente

Nunca coloque valores reais no repositório. Copie `.env.example` e preencha localmente.

| Variável | Usada por | Descrição |
|---|---|---|
| DB_URL | backend | URL JDBC do PostgreSQL |
| DB_USER | backend | usuário do banco |
| DB_PASSWORD | backend | senha do banco |
| OPENAI_API_KEY | backend e ingestion | chave da API da OpenAI |
| OPENAI_MODEL | backend e ingestion | modelo usado nas classificações |
| PROMPT_VERSION | ingestion | versão do prompt registrada com cada resultado |
| API_BASE_URL | ingestion | endereço da API local |
| VITE_API_URL | frontend | endereço da API local |

A chave da OpenAI nunca vai para o frontend. Variáveis com prefixo `VITE_` ficam visíveis no navegador, por isso só o endereço da API entra ali.

## Como rodar

Use três terminais, com o PostgreSQL já iniciado.

Terminal 1, API (Windows):

```
cd backend
mvnw.cmd spring-boot:run
```

Terminal 1, API (macOS e Linux):

```
cd backend
./mvnw spring-boot:run
```

Terminal 2, frontend:

```
cd frontend
npm install
npm run dev
```

Abra o endereço local que o Vite mostrar no terminal.

Terminal 3, ingestão (Windows):

```
cd ingestion
py -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Terminal 3, ingestão (macOS e Linux):

```
cd ingestion
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Rode primeiro em modo de teste, sem enviar nada, e depois importe. Os nomes exatos dos comandos dependem da CLI implementada, ajuste aqui quando ela existir:

```
python -m ingestion process ./data/entrada --dry-run
python -m ingestion import ./data/entrada
```

Rodar o mesmo lote duas vezes não duplica vagas.

## Análise de uma vaga avulsa

Na tela de análise, cole o texto de uma vaga e o sistema devolve habilidades, senioridade, área e requisitos. A chamada à OpenAI é feita pelo backend, com a chave guardada em variável de ambiente. O texto tem limite de tamanho e a resposta é validada antes de ser exibida. Por padrão, o texto colado não é gravado no banco.

## Endpoints principais

| Método | Rota | Função |
|---|---|---|
| POST | /api/v1/imports | recebe um lote de vagas, valida e grava sem duplicar |
| GET | /api/v1/jobs | lista vagas com paginação e filtros |
| GET | /api/v1/jobs/{id} | detalhe da vaga e sua classificação |
| GET | /api/v1/analytics/skills | contagem de habilidades para os gráficos |
| POST | /api/v1/analyses | analisa o texto de uma vaga avulsa |
| GET | /api/v1/health | verificação de saúde da API |

## Testes

```
# ingestion
cd ingestion && pytest

# backend
cd backend && ./mvnw test

# frontend
cd frontend && npm test
```

## Uso de IA, custo e qualidade

A classificação é automática e pode errar. Cada resultado guarda o modelo usado, a versão do prompt, a data e o estado da operação. Respostas inválidas são registradas e nunca salvas como se fossem corretas. Resultados são reaproveitados por cache quando o texto, o modelo e a versão do prompt não mudam. Para controlar gastos, comece com poucas vagas, use a opção de limite da CLI e configure avisos de orçamento no painel da OpenAI. A taxa de acerto medida em uma amostra rotulada manualmente fica em `docs/ai-evaluation.md`.

## Dados e privacidade

O repositório contém apenas dados sintéticos de demonstração em `docs/demo-data/`. Currículos, PDFs de terceiros, chaves e dados pessoais ficam fora do Git. As fontes de vagas e seus termos de uso estão documentados em `docs/sources.md`.

## Limitações conhecidas

PDFs escaneados não são lidos (sem OCR) e viram `needs_review`. A normalização de nomes de habilidades é simples e pode separar variações do mesmo termo. A qualidade da classificação depende do modelo e do prompt. Não há agendamento automático nem autenticação, pois o projeto é pensado para uso local.

## Próximos passos

Leitor de currículo com consentimento e exclusão de dados, OCR, agendamento, Docker e publicação.

## Licença

<!-- Defina antes de publicar. -->
