<div align="center">

# dopShowCaseNet

**Roadmap de implementação — plataforma full stack (.NET)**

[![CI - Build & Test](https://github.com/dop-s/dopShowCaseNet/actions/workflows/ci.yml/badge.svg)](https://github.com/dop-s/dopShowCaseNet/actions/workflows/ci.yml)
[![Dependencies Check & Update](https://github.com/dop-s/dopShowCaseNet/actions/workflows/dependencies.yml/badge.svg)](https://github.com/dop-s/dopShowCaseNet/actions/workflows/dependencies.yml)

Repositório orientado por um roadmap de evolução para stack completo: API .NET, múltiplos front-ends, persistência multi-provider, cache/NoSQL, observabilidade, segurança, Kubernetes, CI/CD e mensageria.

**Documento de referência:** [`Roadmap_FullStack_Senior_NET.pdf`](Roadmap_FullStack_Senior_NET.pdf)

</div>

---

## Índice

1. [Contexto](#contexto)
2. [Domínio base](#domínio-base)
3. [Roadmap por fases](#roadmap-por-fases)
4. [Estrutura prevista do repositório](#estrutura-prevista-do-repositório)
5. [CI/CD](#cicd)
6. [Instalação e uso](#instalação-e-uso)
7. [Contribuições](#contribuições)
8. [Licença](#licença)

## Contexto

O objetivo é partir de uma **API .NET** com **Clean Architecture** e um **CRUD de Pessoas**, evoluindo para uma plataforma full stack com:

- Front-ends múltiplos (React, Angular, Blazor WebAssembly)
- Mensageria e eventos
- Observabilidade e segurança (incluindo LGPD)
- Operação em **Kubernetes** e integração com **cloud** (Azure preferencial)

A persistência padrão começa **em memória**, com opção de alternar para **SQLite**, **SQL Server** ou **PostgreSQL**, e repositório alternativo para **MongoDB** onde o roadmap previr.

## Domínio base

Entidade **Pessoa** com os campos planejados:

| Campo | Descrição |
|--------|-----------|
| FirstName, LastName | Nome e sobrenome |
| DateOfBirth | Data de nascimento |
| GovernmentId | Identificador governamental |
| FatherName, MotherName | Filiação |
| Email, Phone | Contato |
| Address | Endereço |

Endpoints alvo da API: **`/api/v1/people`** (CRUD), com status HTTP adequados, validação e documentação OpenAPI em desenvolvimento.

## Roadmap por fases

Resumo alinhado ao PDF oficial. Cada fase lista **passos** e **critérios de aceitação** planejados.

### Fase 0 — Fundamentos e setup (semana 0)

**Passos:** monorepo (`backend/`, `frontend/`, `infra/`, `docs/`); DX (EditorConfig, `.gitignore`, lint/format, Conventional Commits); qualidade (analyzers C#, ESLint/Prettier, pre-commit); feature flags (`Microsoft.FeatureManagement`) e configuração 12-factor; DoR/DoD e ADRs iniciais.

**Critérios:** repositório inicial; build/test local com um comando; hooks ativos; padrões e **ADR-000** em `/docs`.

### Fase 1 — API monolítica modular (semanas 1–2)

**Passos:** Clean Architecture (Domain / Application / Infrastructure / Api); Minimal APIs + FluentValidation + Swagger; repositório com troca de provider (InMemory / SQLite / SQL Server / Postgres); paginação, validação e idempotência opcional (`Idempotency-Key`).

**Critérios:** CRUD em `/api/v1/people`; Swagger em DEV; testes unitários e de integração básicos.

### Fase 2 — Front-end ×3 (semanas 2–3)

**Passos:** três clientes — **people-web-react** (Vite + React Query + Router), **people-web-angular** (Angular standalone), **people-web-blazor** (WASM); design system comum (tokens) e acessibilidade (WCAG); fluxos CRUD com loading/empty/error; integração com a API e ambientes.

**Critérios:** as três UIs com CRUD completo contra `/api/v1/people`; testes de componentes essenciais em cada stack.

### Fase 3 — Persistência multi-provider (semanas 3–4)

**Passos:** EF Core Migrations; conexão por ambiente; troca de provider via `appsettings` sem recompilar; seed para DEV/testes; índices e planos revisados.

**Critérios:** migrações criadas e aplicadas; documentação de EXPLAIN / query plan das consultas principais.

### Fase 4 — NoSQL e cache (semanas 4–5)

**Passos:** **Redis** (cache por id e listagens, locks/idempotência); **MongoDB** (repositório alternativo para leitura/observabilidade/auditoria); **Polly** (retry, timeout, circuit breaker) para Redis/Mongo.

**Critérios:** hit ratio ≥ 70% em cenários simulados; operação resiliente a falhas momentâneas de cache/NoSQL.

### Fase 5 — Observabilidade e segurança (semanas 5–6)

**Passos:** Serilog + OpenTelemetry (traces e métricas) + correlation-id; rate limiting por rota; headers de segurança (CSP/HSTS); OIDC/OAuth2 (Authorization Code + PKCE); LGPD (minimização, mascaramento de PII, retenção).

**Critérios:** dashboards (p95, erros, RPS) e traces full-stack; rotas protegidas com JWT (scopes/roles) e validação de claims.

### Fase 6 — Docker e Kubernetes (semanas 6–7)

**Passos:** Dockerfiles multi-stage, imagens slim e não-root; K8s (Deployments, Services, Ingress, ConfigMaps, Secrets, probes, HPA); NetworkPolicies, PodDisruptionBudget, affinidades (básico).

**Critérios:** deploy em kind/minikube com TLS no ingress; simulação de falhas demonstrando resiliência (readiness/liveness).

### Fase 7 — CI/CD (semanas 7–8)

**Passos:** GitHub Actions ou Azure DevOps: lint + test + build + scan → imagem → deploy; ambientes (DEV/HML/PRD), secrets e approvals; canary / blue-green (Argo Rollouts / Flagger quando disponível).

**Critérios:** push em `main` publica em DEV; tags promovem PRD via canary; rollback automatizado em erro.

### Fase 8 — Alta disponibilidade e resiliência (semanas 8–9)

**Passos:** SLO/SLI; chaos drills (pods, latência); backups e restore testados; runbooks de incidentes; calibrar timeouts, retries, circuit breakers e bulkheads.

**Critérios:** MTTR médio inferior a 20 minutos em simulações; runbooks versionados e testados.

### Fase 9 — Microsserviços e mensageria (semanas 9–10)

**Passos:** extrair bounded context **People-Events** mantendo monólito modular no restante; dois serviços (producer .NET publicando PersonCreated/Updated/Deleted; consumer .NET ou Node); **Kafka** (streaming, retenção/replay) e **RabbitMQ** (filas, DLQ); padrões Outbox/Saga; idempotência por message-id (dedup Redis/Mongo); **SignalR** para tempo real nos front-ends.

**Critérios:** eventos fluindo por Kafka e RabbitMQ com reprocessamento controlado; SignalR nas UIs; consistência eventual comprovada em testes.

### Fase 10 — Testes automatizados (semanas 10–11)

**Passos:** unitários (domínio); integração com Testcontainers (DB/mensageria); contrato (Pact); E2E web (Playwright/Cypress); mobile opcional (Detox/Appium).

**Critérios:** cobertura útil ≥ 80% nas regras críticas; pipeline estável, sem flakiness crítico.

### Fase 11 — Cloud e certificações (semana 11)

**Passos:** Azure preferencial (AKS, App Service, Azure SQL/Postgres, Cache for Redis, Cosmos com API Mongo opcional, Key Vault, Monitor); mapear certificações (AZ-204, AZ-400, AZ-305) e alternativas AWS/GCP.

**Critérios:** deploy em cloud com observabilidade e Key Vault; plano de estudos e datas-alvo para certificações.

### Fase 12 — Hardening e go-live (semana 12)

**Passos:** hardening de segurança (headers, TLS), rotação de secrets/tokens; LGPD DSR (acesso, retificação, remoção) e retenção; checklists de go-live, custos, on-call e rollback.

**Critérios:** pentest interno sem issues críticas; go-live playbook aprovado por engenharia e produto.

## Estrutura prevista do repositório

Conforme a Fase 0:

| Pasta | Função |
|--------|--------|
| `backend/` | API .NET e serviços |
| `frontend/` | Apps React, Angular, Blazor |
| `infra/` | Docker, Kubernetes, IaC |
| `docs/` | ADRs, padrões, runbooks |

A árvore atual do repositório pode evoluir para esse layout à medida que as fases forem implementadas.

## CI/CD

Este repositório inclui workflows em [`.github/workflows/`](.github/workflows/) (por exemplo `ci.yml`, `cd.yml`, `dependencies.yml`, `pr-automation.yml`, entre outros). Para detalhes dos pipelines, consulte [`.github/README_WORKFLOWS.md`](.github/README_WORKFLOWS.md) e [`.github/QUICKSTART.md`](.github/QUICKSTART.md).

## Instalação e uso

Instruções específicas de build e execução serão preenchidas conforme o `backend/` e os front-ends forem adicionados ao monorepo. Enquanto a estrutura estiver em transição, use os scripts e arquivos de solução do projeto .NET quando existirem na raiz ou em subpastas.

```bash
git clone https://github.com/dop-s/dopShowCaseNet.git
cd dopShowCaseNet
# Exemplo quando a API estiver disponível:
# dotnet restore && dotnet build && dotnet test
```

## Contribuições

Contribuições são bem-vindas. Consulte [CONTRIBUTING.md](CONTRIBUTING.md) e [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). Para reportar vulnerabilidades, siga [SECURITY.md](SECURITY.md).

## Licença

Veja [LICENSE.md](LICENSE.md).

## Contato

### 💬 Suporte Técnico
Para questões técnicas, problemas ou sugestões:
- **Issues**: [GitHub Issues](https://github.com/daniloopinheiro/dopNetHL7/issues)
- **Discussions**: [GitHub Discussions](https://github.com/daniloopinheiro/dopNetHL7/discussions)

### 👨‍💻 Autor
**Danilo O. Pinheiro**  
Especialista em .NET, Clean Architecture e Interoperabilidade em Saúde

- **Email Pessoal**: [daniloopro@gmail.com](mailto:daniloopro@gmail.com)
- **Email Empresarial**: [devsfree@devsfree.com.br](mailto:devsfree@devsfree.com.br)
- **Consultoria**: [contato@dopme.io](mailto:contato@dopme.io)
- **LinkedIn**: [Danilo O. Pinheiro](https://www.linkedin.com/in/daniloopinheiro/)

### 🏢 Empresas
- **[DevsFree](https://devsfree.com.br)**: Desenvolvimento de Software
- **[dopme.io](https://dopme.io)**: Consultoria e Soluções Tecnológicas

<div align="center">

**⭐ Se este projeto foi útil, deixe uma estrela no GitHub! ⭐**

<p>
Feito com ❤️ por <strong>Danilo O. Pinheiro</strong><br/>  
<a href="https://devsfree.com.br" target="_blank">DevsFree</a> • <a href="https://dopme.io" target="_blank">dopme.io</a>  
</p>

</div>