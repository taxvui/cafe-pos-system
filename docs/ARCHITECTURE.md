# Cafe Harmony - System Architecture

**Version**: 0.1.0  
**Last Updated**: May 19, 2026

---

## 📐 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT LAYER (Multi-Platform)           │
├─────────────────────────────────────────────────────────────┤
│  Desktop (Windows/macOS)  │  Web Browser  │  Mobile/Tablet │
│  - Electron wrapper       │  - Progressive │  - PWA Web App  │
│  - Native performance     │  - Responsive  │  - Install able │
└──────────────┬────────────────┬────────────────┬────────────┘
               │ (WebSocket/REST API / IndexedDB)
               │ (Socket.IO Real-time Events)
               │
┌──────────────▼─────────────────────────────────────────────┐
│              FRONTEND LAYER (Next.js 14)                   │
├──────────────────────────────────────────────────────────┬─┤
│ • Next.js App Router (File-based routing)               │ │
│ • TypeScript (Type Safety)                              │ │
│ • TailwindCSS (Styling)                                 │ │
│ • shadcn/ui (Component Library)                         │ │
│ • Zustand (State Management)                            │ │
│ • TanStack Query (Server State)                         │ │
│ • Socket.IO Client (Real-time)                          │ │
│ • Service Worker (PWA/Offline)                          │ │
│ • IndexedDB (Local Cache)                               │ │
└──────────────┬─────────────────────────────────────────────┘
               │ (REST API / WebSocket)
               │
┌──────────────▼──────────────────────────────────────────────┐
│             API GATEWAY & MIDDLEWARE LAYER                 │
├──────────────────────────────────────────────────────────┬──┤
│ • Rate Limiting (express-rate-limit)                    │  │
│ • CORS Configuration                                     │  │
│ • JWT Verification Middleware                           │  │
│ • Request Logging (Pino)                                │  │
│ • Error Handling                                         │  │
│ • Request Validation (Zod)                              │  │
│ • Audit Logging                                          │  │
│ • RBAC Authorization                                     │  │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│           BUSINESS LOGIC LAYER (Services)                  │
├──────────────────────────────────────────────────────────┬──┤
│ ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │  │
│ │Auth Service │  │Order Service │  │Payment Service  │ │  │
│ ├─────────────┤  ├──────────────┤  ├─────────────────┤ │  │
│ │• JWT        │  │• Table Mgmt  │  │• VietQR Handler │ │  │
│ │• RBAC       │  │• Order CRUD  │  │• Webhook Verify │ │  │
│ │• Permissions│  │• Status Flow │  │• Payment Status │ │  │
│ └─────────────┘  └──────────────┘  └─────────────────┘ │  │
│                                                          │  │
│ ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │  │
│ │Product Srvc │  │Print Service │  │Report Service   │ │  │
│ ├─────────────┤  ├──────────────┤  ├─────────────────┤ │  │
│ │• Menu CRUD  │  │• ESC/POS Cmd │  │• Analytics      │ │  │
│ │• Pricing    │  │• Queue Mgmt  │  │• Dashboards     │ │  │
│ │• Stock      │  │• Device Ctrl │  │• Data Export    │ │  │
│ └─────────────┘  └──────────────┘  └─────────────────┘ │  │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│         DATA ACCESS LAYER (ORM - Prisma)                   │
├──────────────────────────────────────────────────────────┬──┤
│ • Query Builder                                         │  │
│ • Migration Engine                                      │  │
│ • Relation Resolution                                   │  │
│ • Type Safety                                           │  │
│ • Connection Pooling                                    │  │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│           DATA STORAGE LAYER (Databases)                   │
├──────────────────────────────────────────────────────────┬──┤
│ ┌──────────────────┐ ┌──────────────┐ ┌──────────────┐ │  │
│ │  PostgreSQL 15   │ │    Redis     │ │  IndexedDB   │ │  │
│ ├──────────────────┤ ├──────────────┤ ├──────────────┤ │  │
│ │ • Primary DB     │ │ • Session    │ │ • Offline    │ │  │
│ │ • Transactional  │ │ • Caching    │ │ • Local Sync │ │  │
│ │ • 18 Tables      │ │ • Real-time  │ │ • Queue      │ │  │
│ │ • ACID Compliant │ │ • Pub/Sub    │ │ • PWA Data   │ │  │
│ └──────────────────┘ └──────────────┘ └──────────────┘ │  │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔄 Component Interaction Flow

### 1. **Order Creation Flow**

```
User (POS Screen)
    ↓
Order Component (React)
    ↓
Zustand Store (State Update)
    ↓
TanStack Query (Mutation)
    ↓
POST /api/orders (Next.js Route)
    ↓
Middleware Chain
├─ Rate Limiting
├─ JWT Verification
├─ RBAC Authorization
└─ Request Validation (Zod)
    ↓
OrderService.create()
    ↓
Prisma ORM
    ↓
PostgreSQL (INSERT)
    ↓
Response + Socket.IO Emit
    ↓
Other Connected Clients
├─ Table Status Update
├─ Kitchen Display
└─ Real-time UI Sync
```

### 2. **VietQR Payment Flow**

```
Payment UI
    ↓
POST /api/payments/create-vietqr
    ↓
PaymentService.generateVietQR()
    ├─ Call Napas VietQR API
    ├─ Generate Dynamic QR
    └─ Store Transaction Ref
    ↓
Return QR Code to Client
    ↓
Display QR (30s timeout)
    ↓
Customer Scans & Transfers
    ↓
Bank webhook → Napas webhook
    ↓
POST /api/webhooks/vietqr
    ↓
PaymentService.verifyWebhook()
    ├─ Validate signature
    ├─ Update payment status
    └─ Update order status
    ↓
Socket.IO Emit Payment Confirmed
    ↓
Update UI (Remove QR, Show Success)
```

### 3. **Real-time Table Synchronization**

```
Admin: Merge Table 1 + 2
    ↓
TableService.mergeTables()
    ↓
Update Database
    ↓
Socket.IO Event: "table:merged"
    ↓
Connected Clients receive event
├─ POS Screen A
├─ POS Screen B
├─ Kitchen Display
└─ Reports Dashboard
    ↓
All UIs update immediately
```

---

## 🔐 Security Architecture

### Authentication Flow

```
┌─────────────────────────────────────────┐
│  User Login                             │
│  POST /api/auth/login                   │
├─────────────────────────────────────────┤
│  Credentials → bcrypt verify            │
│  ↓                                      │
│  Generate JWT Token                     │
│  ├─ Header: algorithm, type             │
│  ├─ Payload: userId, role, permissions  │
│  └─ Signature: HS256 (secret)           │
│  ↓                                      │
│  Return Token (1 hour TTL)              │
│  + Refresh Token (7 days TTL)           │
└─────────────────────────────────────────┘

Subsequent Requests:
┌─────────────────────────────────────────┐
│  Authorization: Bearer {JWT}            │
│  ↓                                      │
│  Middleware: Verify & Decode Token      │
│  ↓                                      │
│  Check Expiration                       │
│  ├─ Valid → Continue                    │
│  └─ Expired → Refresh or Re-login       │
│  ↓                                      │
│  Extract userId, role, permissions      │
│  ↓                                      │
│  RBAC Check: Required permissions       │
└─────────────────────────────────────────┘
```

### RBAC (Role-Based Access Control)

```
Roles:
├─ ADMIN
│  └─ All permissions
├─ CASHIER
│  ├─ View orders
│  ├─ Process payments
│  ├─ Print receipts
│  └─ View reports
├─ WAITER
│  ├─ Create orders
│  ├─ View table status
│  ├─ Send orders to kitchen
│  └─ Merge/split bills
└─ BARISTA
   ├─ View kitchen display
   ├─ Mark items ready
   └─ View order notes

Middleware Authorization:
Route Handler
    ↓
@RequireRole(['ADMIN', 'CASHIER'])
    ↓
Check user.role in decorator
├─ Match → Allow
└─ No Match → 403 Forbidden
```

### Data Security

```
Sensitive Fields:
├─ Passwords: bcrypt with salt rounds (10)
├─ Payment Info: VietQR transaction ID only (no card data)
├─ Customer Phone: PII protected
├─ Staff Email: PII protected

Encryption at Rest (Optional):
├─ Sensitive values in environment variables
├─ Redis password-protected
└─ PostgreSQL SSL connections

Audit Trail:
├─ All CRUD operations logged
├─ User IP & User-Agent tracked
├─ Timestamp on all changes
└─ Cannot be deleted (immutable log)
```

---

## 🏗️ Module Architecture

```
cafe-pos-system/
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (auth)/               # Auth pages (protected)
│   │   ├── (pos)/                # POS pages (protected)
│   │   ├── (admin)/              # Admin pages (protected)
│   │   ├── api/                  # API Routes
│   │   │   ├── auth/             # Authentication endpoints
│   │   │   ├── products/         # Product CRUD
│   │   │   ├── orders/           # Order management
│   │   │   ├── payments/         # Payment processing
│   │   │   ├── tables/           # Table management
│   │   │   ├── reports/          # Analytics
│   │   │   ├── webhooks/         # VietQR webhooks
│   │   │   └── printers/         # Printer management
│   │   └── layout.tsx            # Root layout
│   │
│   ├── components/               # Reusable UI components
│   │   ├── pos/                  # POS-specific
│   │   │   ├── TableFloorPlan.tsx
│   │   │   ├── OrderEntry.tsx
│   │   │   ├── PaymentModal.tsx
│   │   │   └── KitchenDisplay.tsx
│   │   ├── ui/                   # shadcn/ui components
│   │   ├── common/               # Shared components
│   │   └── layouts/              # Page layouts
│   │
│   ├── hooks/                    # Custom React hooks
│   │   ├── useAuth.ts            # Auth context hook
│   │   ├── useSocket.ts          # WebSocket hook
│   │   ├── useOfflineSync.ts     # Offline sync
│   │   └── useLocalStorage.ts    # Storage wrapper
│   │
│   ├── lib/                      # Utilities
│   │   ├── api.ts                # API client
│   │   ├── socket.ts             # Socket.IO setup
│   │   ├── db.ts                 # Prisma client
│   │   ├── auth.ts               # JWT helpers
│   │   ├── vietqr.ts             # VietQR integration
│   │   ├── printer.ts            # Printer commands
│   │   ├── storage.ts            # IndexedDB wrapper
│   │   └── validators.ts         # Zod schemas
│   │
│   ├── services/                 # Business logic
│   │   ├── auth.service.ts       # Auth logic
│   │   ├── order.service.ts      # Order CRUD
│   │   ├── payment.service.ts    # Payment processing
│   │   ├── table.service.ts      # Table management
│   │   ├── product.service.ts    # Product management
│   │   ├── printer.service.ts    # Print jobs
│   │   ├── report.service.ts     # Analytics
│   │   └── vietqr.service.ts     # VietQR API
│   │
│   ├── stores/                   # Zustand state
│   │   ├── useAuthStore.ts
│   │   ├── useOrderStore.ts
│   │   ├── useTableStore.ts
│   │   └── useUIStore.ts
│   │
│   ├── middleware/               # Express/Next middleware
│   │   ├── auth.middleware.ts    # JWT verification
│   │   ├── rbac.middleware.ts    # Role check
│   │   ├── logging.middleware.ts # Request logging
│   │   └── errorHandler.ts       # Error handling
│   │
│   ├── types/                    # TypeScript types
│   │   ├── index.ts              # Common types
│   │   ├── api.ts                # API request/response
│   │   ├── models.ts             # Database models
│   │   └── enums.ts              # Enumerations
│   │
│   └── styles/                   # Global styles
│       └── globals.css
│
├── prisma/
│   ├── schema.prisma             # Database schema
│   ├── migrations/               # Schema migrations
│   └── seed.ts                   # Database seeding
│
├── tests/
│   ├── unit/                     # Unit tests
│   │   └── services/
│   ├── integration/              # Integration tests
│   │   └── api/
│   └── e2e/                      # End-to-end tests
│       └── pos.spec.ts
│
├── public/
│   ├── manifest.json             # PWA manifest
│   ├── sw.js                     # Service worker
│   ├── images/                   # Static images
│   └── icons/                    # App icons
│
└── docker/
    ├── nginx.conf
    └── ssl/
```

---

## 📊 Database Design Principles

### 1. **Normalization**
- Third Normal Form (3NF)
- Eliminating redundancy
- Maintaining referential integrity

### 2. **Indexing Strategy**

```sql
-- Primary Keys (AUTO)
├─ All tables have PRIMARY KEY (id CUID)

-- Foreign Keys (REFERENCED)
├─ user.roleId → role.id
├─ order.tableId → table.id
├─ orderItem.orderId → order.id
└─ (All relationships indexed)

-- Query Optimization
├─ Status fields: INDEX (status)
├─ Date fields: INDEX (createdAt)
├─ Frequently joined: INDEX (userId, orderId)
└─ Full-text search: Specialized indexes

-- Composite Indexes (Multi-column)
├─ (userId, createdAt) for user's orders
├─ (tableId, status) for table state
└─ (productId, orderDate) for sales analysis
```

### 3. **Data Types**

```
├─ CUID: Unique identifiers (distributed-friendly)
├─ Decimal(10,2): Money fields (precision)
├─ DateTime: Timestamps (consistency)
├─ Enum: Status fields (type safety)
├─ Json: Flexible data (VietQR responses)
├─ String: Text fields (variable length)
└─ Int: Quantities (inventory)
```

### 4. **Cascade Behavior**

```
Order DELETED
├─ OrderItems → DELETED (Cascade)
├─ Payments → DELETED (Cascade)
└─ PrintJobs → DELETED (Cascade)

Product DELETED
├─ OrderItems → SET NULL (Soft reference)
└─ ComboProducts → DELETED (Cascade)

User DELETED
├─ Orders → SET NULL (Preserve history)
├─ Shifts → DELETED (Cascade)
└─ AuditLogs → KEPT (Audit trail)
```

---

## 🔄 Real-time Architecture (Socket.IO)

```
Server Setup:
├─ Socket.IO Server (Port 3000 + WebSocket)
├─ Socket Middleware:
│  ├─ JWT Verification
│  ├─ User identification
│  └─ Room assignment
└─ Event Handlers

Client Setup:
├─ Socket.IO Client
├─ Auto-reconnect with exponential backoff
├─ Local queue for offline events
└─ Sync on reconnect

Event Rooms:
├─ /orders           # Order updates
├─ /tables           # Table status
├─ /payments         # Payment updates
├─ /kitchen          # Kitchen display
├─ /reports          # Dashboard updates
└─ /user:{userId}    # User-specific

Event Types:
├─ order:created
├─ order:updated
├─ order:item:ready
├─ table:status-changed
├─ table:merged
├─ payment:confirmed
├─ payment:failed
├─ printer:error
└─ report:updated
```

---

## 🌐 API Design (REST)

### Endpoint Structure

```
/api/v1/
├─ /auth
│  ├─ POST /login
│  ├─ POST /logout
│  ├─ POST /refresh
│  └─ GET /me
│
├─ /products
│  ├─ GET / (list, filtered)
│  ├─ GET /:id
│  ├─ POST / (admin)
│  ├─ PUT /:id (admin)
│  └─ DELETE /:id (admin)
│
├─ /categories
│  ├─ GET /
│  ├─ POST / (admin)
│  └─ PUT /:id (admin)
│
├─ /orders
│  ├─ POST / (create)
│  ├─ GET / (list with filters)
│  ├─ GET /:id
│  ├─ PUT /:id (update items)
│  ├─ PUT /:id/status (status change)
│  ├─ POST /:id/merge (merge tables)
│  └─ POST /:id/split (split bill)
│
├─ /tables
│  ├─ GET / (floor plan)
│  ├─ PUT /:id/status
│  ├─ PUT /:id/transfer
│  └─ POST /:id/merge
│
├─ /payments
│  ├─ POST / (create)
│  ├─ POST /vietqr (generate QR)
│  ├─ GET /:id
│  ├─ POST /verify (webhook)
│  └─ POST /:id/refund
│
└─ /reports
   ├─ GET /revenue/day
   ├─ GET /revenue/month
   ├─ GET /items/bestsellers
   └─ GET /staff/performance
```

### Response Format

```json
{
  "success": true,
  "data": {...},
  "meta": {
    "timestamp": "2026-05-19T10:00:00Z",
    "requestId": "uuid"
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [{...}]
  },
  "meta": {
    "timestamp": "2026-05-19T10:00:00Z",
    "requestId": "uuid"
  }
}
```

---

## 🧪 Testing Strategy

### Unit Tests
- Service logic isolation
- Mock database layer
- Edge cases & error handling
- Coverage target: >80%

### Integration Tests
- API endpoints with real database
- Authentication flow
- Business logic workflows
- Database transactions

### E2E Tests
- Full user workflows
- UI interaction simulation
- Real browser environment
- Critical paths only

### Test Examples

```typescript
// Unit test
describe('OrderService', () => {
  it('should create order with valid items', async () => {
    // Arrange
    // Act
    // Assert
  });
});

// Integration test
describe('POST /api/orders', () => {
  it('should create order and update table status', async () => {
    // Full flow test
  });
});

// E2E test
test('User can complete full POS workflow', async () => {
  // Login → Create order → Payment → Success
});
```

---

## 🚀 Scalability Considerations

### Horizontal Scaling

```
Multiple Next.js Instances
        ↓
Load Balancer (Nginx)
        ↓
┌───────────┬───────────┬───────────┐
│ Instance1 │ Instance2 │ Instance3 │
└───────────┴───────────┴───────────┘
        ↓       ↓           ↓
        └─────PostgreSQL────┘
               (shared)
        
        └─────Redis────┘
               (shared)
```

### Connection Pooling
- PostgreSQL: PgBouncer (50 connections)
- Redis: Connection pooling (10-20)
- Next.js: Built-in connection handling

### Caching Strategy
```
Cache Layers:
1. Browser Cache (Static assets)
2. Redis Cache (API responses)
3. Database Query Cache (Prisma)
4. CDN (Images, static files)

Cache Invalidation:
├─ Time-based (5-10 minutes)
├─ Event-based (On data change)
└─ Manual (Admin clearing)
```

---

## 📈 Performance Optimization

### Frontend
- Code splitting by route
- Image optimization (WebP, lazy loading)
- CSS purging with TailwindCSS
- Compression (gzip, brotli)
- CDN for static assets

### Backend
- Database query optimization
- Connection pooling
- Redis caching
- Rate limiting
- Pagination (max 100 items per page)

### Monitoring
- Application Performance Monitoring (APM)
- Error tracking (Sentry)
- Log aggregation (CloudWatch)
- Uptime monitoring

---

## 🔒 Deployment Architecture

```
Development
└─ docker-compose.yml (Local)

Staging
├─ Docker containers
├─ AWS RDS (PostgreSQL)
├─ ElastiCache (Redis)
└─ ALB (Load Balancer)

Production
├─ Kubernetes cluster (Optional)
├─ AWS RDS Multi-AZ
├─ Redis Cluster
├─ CloudFront (CDN)
├─ WAF (Web Application Firewall)
└─ S3 (Static assets)
```

---

## 📋 Summary

| Aspect | Technology | Purpose |
|--------|-----------|---------|
| Frontend | Next.js 14 + React | Multi-platform UI |
| Styling | TailwindCSS + shadcn/ui | Responsive design |
| State | Zustand + TanStack Query | Local & server state |
| Real-time | Socket.IO | Live synchronization |
| Backend | Next.js API Routes | REST API |
| Database | PostgreSQL + Redis | Data persistence & cache |
| ORM | Prisma | Type-safe queries |
| Auth | JWT + RBAC | Secure access control |
| Testing | Jest + Playwright | Quality assurance |
| Deployment | Docker + VPS | Infrastructure |

---

**This architecture is production-ready and designed for Vietnamese coffee shop operations. It prioritizes:**

✅ **Reliability** - ACID compliance, data consistency  
✅ **Performance** - Caching, indexing, optimization  
✅ **Security** - RBAC, audit logs, encrypted transmission  
✅ **Scalability** - Horizontal scaling, connection pooling  
✅ **Maintainability** - Clean architecture, modular design  
✅ **Real-time** - Instant updates across devices  

