# 🚀 FlowDash v2

> **A evolução enterprise do FlowDash:** migração de uma solução local (Streamlit) para uma plataforma web escalável, multiutilizador e de alta performance.

## 📋 Índice
- [Visão Geral](#-visão-geral)
- [O que muda na v2?](#-o-que-muda-na-v2)
- [Funcionalidades Core](#-funcionalidades-core)
- [Stack Tecnológica](#-stack-tecnológica)
- [Como rodar (dev)](#-como-rodar-dev)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Segurança e Integridade](#-segurança-e-integridade)
- [Contribuição](#-contribuição)

---

## 🔭 Visão Geral
**FlowDash v2** é a refatoração completa do sistema de gestão financeira. Elimina limitações da versão anterior (travas de ficheiro e concorrência limitada) adotando uma arquitetura de microsserviços monolíticos. O produto segue como auditor financeiro em tempo real para retalho físico, agora preparado para crescer.

## 🔄 O que muda na v2?

| Característica | Versão 1 (Legado) | Versão 2 (Nova Stack) |
| :--- | :--- | :--- |
| **Tecnologia** | Python Streamlit | Next.js (React) + FastAPI |
| **Base de dados** | SQLite (arquivo local) | PostgreSQL (cloud) |
| **Acesso** | Single-thread / travas manuais | Multiusuário / concorrência real |
| **Mobile** | Responsividade limitada | PWA mobile-first (PDV) |
| **Lógica** | Misturada com UI | Clean Architecture (services) |

## 💎 Funcionalidades Core

### 💰 Gestão Financeira (Ledger)
- **Princípio da dupla entrada:** cada movimento tem origem e destino (ex.: sai de Caixa, entra em Banco Inter).
- **Contas a pagar:** motor de amortização para empréstimos, boletos e faturas de cartão.
- **Idempotência:** evita lançamentos duplicados mesmo com falhas de conexão.

### 📱 PDV Ágil (Mobile)
- Interface simplificada para vendedores.
- Login rápido via PIN de 4 dígitos.
- Cálculo automático de taxas dos terminais de pagamento.
- **Offline first:** PWA que aceita lançamentos essenciais sem internet.

### 📊 Analytics e DRE
- DRE em tempo real com receita líquida, CMV e margem de contribuição.
- Previsão de faturação com IA (Prophet) para projetar fecho do mês.
- Metas dinâmicas (bronze/prata/ouro).

## 🛠 Stack Tecnológica

### Frontend (`/frontend`)
- **Next.js 14 (App Router)** com renderização híbrida (SSR para dashboards, CSR para PDV).
- **Tailwind CSS + Shadcn UI** como design system.
- **TanStack Query** para estado assíncrono e cache.

### Backend (`/backend`)
- **FastAPI** com validação Pydantic.
- **SQLAlchemy 2.0 (async)** como ORM.
- **Alembic** para migrações.
- **Python 3.11+** com tipagem forte.

### Infraestrutura
- **PostgreSQL 15+** como base relacional.
- **Docker** para padronização de ambiente.

## � Como rodar (dev)

### Pré-requisitos
- Docker e Docker Compose
- Node.js 18+
- Python 3.11+

### Passo a passo sugerido

1.  **Clonar o repositório:**
    ```bash
    git clone https://github.com/alexmabud/flowdash_v2
    cd flowdash_v2
    ```

2.  **Copiar variáveis de ambiente base:**
    ```bash
    cp .env.example .env
    ```

3.  **Levantar a stack local** (ajuste conforme seus serviços):
    ```bash
    docker compose up --build
    ```

### Acessar serviços
- **Frontend:** [http://localhost:3000](http://localhost:3000)
- **API Docs (Swagger):** [http://localhost:8000/docs](http://localhost:8000/docs)
- **PgAdmin/Adminer (opcional):** [http://localhost:8080](http://localhost:8080)

## 📂 Estrutura do Projeto
```plaintext
flowdash-v2/
├── backend/                  # API FastAPI (cérebro do sistema)
│   ├── app/
│   │   ├── api/              # Endpoints REST
│   │   ├── models/           # Tabelas (SQLAlchemy ORM)
│   │   ├── services/         # Regras de negócio (Ledger, PDV, Cálculos)
│   │   └── schemas/          # DTOs e validação (Pydantic)
│   └── alembic/              # Migrações do BD
│
├── frontend/                 # Next.js App
│   └── src/
│       ├── app/              # Páginas (Admin, PDV, Login)
│       ├── components/       # Componentes reutilizáveis
│       └── lib/              # Cliente API + utils
│
└── DESIGN_DOCUMENT_V2.md     # Blueprint técnico completo
```

## 🔒 Segurança e Integridade

### 🔑 Autenticação
JWT com refresh tokens e rotação automática.

### 🔐 Trava de Fechamento
Middleware impede alterações em dias com caixa encerrado, garantindo imutabilidade do passado.

### 📝 Auditoria
Todas as transações possuem `user_id`, `created_at`, `updated_at`.

## 🤝 Contribuição
Antes de qualquer desenvolvimento, consulte **[`DESIGN_DOCUMENT_V2.md`](./DESIGN_DOCUMENT_V2.md)**.

Ele contém:
- Especificações técnicas
- Regras de negócio migradas
- Arquitetura detalhada
- Orientações para agentes de IA

## ✍️ Autor
**Alex Abud**  
*Projeto: FlowDash v2 — Enterprise Financial System*