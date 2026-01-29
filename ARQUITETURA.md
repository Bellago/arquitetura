# Arquitetura do Projeto iLace

## Diagrama de Arquitetura

```mermaid
graph TB
    subgraph "Usuários"
        U1[👤 Cliente Web]
        U2[📱 Cliente Mobile]
    end

    subgraph "Vercel"
        FE[🌐 bellago-web<br/>Next.js 15 + React 19<br/>TailwindCSS<br/>Zustand + TanStack Query]
    end

    subgraph "Oracle OCI"
        BE[⚙️ ilace-backend<br/>NestJS + TypeORM<br/>Node.js<br/>Docker]
    end

    subgraph "Supabase"
        DB[(🗄️ PostgreSQL<br/>Database)]
        AUTH[🔐 Supabase Auth]
        STORAGE[📦 Storage]
    end

    subgraph "Serviços Externos"
        N8N[🔄 N8N<br/>Automação/Webhooks]
    end

    %% Conexões dos usuários
    U1 -->|HTTPS| FE
    U2 -->|HTTPS| FE

    %% Conexões do Frontend
    FE -->|REST API<br/>Axios| BE
    FE -->|Supabase Client<br/>@supabase/ssr| AUTH
    FE -->|Supabase Client| DB
    FE -->|Supabase Client| STORAGE

    %% Conexões do Backend
    BE -->|TypeORM<br/>PostgreSQL Driver| DB
    BE -->|Supabase Client<br/>@supabase/supabase-js| AUTH
    BE -->|Webhooks| N8N

    %% Estilos
    classDef vercel fill:#000,stroke:#fff,stroke-width:2px,color:#fff
    classDef oracle fill:#f80000,stroke:#fff,stroke-width:2px,color:#fff
    classDef supabase fill:#3ecf8e,stroke:#fff,stroke-width:2px,color:#000
    classDef external fill:#6366f1,stroke:#fff,stroke-width:2px,color:#fff
    classDef user fill:#8b5cf6,stroke:#fff,stroke-width:2px,color:#fff

    class FE vercel
    class BE oracle
    class DB,AUTH,STORAGE supabase
    class N8N external
    class U1,U2 user
```

## Descrição dos Componentes

### 🌐 Frontend - bellago-web (Vercel)
- **Framework**: Next.js 15 com React 19
- **Estilização**: TailwindCSS + Radix UI
- **Estado**: Zustand + TanStack Query
- **Comunicação**: 
  - REST API via Axios para o backend
  - Supabase Client para autenticação e banco de dados
- **Deploy**: Vercel (CI/CD automático)

### ⚙️ Backend - ilace-backend (Oracle OCI)
- **Framework**: NestJS
- **ORM**: TypeORM
- **Runtime**: Node.js
- **Containerização**: Docker
- **Principais Módulos**:
  - Appointments (Agendamentos)
  - Auth (Autenticação)
  - Business (Negócios)
  - Professionals (Profissionais)
  - Clients (Clientes)
  - Services (Serviços)
  - Payments (Pagamentos)
  - Notifications (Notificações)
- **Deploy**: Oracle Cloud Infrastructure (OCI)

### 🗄️ Banco de Dados - Supabase
- **Tipo**: PostgreSQL
- **Principais Tabelas**:
  - `business_appointments` - Agendamentos
  - `business_professional_working_hours` - Horários de trabalho
  - `business` - Negócios
  - `business_services` - Serviços oferecidos
  - `clients` - Clientes
  - `professionals` - Profissionais
- **Recursos Adicionais**:
  - Supabase Auth para gerenciamento de usuários
  - Supabase Storage para arquivos

### 🔄 Integrações Externas
- **N8N**: Automação de workflows e webhooks
  - Notificações
  - Integrações com serviços externos

## Fluxo de Dados

1. **Autenticação**: 
   - Usuário faz login pelo frontend
   - Frontend usa Supabase Auth
   - Backend valida tokens JWT

2. **Operações CRUD**:
   - Frontend envia requisições REST para o backend
   - Backend processa e valida
   - Backend persiste no PostgreSQL via TypeORM

3. **Consultas Diretas**:
   - Algumas operações de leitura podem ir direto do frontend para o Supabase
   - Otimiza performance e reduz carga no backend

4. **Webhooks/Notificações**:
   - Backend dispara eventos para N8N
   - N8N executa automações configuradas

## Segurança

- ✅ Autenticação JWT via Supabase
- ✅ HTTPS em todas as conexões
- ✅ Row Level Security (RLS) no Supabase
- ✅ Validações no backend com class-validator
- ✅ Variáveis de ambiente para secrets
- ✅ CORS configurado

## Escalabilidade

- **Frontend**: Escala automaticamente na Vercel (serverless)
- **Backend**: Containerizado com Docker, pode escalar horizontalmente na OCI
- **Banco de Dados**: Supabase gerencia escalabilidade do PostgreSQL
