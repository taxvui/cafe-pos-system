# Database Design

## 📊 Entity Relationship Diagram

```
User (auth) ──┐
              │
Role (rbac)   │
              ├─→ Order ──→ Table (20 bàn)
              │              │
              │              ├─→ OrderItem ──→ Product ──→ Category
              │              │
              │              ├─→ Payment (VietQR, Cash, etc)
              │              │
              │              └─→ Customer
              │
              ├─→ Shift (attendance)
              │
              └─→ AuditLog (tracking)

Product ──→ ComboProduct ←── Combo
Product ──→ Inventory (stock)
Printer ──→ PrintJob (queue)
SystemConfig (settings)
```

## 🗄️ Table Specifications

### Users Table
```sql
user (
  id: CUID (Primary Key)
  email: String (Unique) - Email đăng nhập
  password: String - Hashed password (bcrypt)
  name: String - Tên nhân viên
  avatar: String? - URL hình đại diện
  roleId: String (FK) - Tham chiếu Role
  isActive: Boolean - Active/inactive status
  lastLoginAt: DateTime? - Lần đăng nhập cuối
  createdAt: DateTime - Created timestamp
  updatedAt: DateTime - Updated timestamp
  
  Indexes: roleId, email
)
```

### Roles Table (RBAC)
```sql
role (
  id: CUID
  name: String (Unique) - "admin", "cashier", "waiter", "barista"
  description: String? - Mô tả role
  permissions: String[] - JSON array ["orders.create", "orders.view", ...]
  isActive: Boolean
  createdAt: DateTime
  updatedAt: DateTime
  
  Roles:
  - Admin: Toàn quyền hệ thống
  - Cashier (Thu ngân): Thanh toán, hoàn tiền, xem báo cáo
  - Waiter (Phục vụ): Gọi món, quản lý bàn, in bill
  - Barista: Xem order, update trạng thái, in order
)
```

### Tables Table (Sơ đồ 20 bàn)
```sql
table (
  id: CUID
  tableNumber: Int (Unique) - 1 to 20
  status: String - EMPTY | OCCUPIED | WAITING_PAYMENT | RESERVED
  capacity: Int - Sức chứa (thường 4)
  notes: String? - Ghi chú về bàn
  floorPlan: String - "main", "outdoor", "vip" (if multiple areas)
  
  // UI Layout
  posX, posY: Int - Vị trí trên sơ đồ (pixel)
  width, height: Int - Kích thước
  
  createdAt, updatedAt: DateTime
  
  Status Flow:
  EMPTY → OCCUPIED → WAITING_PAYMENT → EMPTY
            ↓
         RESERVED (đặt trước)
  
  Indexes: status, tableNumber, floorPlan
)
```

### Products Table
```sql
product (
  id: CUID
  name: String - Tên món (Cà phê đen, Trà xanh, etc)
  description: String? - Mô tả
  categoryId: String (FK) - Nhóm món
  price: Decimal(10,2) - Giá bán cho khách
  costPrice: Decimal(10,2) - Giá vốn
  image: String? - URL hình ảnh
  isActive: Boolean - Bật/tắt món
  displayOrder: Int - Thứ tự hiển thị
  createdAt, updatedAt: DateTime
  
  Indexes: categoryId, name, isActive, displayOrder
)
```

### Categories Table
```sql
category (
  id: CUID
  name: String (Unique) - Cafe, Trà, Đá xay, Topping, Bánh ngọt
  description: String?
  icon: String? - Icon tên
  displayOrder: Int - Thứ tự hiển thị
  isActive: Boolean
  createdAt, updatedAt: DateTime
)
```

### Orders Table (Core)
```sql
order (
  id: CUID
  orderNumber: String (Unique) - "ORD-20260519-001" (human readable)
  tableId: String? (FK) - NULL nếu take-away/delivery
  customerId: String? (FK) - Tên khách
  userId: String (FK) - Nhân viên tạo order
  
  orderType: String - DINE_IN | TAKE_AWAY | DELIVERY
  status: String - OPEN | PREPARING | READY | COMPLETED | CANCELLED
  
  // Financial
  subtotal: Decimal - Tổng giá
  discount: Decimal - Giảm tiền (e.g., 20000)
  discountPercent: Decimal - Giảm % (e.g., 10%)
  tax: Decimal - Thuế
  totalAmount: Decimal - Tổng thanh toán
  
  notes: String? - Ghi chú chung
  kitchenNotes: String? - Ghi chú cho bếp
  
  estimatedReadyTime: DateTime? - Dự kiến sẵn sàng
  completedAt: DateTime? - Hoàn tất lúc
  createdAt, updatedAt: DateTime
  
  Status Flow:
  OPEN → PREPARING → READY → COMPLETED
    ↓
  CANCELLED (có thể cancel bất kỳ lúc nào)
  
  Indexes: orderNumber, tableId, customerId, userId, status, orderType, createdAt
)
```

### OrderItems Table
```sql
order_item (
  id: CUID
  orderId: String (FK) - Order này thuộc về
  productId: String (FK) - Sản phẩm
  
  quantity: Int - Số lượng
  unitPrice: Decimal - Giá từng cái
  totalPrice: Decimal - quantity * unitPrice
  
  notes: String? - "ít đá, ít đường, thêm shot"
  status: String - PENDING | PREPARING | READY | SERVED | CANCELLED
  
  createdAt, updatedAt: DateTime
  
  Indexes: orderId, productId, status
)
```

### Combos Table (Khuyến mãi)
```sql
combo (
  id: CUID
  name: String - "Combo Sáng", "Happy Hour"
  description: String?
  discountType: String - PERCENTAGE | FIXED
  discountValue: Decimal - 20% hoặc 50000đ
  image: String?
  isActive: Boolean
  startDate: DateTime? - Ngày bắt đầu
  endDate: DateTime? - Ngày kết thúc
  displayOrder: Int
  createdAt, updatedAt: DateTime
  
  Many-to-Many: ComboProduct
  - Combo 1 → [Product 1, Product 2, Product 3]
)
```

### Payments Table (VietQR + Cash)
```sql
payment (
  id: CUID
  orderId: String (FK) - Tính tiền cho order này
  
  method: String - CASH | BANK_TRANSFER | E_WALLET | VIETQR
  amount: Decimal - Số tiền thanh toán
  status: String - PENDING | PROCESSING | SUCCESS | FAILED | REFUNDED
  
  // VietQR Specific (Napas VietQR API)
  vietqrId: String? (Unique) - Transaction ID từ Napas
  vietqrData: String? - Raw QR data
  qrContent: String? - URL to QR code image
  bankInfo: String? - Tài khoản ngân hàng
  
  // Refund
  refundAmount: Decimal - Tiền hoàn
  refundReason: String? - Lý do hoàn
  refundedAt: DateTime?
  
  paymentRef: String? - Reference từ payment gateway
  notes: String?
  
  createdAt, updatedAt: DateTime
  
  Indexes: orderId, status, method, vietqrId, createdAt
)
```

### Customers Table
```sql
customer (
  id: CUID
  name: String - Tên khách
  phone: String (Unique) - Số điện thoại
  email: String? - Email
  address: String? - Địa chỉ (cho delivery)
  notes: String? - Ghi chú
  
  totalOrders: Int - Số lần mua
  totalSpent: Decimal - Tổng chi tiêu
  
  isBlacklisted: Boolean - Khách hàng bị cấm
  lastOrderAt: DateTime? - Lần order cuối
  createdAt, updatedAt: DateTime
)
```

### Printers Table (Máy in nhiệt)
```sql
printer (
  id: CUID
  name: String (Unique) - "Printer Bếp 1", "Printer Quầy"
  type: String - THERMAL_58 | THERMAL_80 | NETWORK | BLUETOOTH
  connectionType: String - USB | NETWORK | BLUETOOTH
  
  // Connection Info
  address: String? - IP address (network)
  macAddress: String? - MAC address (bluetooth)
  vendorId, productId: String? - USB IDs
  
  location: String - KITCHEN | BAR | CASHIER
  status: String - ONLINE | OFFLINE
  isActive: Boolean
  
  // Printer Settings
  paperWidth: Int - 58 or 80 (mm)
  autoFeed: Boolean - Tự động cắt giấy
  fontSize: Int - 1=small, 2=normal, 3=large
  
  lastHeartbeat: DateTime?
  createdAt, updatedAt: DateTime
)
```

### PrintJobs Table (Queue in ấn)
```sql
print_job (
  id: CUID
  printerId: String (FK) - Máy in nào
  
  jobType: String - KITCHEN_TICKET | BAR_TICKET | RECEIPT | TEST
  content: String - ESC/POS raw commands hoặc data
  
  status: String - PENDING | PRINTING | SUCCESS | FAILED
  errorMessage: String? - Lỗi nếu có
  retryCount: Int - Số lần retry
  maxRetries: Int - Tối đa retry (thường 3)
  
  createdAt: DateTime
  completedAt: DateTime?
)
```

### Shifts Table (Chấm công)
```sql
shift (
  id: CUID
  userId: String (FK) - Nhân viên
  startTime: DateTime - Giờ bắt đầu ca
  endTime: DateTime? - Giờ kết thúc ca
  status: String - ACTIVE | CLOSED
  
  cashStart: Decimal - Tiền mặt lúc bắt đầu
  cashEnd: Decimal? - Tiền mặt lúc kết thúc
  cashDifference: Decimal? - Chênh lệch
  
  notes: String?
  createdAt, updatedAt: DateTime
)
```

### AuditLogs Table (Tracking)
```sql
audit_log (
  id: CUID
  userId: String? (FK) - Người thực hiện
  
  action: String - CREATE | UPDATE | DELETE | PAYMENT | REFUND
  entity: String - Order, Payment, Table, Product
  entityId: String - ID của entity
  
  changes: String? - JSON diff
  oldValues: String? - JSON old values
  newValues: String? - JSON new values
  
  ipAddress: String? - IP address
  userAgent: String? - Browser info
  
  createdAt: DateTime
  
  Indexes: userId, action, entity, entityId, createdAt
)
```

### Inventory Table (Tồn kho)
```sql
inventory (
  id: CUID
  productId: String (FK) (Unique) - Sản phẩm
  quantity: Decimal - Số lượng hiện tại
  minThreshold: Decimal - Ngưỡng cảnh báo
  unit: String - cups, ml, kg, etc
  lastRestocked: DateTime? - Lần nhập kho cuối
  createdAt, updatedAt: DateTime
)
```

### SystemConfig Table (Settings)
```sql
system_config (
  id: CUID
  key: String (Unique) - "vietqr.merchant.id", "tax.rate", etc
  value: String - JSON format
  description: String?
  updatedBy: String?
  updatedAt: DateTime
)
```

## 🔑 Key Relationships

### Order Lifecycle
```
Order (OPEN)
  ├─ OrderItem 1 (PENDING) → (PREPARING) → (READY) → (SERVED)
  ├─ OrderItem 2 (PENDING) → (PREPARING) → (READY) → (SERVED)
  └─ OrderItem N
  
  Order (OPEN) → (PREPARING) → (READY) → (COMPLETED)
         ↓
      Payment (PENDING) → (SUCCESS)
         ↓
      Table status → WAITING_PAYMENT → EMPTY
```

### Payment Flow
```
Payment
  ├─ Method: CASH
  │   └─ Status: SUCCESS (immediate)
  │
  ├─ Method: VIETQR
  │   ├─ vietqrId: "xxx" (from Napas)
  │   ├─ Status: PENDING → PROCESSING → SUCCESS
  │   └─ Webhook callback updates status
  │
  └─ Method: E_WALLET
      └─ Status: PENDING → PROCESSING → SUCCESS
```

## 📈 Scalability Features

### Indexing Strategy
```
High-frequency queries:
- Order.status (filter by status)
- Order.createdAt (time range queries)
- Payment.status (reporting)
- OrderItem.orderId (JOIN optimization)
- Product.categoryId (navigation)
- Table.status (dashboard)
- User.email (login)
- Printer.location (finding printers)
- AuditLog.createdAt (compliance)
```

### Partitioning Strategy (Future)
```
AuditLog table (very large over time):
- Partition by month: audit_log_202605, audit_log_202606, etc
- Archive old partitions to cold storage

Orders table (high volume):
- Partition by date: orders_20260519, orders_20260520, etc
- Archive closed orders annually
```

### Connection Pooling
```
PgBouncer configuration:
- Pool mode: transaction (release after each query)
- Max clients: 100
- Default pool size: 20
- Reserve pool: 5
- Timeout: 300 seconds
```

## 🔐 Data Security

### Sensitive Data
```
User.password
- Hashed with bcrypt (never store plaintext)
- Min 12 characters required

Payment.bankInfo
- Encrypted at rest (Column-level encryption)
- Masked in UI (show only last 4 digits)

Customer.phone, email
- Compliant with GDPR/local regulations
- Soft delete capability (archive)
```

### Audit Trail
```
Every financial operation is logged:
- Payment creation/update
- Refund processing
- Order modifications
- Discount application
- User login/logout

Immutable audit logs for compliance
```

## 🧪 Testing Data Seeds

### Sample Data Quantities
```
- 5 Users (admin, 2 cashiers, 1 waiter, 1 barista)
- 4 Roles (predefined)
- 20 Tables (numbered 1-20)
- 50+ Products (5 categories)
- 10 Combos (active promotions)
- 100+ Sample Orders (past 30 days)
- 1000+ AuditLogs (sample operations)
```

---

**Database Design Version**: 1.0
**Last Updated**: 2026-05-19
