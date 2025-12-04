# Niaga Platform

> Malaysian Fabric E-Commerce Platform - Multi-Service Architecture

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTENDS                             │
├─────────────────┬─────────────────┬─────────────────────────┤
│   Storefront    │     Admin       │      Warehouse          │
│   (Next.js)     │   (Next.js)     │      (Next.js)          │
│   Port: 3000    │   Port: 3001    │      Port: 3002         │
└────────┬────────┴────────┬────────┴───────────┬─────────────┘
         │                 │                    │
         ▼                 ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                      API GATEWAY                             │
│                    (Port: 8080)                              │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│                    MICROSERVICES (Go)                        │
├──────────┬──────────┬──────────┬──────────┬─────────────────┤
│   Auth   │ Catalog  │  Order   │ Customer │   Inventory     │
│  :8001   │  :8002   │  :8004   │  :8003   │    :8005        │
├──────────┴──────────┼──────────┴──────────┼─────────────────┤
│    Notification     │    Reporting        │     Agent       │
│       :8006         │      :8007          │     :8008       │
└─────────────────────┴─────────────────────┴─────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│                     INFRASTRUCTURE                           │
├────────────┬────────────┬────────────┬────────────┬─────────┤
│ PostgreSQL │   Redis    │ Meilisearch│   MinIO    │  NATS   │
│   :5432    │   :6379    │   :7700    │   :9000    │  :4222  │
└────────────┴────────────┴────────────┴────────────┴─────────┘
```

---

## 📦 Repository Structure

| Repository | Type | Description |
|------------|------|-------------|
| `frontend-storefront` | Next.js 14 | Customer shopping website |
| `frontend-admin` | Next.js 16 | Admin dashboard |
| `frontend-warehouse` | Next.js 14 | Warehouse operations |
| `frontend-agent` | Components | Shared agent components |
| `service-auth` | Go | Authentication & RBAC |
| `service-catalog` | Go | Products, categories, designs |
| `service-customer` | Go | Customer profiles |
| `service-order` | Go | Orders & payments |
| `service-inventory` | Go | Stock management |
| `service-notification` | Go | Email/SMS/Push |
| `service-reporting` | Go | Analytics & reports |
| `service-agent` | Go | Agent/reseller system |
| `lib-common` | Go | Shared utilities |
| `database` | SQL | Migration scripts |

---

## 🚀 Quick Start

### 1. Clone All Repos
```bash
gh repo list niaga-platform --limit 50 | while read repo _; do
  gh repo clone "$repo"
done
```

### 2. Start Infrastructure
```bash
docker compose up -d postgres redis meilisearch minio nats
```

### 3. Start Backend Services
```bash
# From niaga-platform root
docker compose up -d service-auth service-catalog service-order
```

### 4. Start Frontend
```bash
cd frontend-storefront && npm install && npm run dev
cd frontend-admin && npm install && npm run dev
```

---

## 🔧 Environment Variables

### Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### Backend (.env)
```env
APP_ENV=development
APP_PORT=8001
DATABASE_URL=postgres://niaga:password@localhost:5432/niaga?sslmode=disable
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-secret-key
```

---

## 🗄️ Database Schema

### Core Tables
- `users` - Customer accounts
- `admin_users` - Admin accounts with roles
- `products` - Product catalog
- `categories` - Product categories
- `orders` - Customer orders
- `order_items` - Order line items
- `payments` - Payment records
- `inventory` - Stock levels

### Agent Tables
- `agents` - Reseller accounts
- `agent_commissions` - Commission tracking
- `referrals` - Referral links

---

## 🔐 Authentication

### JWT Flow
1. Login → `POST /api/v1/auth/login`
2. Receive access + refresh tokens
3. Include `Authorization: Bearer <token>` in requests
4. Refresh when expired → `POST /api/v1/auth/refresh`

### RBAC Roles
- `SUPER_ADMIN` - Full access
- `MANAGER` - Most features except system settings
- `STAFF_ORDERS` - Order management only
- `STAFF_PRODUCTS` - Product management only
- `AGENT` - Agent dashboard only

---

## 📡 API Endpoints

### Auth Service (8001)
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
GET    /api/v1/auth/me
```

### Catalog Service (8002)
```
GET    /api/v1/catalog/products
GET    /api/v1/catalog/products/:id
GET    /api/v1/catalog/categories
GET    /api/v1/catalog/fabric-designs
GET    /api/v1/catalog/colors
```

### Order Service (8004)
```
POST   /api/v1/orders
GET    /api/v1/orders/:id
PATCH  /api/v1/orders/:id/status
POST   /api/v1/orders/:id/payment
```

---

## 🐳 Docker Commands

```bash
# Build all services
docker build -f service-auth/Dockerfile -t service-auth .
docker build -f service-catalog/Dockerfile -t service-catalog .

# Run single service
docker run -p 8001:8001 --env-file .env service-auth

# View logs
docker logs -f service-auth
```

---

## ✅ Build Status

| Component | Status |
|-----------|--------|
| frontend-storefront | ✅ Pass |
| frontend-admin | ✅ Pass |
| frontend-warehouse | ✅ Pass |
| service-auth | ✅ Pass |
| service-catalog | ✅ Pass |
| service-order | ✅ Pass |
| service-customer | ✅ Pass |
| service-inventory | ✅ Pass |
| service-notification | ✅ Pass |
| service-reporting | ✅ Pass |
| service-agent | ✅ Pass |

---

## 📞 Support

- **GitHub**: github.com/niaga-platform
- **Docs**: See `/niaga-docs` repository
