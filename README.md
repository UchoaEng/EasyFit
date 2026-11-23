<div align="center">
  <img src="./public/icon.svg" width="88" height="88" alt="EasyFit icon" />

  # EasyFit

  ### Nutrition, body progress and training in one place.

  A mobile-first PWA that makes health tracking simple, fast and honest: it never invents nutrients, never hides incomplete data and never depends on paid APIs.

  [![EasyFit CI](https://github.com/UchoaEng/EasyFit/actions/workflows/ci.yml/badge.svg)](https://github.com/UchoaEng/EasyFit/actions/workflows/ci.yml)
  [![Next.js](https://img.shields.io/badge/Next.js-16-111111?logo=nextdotjs)](https://nextjs.org/)
  [![React](https://img.shields.io/badge/React-19-149ECA?logo=react&logoColor=white)](https://react.dev/)
  [![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
  [![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
  [![Prisma](https://img.shields.io/badge/Prisma-6-2D3748?logo=prisma)](https://www.prisma.io/)
  [![PWA](https://img.shields.io/badge/PWA-installable-166534?logo=pwa&logoColor=white)](https://web.dev/explore/progressive-web-apps)

  **[Open the app](https://easy-fit-phi.vercel.app)** · [Run locally](#running-locally) · [Report an issue](https://github.com/UchoaEng/EasyFit/issues) · [🇧🇷 Português](#-português)

  <sub>Built for phones, flaky connections and real routines.</sub>
</div>

---

## About

EasyFit brings together a food diary, nutrition goals, body measurements and workout planning in an experience designed for the phone first.

It is a full-stack app: the frontend and the API routes live in the same Next.js project, Prisma handles the data layer and PostgreSQL runs either locally with Docker or remotely on Supabase.

| 📱 Mobile-first | 🥗 No-guess nutrition | 🏋️ Adaptive training | 🛜 Resilient |
| :---: | :---: | :---: | :---: |
| One-handed interface | Incomplete data stays visible | Plans reviewable by split and goal | PWA, cache and offline queue |

### What works today

| Area | Features |
| --- | --- |
| **Diet** | Planned and consumed entries, custom meals, corrections with history, meal copying and macro summary. Imported diet plans can be edited item by item, including swapping a food for a catalog match. |
| **Foods** | Manual search, favorites, recents, private catalog, TACO (Brazilian table), USDA FoodData Central and free Open Food Facts lookup. USDA food names are shown translated to Portuguese. |
| **Barcode** | On-device camera scanning, GTIN lookup and manual entry when the product doesn't exist. |
| **Workouts** | Reviewable Full body, AB, ABC, ABCD and ABCDE suggestions for strength or hypertrophy; manual building, search, JSON import, sessions, sets and substitutions. Exercises can be moved between workouts, and editing is optimized for mobile. |
| **Live session** | Estimated total and remaining workout time (sets plus rest) and estimated calories based on duration, body weight and volume. |
| **Progress** | Nutrition reports, workout adherence, volume per exercise and body measurement trends. |
| **Account** | Sessions, data export and deletion, consents, reminders and personal preferences. |
| **Experience** | Installable PWA, offline queue, recoverable conflicts, skeleton loading, responsive navigation and bottom sheets on mobile. |

## Tech stack

- **Next.js 16** with App Router, Server Components and Route Handlers
- **React 19** and **TypeScript 5**
- **Tailwind CSS 4** for the UI
- **PostgreSQL 17** and **Prisma 6**
- **ZXing** for on-device barcode scanning
- **Open Food Facts**, **TACO** and **USDA FoodData Central** as open nutrition sources
- **Vitest**, **Playwright** and **axe-core** for quality and accessibility
- **Vercel** for the app and APIs; **Supabase** as managed PostgreSQL

> [!TIP]
> No paid API is required. Barcode reading happens on the device, and lookups use open sources.

## Architecture

```mermaid
flowchart LR
    U["PWA on phone or browser"] --> N["Next.js App Router"]
    N --> A["Route Handlers / API"]
    A --> P["Prisma ORM"]
    P --> D[("PostgreSQL / Supabase")]
    A --> O["Open Food Facts"]
    A --> T["TACO / USDA FoodData Central"]
    N --> Q["Offline queue on device"]
    Q --> A
```

```text
app/          pages, layouts and HTTP endpoints
components/   UI components and interactive flows
lib/          domain, auth, integrations and data access
prisma/       schema and versioned migrations
public/       manifest, service worker and PWA icon
tests/        unit, integration and E2E tests
```

## Running locally

### Prerequisites

- [Node.js 24.x](https://nodejs.org/) and npm 11+
- PostgreSQL 17, local or remote (Docker is optional for a local database)

### 1. Clone and install

```bash
git clone https://github.com/UchoaEng/EasyFit.git
cd EasyFit
npm install
```

### 2. Set up the database

Use Supabase or a local PostgreSQL. For a disposable database with Docker:

```bash
docker run --name easyfit-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=easyfit -p 5432:5432 -d postgres:17-alpine
```

The security migrations also lock down Supabase's API roles. On a fresh local PostgreSQL, create these two compatible roles once:

```bash
docker exec easyfit-postgres psql -U postgres -d easyfit -c "CREATE ROLE anon NOLOGIN;"
docker exec easyfit-postgres psql -U postgres -d easyfit -c "CREATE ROLE authenticated NOLOGIN;"
```

Without Docker, run the same SQL as a database admin before the migrations.

### 3. Create the environment file

```bash
cp .env.example .env        # PowerShell: Copy-Item .env.example .env
```

For a local PostgreSQL, adjust the top of `.env`:

```dotenv
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/easyfit?schema=public"
DIRECT_URL="postgresql://postgres:postgres@localhost:5432/easyfit?schema=public"
SESSION_SECRET="replace-with-a-random-key-of-at-least-32-characters"
APP_URL="http://localhost:3000"
OPEN_FOOD_FACTS_CONTACT="http://localhost:3000"
```

> [!IMPORTANT]
> Use a single pair of quotes per value. An extra `"` at the end of `DATABASE_URL` makes Prisma misread `schema=public`.

Generate a secure secret with `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`.

### 4. Prepare Prisma

```bash
npm run prisma:generate
npm run prisma:deploy
```

`prisma:deploy` applies the migrations already in the project. For a new schema change, use `npm run prisma:migrate -- --name change_name`.

### 5. Start the app

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## Environment variables

[`.env.example`](./.env.example) has the full list with safe sample values.

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | App connection; on Supabase/Vercel use the Transaction Pooler. |
| `DIRECT_URL` | Direct connection for migrations; on Supabase use the Session Pooler. |
| `SESSION_SECRET` | Signs sessions; at least 32 random characters. |
| `SESSION_TTL_DAYS` / `SESSION_ROTATION_HOURS` | Absolute session lifetime and token rotation interval. |
| `APP_URL` / `ALLOWED_ORIGINS` | Canonical origin and extra allowed origins (comma-separated) for security checks. |
| `OPEN_FOOD_FACTS_CONTACT` / `OPEN_FOOD_FACTS_*` | Contact sent to the open database; timeout, retries, circuit breaker and limits. |
| `USDA_FDC_API_KEY` / `USDA_FDC_*` | Free FoodData Central key (`DEMO_KEY` works for testing with a lower quota); timeout, retries and limits. |

> [!CAUTION]
> Never commit `.env`, passwords, keys or real database URLs. `.env` is already in `.gitignore`.

## Supabase + Vercel

Supabase is used only as managed PostgreSQL. Auth, sessions and business rules stay in EasyFit's backend.

1. Create a Supabase project.
2. In **Connect → ORM → Prisma**, copy the Transaction Pooler (port `6543`) to `DATABASE_URL` and the Session Pooler (port `5432`) to `DIRECT_URL`.
3. Fill in `.env` locally and run `npm run prisma:deploy` once.
4. Import this repository on Vercel with `./` as the root directory.
5. Add the `.env.example` variables to Vercel's **Production** environment, with `APP_URL` set to the final HTTPS URL (no trailing slash).
6. Deploy. [`vercel.json`](./vercel.json) already pins functions to São Paulo (`gru1`).

> [!NOTE]
> Frontend and backend deploy together. Code-only changes just need a new deploy; the database only needs `prisma:deploy` when there's a new migration. Use a separate database for Preview environments.

## Useful commands

| Command | Description |
| --- | --- |
| `npm run dev` | Development server. |
| `npm run build` / `npm start` | Production build and run. |
| `npm run lint` / `npm run typecheck` | Lint and type checks. |
| `npm test` | Unit tests. |
| `npm run test:integration` | Integration suite (needs `RUN_INTEGRATION_TESTS=1`). |
| `npm run test:e2e` | Playwright E2E flows, including mobile layout. |
| `npm run prisma:generate` / `prisma:deploy` / `prisma:migrate` | Prisma client, apply migrations, create a migration. |

Run integration and E2E tests only against a disposable PostgreSQL. CI provisions PostgreSQL, applies migrations and runs lint, types, unit, integration, mobile E2E, axe audit and build.

## Security and privacy

- Passwords hashed with **Argon2id**; session tokens stored only as hashes and rotated periodically.
- CSP, HSTS, anti-framing and a restrictive permissions policy.
- Origin validation for mutations, internal roles, support scopes and an audit trail.
- Data export and deletion requests for the account owner.
- A lockdown migration that blocks Supabase's public roles from Prisma's tables.

## Troubleshooting

<details>
<summary><strong>Prisma P1001: Can't reach database server</strong></summary>

Check `docker ps`, make sure the URL uses `localhost:5432` (not a literal `HOST`), and confirm user, password, database and port. On Supabase, check the pooler host and that the project is active.

</details>

<details>
<summary><strong>Quoted identifier error near "public"</strong></summary>

Remove duplicated quotes at the end of the URL. It should end with `...?schema=public"`.

</details>

<details>
<summary><strong>Sign-up returns 403 on Vercel</strong></summary>

Set `APP_URL` to the exact public origin (e.g. `https://easy-fit-phi.vercel.app`, no trailing slash), check `ALLOWED_ORIGINS` for other domains, and redeploy after changing variables.

</details>

<details>
<summary><strong>Product not found by barcode</strong></summary>

Open Food Facts is collaborative and may not have every Brazilian product. Search by name, use the national catalog or enter the label values manually.

</details>

---

## 🇧🇷 Português

### Alimentação, evolução corporal e treino em um só lugar.

PWA mobile-first para tornar o acompanhamento de saúde simples, rápido e honesto: sem inventar nutrientes, sem esconder dados incompletos e sem depender de APIs pagas. **[Abrir aplicação](https://easy-fit-phi.vercel.app)**

### Sobre o projeto

O EasyFit reúne diário alimentar, metas nutricionais, medidas corporais e planejamento de treinos em uma experiência pensada primeiro para o celular. É full-stack: frontend e rotas de API ficam no mesmo projeto Next.js, o Prisma cuida dos dados e o PostgreSQL roda localmente via Docker ou no Supabase.

| Área | Recursos |
| --- | --- |
| **Dieta** | Registros planejados e consumidos, refeições personalizadas, correção com histórico, cópia de refeições e resumo de macros. Planos de dieta importados podem ser editados item a item, inclusive trocando o alimento por um do catálogo. |
| **Alimentos** | Pesquisa manual, favoritos, recentes, catálogo privado, TACO, USDA FoodData Central e Open Food Facts gratuito. Os nomes de alimentos da USDA aparecem traduzidos para o português. |
| **Código de barras** | Leitura pela câmera no próprio aparelho, busca por GTIN e cadastro manual quando o produto não existe. |
| **Treinos** | Sugestões revisáveis Full body, AB, ABC, ABCD e ABCDE para força ou hipertrofia; montagem manual, pesquisa, importação JSON, sessões, séries e substituições. Exercícios podem ser movidos entre treinos, e a edição foi otimizada para o celular. |
| **Sessão ao vivo** | Estimativa do tempo total e restante do treino (séries mais descanso) e estimativa de calorias com base em duração, peso corporal e volume. |
| **Evolução** | Relatórios nutricionais, aderência aos treinos, volume por exercício e evolução das medidas. |
| **Conta** | Sessões, exportação e exclusão de dados, consentimentos, lembretes e preferências. |
| **Experiência** | PWA instalável, fila offline, conflitos recuperáveis, skeleton loading, navegação responsiva e bottom sheets no mobile. |

### Tecnologias

Next.js 16 (App Router, Server Components, Route Handlers), React 19, TypeScript 5, Tailwind CSS 4, PostgreSQL 17, Prisma 6, ZXing, Open Food Facts, TACO, USDA FoodData Central, Vitest, Playwright, axe-core, Vercel e Supabase. Nenhuma API paga é obrigatória.

### Rodando localmente

Pré-requisitos: Node.js 24.x, npm 11+ e PostgreSQL 17 (Docker opcional).

```bash
git clone https://github.com/UchoaEng/EasyFit.git
cd EasyFit
npm install
docker run --name easyfit-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=easyfit -p 5432:5432 -d postgres:17-alpine
docker exec easyfit-postgres psql -U postgres -d easyfit -c "CREATE ROLE anon NOLOGIN;"
docker exec easyfit-postgres psql -U postgres -d easyfit -c "CREATE ROLE authenticated NOLOGIN;"
cp .env.example .env        # PowerShell: Copy-Item .env.example .env
npm run prisma:generate
npm run prisma:deploy
npm run dev
```

As roles `anon` e `authenticated` existem porque as migrations de segurança também protegem as roles da API do Supabase. No `.env`, use o bloco de exemplo da versão em inglês acima para o PostgreSQL local, com **apenas um par de aspas** por valor (um `"` extra no fim da `DATABASE_URL` quebra o `schema=public`). Gere o `SESSION_SECRET` com `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`. Acesse [http://localhost:3000](http://localhost:3000).

### Variáveis de ambiente

A lista completa está no [`.env.example`](./.env.example). As principais: `DATABASE_URL` (Transaction Pooler no Supabase), `DIRECT_URL` (Session Pooler, usada nas migrations), `SESSION_SECRET` (mínimo de 32 caracteres), `APP_URL` e `ALLOWED_ORIGINS` (validação de origem), `OPEN_FOOD_FACTS_*` e `USDA_FDC_*` (integrações; `DEMO_KEY` serve para testes). Nunca envie `.env`, senhas ou URLs reais de banco para o Git.

### Supabase + Vercel

O Supabase é usado só como PostgreSQL gerenciado; autenticação e regras ficam no backend do EasyFit.

1. Crie um projeto no Supabase e, em **Connect → ORM → Prisma**, copie o Transaction Pooler (porta `6543`) para `DATABASE_URL` e o Session Pooler (porta `5432`) para `DIRECT_URL`.
2. Rode `npm run prisma:deploy` uma vez com o `.env` preenchido.
3. Importe o repositório na Vercel (raiz `./`), cadastre as variáveis no ambiente **Production** e defina `APP_URL` com a URL HTTPS final, sem barra no fim.
4. Faça o deploy. O [`vercel.json`](./vercel.json) já direciona as funções para São Paulo (`gru1`).

Mudanças só de código precisam apenas de um novo deploy; o banco só precisa de `prisma:deploy` quando houver migration nova. Use outro banco para os ambientes de Preview.

### Comandos úteis

`npm run dev` (desenvolvimento), `npm run build` / `npm start` (produção), `npm run lint` / `npm run typecheck`, `npm test` (unitários), `npm run test:integration` (com `RUN_INTEGRATION_TESTS=1`), `npm run test:e2e` (Playwright, incluindo layout mobile) e `npm run prisma:generate` / `prisma:deploy` / `prisma:migrate`. Rode integração e E2E só em um PostgreSQL descartável.

### Segurança e privacidade

Senhas com **Argon2id**, tokens de sessão guardados só como hash e rotacionados, CSP, HSTS, proteção contra framing, validação de origem, papéis internos com trilha de auditoria, exportação e exclusão de dados do titular e uma migration de lockdown que bloqueia as roles públicas do Supabase.

### Solução de problemas

- **Prisma P1001:** confira `docker ps`, se a URL usa `localhost:5432`, e usuário, senha, banco e porta. No Supabase, confira o host do pooler.
- **Erro de identificador perto de `public`:** remova aspas duplicadas no fim da URL.
- **Cadastro retorna 403 na Vercel:** defina `APP_URL` exatamente com a origem pública, sem barra no fim, confira `ALLOWED_ORIGINS` e faça um novo deploy.
- **Produto não encontrado pelo código de barras:** o Open Food Facts é colaborativo; pesquise pelo nome, use o catálogo nacional ou cadastre os valores do rótulo.

---

<div align="center">
  Built for a real routine: phone in hand, imperfect internet and no-guess information.<br>
  <sub>Feito para uma rotina real: celular na mão, internet imperfeita e informação sem chute.</sub>
</div>
