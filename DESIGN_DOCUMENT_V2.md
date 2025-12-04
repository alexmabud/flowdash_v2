📘 FlowDash v2 — Blueprint & Technical Specification

Contexto: Migração de Monolito Python (Streamlit/SQLite) para Arquitetura Enterprise (Web/API).
Stack: Next.js 14 (Frontend) + FastAPI (Backend) + PostgreSQL (Database).
Leitor Alvo: Desenvolvedores Humanos & Agentes de IA (Gemini, GPT, Claude).
Status: Aprovado para Execução.

🤖 Meta-Instruções para Agentes de IA (LEIA ANTES DE CODAR)

Se você é um Agente de IA lendo este documento para implementar código, siga estas diretrizes rigorosamente:

Respeite a Camada de Serviço (Service Layer):

NUNCA coloque regra de negócio (cálculo de juros, datas de liquidação, validação de saldo) nas Rotas (api/) ou nos Modelos (models/).

Toda lógica deve residir em backend/app/services/.

Exemplo: O cálculo de "D+1 útil" para vendas a crédito pertence a SalesService, não ao endpoint da API.

Integridade do Ledger (Sagrado):

O sistema usa o princípio de Dupla Entrada (Double Entry).

Ao mexer em services/ledger, lembre-se: uma transferência deve sempre gerar um Débito e um Crédito atômicos.

A tabela payable_events é Append-Only. Nunca edite o saldo de uma dívida diretamente; insira um evento de pagamento ou ajuste.

Idempotência é Obrigatória:

O Frontend (PDV) pode falhar ou enviar dados duplicados.

Sempre verifique se a entidade possui um uid (hash) antes de inserir. Use a lógica herdada de shared/ids.py.

Consistência de Nomes:

Código (Variáveis, Funções, Tabelas): Inglês.

Comentários explicativos e Documentação: Português.

Exemplo: Tabela bank_accounts, mas comentário # Saldo consolidado do banco.

1. Sumário Executivo

O FlowDash v2 visa resolver os gargalos de performance e concorrência da versão Streamlit (V1), introduzindo um banco de dados relacional robusto e uma API stateless. Isso permitirá que múltiplos vendedores (PDV) e administradores (Dashboard) operem simultaneamente sem conflitos de travamento de arquivo (SQLite locking), mantendo a lógica financeira complexa já existente.

2. Arquitetura da Solução

Adotamos o padrão Modular Monolith com separação Client-Server.

2.1. Backend (Server-Side)

Framework: FastAPI (Python 3.11+).

Responsabilidade: Fonte da verdade, cálculos financeiros, persistência e autenticação.

ORM: SQLAlchemy 2.0 (Async) - Mapeamento Objeto-Relacional.

Migrations: Alembic - Controle de versão do esquema do banco.

2.2. Frontend (Client-Side)

Framework: Next.js 14 (App Router).

Responsabilidade: UX, PWA (Offline capabilities iniciais), Cache de visualização.

Estilização: Tailwind CSS + ShadcnUI.

State Management: React Query (TanStack Query) para dados do servidor; Zustand para estado local (carrinho do PDV).

2.3. Banco de Dados

Engine: PostgreSQL 15+.

Hospedagem: Railway, Render ou Neon (Serverless).

Benefício: Suporte a transações concorrentes, tipos decimais precisos e backups automáticos.

3. Modelagem de Dados (Schema Core)

As tabelas do V1 foram normalizadas para a nova arquitetura.

3.1. Acesso e Identidade

users

id: PK (Integer)

email: Unique Index.

password_hash: String.

role: Enum ('admin', 'gerente', 'vendedor').

pin: String(4) (Nullable) - Para login rápido no PDV (herdado de pin_utils.py).

active: Boolean.

3.2. Financeiro (Core)

bank_accounts

Substitui as colunas soltas de saldo.

id: PK

name: 'Inter', 'Caixa', 'Caixa 2' (Físico), 'Bradesco'.

is_cash: Boolean (Define se é dinheiro em espécie).

transactions (A Tabela Mestra)

Unifica movimentacoes_bancarias, entrada (vendas), saida e correcao_caixa.

id: PK

uid: Hash Idempotente (Unique Index) - Garante que não haja duplicidade.

type: Enum ('credit', 'debit').

amount: Decimal(12,2).

date_competence: Data do fato gerador (Venda).

date_liquidation: Data da disponibilidade financeira (D+N).

account_id: FK -> bank_accounts.

category_id: FK -> categories.

user_id: FK -> users (Quem fez o lançamento).

3.3. Ledger de Dívidas (Contas a Pagar)

Reflete a lógica de repository/contas_a_pagar_mov_repository.

payables (A Obrigação Pai)

Representa o contrato de empréstimo, a fatura do mês ou o boleto.

original_amount, due_date, creditor, status (Calculado).

payable_events (O Histórico)

Registra cada alteração na dívida.

event_type: 'LANCAMENTO', 'JUROS', 'MULTA', 'DESCONTO', 'PAGAMENTO'.

transaction_id (FK): Se for pagamento, linka com a saída real do dinheiro (transactions).

3.4. Auxiliares

categories: Árvore de categorias financeiras.

machine_rates: Configuração de taxas (Maquineta + Bandeira + Parcelas -> Taxa %).

4. Módulos e Serviços (Backend)

Aqui reside a inteligência do sistema. Cada serviço deve mapear uma lógica do V1.

Serviço

Responsabilidade

Origem V1 (Referência)

AuthService

Login (Senha/PIN), Geração e validação de JWT.

auth/auth.py, utils/pin_utils.py

SalesService

Registrar venda, aplicar taxas de maquineta, calcular data de liquidação (Workalendar).

services/vendas.py, flowdash_pages/lancamentos/actions_venda.py

FinanceService

Registrar saídas simples, transferências entre contas (atomicidade), correções.

actions_saida.py, actions_transferencia.py, actions_deposito.py

LedgerService

O coração contábil. Baixa de faturas, amortização de empréstimos, cálculo de saldo devedor.

services/ledger/*.py (Todos os mixins)

ReportService

Agregação de dados para DRE e Dashboard (Queries SQL otimizadas).

dre/dre.py, dashboard/dashboard.py

LockService

Guardião. Impede alterações em datas já fechadas.

fechamento/lock_manager.py

5. Integração Frontend (Next.js)

5.1. Estrutura de Rotas (App Router)

/login: Autenticação unificada.

/app (Layout Protegido):

/dashboard: Visão gerencial (Gráficos).

/pdv: Visão de venda rápida (Mobile-first, botões grandes).

/lancamentos: Tabela completa (Data Grid com filtros).

/financeiro/contas-pagar: Gestão de passivos e pagamentos.

/configuracoes: Cadastros (Taxas, Categorias, Usuários).

5.2. Tecnologias

UI: Tailwind CSS (Estilização rápida), ShadcnUI (Componentes acessíveis), Lucide React (Ícones).

Charts: Recharts (Substituindo Plotly para melhor performance em React).

Forms: React Hook Form + Zod (Validação).

6. Estratégia de Migração (ETL)

Para levar os dados do V1 para o V2, criaremos o script scripts/migrate_v1_to_v2.py:

Ler o SQLite flowdash_data.db usando Pandas.

Transformação:

Converter strings de data ('YYYY-MM-DD') para objetos Python date.

Normalizar nomes de bancos (ex: "Inter" e "Banco Inter" viram o mesmo ID).

Recalcular hashes de UID se necessário.

Carga: Inserir no PostgreSQL via SQLAlchemy Models, respeitando a ordem: Users -> Accounts -> Categories -> Payables -> Transactions.

7. Roadmap de Implementação

Setup & Infra: Criar estrutura de pastas, Docker, configurar DB. (✅ Feito)

Backend Core: Implementar Models (SQLAlchemy) e Migrations (Alembic).

Migration ETL: Portar os dados reais do SQLite para o Postgres para ter base de teste.

Services API: Implementar AuthService, SalesService e LedgerService.

Frontend MVP: Telas de Login e Lançamento de Vendas (PDV).

Frontend Admin: Dashboard, DRE e Telas de Configuração.

Validação Final: Comparar saldo final do V1 com V2.

8. 🗺️ Mapa do Território (Árvore de Arquivos V2)

Esta é a estrutura canônica que o projeto terá. Use-a como guia para criar novos arquivos.

flowdash-v2/
├── .env                        # Variáveis de ambiente (DB URL, Secret Key)
├── docker-compose.yml          # Orquestração (App + DB local)
├── requirements.txt            # Dependências Python
├── README.md                   # Documentação geral
├── DESIGN_DOCUMENT_V2.md       # ESTE ARQUIVO (A Bíblia do Projeto)
│
├── backend/                    # --- CÉREBRO DO SISTEMA (FastAPI) ---
│   ├── alembic/                # Scripts de Migração de Banco de Dados
│   ├── alembic.ini             # Config do Alembic
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py             # Entrypoint da API (FastAPI App)
│   │   │
│   │   ├── api/                # Camada de Apresentação (Rotas/Endpoints)
│   │   │   └── v1/
│   │   │       ├── api.py      # Router aggregator
│   │   │       └── endpoints/
│   │   │           ├── auth.py       # POST /login, /refresh
│   │   │           ├── sales.py      # POST /sales (Vendas)
│   │   │           ├── finance.py    # POST /transfer, POST /expense
│   │   │           ├── ledger.py     # GET /payables, POST /pay
│   │   │           └── reports.py    # GET /dre, GET /dashboard-stats
│   │   │
│   │   ├── core/               # Configurações Centrais
│   │   │   ├── config.py       # Settings (Pydantic, env vars)
│   │   │   └── security.py     # Hash de senha, JWT generator, PIN utils
│   │   │
│   │   ├── db/                 # Acesso a Dados
│   │   │   ├── session.py      # Engine SQLAlchemy Async
│   │   │   └── base.py         # Import central de todos os models
│   │   │
│   │   ├── models/             # --- DEFINIÇÃO DAS TABELAS (ORM) ---
│   │   │   ├── user.py         # Users, Roles
│   │   │   ├── finance.py      # Transactions, BankAccounts
│   │   │   ├── ledger.py       # Payables, PayableEvents (Complexo!)
│   │   │   └── auxiliary.py    # Categories, MachineRates
│   │   │
│   │   ├── schemas/            # --- DTOs (Pydantic) ---
│   │   │   ├── user.py         # UserCreate, UserResponse
│   │   │   ├── transaction.py  # TransactionCreate, TransactionResponse
│   │   │   └── payable.py      # PayableSchema
│   │   │
│   │   └── services/           # --- LÓGICA DE NEGÓCIO (Antigo V1 Logic) ---
│   │       ├── auth_service.py # Lógica de autenticação
│   │       ├── sales_service.py# Lógica de Vendas (Taxas, Datas)
│   │       ├── report_service.py # Lógica de DRE e Dashboard
│   │       └── ledger/         # O Coração Financeiro (Portado de services/ledger V1)
│   │           ├── orchestrator.py # Coordena transações
│   │           ├── payments.py     # Regras de baixa e status
│   │           └── amortization.py # Regras de empréstimo
│   │
│   └── scripts/                # Scripts utilitários
│       └── migrate_v1.py       # Script ETL (SQLite -> Postgres)
│
└── frontend/                   # --- INTERFACE DO USUÁRIO (Next.js) ---
    ├── public/                 # Assets estáticos (Icons, Logos)
    ├── src/
    │   ├── app/                # App Router (Páginas)
    │   │   ├── layout.tsx      # Root Layout (Providers)
    │   │   ├── page.tsx        # Redirect / Landing
    │   │   │
    │   │   ├── (auth)/         # Grupo de rotas de autenticação
    │   │   │   └── login/page.tsx
    │   │   │
    │   │   ├── (admin)/        # Layout Administrativo (Sidebar + Header)
    │   │   │   ├── layout.tsx
    │   │   │   ├── dashboard/page.tsx
    │   │   │   ├── lancamentos/page.tsx
    │   │   │   └── dre/page.tsx
    │   │   │
    │   │   └── (pdv)/          # Layout Operacional (Minimalista)
    │   │       ├── layout.tsx
    │   │       └── venda/page.tsx
    │   │
    │   ├── components/         # Biblioteca de Componentes
    │   │   ├── ui/             # ShadcnUI (Button, Input, Card, Dialog)
    │   │   ├── finance/        # StatCard, TransactionTable, MoneyInput
    │   │   └── layout/         # Sidebar, Header, MobileNav
    │   │
    │   ├── lib/                # Utilitários do Frontend
    │   │   ├── api.ts          # Cliente Axios configurado (Interceptors)
    │   │   └── utils.ts        # Formatadores (Moeda BRL, Data)
    │   │
    │   ├── hooks/              # Custom React Hooks
    │   │   ├── useAuth.ts      # Gestão de sessão
    │   │   └── useSales.ts     # Lógica de carrinho do PDV
    │   │
    │   └── types/              # Tipagem TypeScript (Interfaces)
    │
    ├── package.json
    ├── next.config.js
    └── tailwind.config.ts

