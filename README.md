# 💰 FinAPI — Financial Management API

API REST para gerenciamento financeiro pessoal, construída com **Bun**, **Hono** e **Prisma**.

## Stack

| Tecnologia | Papel |
|------------|-------|
| [Bun](https://bun.sh) | Runtime + package manager |
| [Hono](https://hono.dev) | Framework HTTP ultrarrápido |
| [Prisma](https://prisma.io) | ORM + migrations |
| [PostgreSQL](https://postgresql.org) | Banco de dados |
| [Zod](https://zod.dev) | Validação de schemas |
| [bcryptjs](https://github.com/dcodeIO/bcrypt.js) | Hash de senhas |
| [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) | JWT access + refresh tokens |

---

## Instalação

```bash
# 1. Instalar dependências
bun install

# 2. Configurar variáveis de ambiente
cp .env.example .env
# Edite o .env com suas configurações

# 3. Criar e migrar o banco
bun run db:migrate

# 4. (Opcional) Popular com dados de teste
bun run db:seed

# 5. Iniciar em desenvolvimento
bun run dev

# Produção
bun run start
```

---

## Endpoints

### 🔐 Autenticação (`/api/v1/auth`)
| Método | Rota | Descrição |
|--------|------|-----------|
| POST | `/register` | Cadastro de usuário |
| POST | `/login` | Login (retorna tokens) |
| POST | `/refresh` | Renovar access token |
| POST | `/logout` | Logout (invalida refresh token) |
| GET | `/me` | Dados do usuário autenticado |
| PATCH | `/me` | Atualizar perfil |

### 🏦 Contas (`/api/v1/accounts`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Listar contas + saldo total |
| GET | `/:id` | Detalhes da conta |
| POST | `/` | Criar conta |
| PUT | `/:id` | Atualizar conta |
| DELETE | `/:id` | Desativar conta (soft delete) |
| GET | `/:id/statement` | Extrato com filtro de período |

**Tipos de conta:** `CHECKING`, `SAVINGS`, `CREDIT_CARD`, `INVESTMENT`, `CASH`, `OTHER`

### 💸 Transações (`/api/v1/transactions`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Listar com filtros e paginação |
| GET | `/:id` | Detalhes |
| POST | `/` | Criar transação |
| PUT | `/:id` | Atualizar |
| DELETE | `/:id` | Excluir (reverte saldo) |
| POST | `/bulk` | Importar múltiplas (até 100) |

**Filtros disponíveis:** `type`, `categoryId`, `accountId`, `from`, `to`, `search`, `tags`, `minAmount`, `maxAmount`, `page`, `limit`

### 🏷️ Categorias (`/api/v1/categories`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Listar (filtrar por `?type=`) |
| POST | `/` | Criar categoria |
| PUT | `/:id` | Atualizar |
| DELETE | `/:id` | Excluir (apenas sem transações) |

### 💱 Transferências (`/api/v1/transfers`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Histórico de transferências |
| POST | `/` | Transferir entre contas |

### 📊 Orçamentos (`/api/v1/budgets`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/?month=&year=` | Orçamentos do período |
| POST | `/` | Criar/atualizar orçamento |
| PUT | `/:id` | Atualizar valor |
| DELETE | `/:id` | Excluir |
| GET | `/alerts` | Orçamentos em alerta (≥80%) |

### 🎯 Metas (`/api/v1/goals`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Listar metas ativas |
| POST | `/` | Criar meta |
| PATCH | `/:id/deposit` | Adicionar valor à meta |
| PUT | `/:id` | Atualizar meta |
| DELETE | `/:id` | Excluir |

### 🔄 Recorrentes (`/api/v1/recurring`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Listar recorrentes |
| POST | `/` | Criar transação recorrente |
| POST | `/:id/process` | Processar manualmente |
| DELETE | `/:id` | Desativar |

**Frequências:** `DAILY`, `WEEKLY`, `BIWEEKLY`, `MONTHLY`, `QUARTERLY`, `YEARLY`

### 📈 Dashboard (`/api/v1/dashboard`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Resumo completo: saldo, receitas, despesas, alertas, metas, vencimentos |

### 📉 Relatórios (`/api/v1/reports`)
| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/monthly?year=` | Breakdown mensal do ano |
| GET | `/by-category?from=&to=&type=` | Gastos por categoria |
| GET | `/cash-flow?months=6` | Fluxo de caixa histórico |
| GET | `/net-worth` | Patrimônio líquido |

---

## Autenticação

Todas as rotas (exceto `/auth/register` e `/auth/login`) exigem o header:

```
Authorization: Bearer <access_token>
```

### Fluxo de tokens
1. `/login` retorna `accessToken` (15min) + `refreshToken` (7 dias)
2. Quando o access token expirar, chame `/auth/refresh` com o refresh token
3. O refresh token é rotacionado a cada uso (invalidado e um novo emitido)

---

## Boas práticas implementadas

- ✅ **Transações ACID** — Saldo de contas atualizado atomicamente via `db.$transaction`
- ✅ **Soft delete** — Contas desativadas, não excluídas
- ✅ **Validação Zod** — Todos os inputs validados com tipos estritos
- ✅ **Rate limiting** — 100 req/min por IP
- ✅ **CORS configurável** — Via variável de ambiente
- ✅ **Secure headers** — Via `hono/secure-headers`
- ✅ **Refresh token rotation** — Revoga o token antigo a cada uso
- ✅ **Paginação** — Todas as listagens com `page` e `limit`
- ✅ **Variáveis de ambiente validadas** — App não sobe com configuração inválida
- ✅ **Prisma singleton** — Evita conexões duplicadas em desenvolvimento
- ✅ **Índices no banco** — `(userId, date)` e `(userId, categoryId)` para performance

---

## Estrutura do projeto

```
src/
├── index.ts              # Entry point e configuração do app
├── routes/
│   ├── auth.ts           # Autenticação e perfil
│   ├── accounts.ts       # Contas bancárias
│   ├── transactions.ts   # Transações
│   ├── categories.ts     # Categorias
│   ├── budgets.ts        # Orçamentos mensais
│   ├── goals.ts          # Metas financeiras
│   ├── transfers.ts      # Transferências entre contas
│   ├── recurring.ts      # Transações recorrentes
│   ├── dashboard.ts      # Dashboard unificado
│   └── reports.ts        # Relatórios e analytics
├── middleware/
│   ├── auth.ts           # Verificação de JWT
│   ├── error-handler.ts  # Handler global de erros
│   └── rate-limiter.ts   # Rate limiting por IP
└── utils/
    ├── db.ts             # Prisma client singleton
    ├── env.ts            # Validação de variáveis de ambiente
    ├── jwt.ts            # Utilitários de JWT
    └── pagination.ts     # Helpers de paginação
```
