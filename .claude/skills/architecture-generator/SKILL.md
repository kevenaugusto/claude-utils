---
name: architecture-generator
description: "Conduz brainstorming arquitetural estruturado (12 dimensões) e gera documento <projeto>-architecture.md completo"
argument-hint: "<ideia + restrições + funcionalidades>"
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, Task, Agent]
disable-model-invocation: true
---

# /architecture-generator

Conduz um **brainstorming arquitetural interativo** dimensão por dimensão e produz um arquivo `docs/architecture/<slug>-architecture.md` no padrão do `finance-app-architecture.md`.

---

## Como usar

```bash
/architecture-generator "App de finanças local-only: importação OFX/OCR, classificação automática, alertas de limite, relatórios PDF. Stack: Python + React. Desktop. Sem nuvem."
```

---

## O que faz (fluxo)

1. **Parse** do input → extrai ideia, features, restrições, contexto
2. **Inferir slug** do projeto (ex.: `finance-app`, `saas-agendamento`)
3. **Loop 12 dimensões** (uma por vez, interativo):
   - Mostra decisões já tomadas (contexto)
   - Apresenta 2-3 opções com tabela Prós/Contras + recomendação fundamentada
   - Você responde: `A`, `B`, `C`, `Outro`, ou `TBD`
   - Valida conflitos com escolhas anteriores (ex.: serverless + SQLite local)
4. **Resumo consolidado** → tabela única com todas decisões
5. **Confirmação** → "Está tudo correto? gero o arquivo?"
6. **Gera** `docs/architecture/<slug>-architecture.md` (~700 linhas)
7. **Salva** e sugere próximos passos (ADRs, Incremento 1, etc.)

---

## 12 Dimensões

| # | Dimensão | Pergunta-chave |
|---|---|---|
| 1 | **Modelo da Aplicação** | "O usuário final interage como?" (Web/Desktop/Mobile/CLI/API/Híbrido) |
| 2 | **Arquitetura de Alto Nível** | "Como componentes se organizam?" (Modular Monolith/Microservices/Serverless/Clean/Hexagonal/Event-driven) |
| 3 | **Linguagem / Backend** | "Qual stack backend?" (Python/FastAPI, Node/NestJS, Go, Rust, Java, .NET) |
| 4 | **Frontend** | "Qual stack frontend?" (React/Vue/Svelte + TS, Next/Remix/Astro, nativo) |
| 5 | **Banco de Dados** | "Onde dados vivem?" (SQL/NoSQL/NewSQL/embedded + ORM) |
| 6 | **Infraestrutura / Deploy** | "Onde roda?" (Local-only/Docker/VPS/Cloud/Serverless/K8s) |
| 7 | **Auth / Autorização** | "Quem acessa o quê?" (Próprio/JWT, OAuth/OIDC, mTLS, RBAC/ABAC) |
| 8 | **Comunicação** | "Como serviços conversam?" (REST/GraphQL/gRPC/tRPC/WS/SSE/MQ) |
| 9 | **Observabilidade** | "Como observamos?" (Logs/Métricas/Tracing/Alerting + nível investimento) |
| 10 | **Qualidade** | "Como garantimos?" (Testes/Lint/CI/CD/Contract Testing/ADRs) |
| 11 | **Segurança** | "Como protegemos?" (STRIDE/Secrets/Rate Limit/CORS/CSP/Scanning/Compliance) |
| 12 | **Dados Sensíveis / LGPD** | "Como tratamos dados pessoais?" (Criptografia/Retenção/Anonimização/Consentimento/DSR) |

---

## Princípios do Brainstorming

| Princípio | Prática |
|---|---|
| **Uma dimensão por vez** | Não sobrecarregar; avançar só após decisão confirmada |
| **Sugerir com fundamentação** | "Recomendo X porque Y; alternativa viável: Z" |
| **Nunca decidir pelo usuário** | Opções claras; usuário escolhe |
| **Validar consistência** | Conflitos explícitos: "Escolha X na D2 conflita com Y na D3. Sugiro..." |
| **Permitir "pular por agora"** | Marcar `TBD` e retomar no final |
| **Revisar tudo junto ao final** | Resumo consolidado → aprovação → geração |

---

## Output Final: `<slug>-architecture.md`

```markdown
# Arquitetura de <Nome do Projeto>

> Documento gerado após sessão de brainstorming técnico.
> Data: YYYY-MM-DD

---

## 1. Visão Geral da Arquitetura
- Parágrafo propósito/escopo/premissas
- Diagrama ASCII (box-drawing ou mermaid-style)
- Premissas e Restrições (tabela)

---

## 2. Stack Tecnológica
| Camada | Tecnologia | Versão | Justificativa |
|---|---|---|---|
| Runtime | ... | ... | ... |
| Framework HTTP | ... | ... | ... |
| Banco de Dados | ... | ... | ... |
| ORM | ... | ... | ... |
| Migrações | ... | ... | ... |
| Frontend | ... | ... | ... |
| Build Tool | ... | ... | ... |
| Estado Server | ... | ... | ... |
| Estado UI | ... | ... | ... |

---

## 3. Modelagem de Dados
- Entidades principais (tabelas com campos, tipos, constraints)
- Relacionamentos e regras de negócio (FKs, enums, triggers)
- Diagrama ER ASCII

---

## 4. Componentes e Responsabilidades
### 4.1 Backend (<Framework>)
- Módulos de API (tabela endpoints)
- Serviços (camada de negócio)
- Outros: Bot, Workers, etc.

### 4.2 Frontend (<Framework>)
- Páginas/Rotas
- Hooks Customizados
- Stores (Estado Global)

---

## 5. Fluxos de Integração (ASCII + detalhes)
5.1 Importação OFX → Parser → Classifier → Save → Review
5.2 OCR Comprovante → Upload → Extração → Classificação → Save
5.3 Alerta de Limite → Scheduler → Verificação → Notificação
5.4 Geração PDF → Template → Dados → Render → Download

---

## 6. Decisões Arquiteturais e Trade-offs
| # | Decisão | Escolha | Alternativas | Trade-off Principal |
|---|---|---|---|---|
| D1 | ... | ... | ... | ... |

---

## 7. Estrutura do Projeto (tree ASCII)

---

## 8. Segurança e Conformidade
- Threat Model (STRIDE resumo)
- Secrets, Rate Limiting, CORS, CSP, Dependency Scanning
- LGPD/GDPR: criptografia, retenção, consentimento, DSR

---

## 9. Observabilidade
| Pilar | Ferramenta | Configuração |
|---|---|---|
| Logs | ... | structured JSON, correlation IDs |
| Métricas | ... | RED/USE dashboards |
| Tracing | ... | sampling rate |
| Alerting | ... | canais, runbooks |

---

## 10. Roadmap de Incrementos
| Inc | Foco | Entregáveis-chave | Estimativa |
|---|---|---|---|
| 1 | Fundação | Setup, models, CRUD base, auth | ... |
| 2 | Core A | ... | ... |
| 3 | Core B | ... | ... |

---

## 11. Riscos e Mitigações
| Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|

---

## 12. Decisões Pendentes (TBD)
| Item | Contexto | Bloqueador |
|---|---|---|