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

    subgraph "AWS (us-east-1)"
        BE[⚙️ ilace-backend<br/>NestJS + TypeORM<br/>Node.js<br/>Docker]
        ECS[☁️ ECS Fargate<br/>Cluster: ilace-cluster<br/>Service: ilace-backend-service]
        ECR[📦 ECR<br/>ilace-backend]
        SM[🔑 Secrets Manager<br/>ilace-backend/env]
        CW[📊 CloudWatch Logs<br/>/ecs/ilace-backend]
        BE --- ECS
    end

    subgraph "Supabase"
        DB[(🗄️ PostgreSQL<br/>Database)]
        AUTH[🔐 Supabase Auth]
        STORAGE[📦 Storage]
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

    %% Infraestrutura AWS
    ECR -.->|pull image| ECS
    SM -.->|inject secrets| BE
    BE -.->|logs| CW

    %% Estilos
    classDef vercel fill:#000,stroke:#fff,stroke-width:2px,color:#fff
    classDef aws fill:#ff9900,stroke:#232f3e,stroke-width:2px,color:#000
    classDef supabase fill:#3ecf8e,stroke:#fff,stroke-width:2px,color:#000
    classDef user fill:#8b5cf6,stroke:#fff,stroke-width:2px,color:#fff

    class FE vercel
    class BE,ECS,ECR,SM,CW aws
    class DB,AUTH,STORAGE supabase
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

### ⚙️ Backend - ilace-backend (AWS)
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
- **Deploy**: AWS ECS Fargate
  - **Região**: `us-east-1`
  - **Cluster**: `ilace-cluster`
  - **Service**: `ilace-backend-service`
  - **Task Definition Family**: `ilace-backend`
  - **Registry**: Amazon ECR (`ilace-backend`)
  - **Secrets**: AWS Secrets Manager (`ilace-backend/env`)
  - **Logs**: Amazon CloudWatch Logs (`/ecs/ilace-backend`)
  - **CI/CD**: GitHub Actions

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

## Segurança

- ✅ Autenticação JWT via Supabase
- ✅ HTTPS em todas as conexões
- ✅ Row Level Security (RLS) no Supabase
- ✅ Validações no backend com class-validator
- ✅ Secrets gerenciados via AWS Secrets Manager
- ✅ CORS configurado

## Escalabilidade

- **Frontend**: Escala automaticamente na Vercel (serverless)
- **Backend**: Containerizado com Docker, escala horizontalmente no AWS ECS Fargate (cluster `ilace-cluster`)
- **Banco de Dados**: Supabase gerencia escalabilidade do PostgreSQL
