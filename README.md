# Cafe Harmony - Professional POS System for Vietnamese Coffee Shops

[English](#english) | [Tiếng Việt](#tiếng-việt)

---

## English

### ☕ Overview

**Cafe Harmony** is a modern, production-ready Point of Sale (POS) system designed specifically for Vietnamese coffee shops. Built with cutting-edge technology, it provides a complete solution for managing tables, orders, payments, and operations in a multi-platform environment.

### 🎯 Key Features

#### Core POS Management
- ✅ **20 Table Management** - Visual floor plan with drag-and-drop
- ✅ **Multi-Order Types** - Dine-in, Take-away, Delivery
- ✅ **Merge/Split Tables** - Complex billing scenarios
- ✅ **Real-time Updates** - Instant synchronization across devices

#### Ordering System
- ✅ **Smart Menu** - 5+ categories, quick search
- ✅ **Combo Bundles** - Pre-configured packages
- ✅ **Custom Notes** - "ít đá", "thêm shot", "ít đường"
- ✅ **Upsell Suggestions** - Automatic recommendations

#### Payment Processing
- ✅ **Multiple Methods** - Cash, Bank Transfer, E-wallet
- ✅ **VietQR Dynamic QR** - Napas integration, auto-verification
- ✅ **Webhook Support** - Real-time payment confirmation
- ✅ **Receipt Printing** - Thermal printer support

#### Kitchen Operations
- ✅ **Kitchen Display** - Real-time order status
- ✅ **Thermal Printing** - ESC/POS (58mm, 80mm)
- ✅ **Network Printers** - Multi-device printing
- ✅ **Bluetooth Support** - Android tablet printers

#### Reporting & Analytics
- ✅ **Revenue Dashboard** - Daily/monthly KPIs
- ✅ **Bestselling Items** - Real-time analytics
- ✅ **Staff Performance** - Individual metrics
- ✅ **Data Export** - CSV, PDF formats

#### Security & Compliance
- ✅ **RBAC Authorization** - 4 roles, 19 permissions
- ✅ **Audit Logging** - Complete activity trail
- ✅ **JWT Authentication** - Secure token-based auth
- ✅ **Rate Limiting** - DDoS protection

### 📱 Multi-Platform Support

| Platform | Support | Features |
|----------|---------|----------|
| **Desktop (Windows/macOS)** | ✅ Web + Electron | Full features, offline mode |
| **iPad** | ✅ PWA | Responsive, installable |
| **Android Tablet** | ✅ PWA | Bluetooth printer support |
| **Web Browser** | ✅ Chrome, Firefox, Safari | Responsive UI |
| **Mobile Phone** | ✅ PWA | Limited features, responsive |

### 🏗️ Architecture

```
Frontend (Next.js 14 + React 18)
├─ TypeScript for type safety
├─ TailwindCSS for responsive design
├─ Zustand for state management
├─ TanStack Query for server state
├─ Socket.IO for real-time sync
└─ PWA with offline support

Backend (Next.js API Routes)
├─ JWT authentication
├─ RBAC authorization
├─ VietQR payment integration
├─ Thermal printer support
└─ WebSocket real-time

Database
├─ PostgreSQL (primary)
├─ Redis (cache & real-time)
└─ IndexedDB (client-side offline)
```

### 🛠️ Technology Stack

| Category | Technology | Version |
|----------|-----------|---------|
| **Frontend** | Next.js | 14.0.0 |
| **UI Library** | React | 18.2.0 |
| **Styling** | TailwindCSS | 3.3.0 |
| **Components** | shadcn/ui | Latest |
| **State** | Zustand | 4.4.1 |
| **Server State** | TanStack Query | 5.0.0 |
| **Real-time** | Socket.IO | 4.7.0 |
| **Database** | PostgreSQL | 15 |
| **Cache** | Redis | 7 |
| **ORM** | Prisma | 5.3.0 |
| **Auth** | JWT | - |
| **Validation** | Zod | 3.22.0 |
| **Testing** | Jest + Playwright | Latest |
| **Deployment** | Docker | Latest |

### 🚀 Quick Start

#### Prerequisites
- Node.js 18+
- Docker & Docker Compose
- PostgreSQL 15
- Redis 7

#### Setup (3 minutes)

```bash
# 1. Clone repository
git clone https://github.com/taxvui/cafe-pos-system.git
cd cafe-pos-system

# 2. Install dependencies
npm install

# 3. Copy environment configuration
cp .env.example .env.local

# 4. Start Docker services
docker-compose up -d

# 5. Run database migrations
npm run db:migrate

# 6. Seed sample data
npm run db:seed

# 7. Start development server
npm run dev

# 8. Open browser
# http://localhost:3000
```

#### Login Credentials (Demo)

```
Admin:   admin@cafeharmony.local / admin123
Cashier: cashier@cafeharmony.local / cashier123
Waiter:  waiter@cafeharmony.local / waiter123
Barista: barista@cafeharmony.local / barista123
```

### 📁 Project Structure

```
cafe-pos-system/
├── src/
│   ├── app/                 # Next.js App Router
│   ├── components/          # Reusable components
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Utilities & helpers
│   ├── services/           # Business logic
│   ├── stores/             # Zustand stores
│   ├── middleware/         # API middleware
│   ├── types/              # TypeScript types
│   └── styles/             # Global styles
├── prisma/
│   ├── schema.prisma       # Database schema
│   └── seed.js             # Sample data
├── docker-compose.yml      # Development environment
├── Dockerfile              # Production image
├── package.json            # Dependencies
└── README.md              # This file
```

### 🗄️ Database Schema

**18 Production-Ready Tables**:

```
Authentication
├─ User
├─ Role (with RBAC)
└─ Permission

Floor Plan
└─ Table (20 tables)

Products
├─ Category
├─ Product
├─ Combo
└─ ComboProduct

Orders
├─ Order
└─ OrderItem

Payments
└─ Payment (VietQR ready)

Operations
├─ Printer
├─ PrintJob
├─ Inventory
└─ Shift

Customers
└─ Customer

Compliance
├─ AuditLog
└─ SystemConfig
```

### 🔐 Security Features

- ✅ **JWT Tokens** - Secure, stateless authentication
- ✅ **RBAC** - 4 roles with 19 granular permissions
- ✅ **Bcrypt Passwords** - Industry-standard hashing
- ✅ **Audit Logging** - Complete activity trail
- ✅ **Rate Limiting** - API protection
- ✅ **CORS** - Cross-origin control
- ✅ **HTTPS** - TLS/SSL support
- ✅ **Input Validation** - Zod schema validation

### 📊 API Endpoints (40+ endpoints)

```
Authentication
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
GET    /api/auth/me

Orders
POST   /api/orders
GET    /api/orders
GET    /api/orders/:id
PUT    /api/orders/:id
POST   /api/orders/:id/merge
POST   /api/orders/:id/split

Payments
POST   /api/payments
POST   /api/payments/vietqr
GET    /api/payments/:id
POST   /api/webhooks/vietqr

Tables
GET    /api/tables
PUT    /api/tables/:id/status
PUT    /api/tables/:id/transfer

Products
GET    /api/products
POST   /api/products
PUT    /api/products/:id
DELETE /api/products/:id

Reports
GET    /api/reports/revenue/day
GET    /api/reports/revenue/month
GET    /api/reports/bestsellers
GET    /api/reports/staff-performance
```

### 🧪 Testing

```bash
# Unit tests
npm run test

# Watch mode
npm run test:watch

# Coverage
npm run test:coverage

# E2E tests
npm run test:e2e
```

### 🐳 Docker Commands

```bash
# Start all services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f app

# Rebuild image
docker-compose build

# Execute migration
docker-compose exec app npm run db:migrate

# Seed data
docker-compose exec app npm run db:seed

# Database shell
docker-compose exec postgres psql -U postgres -d cafe_pos_dev
```

### 📚 Documentation

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System design, component interactions
- **[DATABASE.md](docs/DATABASE.md)** - Schema documentation
- **[API.md](docs/API.md)** - API reference guide
- **[DEPLOYMENT.md](docs/DEPLOYMENT.md)** - Production deployment guide

### 🚀 Deployment

#### VPS Deployment (Linux)

```bash
# 1. SSH to server
ssh root@your-server.com

# 2. Install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose git nodejs npm -y

# 3. Clone repository
git clone https://github.com/yourusername/cafe-pos-system.git
cd cafe-pos-system

# 4. Configure environment
cp .env.example .env.production
# Edit .env.production with production settings

# 5. Start services
docker-compose up -d

# 6. Setup reverse proxy (Nginx)
# See DEPLOYMENT.md for detailed instructions
```

#### Docker Hub Deployment

```bash
# Build image
docker build -t youruser/cafe-pos:latest .

# Push to Docker Hub
docker push youruser/cafe-pos:latest

# Pull on server
docker pull youruser/cafe-pos:latest
docker run -d -p 3000:3000 youruser/cafe-pos:latest
```

### 🔗 VietQR Integration

#### Setup Steps

1. **Register with Napas VietQR**
   - Visit https://vietqr.io
   - Create account and get API credentials
   - Store API key securely in `.env.production`

2. **Configure Webhook**
   - Set webhook URL in .env: `PAYMENT_WEBHOOK_URL`
   - Webhook receives payment confirmation from Napas
   - Auto-updates order payment status

3. **Generate Dynamic QR**
   ```javascript
   // Backend automatically generates QR on payment creation
   POST /api/payments/vietqr
   {
     orderId: "order-uuid",
     amount: 500000
   }
   
   // Response: QR code image + transaction reference
   ```

4. **Payment Confirmation**
   ```
   Customer scans QR → Bank transfer
   ↓
   Napas webhook → Your server
   ↓
   Auto-verify & update status
   ↓
   Real-time UI update (Socket.IO)
   ```

### 🖨️ Thermal Printer Setup

#### Supported Printers

- ✅ **Thermal 58mm** - Standard receipt printer
- ✅ **Thermal 80mm** - Wide format receipts
- ✅ **Network Printers** - LAN-connected via IP:Port
- ✅ **Bluetooth** - Android tablet support
- ✅ **WebUSB** - Direct browser connection

#### Configuration

```javascript
// Admin panel → Printer Settings
{
  name: "Kitchen Printer",
  type: "THERMAL_58MM",
  ipAddress: "192.168.1.100",
  port: 9100,
  purpose: "KITCHEN"
}
```

#### Automatic Printing

```javascript
// Auto-print on order confirmation
order:created → Generate ESC/POS
↓
Find active printers by purpose
↓
Queue print jobs
↓
Send to network printer
↓
Real-time status updates
```

### 📞 Support & Contributing

- **Issues**: [GitHub Issues](https://github.com/taxvui/cafe-pos-system/issues)
- **Discussions**: [GitHub Discussions](https://github.com/taxvui/cafe-pos-system/discussions)
- **Pull Requests**: Welcome!

### 📄 License

MIT License - See LICENSE.md

### 👨‍💼 Author

**Tạ Xuân Vũ** - Senior Fullstack Architect  
Email: taxvui@example.com  
GitHub: [@taxvui](https://github.com/taxvui)

---

## Tiếng Việt

### ☕ Giới Thiệu

**Cafe Harmony** là hệ thống Điểm Bán Hàng (POS) hiện đại, sẵn sàng cho production, được thiết kế đặc biệt cho các quán cà phê Việt Nam. Xây dựng với công nghệ tiên tiến nhất, nó cung cấp giải pháp hoàn chỉnh để quản lý bàn, đơn hàng, thanh toán và hoạt động kinh doanh trên nhiều nền tảng.

### 🎯 Tính Năng Chính

#### Quản Lý POS
- ✅ **Quản Lý 20 Bàn** - Sơ đồ trực quan với drag-and-drop
- ✅ **Nhiều Kiểu Đơn** - Tại chỗ, Mang về, Giao hàng
- ✅ **Gộp/Tách Bàn** - Tính toán hóa đơn phức tạp
- ✅ **Cập Nhật Real-time** - Đồng bộ tức thời trên nhiều thiết bị

#### Gọi Món
- ✅ **Menu Thông Minh** - 5+ danh mục, tìm kiếm nhanh
- ✅ **Combo Dạo** - Gói combo được cấu hình sẵn
- ✅ **Ghi Chú Tùy Chỉnh** - "ít đá", "thêm shot", "ít đường"
- ✅ **Gợi Ý Upsell** - Gợi ý tự động

#### Thanh Toán
- ✅ **Nhiều Phương Thức** - Tiền mặt, Chuyển khoản, Ví điện tử
- ✅ **VietQR Động** - Tích hợp Napas, xác minh tự động
- ✅ **Webhook Support** - Xác nhận thanh toán real-time
- ✅ **In Hóa Đơn** - Hỗ trợ máy in nhiệt

#### Quản Lý Bếp
- ✅ **Màn Hình Bếp** - Trạng thái đơn real-time
- ✅ **In Nhiệt** - ESC/POS (58mm, 80mm)
- ✅ **Máy In Mạng** - In từ nhiều thiết bị
- ✅ **Hỗ Trợ Bluetooth** - Máy in Android

#### Báo Cáo & Phân Tích
- ✅ **Bảng Điều Khiển Doanh Thu** - KPI hàng ngày/tháng
- ✅ **Món Bán Chạy** - Phân tích real-time
- ✅ **Hiệu Suất Nhân Viên** - Chỉ số cá nhân
- ✅ **Xuất Dữ Liệu** - Định dạng CSV, PDF

#### Bảo Mật & Compliance
- ✅ **Kiểm Soát RBAC** - 4 vai trò, 19 quyền hạn
- ✅ **Ghi Nhật Ký Audit** - Theo dõi hoạt động hoàn chỉnh
- ✅ **Xác Thực JWT** - Xác thực dựa trên token an toàn
- ✅ **Giới Hạn Tốc Độ** - Bảo vệ DDoS

### 💻 Hỗ Trợ Đa Nền Tảng

| Nền Tảng | Hỗ Trợ | Tính Năng |
|----------|--------|----------|
| **Desktop (Windows/macOS)** | ✅ Web + Electron | Đầy đủ, chế độ offline |
| **iPad** | ✅ PWA | Responsive, có thể cài đặt |
| **Android Tablet** | ✅ PWA | Hỗ trợ máy in Bluetooth |
| **Trình Duyệt Web** | ✅ Chrome, Firefox, Safari | Giao diện responsive |
| **Điện Thoại** | ✅ PWA | Tính năng hạn chế, responsive |

### 🔧 Cài Đặt Nhanh (3 phút)

```bash
# 1. Clone repository
git clone https://github.com/taxvui/cafe-pos-system.git
cd cafe-pos-system

# 2. Cài đặt dependencies
npm install

# 3. Copy cấu hình
cp .env.example .env.local

# 4. Khởi chạy Docker
docker-compose up -d

# 5. Chạy migration
npm run db:migrate

# 6. Seed dữ liệu
npm run db:seed

# 7. Khởi chạy dev
npm run dev

# 8. Mở trình duyệt
# http://localhost:3000
```

### 👤 Tài Khoản Demo

```
Admin:   admin@cafeharmony.local / admin123
Cashier: cashier@cafeharmony.local / cashier123
Waiter:  waiter@cafeharmony.local / waiter123
Barista: barista@cafeharmony.local / barista123
```

### 🚀 Triển Khai Production

#### Trên VPS Linux

```bash
# SSH vào server
ssh root@your-server.com

# Cài đặt Docker
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose -y

# Clone project
git clone https://github.com/yourusername/cafe-pos-system.git
cd cafe-pos-system

# Cấu hình môi trường
cp .env.example .env.production
# Chỉnh sửa .env.production

# Khởi chạy
docker-compose up -d

# Xem logs
docker-compose logs -f app
```

### 📞 Hỗ Trợ

- **Issues**: [GitHub Issues](https://github.com/taxvui/cafe-pos-system/issues)
- **Thảo Luận**: [GitHub Discussions](https://github.com/taxvui/cafe-pos-system/discussions)
- **Email**: support@cafeharmony.local

### 📄 Giấy Phép

MIT License

---

**Made with ❤️ for Vietnamese Coffee Shops**
