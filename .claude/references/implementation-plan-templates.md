# Templates de Planos de Implementação

> **Documento de referência** — não é invocável. Lido por path pelo agente
> `agents/implementation-planner.md` (templates A, B, C, D).

# /implementation-planner

Gera planos de implementação detalhados a partir de um documento de arquitetura gerado pelo `/architecture-generator`, seguindo o padrão do `finance-app`.

## Input Obrigatório
- Caminho para `<projeto>-architecture.md` (ex.: `docs/architecture/finance-app-architecture.md`)
- Número do incremento alvo (ex.: `1`, `2`, ... `6`)

## Fluxo

### 1. Parse do Arquitetura
Ler o arquivo e extrair:
- **Seção 8 (Roadmap)** → checklist do incremento N
- **Seção 3 (Modelagem)** → entidades, enums, FKs, constraints
- **Seção 4 (Componentes)** → endpoints, serviços, páginas, hooks, stores do incremento
- **Seção 5 (Fluxos)** → fluxos que pertencem a este incremento
- **Seção 7 (Estrutura)** → tree de arquivos alvo (filtrar pelo incremento)
- **Seção 2 (Stack)** → decisões tecnológicas fixas
- **Seção 6 (Decisões)** → regras não-funcionais (D1-D12)
- **Seção 11 (Riscos)** → riscos relevantes ao incremento

### 2. Gerar 3-4 Arquivos em `docs/plans/`

#### A. `incremento-N-visao-geral.md`
~~~markdown
# Incremento N — <Título>: Plano de Implementação (Visão Geral)

> Alinhado à arquitetura em `docs/architecture/<projeto>-architecture.md` (tópico 8).
> Objetivo do incremento: <uma frase do roadmap>.

---

## 0. Decisões técnicas adotadas (pré-requisito do plano)
| Tema | Decisão | Por quê |
|------|---------|---------|
| <extrair de Seção 2 + 6> | <valor> | <justificativa> |

---

## 1. Divisão do plano
| Arquivo | Conteúdo |
|---------|----------|
| `incremento-N-visao-geral.md` (este) | Visão geral, decisões, sequência, DoD |
| `incremento-N-backend.md` | Plano detalhado backend |
| `incremento-N-frontend.md` | Plano detalhado frontend |
| `incremento-N-testes.md` | Plano de testes (opcional) |

---

## 2. Sequência global de execução
1. **Backend — fundação**: <passos da Seção 7 backend>
2. **Backend — dados**: models + migrações + schemas
3. **Backend — CRUD**: rotas + deps + CORS
4. **Backend — fumaça**: validar no Swagger
5. **Frontend — fundação**: setup + tipos + API client + React Query + Zustand + Layout + Router
6. **Frontend — CRUD**: páginas <listar da Seção 4>
7. **Integração**: conectar FE ↔ BE, validar E2E, servir estático em prod
8. **Documentação OpenAPI**: conferir endpoints no `/docs`

> **Por que backend primeiro?** O frontend depende dos contratos da API (schemas, rotas, formatos). Com o backend estável, o frontend consome contratos já validados no Swagger — menos idas e vindas.

---

## 3. Escopo do Incremento N (checklist resumido)
- [ ] <item do roadmap Seção 8>
- [ ] ...

---

## 4. O que fica explicitamente de fora (incrementos futuros)
- <itens do roadmap Seção 8 dos incrementos N+1...>
- <itens da Seção 12 "Futuras Evoluções" se relevante>

---

## 5. Definição de pronto (Incremento N completo quando…)
- [ ] <critérios mensuráveis: build passa, migrações rodam, Swagger documenta, FE CRUD funciona, etc.>

---

## 6. Próximos passos
1. Detalhar e executar **backend** (`incremento-N-backend.md`)
2. Detalhar e executar **frontend** (`incremento-N-frontend.md`)
3. Integração ponta a ponta
~~~

#### B. `incremento-N-backend.md`
~~~markdown
# Incremento N — Plano de Implementação: Backend

> Parte do plano do Incremento N. Visão geral em `incremento-N-visao-geral.md`.
> Objetivo: <descrição do objetivo backend do roadmap>.

---

## 1. Stack e decisões (herdadas da arquitetura)
- Runtime: <Seção 2>
- Gerenciador: <Seção 2>
- HTTP: <Seção 2>
- ORM: <Seção 2> + driver async
- Migrações: <Seção 2>
- Validação: <Seção 2>
- Monetário: **Decimal internamente, string na API** (Seção 6, D<X>)

---

## 2. Estrutura de arquivos a criar (filtrar Seção 7 pelo incremento)

```text
<projeto>/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── enums.py
│   │   │   ├── <entity1>.py
│   │   │   └── ...
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── common.py
│   │   │   └── <entity>.py (Create/Update/Response)
│   │   └── api/
│   │       ├── __init__.py
│   │       ├── deps.py
│   │       ├── responses.py
│   │       └── routes/
│   │           ├── __init__.py
│   │           └── <resource>.py
│   ├── alembic/
│   │   ├── env.py
│   │   └── versions/
│   │       └── 0001_initial.py
│   ├── alembic.ini
│   ├── pyproject.toml
│   └── .env.example
├── data/
└── .gitignore
```

---

## 3. Passo a passo (sequencial, acionável)

### Passo 1 — Setup do projeto backend
- Criar `backend/pyproject.toml` com deps: <listar de Seção 2>
- Opcional: `ruff` como dev dependency
- `uv sync` (executa por você)
- Criar `backend/.env.example` (DATABASE_URL, CORS_ORIGINS, SERVE_FRONTEND)

### Passo 2 — Configuração (`config.py`, `database.py`)
- `config.py`: BaseSettings lendo `.env`:
  - `database_url` (default relativo à raiz do projeto)
  - `cors_origins` (dev: localhost:5173/8080)
  - `serve_frontend` (bool)
  - `PROJECT_ROOT` resolvido em runtime (2 níveis acima)
  - `absolute_database_url`: property que resolve caminho relativo → absoluto
  - `env_file` aponta para `backend/.env` (absoluto em runtime)
- `database.py`:
  - `engine = create_async_engine(settings.absolute_database_url, echo=False, future=True)`
  - `AsyncSessionLocal = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False, autoflush=False)`
  - `Base(DeclarativeBase)`
  - `get_db` **apenas** em `app/api/deps.py`

### Passo 3 — Models SQLAlchemy (Seção 3 + 7)
Mapear conforme modelagem:
- **<Entity>**: <campos com tipos, constraints, enums como StrEnum>
- Usar `Mapped[]` + `mapped_column()` (estilo 2.0)
- Enums como `StrEnum` do Python (não `Enum` do SQLAlchemy)
- Relacionamentos com `lazy="selectin"`
- Default de `created_at` em app (`_utcnow()`), não `server_default`

### Passo 4 — Migrações Alembic
- `alembic.ini` + `alembic/env.py` configurado para **async**
  - Sobrescrever `sqlalchemy.url` com `settings.absolute_database_url` **antes** de `async_engine_from_config`
- Migrações:
  - `0001_initial.py` — cria todas as tabelas do incremento
  - `0002_<ajuste>.py` — se houver (ex.: created_at adicionado depois)
- Comando: `alembic upgrade head` (executa por você)

### Passo 5 — Pydantic Schemas (Create/Update/Response por entidade)
Para cada entidade:
- `<Entity>Create`, `<Entity>Update` (parcial), `<Entity>Response`
- **Monetário**: `Decimal` nos schemas + `field_serializer` → string `"150.90"` (2 casas)
- `date` como `date`; `created_at` como `datetime`
- Validações: `name` trim, `keyword` uppercase, `priority` 0-10000, `amount` range

### Passo 6 — Dependency de sessão (`api/deps.py`)

```python
async def get_db() -> AsyncIterator[AsyncSession]:
    async with AsyncSessionLocal() as session:
        yield session
```

### Passo 7 — Rotas CRUD (Seção 4.1 endpoints do incremento)
Para cada recurso (`/api/<resource>`):
- `GET /` (lista, paginação `limit`/`offset`, ordenação padrão)
- `POST /` (cria, 201, `response_model=<Entity>Response`)
- `PUT /{id}` (atualiza parcial via `exclude_unset=True`)
- `DELETE /{id}` (200 com body `{"detail": "...", "id": id}`)
- Tratar: 404 não encontrado, 422 FK inválida, validação de schema
- `response_model` em todos

### Passo 8 — `main.py` (FastAPI app)
- Título, versão, descrição
- **CORS** via `CORSMiddleware` com `settings.cors_origins`
- Incluir routers com prefixo `/api`
- **Produção**: montar `../frontend/dist` em `/` se `serve_frontend=True` e diretório existir
- Healthcheck: `GET /api/health` → `{"status": "ok"}`

### Passo 9 — `.gitignore` e `data/`
- Ignorar: `data/`, `.venv/`, `*.pyc`, `__pycache__/`, `.env`
- Criar diretório `data/` vazio

### Passo 10 — Validação (fumaça)
- Subir: `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`
- Acessar `/docs` e testar **cada endpoint CRUD** com dados de exemplo
- Confirmar: monetário como string; CRUD funciona; 404 em IDs inexistentes

---

## 4. Definição de pronto (backend)
- `uv sync` resolve deps; `alembic upgrade head` cria schema sem erros
- Models, schemas e rotas das <N> entidades implementadas
- Monetário persistido como `DECIMAL(12,2)` e trafegado como string
- Swagger (`/docs`) com todos os endpoints CRUD documentados e testáveis
- CORS liberando localhost; estático de produção montado (quando houver `dist/`)

---

## 5. Riscos/mitos específicos do backend
| Risco | Mitigação |
|-------|-----------|
| Alembic async mal configurado | Usar padrão `asyncio.run` no `env.py` oficial do SQLAlchemy async |
| Decimal serializado como float | Testar no Swagger desde o início; ajustar `field_serializer` |
| Caminho do SQLite relativo quebra | Definir `DATABASE_URL` absoluto via `.env` ou resolver em `config.py` |
~~~

#### C. `incremento-N-frontend.md`
~~~markdown
# Incremento N — Plano de Implementação: Frontend

> Parte do plano do Incremento N. Visão geral em `incremento-N-visao-geral.md`.
> Objetivo: <descrição do objetivo frontend do roadmap>.

---

## 1. Stack e decisões (herdadas)
- Scaffold: Vite `react-ts`
- Roteamento: React Router v6
- Server-state: TanStack Query v5
- UI-state: Zustand
- HTTP: **fetch wrapper nativo** (sem axios) — decisão do plano
- Monetário: string na API; exibir via `Intl.NumberFormat('pt-BR', {style:'currency', currency:'BRL'})`
- Build/Prod: `dist/` servido como estático pelo FastAPI (mesma origem)

---

## 2. Decisão pendente: cliente HTTP
| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| A) `fetch` wrapper (recomendado) | Zero deps; nativo; suficiente para CRUD | Sem interceptadores; tratamento manual de erros |
| B) `axios` | Interceptadores, cancelamento, erros padronizados | Dependência extra |

**Recomendação: Opção A**. Para CRUD simples, `fetch` basta e mantém bundle enxuto.

---

## 3. Estrutura de arquivos a criar (filtrar Seção 7 frontend pelo incremento)

```text
frontend/
├── public/
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── vite-env.d.ts
│   ├── pages/
│   │   ├── Dashboard.tsx (placeholder)
│   │   ├── <Resource1>.tsx
│   │   ├── <Resource2>.tsx
│   │   └── Placeholder.tsx (páginas futuras)
│   ├── components/
│   │   └── Layout.tsx (sidebar + <Outlet/>)
│   ├── hooks/
│   │   ├── use<Resource1>.ts
│   │   └── ...
│   ├── services/
│   │   └── api.ts (fetch wrapper + funções por recurso)
│   ├── stores/
│   │   └── uiStore.ts (Zustand: sidebarOpen, theme?, filtros?)
│   ├── types/
│   │   ├── <resource1>.ts
│   │   └── ...
│   └── styles/
│       └── globals.css
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── vitest.config.ts (se testes no mesmo incremento)
```

---

## 4. Passo a passo

### Passo 1 — Setup do projeto frontend
- `npm create vite@latest frontend -- --template react-ts` (executa por você)
- `npm install react-router-dom @tanstack/react-query zustand`
- `npm install` (executa por você)
- `vite.config.ts`: dev server porta 5173; **proxy opcional** ou URL absoluta via env

> **Proxy vs URL absoluta**: Em dev, duas opções — (a) Vite proxy encaminha `/api` pro backend (evita CORS); (b) URL absoluta `http://localhost:8000/api` com CORS. Recomendo **URL absoluta via env** (`VITE_API_URL`) para espelhar produção (mesma origem). Fica a seu critério.

### Passo 2 — Tipos TypeScript (`types/`)
Espelhar contratos da API (strings para monetário):
- `<Resource>`: `{ id: number; <campos>; created_at: string }`
- `<Resource>Input`: `{ <campos create/update> }`
- Criar também tipos de input quando diferirem do tipo base

### Passo 3 — API Client (`services/api.ts`)
- `BASE_URL = import.meta.env.VITE_API_URL ?? "http://localhost:8000/api"`
- `class ApiError extends Error` com `status`, `detail`
- `request<T>(method, path, body?)` genérico:
  - `Content-Type: application/json` + `JSON.stringify(body)`
  - Não-2xx: extrai `detail` do JSON (ou texto) e lança `ApiError`
  - 204 → retorna `undefined`
- Funções por recurso: `listX`, `createX`, `updateX`, `deleteX`

### Passo 4 — Stores Zustand (`stores/uiStore.ts`)
- Estado UI enxuto: `sidebarOpen`, `theme` (opcional), `filtrosAtivos`
- Ações: `toggleSidebar`, `setTheme`, etc.
- Começar mínimo; expandir nos incrementos

### Passo 5 — Hooks com React Query (`hooks/`)
Padrão idêntico por recurso:
- `useX()` → `useQuery({ queryKey: ['x'], queryFn: api.listX })`
- `useXMutations()` → `useMutation` para create/update/remove com `onSuccess: () => qc.invalidateQueries({ queryKey: ['x'] })`
- `update` recebe `{ id, payload }`; `remove` recebe `id`
- `QueryClientProvider` em `main.tsx` com `staleTime: 30000`, `refetchOnWindowFocus: false`, `retry: 1`

### Passo 6 — Layout e navegação (`components/Layout.tsx`, `App.tsx`)
- `Layout`: sidebar com links (habilitados: <páginas do incremento>; desabilitados: futuras) + `<Outlet />`
- Rotas React Router v6 aninhadas sob `<Layout/>`:
  - `/` → Dashboard (placeholder)
  - `/<resource1>` → `<Resource1>`
  - `/<resource2>` → `<Resource2>`
  - Rotas futuras → `Placeholder` (desabilitadas na sidebar)

### Passo 7 — Páginas CRUD
Padrão comum nas <N> páginas:
- `useQuery` para lista; estados `isLoading` ("Carregando…"), `isError` (banner), lista vazia (card "Crie o primeiro")
- Modal (`modal-backdrop` + `modal`, fecha no backdrop; `stopPropagation` no conteúdo) com form criar/editar
- Validação local antes do submit (nome obrigatório; em Transactions: descrição, conta, valor válidos)
- `handleSubmit` chama `create`/`update` mutation; erro de `ApiError` vira banner local; sucesso fecha modal e reseta form
- `handleDelete` com `confirm()` nativo; erro vira `alert()`
- **Transactions**: formatação valor com `Intl.NumberFormat`; positivos (verde/income), negativos (vermelho/expense)

### Passo 8 — Dashboard placeholder
- Cards simples (contagem de contas, categorias) ou placeholder
- Gráficos/saldos reais ficam para Incremento 2

### Passo 9 — Validação da integração
- Subir backend + frontend
- Em cada página: listar (vazio), criar, editar, excluir → conferir persistência no backend (recarrega no Swagger/DB)
- Confirmar: monetário como string; datas `YYYY-MM-DD`

---

## 5. Definição de pronto (frontend)
- Vite dev server sobe; `npm run build` gera `dist/`
- Tipos TS alinhados aos schemas da API
- API client com `fetch` wrapper e funções por recurso
- Layout + React Router com navegação entre páginas
- Páginas <Resources> com CRUD completo contra o backend
- Valores monetários formatados em BRL no frontend
- Dashboard como placeholder

---

## 6. Riscos/mitos específicos do frontend
| Risco | Mitigação |
|-------|-----------|
| Tipos dessincronizados da API | Revisar contratos no Swagger antes de codar os tipos |
| Decimal como float no JS | Nunca calcular com valor da API; tratar como string e só formatar para exibir |
| CORS em dev | Usar `VITE_API_URL` absoluto + CORS no backend, ou Vite proxy — escolher uma e documentar |
| React Query stale state | `invalidateQueries` após **toda** mutação de escrita |
~~~

#### D. `incremento-N-testes.md` (opcional, padrão finance-app)
~~~markdown
# Plano de Testes Unitários e de Integração — Frontend (Incremento N)

> Baseado em `docs/project-state/frontend.md` e `docs/plans/incremento-N-frontend.md`.

---

## 1. Escopo e decisão
**O que testar (unitário/integração leve):**
- `lib/format.ts` — funções puras de formatação monetária
- `services/api.ts` — wrapper `request<T>`, parsing de erro, serialização
- `hooks/use<Resource>.ts` — lógica de query/mutation + invalidação

**O que NÃO testar unitariamente (coberto por E2E/integração):**
- Tipos TypeScript (`types/*.ts`) — validação em tempo de compilação
- Store Zustand trivial (`stores/uiStore.ts`)
- Páginas completas — fluxo UI + API mockado → teste de integração com MSW (fora deste plano, sugerido para Inc. N+1)

**Por que não E2E headless agora**: superfície pequena, backend já tem smoke test, pirâmide invertida sem unit tests base.

---

## 2. Ferramentas
| Ferramenta | Versão | Papel |
|------------|--------|-------|
| Vitest | ^2.0 | Test runner nativo Vite, TS/ESM, watch, coverage (v8) |
| @testing-library/react | ^16.0 | Renderização de componentes para testes de integração |
| @testing-library/user-event | ^14.0 | Interações reais (digitação, clique, select) |
| @tanstack/react-query testing utils | ^5.51 | `createWrapper`, `renderHook` para hooks com QueryClient real |
| MSW (Mock Service Worker) | ^2.0 | **Opcional** — para testes de integração de páginas (mock da API) |
| jsdom | (via Vitest) | Ambiente DOM para @testing-library/react |

### Deps a adicionar (`frontend/package.json`)

```json
{
  "devDependencies": {
    "vitest": "^2.0.0",
    "@testing-library/react": "^16.0.0",
    "@testing-library/user-event": "^14.0.0",
    "@testing-library/jest-dom": "^6.4.0",
    "jsdom": "^24.0.0"
  }
}
```

---

## 3. Estrutura de pastas

```text
frontend/
├── src/
│   ├── lib/format.ts
│   ├── services/api.ts
│   ├── hooks/
│   │   ├── use<Resource1>.ts
│   │   └── ...
│   └── test/
│       ├── setup.ts
│       ├── utils/
│       │   └── query-client.tsx
│       ├── lib/
│       │   └── format.test.ts
│       ├── services/
│       │   └── api.test.ts
│       └── hooks/
│           ├── use<Resource1>.test.ts
│           └── ...
├── vitest.config.ts
└── package.json
```

---

## 4. O que testar em cada módulo

### 4.1 `lib/format.ts` — **Cobertura alvo: 100%**
| Função | Cenários |
|--------|----------|
| `formatBRL(value)` | `"150.90" → "R$ 150,90"`; `"-150.90" → "-R$ 150,90"`; `"0" → "R$ 0,00"`; `"1000.00" → "R$ 1.000,00"`; `null/undefined/"" → "—"`; `"abc" → "—"` (NaN) |
| `amountClass(value)` | `"150.90" → "income"`; `"0" → "income"`; `"-0.01" → "expense"`; `"-150.90" → "expense"` |

### 4.2 `services/api.ts` — **Cobertura alvo: 90%**
| Unidade | Cenários |
|---------|----------|
| `request<T>(method, path, body)` | GET sem body; POST/PUT com body (JSON.stringify); 200/201 retorna JSON parseado; 204 retorna `undefined`; 404/422/500 lança `ApiError` com `status`, `message` (de `detail`), `detail`; network error lança `ApiError`; header `Content-Type` correto |
| `ApiError` | Construtor popula `name`, `status`, `detail`; `instanceof Error` true |
| Funções por recurso (`listX`, `createX`, etc.) | Delegam para `request` com path/method corretos; serializam payload |

> **Mock**: `vi.hoisted(() => vi.fn())` + `global.fetch = mockFetch` ou `vi.stubGlobal('fetch', ...)`

### 4.3 `hooks/use<Resource>.ts` — **Cobertura alvo: 85%**
Padrão idêntico nos <N> recursos — testar um serve de template.
| Hook | Cenários |
|------|----------|
| `useX()` | Retorna `{ data, isLoading, isError, error }`; `data` vazio inicialmente; após `waitFor` resolve com array mockado |
| `useXMutations().create` | `mutateAsync(payload)` chama `api.createX(payload)`; em sucesso `invalidateQueries(['x'])` chamado; retorna dado criado |
| `useXMutations().update` | `mutateAsync({ id, payload })` chama `api.updateX(id, payload)`; invalida query; retorna dado atualizado |
| `useXMutations().remove` | `mutateAsync(id)` chama `api.deleteX(id)`; invalida query |
| Error handling | Se API lança `ApiError`, mutation `error` contém a instância; query **não** invalida |

> **Setup**: `renderHook` com `QueryClientProvider` wrapper (ver `test/utils/query-client.tsx`)

---

## 5. Setup e configuração

### `vitest.config.ts`

```ts
/// <reference types="vitest" />
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.test.ts', 'src/**/*.test.tsx'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['src/test/**', 'src/main.tsx', 'src/vite-env.d.ts'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 70,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: { '@': path.resolve(__dirname, './src') },
  },
});
```

### `src/test/setup.ts`

```ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

afterEach(() => {
  cleanup();
  vi.clearAllMocks();
});

// Mock global de fetch (pode ser sobrescrito por teste)
global.fetch = vi.fn();
```

### `src/test/utils/query-client.tsx`

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactNode } from 'react';

export function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
  });
}

export function renderWithQueryClient(
  ui: ReactNode,
  { queryClient = createTestQueryClient() } = {}
) {
  return (
    <QueryClientProvider client={queryClient}>
      {ui}
    </QueryClientProvider>
  );
}
```

---

## 6. Scripts no `package.json`

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

---

## 7. Cobertura mínima sugerida (gate de CI)
| Métrica | Threshold | Rationale |
|---------|-----------|-----------|
| **Lines** | 80% | Código não-UI (lib, services, hooks) é testável e crítico |
| **Functions** | 80% | Cada função exportada tem pelo menos 1 teste |
| **Branches** | 70% | Tratamento de erro (422/404/500, NaN, null) cria branches |
| **Statements** | 80% | Alinhado a lines |

> **Arquivos excluídos da cobertura**: `src/main.tsx`, `src/vite-env.d.ts`, `src/test/**`, páginas/components (fora do escopo unitário).

---

## 8. Exemplo de teste (template)

### `src/test/lib/format.test.ts`

```ts
import { describe, it, expect } from 'vitest';
import { formatBRL, amountClass } from '../../lib/format';

describe('lib/format', () => {
  describe('formatBRL', () => {
    it.each([
      ['150.90', 'R$ 150,90'],
      ['-150.90', '-R$ 150,90'],
      ['0', 'R$ 0,00'],
      ['1000.00', 'R$ 1.000,00'],
      [null, '—'],
      [undefined, '—'],
      ['', '—'],
      ['abc', '—'],
    ])('formatBRL(%s) → %s', (input, expected) => {
      expect(formatBRL(input as string | null)).toBe(expected);
    });
  });

  describe('amountClass', () => {
    it.each([
      ['150.90', 'income'],
      ['0', 'income'],
      ['-0.01', 'expense'],
      ['-150.90', 'expense'],
    ])('amountClass(%s) → %s', (input, expected) => {
      expect(amountClass(input)).toBe(expected);
    });
  });
}
```

### `src/test/services/api.test.ts`

```ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { request, ApiError, listCategories, createCategory } from '../../services/api';

const mockFetch = vi.fn();
global.fetch = mockFetch;

describe('services/api', () => {
  beforeEach(() => mockFetch.mockReset());

  describe('request', () => {
    it('GET 200 retorna JSON parseado', async () => {
      mockFetch.mockResolvedValueOnce({
        ok: true,
        status: 200,
        json: async () => [{ id: 1, name: 'Teste' }],
      });
      const data = await request<{ id: number; name: string }[]>('GET', '/categories');
      expect(data).toEqual([{ id: 1, name: 'Teste' }]);
      expect(mockFetch).toHaveBeenCalledWith('http://localhost:8000/api/categories', expect.objectContaining({ method: 'GET' }));
    });

    it('POST 201 com body serializa JSON', async () => {
      mockFetch.mockResolvedValueOnce({
        ok: true,
        status: 201,
        json: async () => ({ id: 2, name: 'Nova' }),
      });
      const data = await request('POST', '/categories', { name: 'Nova', type: 'expense' });
      expect(data).toEqual({ id: 2, name: 'Nova' });
      expect(mockFetch).toHaveBeenCalledWith(expect.any(String), expect.objectContaining({
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: 'Nova', type: 'expense' }),
      }));
    });

    it('422 lança ApiError com detail', async () => {
      mockFetch.mockResolvedValueOnce({
        ok: false,
        status: 422,
        json: async () => ({ detail: 'name já existe' }),
      });
      await expect(request('POST', '/categories', { name: 'Dup' })).rejects.toThrow(ApiError);
      try {
        await request('POST', '/categories', { name: 'Dup' });
      } catch (e) {
        expect(e).toBeInstanceOf(ApiError);
        expect((e as ApiError).status).toBe(422);
        expect((e as ApiError).message).toBe('name já existe');
      }
    });

    it('204 retorna undefined', async () => {
      mockFetch.mockResolvedValueOnce({ ok: true, status: 204 });
      const data = await request('DELETE', '/categories/1');
      expect(data).toBeUndefined();
    });
  });

  describe('funções por recurso', () => {
    it('listCategories chama GET /categories', async () => {
      mockFetch.mockResolvedValueOnce({ ok: true, status: 200, json: async () => [] });
      await listCategories();
      expect(mockFetch).toHaveBeenCalledWith('http://localhost:8000/api/categories', expect.objectContaining({ method: 'GET' }));
    });

    it('createCategory chama POST /categories', async () => {
      mockFetch.mockResolvedValueOnce({ ok: true, status: 201, json: async () => ({ id: 1 }) });
      await createCategory({ name: 'Cat', type: 'expense' });
      expect(mockFetch).toHaveBeenCalledWith('http://localhost:8000/api/categories', expect.objectContaining({ method: 'POST' }));
    });
  });
});
```

### `src/test/hooks/useCategories.test.ts`

```ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { useCategories, useCategoryMutations } from '../../hooks/useCategories';
import * as api from '../../services/api';
import { createTestQueryClient, renderWithQueryClient } from '../utils/query-client';

vi.mock('../../services/api');

describe('hooks/useCategories', () => {
  let queryClient: ReturnType<typeof createTestQueryClient>;

  beforeEach(() => {
    queryClient = createTestQueryClient();
    vi.resetAllMocks();
  });

  it('useCategories resolve data', async () => {
    vi.mocked(api.listCategories).mockResolvedValueOnce([{ id: 1, name: 'Cat', type: 'expense', monthly_limit: null, created_at: '2024-01-01T00:00:00Z' }]);
    const { result } = renderHook(() => useCategories(), { wrapper: (ui) => renderWithQueryClient(ui, { queryClient }) });
    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(result.current.data).toHaveLength(1);
  });

  it('create mutation invalida query', async () => {
    vi.mocked(api.createCategory).mockResolvedValueOnce({ id: 2, name: 'Nova', type: 'expense', monthly_limit: null, created_at: '2024-01-01T00:00:00Z' });
    const { result } = renderHook(() => useCategoryMutations(), { wrapper: (ui) => renderWithQueryClient(ui, { queryClient }) });
    await result.current.create.mutateAsync({ name: 'Nova', type: 'expense' });
    await waitFor(() => expect(api.createCategory).toHaveBeenCalled());
    // invalidateQueries é interno; verificar via queryClient.getQueryState(['categories'])?.fetchStatus === 'fetching'
  });

  it('update mutation invalida query', async () => {
    vi.mocked(api.updateCategory).mockResolvedValueOnce({ id: 1, name: 'Editada', type: 'expense', monthly_limit: null, created_at: '2024-01-01T00:00:00Z' });
    const { result } = renderHook(() => useCategoryMutations(), { wrapper: (ui) => renderWithQueryClient(ui, { queryClient }) });
    await result.current.update.mutateAsync({ id: 1, payload: { name: 'Editada' } });
    await waitFor(() => expect(api.updateCategory).toHaveBeenCalledWith(1, { name: 'Editada' }));
  });

  it('remove mutation invalida query', async () => {
    vi.mocked(api.deleteCategory).mockResolvedValueOnce({ detail: 'ok', id: 1 });
    const { result } = renderHook(() => useCategoryMutations(), { wrapper: (ui) => renderWithQueryClient(ui, { queryClient }) });
    await result.current.remove.mutateAsync(1);
    await waitFor(() => expect(api.deleteCategory).toHaveBeenCalledWith(1));
  });
});
```

---

## 9. Próximos passos (fora deste plano)
1. **Testes de integração de páginas** com MSW (`msw-handlers.ts` mockando `/api/*`) + `@testing-library/react` + `user-event` — cobre formulários, validação, modal, toast/alert.
2. **E2E headless (Playwright)** no Incremento N+1 — quando houver Dashboard real, gráficos, import OFX/OCR.
3. **Visual regression** (Chromatic/Playwright snapshot) para componentes de UI estáveis.
4. **Contract testing** (Pact) se frontend/backend evoluírem independentemente.

---

## 10. Decisão de adoção
Este plano **deve ser executado** antes ou junto do Incremento N+1, pois:
- Cria rede de segurança para refatoração dos hooks/services (base de features futuras)
- Estabelece cultura de teste no frontend (hoje inexistente)
- Baixo custo: ~50 testes cobrem 3 módulos críticos em < 10s

**Ação imediata sugerida**: instalar deps, criar `vitest.config.ts` + `setup.ts` + 3 arquivos de teste (format, api, um hook), rodar `npm run test:coverage` e confirmar thresholds. O restante (hooks Accounts/Transactions) segue o mesmo padrão.
~~~