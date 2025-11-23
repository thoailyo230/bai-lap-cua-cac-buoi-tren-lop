# Sơ Đồ Hệ Thống - Website Bán Xe Điện

## 1. Use Case Diagram (Tổng Quan Chức Năng)

```mermaid
graph TB
    subgraph "Actors"
        Guest[Khách Hàng<br/>Chưa Đăng Nhập]
        Customer[Khách Hàng<br/>Đã Đăng Nhập]
        Admin[Quản Trị Viên]
    end
    
    subgraph "Use Cases - Khách Hàng"
        UC1[Xem Danh Sách Sản Phẩm]
        UC2[Tìm Kiếm Sản Phẩm]
        UC3[Xem Chi Tiết Sản Phẩm]
        UC4[So Sánh Sản Phẩm]
        UC5[Đăng Ký Tài Khoản]
        UC6[Đăng Nhập]
        UC7[Thêm Vào Giỏ Hàng]
        UC8[Xem Giỏ Hàng]
        UC9[Cập Nhật Giỏ Hàng]
        UC10[Đặt Hàng]
        UC11[Xem Đơn Hàng]
        UC12[Quản Lý Thông Tin Cá Nhân]
        UC13[Chat với AI]
        UC14[Chat với Nhân Viên]
        UC15[Kiểm Tra Bảo Hành]
    end
    
    subgraph "Use Cases - Quản Trị"
        UC16[Quản Lý Sản Phẩm]
        UC17[Quản Lý Danh Mục]
        UC18[Quản Lý Thương Hiệu]
        UC19[Quản Lý Đơn Hàng]
        UC20[Xem Doanh Thu]
        UC21[Quản Lý Người Dùng]
        UC22[Gửi Email Marketing]
        UC23[Quản Lý Live Chat]
    end
    
    Guest --> UC1
    Guest --> UC2
    Guest --> UC3
    Guest --> UC4
    Guest --> UC5
    Guest --> UC6
    Guest --> UC13
    Guest --> UC15
    
    Customer --> UC1
    Customer --> UC2
    Customer --> UC3
    Customer --> UC4
    Customer --> UC7
    Customer --> UC8
    Customer --> UC9
    Customer --> UC10
    Customer --> UC11
    Customer --> UC12
    Customer --> UC13
    Customer --> UC14
    Customer --> UC15
    
    Admin --> UC16
    Admin --> UC17
    Admin --> UC18
    Admin --> UC19
    Admin --> UC20
    Admin --> UC21
    Admin --> UC22
    Admin --> UC23
```

## 2. DFD (Data Flow Diagram) - Mức 0 (Context Diagram)

```mermaid
graph LR
    Customer[Khách Hàng]
    Admin[Quản Trị Viên]
    System[Hệ Thống<br/>Bán Xe Điện]
    Payment[Payment Gateway<br/>ZaloPay]
    Email[Email Server]
    
    Customer -->|Yêu cầu xem sản phẩm| System
    Customer -->|Đặt hàng| System
    Customer -->|Thanh toán| System
    System -->|Thông tin sản phẩm| Customer
    System -->|Xác nhận đơn hàng| Customer
    
    Admin -->|Quản lý dữ liệu| System
    System -->|Báo cáo| Admin
    
    System -->|Yêu cầu thanh toán| Payment
    Payment -->|Kết quả thanh toán| System
    
    System -->|Gửi email| Email
    Email -->|Thông báo| Customer
```

## 3. Activity Diagram - Quy Trình Đặt Hàng

```mermaid
flowchart TD
    Start([Bắt Đầu]) --> Browse[Khách Hàng Duyệt Sản Phẩm]
    Browse --> Select{Chọn Sản Phẩm?}
    Select -->|Có| CheckLogin{Đã Đăng Nhập?}
    Select -->|Không| Browse
    
    CheckLogin -->|Chưa| Login[Đăng Nhập/Đăng Ký]
    Login --> AddToCart[Thêm Vào Giỏ Hàng]
    CheckLogin -->|Rồi| AddToCart
    
    AddToCart --> ViewCart[Xem Giỏ Hàng]
    ViewCart --> SelectColor[Chọn Màu Sắc]
    SelectColor --> UpdateCart{Cập Nhật Số Lượng?}
    UpdateCart -->|Có| ViewCart
    UpdateCart -->|Không| Checkout[Tiến Hành Thanh Toán]
    
    Checkout --> FillInfo[Điền Thông Tin Giao Hàng]
    FillInfo --> SelectPayment{Chọn Phương Thức<br/>Thanh Toán}
    
    SelectPayment -->|COD| CreateOrder1[Tạo Đơn Hàng]
    SelectPayment -->|Chuyển Khoản| CreateOrder2[Tạo Đơn Hàng]
    SelectPayment -->|ZaloPay| ProcessPayment[Xử Lý Thanh Toán]
    
    ProcessPayment --> PaymentResult{Kết Quả<br/>Thanh Toán?}
    PaymentResult -->|Thành Công| CreateOrder3[Tạo Đơn Hàng]
    PaymentResult -->|Thất Bại| Checkout
    
    CreateOrder1 --> SendEmail[Gửi Email Xác Nhận]
    CreateOrder2 --> SendEmail
    CreateOrder3 --> SendEmail
    
    SendEmail --> ClearCart[Xóa Giỏ Hàng]
    ClearCart --> ShowSuccess[Hiển Thị Thông Báo<br/>Thành Công]
    ShowSuccess --> End([Kết Thúc])
```

## 4. Sequence Diagram - Đặt Hàng

```mermaid
sequenceDiagram
    participant C as Customer
    participant BC as Browser/Client
    participant HC as HomeController
    participant CS as CartService
    participant PC as PaymentController
    participant OS as OrderService
    participant ES as EmailService
    participant DB as Database
    
    C->>BC: Chọn sản phẩm, thêm vào giỏ hàng
    BC->>HC: POST /cart/add
    HC->>CS: addToCart(user, productId, quantity, color)
    CS->>DB: Lưu CartItem
    DB-->>CS: CartItem saved
    CS-->>HC: Success
    HC-->>BC: Redirect to /cart
    BC-->>C: Hiển thị giỏ hàng
    
    C->>BC: Click "Thanh Toán"
    BC->>PC: GET /payment/checkout
    PC->>CS: getCartItems(user)
    CS->>DB: Query CartItems
    DB-->>CS: CartItems
    CS-->>PC: CartItems
    PC-->>BC: Render checkout page
    BC-->>C: Form thanh toán
    
    C->>BC: Điền thông tin, submit
    BC->>PC: POST /payment/checkout
    PC->>OS: createOrder(user, orderData)
    OS->>DB: Create Order
    OS->>DB: Create OrderItems
    OS->>CS: clearCart(user)
    CS->>DB: Delete CartItems
    OS->>ES: sendOrderConfirmation()
    ES-->>C: Email xác nhận
    OS-->>PC: Order created
    PC-->>BC: Redirect to order detail
    BC-->>C: Hiển thị đơn hàng
```

## 5. Class Diagram

```mermaid
classDiagram
    class User {
        -Long id
        -String username
        -String email
        -String password
        -String fullName
        -String phone
        -String address
        -Set~Role~ roles
        -Boolean enabled
        +getId()
        +getUsername()
        +getEmail()
    }
    
    class Role {
        -Long id
        -String name
        +getId()
        +getName()
    }
    
    class Product {
        -Long id
        -String name
        -String description
        -BigDecimal price
        -BigDecimal originalPrice
        -String imageUrl
        -Double rating
        -Integer stock
        -Category category
        -Brand brand
        -List~ProductImage~ images
        +getId()
        +getName()
        +getPrice()
    }
    
    class Category {
        -Long id
        -String name
        -String slug
        -Integer displayOrder
        +getId()
        +getName()
    }
    
    class Brand {
        -Long id
        -String name
        -String description
        +getId()
        +getName()
    }
    
    class ProductImage {
        -Long id
        -String imageUrl
        -Integer displayOrder
        -Product product
        +getId()
        +getImageUrl()
    }
    
    class ProductColor {
        -Long id
        -String colorName
        -String colorCode
        -String imageUrl
        -Product product
        +getId()
        +getColorName()
    }
    
    class Cart {
        -Long id
        -User user
        -List~CartItem~ items
        -LocalDateTime createdAt
        +getId()
        +getItems()
    }
    
    class CartItem {
        -Long id
        -Cart cart
        -Product product
        -Integer quantity
        -BigDecimal price
        -String selectedColor
        +getId()
        +getQuantity()
        +getSubtotal()
    }
    
    class Order {
        -Long id
        -User user
        -List~OrderItem~ items
        -BigDecimal totalAmount
        -String status
        -String paymentMethod
        -LocalDateTime createdAt
        +getId()
        +getTotalAmount()
        +getStatus()
    }
    
    class OrderItem {
        -Long id
        -Order order
        -Product product
        -Integer quantity
        -BigDecimal price
        -String selectedColor
        +getId()
        +getSubtotal()
    }
    
    class ChatMessage {
        -Long id
        -User user
        -String message
        -String response
        -String chatType
        -LocalDateTime createdAt
        +getId()
        +getMessage()
    }
    
    class ProductService {
        +getAllProducts()
        +getProductById()
        +createProduct()
        +updateProduct()
        +deleteProduct()
        +getProductsByCategory()
    }
    
    class CartService {
        +getOrCreateCart()
        +addToCart()
        +updateCartItemQuantity()
        +removeFromCart()
        +clearCart()
        +getCartTotal()
    }
    
    class OrderService {
        +createOrder()
        +getOrdersByUser()
        +updateOrderStatus()
        +getRevenueByDateRange()
    }
    
    class UserService {
        +findByUsername()
        +createUser()
        +updateUser()
        +getAllUsers()
    }
    
    class EmailService {
        +sendOrderConfirmation()
        +sendOrderStatusUpdate()
        +sendMarketingEmail()
    }
    
    class ChatService {
        +getResponse()
        +transferToStaff()
    }
    
    User "1" --> "*" Role : has
    User "1" --> "1" Cart : owns
    User "1" --> "*" Order : places
    User "1" --> "*" ChatMessage : sends
    
    Cart "1" --> "*" CartItem : contains
    CartItem "*" --> "1" Product : references
    
    Order "1" --> "*" OrderItem : contains
    OrderItem "*" --> "1" Product : references
    
    Product "*" --> "1" Category : belongs to
    Product "*" --> "1" Brand : belongs to
    Product "1" --> "*" ProductImage : has
    Product "1" --> "*" ProductColor : has
    
    ProductService ..> Product : manages
    CartService ..> Cart : manages
    CartService ..> CartItem : manages
    OrderService ..> Order : manages
    UserService ..> User : manages
    EmailService ..> Order : uses
    ChatService ..> ChatMessage : uses
```

## 6. Database Diagram (ERD)

```mermaid
erDiagram
    USERS {
        bigint id PK
        string username UK
        string email UK
        string password
        string fullName
        string phone
        string address
        boolean enabled
        datetime createdAt
        datetime updatedAt
    }
    
    ROLES {
        bigint id PK
        string name UK
    }
    
    USER_ROLES {
        bigint user_id FK
        bigint role_id FK
    }
    
    CATEGORIES {
        bigint id PK
        string name
        string slug
        int displayOrder
    }
    
    BRANDS {
        bigint id PK
        string name
        string description
    }
    
    PRODUCTS {
        bigint id PK
        string name
        text description
        decimal price
        decimal originalPrice
        string imageUrl
        double rating
        int reviewCount
        int viewCount
        int stock
        boolean featured
        bigint category_id FK
        bigint brand_id FK
    }
    
    PRODUCT_IMAGES {
        bigint id PK
        bigint product_id FK
        string imageUrl
        int displayOrder
    }
    
    PRODUCT_COLORS {
        bigint id PK
        bigint product_id FK
        string colorName
        string colorCode
        string imageUrl
        int displayOrder
        boolean isAvailable
    }
    
    CARTS {
        bigint id PK
        bigint user_id FK
        datetime createdAt
        datetime updatedAt
    }
    
    CART_ITEMS {
        bigint id PK
        bigint cart_id FK
        bigint product_id FK
        int quantity
        decimal price
        string selectedColor
    }
    
    ORDERS {
        bigint id PK
        bigint user_id FK
        decimal totalAmount
        string shippingAddress
        string phone
        string fullName
        string status
        string paymentMethod
        text notes
        datetime createdAt
        datetime updatedAt
    }
    
    ORDER_ITEMS {
        bigint id PK
        bigint order_id FK
        bigint product_id FK
        int quantity
        decimal price
        string selectedColor
    }
    
    CHAT_MESSAGES {
        bigint id PK
        bigint user_id FK
        text message
        text response
        string chatType
        datetime createdAt
    }
    
    USERS ||--o{ USER_ROLES : has
    ROLES ||--o{ USER_ROLES : assigned_to
    USERS ||--o| CARTS : owns
    USERS ||--o{ ORDERS : places
    USERS ||--o{ CHAT_MESSAGES : sends
    
    CATEGORIES ||--o{ PRODUCTS : contains
    BRANDS ||--o{ PRODUCTS : manufactures
    PRODUCTS ||--o{ PRODUCT_IMAGES : has
    PRODUCTS ||--o{ PRODUCT_COLORS : has
    PRODUCTS ||--o{ CART_ITEMS : referenced_by
    PRODUCTS ||--o{ ORDER_ITEMS : referenced_by
    
    CARTS ||--o{ CART_ITEMS : contains
    ORDERS ||--o{ ORDER_ITEMS : contains
```

## 7. Deployment Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        Browser[Web Browser<br/>Chrome/Firefox/Edge]
        Mobile[Mobile Browser]
    end
    
    subgraph "Load Balancer"
        LB[Load Balancer<br/>Nginx]
    end
    
    subgraph "Application Layer"
        App1[Spring Boot App<br/>Instance 1<br/>Port 8080]
        App2[Spring Boot App<br/>Instance 2<br/>Port 8081]
    end
    
    subgraph "Database Layer"
        DB[(MySQL Database<br/>Port 3306)]
        DBSlave[(MySQL Replica<br/>Read Only)]
    end
    
    subgraph "External Services"
        ZaloPay[ZaloPay API<br/>Payment Gateway]
        SMTP[SMTP Server<br/>Email Service]
    end
    
    subgraph "Storage"
        FileStorage[File Storage<br/>Product Images]
    end
    
    Browser --> LB
    Mobile --> LB
    LB --> App1
    LB --> App2
    
    App1 --> DB
    App1 --> DBSlave
    App2 --> DB
    App2 --> DBSlave
    
    App1 --> ZaloPay
    App2 --> ZaloPay
    App1 --> SMTP
    App2 --> SMTP
    
    App1 --> FileStorage
    App2 --> FileStorage
    
    DB -.->|Replication| DBSlave
```

## Mô Tả Chi Tiết Các Sơ Đồ

### 1. Use Case Diagram
- **Actors**: Khách hàng chưa đăng nhập, Khách hàng đã đăng nhập, Quản trị viên
- **Use Cases**: Các chức năng chính của hệ thống được phân loại theo từng actor

### 2. DFD (Data Flow Diagram)
- Mô tả luồng dữ liệu giữa các thành phần chính: Khách hàng, Admin, Hệ thống, Payment Gateway, Email Server

### 3. Activity Diagram
- Quy trình đặt hàng từ khi khách hàng duyệt sản phẩm đến khi hoàn tất đơn hàng

### 4. Sequence Diagram
- Tương tác giữa các đối tượng trong quá trình đặt hàng: Customer → Controller → Service → Database

### 5. Class Diagram
- Cấu trúc các lớp trong hệ thống, bao gồm Models và Services với các mối quan hệ

### 6. Database Diagram (ERD)
- Cấu trúc database với các bảng và mối quan hệ giữa chúng

### 7. Deployment Diagram
- Kiến trúc triển khai hệ thống với Load Balancer, Application Servers, Database, và các dịch vụ bên ngoài

