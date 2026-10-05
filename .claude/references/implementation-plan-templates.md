# Templates de Planos de Implementação

> **Documento de referência** — não é invocável. Lido por path pelo agente
> `agents/implementation-planner.md` (templates A, B, C, D).
>
> ⚠️ **Regra de preenchimento:** não copie conteúdo de outros projetos.
> Preencha **exclusivamente** a partir do documento de arquitetura informado.
> Todo marcador `<...>` deve ser substituído por valor real extraído da
> arquitetura — nunca por suposição, palpite ou stack suposta.

---

## Mapa: qual template usa quais seções da arquitetura

| Template | Seções consumidas | Omitir quando |
|---|---|---|
| A — visão geral | 2, 6, 10, 11 | nunca |
| B — backend | 2, 3, 4.1, 7, 10 | Seção 4.1 ausente |
| C — frontend | 2, 4.2, 7, 10 | Seção 4.2 ausente |
| D — testes | 2, 10 | Sem código a testar no incremento |

---

## A — `incremento-N-visao-geral.md`

> Alinhado à arquitetura em `docs/architecture/<projeto>-architecture.md`.
> Objetivo do incremento: <foco declarado na tabela de roadmap, Seção 10>.

---

### 0. Decisões técnicas adotadas

Extrair da Seção 2 (Stack Tecnológica) e da Seção 6 (Decisões e Trade-offs).
Se qualquer convenção estiver ausente na arquitetura, marcá-la como
`<não definida na arquitetura — assumir padrão de mercado>` em vez de inventar.

| Tema | Decisão | Por quê |
|------|---------|---------|
| <extrair de Seção 2> | <valor literal> | <justificativa literal da arquitetura> |
| <extrair de Seção 6> | <valor literal> | <trade-off declarado> |

---

### 1. Divisão do plano

| Arquivo | Conteúdo |
|---------|----------|
| `incremento-N-visao-geral.md` (este) | Escopo, decisões, sequência, DoD |
| `incremento-N-backend.md` | <ou "omitido — sem camada de backend"> |
| `incremento-N-frontend.md` | <ou "omitido — sem camada de frontend"> |
| `incremento-N-testes.md` | Planos de teste |

---

### 2. Escopo do incremento

Resultado do cruzamento Seções 10 + 4 + 3 + 7. **Esta é a seção mais
importante** — é a fronteira do que será feito.

**Componentes necessários** (da Seção 4, cruzados com os entregáveis da Seção 10):
- `<nome do componente>` — <responsabilidade, como consta na Seção 4>

**Entidades envolvidas** (da Seção 3, apenas as tocadas pelos componentes acima):
- `<Entidade>` — <campos-chave e relações declaradas na Seção 3>

**Arquivos de destino** (da Seção 7):
```
<caminhos literais extraídos da árvore da Seção 7>
```

**Entregáveis declarados no roadmap** (Seção 10):
- <entregável, copiado literalmente>

**Lacunas** — entregáveis sem componente correspondente na Seção 4:
- <entregável órfão, se houver>

---

### 3. Sequência global de execução

1. **Dados** — <entidades e migrações deste incremento>
2. **Backend** — <componentes da Seção 4.1>
3. **Frontend** — <componentes da Seção 4.2>
4. **Integração** — <como as pontas se conectam>
5. **Validação** — <como verificar, conforme comandos da Seção 2>

> **Justificativa da ordem:** <por que dados antes da API, conforme as
> dependências declaradas na Seção 10>.

---

### 4. O que fica explicitamente de fora

Incrementos N+1 em diante, conforme a tabela da Seção 10:
- <foco do incremento seguinte>
- <foco do incremento posterior>

Itens marcados como TBD na Seção 12 (Decisões Pendentes):
- <item TBD que afeta este incremento>

---

### 5. Definition of Done

Critérios mensuráveis, derivados do escopo:
- [ ] <entidade X criada com os campos declarados na Seção 3>
- [ ] <componente Y responde conforme a Seção 4>
- [ ] <validação Z passa>
- [ ] <build/teste passa, comando conforme a Seção 2>

---

### 6. Próximos passos

1. Executar `incremento-N-backend.md`
2. Executar `incremento-N-frontend.md`
3. Implementar testes conforme `incremento-N-testes.md`

---

## B — `incremento-N-backend.md`

> Parte do plano do Incremento N. Visão geral em `incremento-N-visao-geral.md`.
>
> **Omitir este arquivo** se a Seção 4.1 da arquitetura não existir.

---

### 1. Stack e decisões

Extrair literalmente da Seção 2. Não assumir versões nem ferramentas.

- Runtime: <da Seção 2>
- Gerenciador de dependências: <da Seção 2>
- Camada HTTP: <da Seção 2>
- Persistência / ORM: <da Seção 2>
- Migrações: <da Seção 2>
- Validação: <da Seção 2>

Convenções não funcionais da Seção 6 que afetam o backend:
- <convenção, copiada literalmente>

---

### 2. Estrutura de arquivos a criar

Subtree da Seção 7, filtrado pelo escopo do incremento. **Copiar os paths
literais da arquitetura**, não inventar uma estrutura convencional.

```
<tree extraído da Seção 7, com apenas os paths do escopo>
```

---

### 3. Passo a passo

Cada passo deriva da stack da Seção 2 e de um componente da Seção 4.1.

### Passo 1 — <etapa derivada da stack>

- Objetivo: <o que este passo entrega>
- Comando(s): <comandos reais daquela stack, extraídos ou derivados da Seção 2>
- Arquivos: <paths literais>
- Critério de aceite: <verificável>

### Passo 2 — <etapa>

<mesma estrutura>

### Passo N — <etapa>

<mesma estrutura>

> A quantidade de passos é determinada pelo escopo do incremento, não fixa.

---

### 4. Componentes do incremento

Da Seção 4.1, filtrada pelo cruzamento.

### `<NomeDoComponente>`

- Responsabilidade: <literal da Seção 4>
- Interfaces expostas: <endpoints, signatures ou eventos — como a Seção 4 definir>
- Dependências: <outros componentes do incremento>
- Arquivos: <paths literais da Seção 7>

---

### 5. Entidades e persistência

Da Seção 3, filtrada pelo escopo.

### `<Entidade>`

| Campo | Tipo | Constraints |
|-------|------|-------------|
| <campo> | <tipo> | <null/not null/unique/default> |

- Relações: <FKs, com cardinalidade>
- Enums: <valores literais>
- Migração: <nome do arquivo de migração conforme convenção da Seção 2>

---

### 6. Definition of Done (backend)

- [ ] <entidades persistidas conforme Seção 3>
- [ ] <componentes da Seção 4.1 respondendo>
- [ ] <validação automatizada passa>

---

### 7. Riscos do backend

| Risco | Mitigação |
|-------|-----------|
| <extrair da Seção 11 da arquitetura, filtrado> | <...> |

---

## C — `incremento-N-frontend.md`

> Parte do plano do Incremento N. Visão geral em `incremento-N-visao-geral.md`.
>
> **Omitir este arquivo** se a Seção 4.2 da arquitetura não existir.

---

### 1. Stack e decisões

- Framework: <da Seção 2>
- Roteamento: <da Seção 2>
- Gerenciamento de estado: <da Seção 2>
- Cliente HTTP: <da Seção 2>
- Build: <da Seção 2>

Convenções da Seção 6 que afetam o frontend:
- <convenção, copiada literalmente>

---

### 2. Estrutura de arquivos a criar

Subtree da Seção 7, filtrado. Paths literais.

```
<tree da Seção 7>
```

---

### 3. Passo a passo

### Passo 1 — <etapa derivada da stack>

- Objetivo: <...>
- Comando(s): <...>
- Arquivos: <paths literais>
- Critério de aceite: <...>

---

### 4. Componentes do incremento

Da Seção 4.2, filtrada pelo escopo.

### `<NomeDoComponente>`

- Responsabilidade: <literal da Seção 4>
- Rotas / entradas: <...>
- Dependências de dados: <endpoints ou hooks, conforme Seção 4>
- Arquivos: <paths literais>

---

### 5. Definition of Done (frontend)

- [ ] <build passa, comando da Seção 2>
- [ ] <componentes da Seção 4.2 funcionais>
- [ ] <formatação e tipos alinhados ao contrato da API>

---

### 6. Riscos do frontend

| Risco | Mitigação |
|-------|-----------|
| <extrair da Seção 11> | <...> |

---

## D — `incremento-N-testes.md`

> Baseado em `incremento-N-backend.md` e `incremento-N-frontend.md`.
>
> **Omitir** se o incremento não introduz código a testar.

---

### 1. Escopo e decisão

**O que testar** — apenas módulos com lógica pura ou efeito verificável:
- <módulo> — <por que é testável>
- <módulo> — <...>

**O que não testar unitariamente** — validado em compilação ou integração:
- <tipos, constantes, wiring>

**Justificativa:** <por que esta pirâmide para este projeto>

---

### 2. Ferramentas

**Derivadas da Seção 2** — não copiar as de outro projeto.

| Ferramenta | Papel |
|------------|-------|
| <framework de teste declarado na Seção 2> | <...> |
| <auxiliar, se declarado> | <...> |

Se a arquitetura não declara estratégia de testes, marcar
`<não definida na arquitetura — definir antes de implementar>`.

---

### 3. Cenários por módulo

Para cada módulo testável do plano:

### `<Módulo>`

| Entrada | Resultado esperado | Cobertura |
|---------|--------------------|-----------|
| <caso> | <...> | <...> |

- Casos de borda: <...>
- Casos de erro: <...>

---

### 4. Configuração

Arquivos de configuração de teste, com os valores que a arquitetura definir.

- Localização: `<paths>`
- Comando de execução: `<comando da Seção 2>`
- Cobertura mínima: `<valor — definir conforme política do time>`

---

### 5. Definition of Done (testes)

- [ ] <todos os módulos da Seção 1 testados>
- [ ] <comando da Seção 4 passa>
- [ ] <relatório de cobertura gerado>