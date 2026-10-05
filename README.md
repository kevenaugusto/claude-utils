# claude-utils

Toolbox de comandos, agentes e skills reutilizáveis para Claude Code. Os arquivos
em `.claude/` são copiados para outros projetos — não são dependência deste
repositório.

## Fluxo principal

```
/architecture-generator "sua ideia"
        ↓
docs/architecture/<projeto>-architecture.md
        ↓
/implementation-planner "<arquivo>" <n>
        ↓
docs/plans/incremento-N-*.md
```

Arquitetura primeiro, planos depois. O segundo passo só aceita documentos
produzidos pelo primeiro.

---

## Comandos

### `/architecture-generator "<ideia + restrições + funcionalidades>"`

Conduz um brainstorming de **12 dimensões, uma por vez**, e gera um documento de
arquitetura de ~700 linhas com 12 seções.

Em cada dimensão o agente mostra 2–3 opções com prós/contras, dá uma
recomendação fundamentada, e devolve o controle: você responde `A`, `B`, `C`,
`Outro` ou `TBD`. Após cada escolha ele checa conflitos conhecidos (serverless +
banco embedded, local-only + cloud DB, offline-first + serverless, etc.) e aponta
antes de avançar.

As 12 dimensões: modelo da aplicação · arquitetura de alto nível · linguagem /
backend · frontend · banco de dados · infraestrutura · auth · comunicação ·
observabilidade · qualidade · segurança · dados sensíveis e LGPD.

No final, resumo consolidado para aprovação, e só então grava o arquivo.

### `/implementation-planner "<caminho-do-architecture-md> <numero-incremento>"`

Lê um documento de arquitetura e gera os planos de implementação do incremento
escolhido.

Funciona como função pura: **não faz perguntas**. Todas as decisões técnicas já
foram tomadas no brainstorming. Se faltar informação para um plano correto,
emite erro em vez de supor.

O escopo do incremento é derivado do cruzamento entre as seções do documento —
roadmap, componentes, modelagem e estrutura —, e os passos são derivados da stack
declarada. Por isso funciona para qualquer stack, sem assumir nada.

Emite de 3 a 4 arquivos em `docs/plans/`:

| Arquivo | Condição |
|---------|----------|
| `incremento-N-visao-geral.md` | sempre |
| `incremento-N-backend.md` | documento tem Seção 4.1 |
| `incremento-N-frontend.md` | documento tem Seção 4.2 |
| `incremento-N-testes.md` | incremento introduz código a testar |

Para CLI, API ou worker — que não têm camada de frontend — são 3 arquivos, não 4.

### `/finance-app-implementation-planner "<caminho> <n>"`

Versão legada, **específica do projeto finance-app**. Mantida porque consome um
documento de arquitetura escrito à mão, não gerado pelo `architecture-generator`,
e carrega convenções específicas (monetário como string, `StrEnum`, SQLAlchemy
2.0, React Query, Vitest).

Não aplique a outros documentos.

---

## Estrutura

Três categorias, com mecânicas distintas:

```
.claude/
├── skills/       slash commands invocáveis por você
│   └── <nome>/SKILL.md
├── agents/       subagentes com a lógica de trabalho
│   └── <nome>.md
└── references/   documentos lidos por path pelos agentes
    └── <nome>.md
```

| Diretório | Como funciona | Como editar |
|-----------|---------------|-------------|
| `skills/` | vira `/nome-do-slug`. `disable-model-invocation: true` impede que o Claude invoque sozinho | frontmatter: `name`, `description`, `argument-hint`, `allowed-tools` |
| `agents/` | invocado pela skill correspondente; contém o mecanismo | corpo do arquivo |
| `references/` | não é invocável; é lido por path via `Read` | corpo do arquivo |

O layout `skills/<nome>/SKILL.md` é o suportado. Arquivo `.md` solto direto em
`skills/` **não carrega** — o agente precisa do subdiretório com `SKILL.md`.

### Inventário

| Slash command | Agente | Referência |
|---------------|--------|------------|
| `/architecture-generator` | `agents/architecture-generator.md` | `references/architecture-generator-guide.md` |
| `/implementation-planner` | `agents/implementation-planner.md` | `references/implementation-plan-templates.md` |
| `/finance-app-implementation-planner` | `agents/finance-app-implementation-planner.md` | `references/finance-app-plan-templates.md` |

---

## Uso em outro projeto

Copie os diretórios necessários:

```bash
# só o fluxo de arquitetura + planos
cp -r .claude/skills/architecture-generator .claude/agents/architecture-generator.md .claude/references/architecture-generator-guide.md /outro-projeto/.claude/
```

Três arquivos por workflow. A skill, o agente e a referência que ele lê.

---

## Notas de manutenção

**Resolução de seções por título.** O `implementation-planner` resolve as seções
do documento por título, com a posição numérica como fallback. Por título
sobrevive a reordenação do template; por número quebra em silêncio — foi o que
quebrou no planner do finance-app, que lia a Seção 8 enquanto o gerador escrevia
o roadmap na Seção 10.

**Isolamento entre workflows.** Os workflows genéricos não referenciam nada do
finance-app. Ao escrever nesses arquivos, não use conhecimento de outro projeto
como exemplo: o guardião é o auto-check do agente, que exige que nenhuma stack
não declarada no documento lido apareça no plano.

**Regenerar documento.** Se rodar o `/architecture-generator` de novo sobre o
mesmo projeto, sobrescreve o `.md`. Guarde o anterior se quiser comparar.

**Convenções técnicas** — thresholds de cobertura, serialização de valores
monetários, StrEnum, escolha de ORM — nos arquivos do finance-app são específicas
daquele projeto e não valem para os workflows genéricos.
