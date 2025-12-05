# FlowDash v2

> **A Evolução Enterprise do FlowDash.**
> [cite_start]Migração de uma solução local (Streamlit) para uma plataforma web escalável, multi-utilizador e de alta performance. [cite: 4]

## Visão Geral

O **FlowDash v2** é a refatoração completa do nosso sistema de gestão financeira. [cite_start]O objetivo é eliminar as limitações da versão anterior (travamento de arquivos, concorrência limitada) adotando uma arquitetura moderna de **Microsserviços Monolíticos**. [cite: 6, 7]

[cite_start]O sistema continua a ser um auditor financeiro em tempo real, focado em retalho físico, mas agora preparado para escalar. [cite: 8]

### O que muda na v2?

| Característica    | Versão 1 (Legado)              | Versão 2 (Nova Stack)          |
|-------------------|--------------------------------|--------------------------------|
| **Tecnologia**    | Python Streamlit               | Next.js (React) + FastAPI      |
| **Base de Dados** | SQLite (Arquivo Local)         | PostgreSQL (Cloud)             |
| **Acesso**        | Single-thread / Travas manuais | Multi-user / Concorrência Real |
| **Mobile**        | Responsividade limitada        | PWA Mobile-First (PDV)         |
| **Lógica**        | Misturada com UI               | Clean Architecture (Services)  |
[cite_start][cite: 10]

---

## Funcionalidades Core

[cite_start]O FlowDash v2 mantém e expande as funcionalidades críticas de negócio: [cite: 12]

### 1. Gestão Financeira (Ledger)
* [cite_start]**Princípio da Dupla Entrada:** Toda a movimentação tem origem e destino (Ex: Sai de Caixa, Entra em Banco Inter). [cite: 14]
* [cite_start]**Contas a Pagar (CAP):** Motor de amortização para Empréstimos, Boletos e Faturas de Cartão. [cite: 15]
* [cite_start]**Idempotência:** O sistema previne lançamentos duplicados mesmo se a internet do vendedor falhar. [cite: 16]

### 2. PDV Ágil (Mobile)
* [cite_start]Interface simplificada para vendedores. [cite: 18]
* [cite_start]Login rápido via PIN de 4 dígitos. [cite: 19]
* [cite_start]Cálculo automático de taxas de terminais de pagamento em tempo real. [cite: 20]
* [cite_start]**Offline First:** Funciona offline (PWA) para lançamentos essenciais. [cite: 21]

### 3. Analytics & DRE
* [cite_start]**DRE em Tempo Real:** Cálculo automático de Receita Líquida, CMV e Margem de Contribuição. [cite: 27]
* [cite_start]**Previsão de Faturação:** Integração com IA (Prophet) para projetar o fecho do mês. [cite: 28]
* [cite_start]**Metas Dinâmicas:** Acompanhamento de objetivos (Bronze/Prata/Ouro). [cite: 28]

---

## Stack Tecnológica

### Frontend (`/frontend`)
* [cite_start]**Next.js 14 (App Router):** Renderização híbrida (SSR para Dashboards, CSR para PDV). [cite: 31]
* [cite_start]**Tailwind CSS + ShadcnUI:** Design System moderno e acessível. [cite: 32]
* [cite_start]**TanStack Query:** Gerenciamento de estado assíncrono e cache. [cite: 33]

### Backend (`/backend`)
* [cite_start]**FastAPI:** Performance extrema e validação automática de dados (Pydantic). [cite: 35]
* [cite_start]**SQLAlchemy 2.0 (Async):** ORM moderno para acesso à base de dados. [cite: 35]
* [cite_start]**Alembic:** Gerenciamento de migrações do esquema da base de dados. [cite: 36]
* [cite_start]**Python 3.11+:** Tipagem forte em todo o código. [cite: 37]

### Infraestrutura
* [cite_start]**PostgreSQL 15+:** Base de dados relacional robusta. [cite: 39]
* [cite_start]**Docker:** Padronização do ambiente de desenvolvimento. [cite: 40]

---

## Como Rodar o Projeto (Dev)

### Pré-requisitos
* Docker & Docker Compose
* Node.js 18+
* Python 3.11+

### 1. Clonar e Configurar

```bash
git clone [https://github.com/alexmabud/flowdash_v2](https://github.com/alexmabud/flowdash_v2)
cd flowdash_v2

# Copie as variáveis de ambiente base
cp .env.example .env