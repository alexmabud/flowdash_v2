💼 FlowDash v2

A Evolução Enterprise do FlowDash.
Migrando de uma solução local para uma plataforma web escalável, multi-usuário e de alta performance.

🎯 Visão Geral

O FlowDash v2 é a refatoração completa do nosso sistema de gestão financeira. O objetivo é eliminar as limitações da versão anterior (travamento de arquivos, concorrência limitada) adotando uma arquitetura moderna de Microsserviços Monolíticos.

O sistema continua sendo um auditor financeiro em tempo real, focado em varejo físico, mas agora preparado para escalar.

🚀 O que muda na v2?

Característica

Versão 1 (Legado)

Versão 2 (Nova Stack)

Tecnologia

Python + Streamlit

Next.js (React) + FastAPI

Banco de Dados

SQLite (Arquivo Local)

PostgreSQL (Cloud)

Acesso

Single-thread / Travas manuais

Multi-user / Concorrência Real

Mobile

Responsividade limitada

PWA Mobile-First (PDV)

Lógica

Misturada com UI

Clean Architecture (Services)

🧠 Funcionalidades Core

O FlowDash v2 mantém e expande as funcionalidades críticas de negócio:

💰 1. Gestão Financeira (Ledger)

Princípio da Dupla Entrada: Toda movimentação tem origem e destino (Ex: Sai de Caixa, Entra em Banco Inter).

Contas a Pagar (CAP): Motor de amortização para Empréstimos, Boletos e Faturas de Cartão.

Idempotência: O sistema previne lançamentos duplicados mesmo se a internet do vendedor falhar.

📱 2. PDV Ágil (Mobile)

Interface simplificada para vendedores.

Login rápido via PIN de 4 dígitos.

Cálculo automático de taxas de maquininha em tempo real.

Funciona offline (PWA) para lançamentos essenciais.

📊 3. Analytics & DRE

DRE em Tempo Real: Cálculo automático de Receita Líquida, CMV e Margem de Contribuição.

Previsão de Faturamento: Integração com IA (Prophet) para projetar o fechamento do mês.

Metas Dinâmicas: Acompanhamento de atingimento (Bronze/Prata/Ouro).

🛠️ Stack Tecnológica

Frontend (/frontend)

Next.js 14 (App Router): Renderização híbrida (SSR para Dashboards, CSR para PDV).

Tailwind CSS + ShadcnUI: Design System moderno e acessível.

TanStack Query: Gerenciamento de estado assíncrono e cache.

Backend (/backend)

FastAPI: Performance extrema e validação automática de dados (Pydantic).

SQLAlchemy 2.0 (Async): ORM moderno para acesso ao banco.

Alembic: Gerenciamento de migrações do esquema do banco.

Python 3.11+: Tipagem forte em todo o código.

Infraestrutura

PostgreSQL 15+: Banco de dados relacional robusto.

Docker: Padronização do ambiente de desenvolvimento.

🏗️ Como Rodar o Projeto (Dev)

Pré-requisitos

Docker & Docker Compose

Node.js 18+

Python 3.11+

1. Clonar e Configurar

git clone [https://github.com/alexmabud/flowdash_v2](https://github.com/alexmabud/flowdash_v2)
cd flowdash_v2

# Copie as variáveis de ambiente
cp .env.example .env


2. Iniciar os Serviços (Docker)

Levanta o Banco de Dados, Backend e Frontend simultaneamente.

docker-compose up -d --build


3. Acessar

Frontend: http://localhost:3000

API Docs (Swagger): http://localhost:8000/docs

Banco (Adminer/PgAdmin): http://localhost:8080 (se configurado)

🗂️ Estrutura do Projeto

flowdash-v2/
├── backend/                # API FastAPI
│   ├── app/
│   │   ├── api/            # Rotas (Endpoints)
│   │   ├── models/         # Tabelas do Banco (SQLAlchemy)
│   │   ├── services/       # Regras de Negócio (Ledger, Vendas)
│   │   └── schemas/        # Validação de Dados (Pydantic)
│   └── alembic/            # Migrações de Banco
│
├── frontend/               # Next.js App
│   ├── src/
│   │   ├── app/            # Páginas (Admin, PDV, Login)
│   │   ├── components/     # UI Reutilizável
│   │   └── lib/            # Utilitários e API Client
│
└── DESIGN_DOCUMENT_V2.md   # Documentação Técnica Completa (Blueprint)


🔐 Segurança

Autenticação: JWT (JSON Web Tokens) com refresh rotation.

Trava de Fechamento: Middleware que impede edições em dias com caixa já encerrado (LockService).

Auditoria: Todas as transações financeiras possuem log de created_at e user_id.

👨‍💻 Contribuição

Consulte o arquivo DESIGN_DOCUMENT_V2.md na raiz antes de iniciar qualquer desenvolvimento. Ele contém as especificações técnicas, regras de negócio migradas e o mapa detalhado dos módulos.

Autor: Alex Abud
Projeto: FlowDash v2 — Enterprise Financial System