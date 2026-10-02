---
name: implementation-planner
description: "Gera planos de implementação por incremento a partir de documento de arquitetura (padrão finance-app)"
argument-hint: "<caminho-architecture-md> <numero-incremento>"
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, Task, Agent]
disable-model-invocation: true
---

# /implementation-planner

Gera planos de implementação detalhados (backend, frontend, testes) a partir de um documento de arquitetura gerado pelo `/architecture-generator`, seguindo o padrão do `finance-app`.

## Uso

```bash
# No terminal, dentro do projeto que tem o arquivo de arquitetura:
/implementation-planner "docs/architecture/meu-projeto-architecture.md" 1
```

**Parâmetros:**
1. **Caminho do arquivo de arquitetura** — relativo à raiz do projeto (ex.: `docs/architecture/finance-app-architecture.md`)
2. **Número do incremento** — inteiro (1, 2, 3, 4, 5, 6...)

## O que acontece

1. **Lê** o documento de arquitetura especificado
2. **Extrai** dados das seções relevantes (Roadmap, Modelagem, Componentes, Fluxos, Estrutura, Stack, Decisões, Riscos)
3. **Gera 3-4 arquivos** em `docs/plans/`:
   - `incremento-N-visao-geral.md` — orquestrador do incremento
   - `incremento-N-backend.md` — plano detalhado backend
   - `incremento-N-frontend.md` — plano detalhado frontend
   - `incremento-N-testes.md` — plano de testes unitários/integração (opcional)

## Exemplo

```bash
# Gerar planos do Incremento 1 para finance-app
/implementation-planner "docs/architecture/finance-app-architecture.md" 1

# Gerar planos do Incremento 2 (quando pronto)
/implementation-planner "docs/architecture/finance-app-architecture.md" 2
```

## Saída esperada (Incremento 1)

```
docs/plans/
├── incremento-1-visao-geral.md     # Visão geral, decisões, sequência, DoD
├── incremento-1-backend.md         # 10 passos: setup → models → migrações → schemas → rotas → validação
├── incremento-1-frontend.md        # 9 passos: setup → tipos → api → store → hooks → layout → páginas → validação
└── incremento-1-testes.md          # Vitest + Testing Library: format, api, hooks (cobertura 80/80/70)
```

## Pré-requisitos

- Documento de arquitetura deve seguir o padrão do `/architecture-generator` (12 seções, Seção 8 com roadmap de incrementos)
- Projeto deve ter estrutura `docs/architecture/` e `docs/plans/` (criadas automaticamente se não existirem)

## Personalização

Para adaptar o estilo dos planos gerados, edite `.claude/references/implementation-plan-templates.md`:
- Seções dos templates (A, B, C, D) → estrutura dos markdowns gerados
- Regras de extração → quais seções da arquitetura mapear para cada parte do plano
- Convenções herdadas → monetário=string, enums=StrEnum, fetch wrapper, etc.