---
name: implementation-planner
description: "Gera planos de implementação por incremento a partir de documentos de arquitetura gerados pelo architecture-generator"
argument-hint: "<caminho-do-architecture-md> <numero-incremento>"
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, Task, Agent]
disable-model-invocation: true
---

# /implementation-planner

Gera planos de implementação por incremento a partir de um documento de
arquitetura produzido pelo `/architecture-generator`.

---

## Como usar

```bash
/implementation-planner "docs/architecture/meu-projeto-architecture.md" 1
```

**Parâmetros:**

1. **Caminho do documento de arquitetura** — relativo à raiz do projeto
2. **Número do incremento** — inteiro, conforme a tabela da Seção 10

---

## O que faz

1. Lê o documento e resolve as seções **por título**
2. Confirma que o incremento N existe na tabela de roadmap
3. **Cruza** roadmap, componentes, modelagem e estrutura para isolar o escopo
4. Deriva os passos a partir da stack declarada na arquitetura
5. Gera os arquivos em `docs/plans/`

Não faz perguntas: todas as decisões técnicas já foram tomadas no brainstorming
do gerador. Se faltar informação, emite erro em vez de supor.

---

## Saída

```
docs/plans/
├── incremento-N-visao-geral.md   ← sempre
├── incremento-N-backend.md       ← se a arquitetura tiver Seção 4.1
├── incremento-N-frontend.md      ← se a arquitetura tiver Seção 4.2
└── incremento-N-testes.md        ← se o incremento introduzir código a testar
```

Para CLI, API ou worker — que não têm camada de frontend — são gerados 3
arquivos, não 4. A regra é a presença real de conteúdo na arquitetura.

---

## Pré-requisitos

O documento de arquitetura precisa ter as seções que o agente usa:

| Seção | Uso |
|-------|-----|
| 10 — Roadmap | Escopo e ordem dos incrementos |
| 4 — Componentes | O que existe e sua responsabilidade |
| 3 — Modelagem | Entidades e campos |
| 7 — Estrutura | Árvore de arquivos |
| 2 — Stack | Comandos e ferramentas |

Documentos sem essas seções não produzem plano — o agente emite erro
especificando qual faltou.

---

## Arquivos relacionados

| Arquivo | Papel |
|---------|-------|
| `.claude/agents/implementation-planner.md` | **Agente** — resolução de seções, cruzamento, derivação, erros |
| `.claude/references/implementation-plan-templates.md` | **Templates** — estrutura dos 4 arquivos de plano |
| `.claude/agents/architecture-generator.md` | Produz o documento que este agente consome |

---

## Personalização

Para adaptar os planos gerados, edite `.claude/references/implementation-plan-templates.md`:
- Seções de cada template → estrutura dos markdowns gerados
- Mapa de seções → quais seções da arquitetura alimentam cada plano
- Regras de ausência → o que fazer quando um dado não está no documento