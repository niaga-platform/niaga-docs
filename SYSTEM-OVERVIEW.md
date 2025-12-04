# NIAGA PLATFORM - Complete System Overview

## 📋 Table of Contents
1. [Project Summary](#1-project-summary)
2. [Tech Stack](#2-tech-stack)
3. [Architecture Overview](#3-architecture-overview)
4. [Microservices](#4-microservices)
5. [Frontend Applications](#5-frontend-applications)
6. [Database Schema](#6-database-schema)
7. [API Endpoints](#7-api-endpoints)
8. [Authentication & Authorization](#8-authentication--authorization)
9. [Key Features](#9-key-features)
10. [File Structure](#10-file-structure)
11. [Environment Variables](#11-environment-variables)
12. [How to Run](#12-how-to-run)

---

## 1. Project Summary

**Niaga Platform** adalah comprehensive e-commerce platform untuk batik fashion business di Malaysia.

| Attribute | Value |
|-----------|-------|
| Business Type | Batik Fashion E-commerce |
| Product Types | Kain (by meter), Ready-made (by size) |
| SKUs | ~11,000 |
| Branches | 10 |
| Agents | 100 |
| Traffic Split | 60% Mobile, 40% Desktop |

### Key Business Features:
- Multi-branch inventory management
- Agent/Team sales with commission tracking
- Multiple payment methods (Online, CDM, Bank Transfer)
- White-label template (can be cloned for different brands)

---

## 2. Tech Stack

### Backend
| Component | Technology |
|-----------|------------|
| Language | Go 1.21+ |
| Framework | Gin |
| ORM | GORM |
| Database | PostgreSQL 16 |
| Cache | Redis |
| Search | Meilisearch |
| Message Queue | NATS |
| File Storage | MinIO |
| Logging | Zap |

### Frontend
| Component | Technology |
|-----------|------------|
| Framework | Next.js 14/16 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| UI Components | shadcn/ui |
| State Management | Zustand |
| Data Fetching | React Query (TanStack) |
| Charts | Recharts |
| Icons | Lucide React |

### Infrastructure
| Component | Technology |
|-----------|------------|
| Containerization | Docker |
| Orchestration | Docker Swarm |
| Reverse Proxy | Traefik |
| Monitoring | Prometheus + Grafana |
| CI/CD | GitHub Actions |

---

## 3. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS                                     │
│         Browser / Mobile App / Admin Dashboard / Agent Portal           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           TRAEFIK (Reverse Proxy)                        │
│                         Load Balancing & SSL                            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        ▼                           ▼                           ▼
┌───────────────┐          ┌───────────────┐          ┌───────────────┐
│   Frontend    │          │   Frontend    │          │   Frontend    │
│  Storefront   │          │    Admin      │          │    Agent      │
│  (Next.js)    │          │  (Next.js)    │          │ (Components)  │
│  Port: 3000   │          │  Port: 3001   │          │               │
└───────────────┘          └───────────────┘          └───────────────┘
        │                           │                           │
        └───────────────────────────┼───────────────────────────┘
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        SERVICE GATEWAY (Go/Gin)                          │
│                           Port: 8080                                     │
│                    API Routing, Auth, Rate Limiting                     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
    ┌─────────┬─────────┬─────────┬─┴───────┬─────────┬─────────┬────────┐
    ▼         ▼         ▼         ▼         ▼         ▼         ▼        ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│ Auth  │ │Catalog│ │ Order │ │Invent.│ │Custom.│ │Content│ │Notif. │ │Report │
│ :8081 │ │ :8082 │ │ :8083 │ │ :8084 │ │ :8085 │ │ :8086 │ │ :8087 │ │ :8088 │
└───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘
    │         │         │         │         │         │         │        │
    └─────────┴─────────┴─────────┴────┬────┴─────────┴─────────┴────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              ▼                        ▼                        ▼
      ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
      │  PostgreSQL   │       │     Redis     │       │     NATS      │
      │   (Primary)   │       │    (Cache)    │       │   (Events)    │
      └───────────────┘       └───────────────┘       └───────────────┘
              │
              ▼
      ┌───────────────┐       ┌───────────────┐
      │  Meilisearch  │       │     MinIO     │
      │   (Search)    │       │   (Storage)   │
      └───────────────┘       └───────────────┘
```

---

## 4. Microservices

### 4.1 Service Details

| Service | Port | Responsibility |
|---------|------|----------------|
| **service-gateway** | 8080 | API Gateway, routing, rate limiting |
| **service-auth** | 8081 | Authentication, JWT, RBAC, user management |
| **service-catalog** | 8082 | Products, categories, variants, pricing |
| **service-order** | 8083 | Orders, cart, checkout, payments |
| **service-inventory** | 8084 | Stock management, warehouses, transfers |
| **service-customer** | 8085 | Customer profiles, addresses, preferences |
| **service-content** | 8086 | CMS, pages, banners, menus, site settings |
| **service-notification** | 8087 | Email, SMS, push notifications |
| **service-reporting** | 8088 | Analytics, reports, dashboards |

### 4.2 Service Structure (Standard)

```
service-{name}/
├── .github/workflows/ci.yml
├── cmd/
│   └── server/
│       └── main.go              # Entry point
├── internal/
│   ├── config/
│   │   └── config.go            # Configuration loading
│   ├── handlers/
│   │   └── {name}_handler.go    # HTTP handlers
│   ├── middleware/
│   │   ├── auth.go              # JWT validation
│   │   └── session.go           # Session handling
│   ├── models/
│   │   └── {name}.go            # Data models
│   ├── repository/
│   │   └── {name}_repository.go # Database operations
│   ├── services/
│   │   └── {name}_service.go    # Business logic
│   ├── events/
│   │   └── nats_publisher.go    # Event publishing
│   └── utils/
├── migrations/
│   └── *.sql                    # Database migrations
├── pkg/
│   └── storage/
│       └── local.go             # File storage
├── Dockerfile
├── go.mod
├── go.sum
└── README.md
```

---

## 5. Frontend Applications

### 5.1 frontend-storefront (Customer-facing)
**Port:** 3000

```
frontend-storefront/
├── app/
│   ├── (shop)/                  # Public shop pages
│   │   ├── page.tsx             # Homepage
│   │   ├── products/            # Product listing & detail
│   │   ├── categories/          # Category pages
│   │   ├── cart/                # Shopping cart
│   │   ├── checkout/            # Checkout flow
│   │   └── tailoring/           # Custom tailoring orders
│   ├── (auth)/                  # Authentication
│   │   ├── login/
│   │   └── register/
│   ├── (account)/               # Customer account
│   │   └── account/
│   │       ├── orders/
│   │       ├── addresses/
│   │       └── profile/
│   └── (agent)/                 # Agent portal (integrated)
│       └── agent/
│           ├── page.tsx         # Agent dashboard
│           ├── orders/          # Agent orders
│           ├── customers/       # Agent customers
│           ├── commissions/     # Commission tracking
│           └── performance/     # Performance metrics
├── components/
├── lib/
└── public/
```

### 5.2 frontend-admin (Admin Dashboard)
**Port:** 3001

```
frontend-admin/
├── src/app/
│   ├── (auth)/login/
│   ├── (dashboard)/
│   │   ├── page.tsx             # Dashboard
│   │   ├── orders/              # Order management
│   │   ├── products/            # Product management
│   │   ├── categories/          # Category management
│   │   ├── customers/           # Customer management
│   │   ├── inventory/           # Inventory management
│   │   │   ├── stock/
│   │   │   ├── transfers/
│   │   │   └── warehouses/
│   │   ├── agents/              # Agent management
│   │   └── reports/             # Reports & analytics
│   ├── payments/
│   │   └── pending/             # Payment verification
│   └── settings/
│       ├── users/               # User management
│       ├── roles/               # Role management
│       └── payment-methods/     # Payment settings
├── components/
│   ├── auth/
│   │   ├── PermissionGate.tsx
│   │   └── RoleGate.tsx
│   ├── layout/
│   │   └── Sidebar.tsx
│   ├── payments/
│   │   ├── PaymentVerifyModal.tsx
│   │   └── ReceiptViewer.tsx
│   └── ui/                      # shadcn components
├── lib/
│   ├── api/rbac.ts
│   ├── constants/
│   │   ├── permissions.ts
│   │   └── roles.ts
│   └── hooks/
│       ├── useAuth.ts
│       └── usePermissions.ts
└── public/
```

### 5.3 frontend-agent (Shared Components)
```
frontend-agent/
├── components/
│   ├── layout/
│   │   ├── AgentSidebar.tsx
│   │   └── AgentHeader.tsx
│   ├── dashboard/
│   │   ├── StatsCards.tsx
│   │   ├── RecentOrders.tsx
│   │   └── PerformanceChart.tsx
│   ├── orders/
│   │   ├── CreateOrderWizard.tsx
│   │   └── OrdersTable.tsx
│   ├── customers/
│   └── commissions/
├── hooks/
│   ├── useAgent.ts
│   └── useCommissions.ts
├── lib/
│   └── api/agent.ts
└── index.ts                     # Exports all
```

---

## 6. Database Schema

### 6.1 Schema Organization

| Schema | Purpose |
|--------|---------|
| `auth` | Users, roles, permissions, sessions |
| `catalog` | Products, categories, variants, pricing |
| `sales` | Orders, cart, payments, coupons |
| `inventory` | Stock, warehouses, transfers |
| `customer` | Customer profiles, addresses |
| `content` | CMS pages, banners, menus |

### 6.2 Key Tables

#### auth schema
```sql
auth.users
├── id (UUID, PK)
├── email (UNIQUE)
├── password_hash
├── name
├── phone
├── user_type (customer/admin/agent)
├── is_active
├── email_verified
├── created_at
└── updated_at

auth.roles
├── id (UUID, PK)
├── code (UNIQUE) -- SUPER_ADMIN, MANAGER, etc
├── name
├── description
├── is_system
└── is_active

auth.permissions
├── id (UUID, PK)
├── code (UNIQUE) -- products.view, orders.create
├── name
├── module
└── action

auth.role_permissions (M:M)
├── role_id (FK)
└── permission_id (FK)

auth.user_roles (M:M)
├── user_id (FK)
└── role_id (FK)
```

#### sales schema
```sql
sales.orders
├── id (UUID, PK)
├── order_number (UNIQUE)
├── customer_id (FK)
├── agent_id (FK, nullable)
├── status
├── payment_status
├── payment_method_code
├── subtotal
├── shipping_cost
├── discount
├── total
├── agent_commission
├── shipping_address (JSONB)
├── notes
├── created_at
└── updated_at

sales.order_items
├── id (UUID, PK)
├── order_id (FK)
├── product_id
├── variant_id
├── product_name
├── variant_name
├── sku
├── quantity
├── unit_price
├── subtotal
└── metadata (JSONB) -- meter, size, color

sales.payments
├── id (UUID, PK)
├── order_id (FK)
├── payment_method_code
├── amount
├── status
├── receipt_url
├── depositor_name
├── deposit_date
├── deposit_reference
├── deposit_bank
├── verified_by (FK)
├── verified_at
├── rejection_reason
└── created_at

sales.payment_methods
├── id (UUID, PK)
├── code (UNIQUE)
├── name
├── name_ms
├── description
├── instructions
├── bank_name
├── bank_account_name
├── bank_account_number
├── requires_receipt
├── requires_verification
├── is_active
└── sort_order

sales.teams
├── id (UUID, PK)
├── code (UNIQUE)
├── name
├── leader_id (FK)
├── target_monthly
├── commission_rate
└── is_active

sales.agents
├── id (UUID, PK)
├── user_id (FK)
├── team_id (FK)
├── code (UNIQUE)
├── name
├── email
├── phone
├── commission_rate
├── status
└── created_at

sales.agent_commissions
├── id (UUID, PK)
├── agent_id (FK)
├── order_id (FK)
├── order_total
├── commission_rate
├── commission_amount
├── status (pending/approved/paid)
├── approved_by
├── approved_at
├── paid_at
└── created_at
```

#### catalog schema
```sql
catalog.products
├── id (UUID, PK)
├── sku (UNIQUE)
├── name
├── slug (UNIQUE)
├── description
├── category_id (FK)
├── product_type (fabric/readymade)
├── base_price
├── is_active
├── metadata (JSONB)
└── created_at

catalog.product_variants
├── id (UUID, PK)
├── product_id (FK)
├── sku (UNIQUE)
├── name
├── color
├── size
├── price
├── images (JSONB)
└── is_active

catalog.categories
├── id (UUID, PK)
├── name
├── slug (UNIQUE)
├── parent_id (FK, self-reference)
├── description
├── image_url
├── sort_order
└── is_active
```

#### inventory schema
```sql
inventory.warehouses
├── id (UUID, PK)
├── code (UNIQUE)
├── name
├── address
├── is_active
└── is_default

inventory.stock
├── id (UUID, PK)
├── warehouse_id (FK)
├── variant_id (FK)
├── quantity
├── reserved_quantity
├── available_quantity (computed)
└── updated_at

inventory.stock_movements
├── id (UUID, PK)
├── warehouse_id (FK)
├── variant_id (FK)
├── movement_type
├── quantity
├── reference_type
├── reference_id
├── notes
└── created_at
```

---

## 7. API Endpoints

### 7.1 Authentication (service-auth)
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh
POST   /api/v1/auth/forgot-password
POST   /api/v1/auth/reset-password
GET    /api/v1/auth/me                    # Returns user + roles + permissions
PUT    /api/v1/auth/me
```

### 7.2 RBAC (service-auth)
```
GET    /api/v1/admin/users
POST   /api/v1/admin/users
GET    /api/v1/admin/users/:id
PUT    /api/v1/admin/users/:id
DELETE /api/v1/admin/users/:id

GET    /api/v1/admin/roles
POST   /api/v1/admin/roles
GET    /api/v1/admin/roles/:id
PUT    /api/v1/admin/roles/:id
DELETE /api/v1/admin/roles/:id

GET    /api/v1/admin/permissions
```

### 7.3 Catalog (service-catalog)
```
GET    /api/v1/products
GET    /api/v1/products/:id
GET    /api/v1/products/slug/:slug
POST   /api/v1/admin/products
PUT    /api/v1/admin/products/:id
DELETE /api/v1/admin/products/:id

GET    /api/v1/categories
GET    /api/v1/categories/:id
POST   /api/v1/admin/categories
PUT    /api/v1/admin/categories/:id
DELETE /api/v1/admin/categories/:id

GET    /api/v1/search?q=batik
```

### 7.4 Orders (service-order)
```
# Cart
GET    /api/v1/cart
POST   /api/v1/cart/items
PUT    /api/v1/cart/items/:id
DELETE /api/v1/cart/items/:id
DELETE /api/v1/cart

# Checkout
POST   /api/v1/checkout
GET    /api/v1/orders
GET    /api/v1/orders/:id

# Admin
GET    /api/v1/admin/orders
GET    /api/v1/admin/orders/:id
PUT    /api/v1/admin/orders/:id/status
```

### 7.5 Payments (service-order)
```
# Public
GET    /api/v1/payment-methods
POST   /api/v1/payments/upload-receipt
GET    /api/v1/payments/:orderId/status

# Admin
GET    /api/v1/admin/payments
GET    /api/v1/admin/payments/pending
GET    /api/v1/admin/payments/:id
PUT    /api/v1/admin/payments/:id/verify
PUT    /api/v1/admin/payments/:id/reject
PUT    /api/v1/admin/payment-methods/:id
```

### 7.6 Agent Portal (service-order/service-auth)
```
GET    /api/v1/agent/profile
GET    /api/v1/agent/dashboard
GET    /api/v1/agent/orders
POST   /api/v1/agent/orders
GET    /api/v1/agent/orders/:id
GET    /api/v1/agent/customers
POST   /api/v1/agent/customers
GET    /api/v1/agent/customers/:id
PUT    /api/v1/agent/customers/:id
GET    /api/v1/agent/commissions
GET    /api/v1/agent/performance
GET    /api/v1/agent/team
```

### 7.7 Inventory (service-inventory)
```
GET    /api/v1/admin/warehouses
POST   /api/v1/admin/warehouses
GET    /api/v1/admin/warehouses/:id
PUT    /api/v1/admin/warehouses/:id

GET    /api/v1/admin/stock
PUT    /api/v1/admin/stock/:variantId
POST   /api/v1/admin/stock/transfer
GET    /api/v1/admin/stock/movements
```

### 7.8 Content/CMS (service-content)
```
GET    /api/v1/pages
GET    /api/v1/pages/:slug
GET    /api/v1/banners
GET    /api/v1/menus
GET    /api/v1/site-settings

POST   /api/v1/admin/pages
PUT    /api/v1/admin/pages/:id
POST   /api/v1/admin/banners
PUT    /api/v1/admin/banners/:id
POST   /api/v1/admin/menus
PUT    /api/v1/admin/menus/:id
PUT    /api/v1/admin/site-settings
```

---

## 8. Authentication & Authorization

### 8.1 User Types
| Type | Access | Description |
|------|--------|-------------|
| `customer` | Storefront | Regular customers, shopping |
| `agent` | Agent Portal | Sales agents, submit orders for customers |
| `admin` | Admin Dashboard | Staff with role-based access |

### 8.2 Login Flow
```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Login Page     │     │    Backend       │     │    Redirect      │
│   /login         │────>│  /auth/login     │────>│                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │  Check user_type      │
                    ├───────────────────────┤
                    │ customer → /account   │
                    │ agent    → /agent     │
                    │ admin    → /admin     │
                    └───────────────────────┘
```

### 8.3 RBAC Roles
| Role | Permissions |
|------|-------------|
| `SUPER_ADMIN` | Full access to everything |
| `MANAGER` | Orders, Products, Customers, Reports |
| `STAFF_ORDERS` | Orders only (view, update status) |
| `STAFF_PRODUCTS` | Products, Inventory only |
| `STAFF_CONTENT` | CMS Content only |
| `ACCOUNTANT` | Orders (view), Reports, Commissions |
| `AGENT_MANAGER` | Agents, Teams, Commissions |

### 8.4 Permission Format
```
module.action

Examples:
- dashboard.view
- products.view
- products.create
- products.update
- products.delete
- orders.view
- orders.update
- orders.export
- payments.verify
- users.roles
```

---

## 9. Key Features

### 9.1 Agent Portal
- Agents login via storefront (`/login`)
- Redirect to `/agent` dashboard
- Submit orders on behalf of customers
- Track personal sales & commissions
- View team leaderboard

### 9.2 Commission System
```
Order Created (with agent_id)
        │
        ▼
Auto-calculate commission
(order_total × agent.commission_rate)
        │
        ▼
Commission status: PENDING
        │
        ▼
Payment Verified
        │
        ▼
Commission status: APPROVED
        │
        ▼
Admin marks as paid
        │
        ▼
Commission status: PAID
```

### 9.3 Payment Methods
| Method | Type | Verification |
|--------|------|--------------|
| FPX (Online Banking) | Redirect | Auto |
| Credit/Debit Card | Redirect | Auto |
| Cash Deposit (CDM) | Manual | Admin verify receipt |
| Bank Transfer | Manual | Admin verify receipt |

### 9.4 Batik-Specific Features
- **Fabric by meter**: Min 1m, increment 0.5m
- **Ready-made by size**: XS, S, M, L, XL, 2XL, 3XL
- **Color selection**: Color swatches with stock indicator
- **Custom tailoring**: Measurement capture, custom orders

---

## 10. File Structure

```
niaga-platform/
├── .github-templates/           # CI workflow templates
├── .vscode/                     # VS Code settings
├── docs/                        # Documentation
│   ├── ARCHITECTURE.md
│   ├── DATABASE-SCHEMA.md
│   ├── DEPLOYMENT.md
│   ├── ENVIRONMENT-VARIABLES.md
│   └── api/openapi.yaml
│
├── frontend-admin/              # Admin dashboard
├── frontend-storefront/         # Customer storefront
├── frontend-agent/              # Agent components (shared)
│
├── service-auth/                # Authentication service
├── service-catalog/             # Product catalog service
├── service-order/               # Order & payment service
├── service-inventory/           # Inventory service
├── service-customer/            # Customer service
├── service-content/             # CMS service
├── service-notification/        # Notification service
├── service-reporting/           # Reports service
├── service-gateway/             # API gateway
│
├── infra-platform/              # Infrastructure
│   ├── docker/
│   ├── traefik/
│   ├── monitoring/
│   │   ├── grafana/
│   │   └── prometheus/
│   └── scripts/
│
├── infra-database/              # Database setup
│   ├── migrations/
│   ├── seeds/
│   └── scripts/
│
├── docker-compose.dev.yml       # Development compose
└── README.md
```

---

## 11. Environment Variables

### Backend Services (Common)
```env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=password
DB_NAME=niaga_db
DB_SSL_MODE=disable

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# NATS
NATS_URL=nats://localhost:4222

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRY=24h

# MinIO
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=niaga-uploads

# Service Ports
AUTH_SERVICE_PORT=8081
CATALOG_SERVICE_PORT=8082
ORDER_SERVICE_PORT=8083
INVENTORY_SERVICE_PORT=8084
```

### Frontend
```env
# API
NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1

# Site
NEXT_PUBLIC_SITE_NAME=Niaga Platform
NEXT_PUBLIC_SITE_URL=http://localhost:3000

# Features
NEXT_PUBLIC_ENABLE_AGENT_PORTAL=true
NEXT_PUBLIC_ENABLE_CDM_PAYMENT=true
```

---

## 12. How to Run

### Prerequisites
- Go 1.21+
- Node.js 18+
- Docker & Docker Compose
- PostgreSQL 16
- Redis

### Development Setup

```bash
# 1. Clone repository
git clone https://github.com/your-org/niaga-platform.git
cd niaga-platform

# 2. Start infrastructure
docker-compose -f docker-compose.dev.yml up -d

# 3. Run database migrations
cd infra-database
./scripts/migrate.sh

# 4. Start backend services (each in separate terminal)
cd service-auth && go run cmd/server/main.go
cd service-catalog && go run cmd/server/main.go
cd service-order && go run cmd/server/main.go
# ... etc

# 5. Start frontend
cd frontend-storefront && npm install && npm run dev
cd frontend-admin && npm install && npm run dev
```

### Docker Commands
```bash
# Build all services
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f service-order

# Stop all
docker-compose down
```

### Database Migration
```bash
# Run migrations
cd infra-database
psql -h localhost -U postgres -d niaga_db -f migrations/001_initial.sql

# Seed data
psql -h localhost -U postgres -d niaga_db -f seeds/001_default_data.sql
```

---

## 📞 Support

For questions or issues:
- Check `docs/TROUBLESHOOTING.md`
- Review `docs/DEVELOPER-ONBOARDING.md`
- Create GitHub issue

---

*Last Updated: December 2024*
*Version: 1.0.0*
