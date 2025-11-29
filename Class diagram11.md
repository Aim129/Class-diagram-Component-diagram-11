```mermaid 
classDiagram
    %% --- Абстрактный пользователь ---
    class User {
      <<abstract>>
      - id: UUID
      - name: String
      - email: String
      - phone: String
      - address: Address
      - role: Role
      + register(): boolean
      + login(email, password): Session
      + updateProfile(data): void
    }
    User <|-- Client
    User <|-- Administrator

    class Client {
      - loyaltyAccount: LoyaltyAccount
      - orderHistory: List<Order>
      + addToCart(productId, qty): void
      + checkout(cart, paymentInfo): Order
      + leaveReview(productId, rating, text): Review
    }

    class Administrator {
      + createProduct(productData): Product
      + updateProduct(productId, data): void
      + deleteProduct(productId): void
      + viewAuditLogs(): List<AuditLog>
    }

    %% --- Продукты и инвентарь ---
    class Product {
      - id: UUID
      - name: String
      - description: String
      - basePrice: Money
      - categoryId: UUID
      - images: List<URL>
      + create(): boolean
      + update(data): boolean
      + delete(): boolean
      + applyDiscount(discount: Discount): Money
    }

    class Category {
      - id: UUID
      - name: String
      - parentCategoryId: UUID
      + addChildCategory(c: Category): void
    }

    %% Мульти-склад
    class Warehouse {
      - id: UUID
      - name: String
      - address: Address
      + getStock(productId): int
    }

    class InventoryItem {
      - id: UUID
      - productId: UUID
      - warehouseId: UUID
      - quantity: int
      + reserve(qty): boolean
      + release(qty): void
      + transferTo(warehouseId, qty): boolean
    }

    %% --- Заказы и корзина ---
    class Cart {
      - id: UUID
      - clientId: UUID
      - items: List<CartItem>
      - promoCode: PromoCode
      + addItem(productId, qty): void
      + removeItem(productId): void
      + applyPromo(code): boolean
      + calculateTotal(): Money
    }

    class CartItem {
      - productId: UUID
      - quantity: int
      - priceAtAdd: Money
    }

    class Order {
      - id: UUID
      - createdAt: DateTime
      - status: OrderStatus
      - clientId: UUID
      - items: List<OrderItem)
      - deliveryId: UUID
      - paymentId: UUID
      - totalAmount: Money
      + place(): boolean
      + cancel(): boolean
      + markPaid(paymentId): boolean
      + updateStatus(status): void
    }

    class OrderItem {
      - productId: UUID
      - quantity: int
      - unitPrice: Money
      - subtotal: Money
    }

    %% --- Платежи ---
    class Payment {
      <<abstract>>
      - id: UUID
      - type: PaymentType
      - amount: Money
      - status: PaymentStatus
      - createdAt: DateTime
      + process(): PaymentResult
      + refund(amount): RefundResult
    }
    Payment <|-- CardPayment
    Payment <|-- EWalletPayment

    class CardPayment {
      - cardLast4: String
      - cardBrand: String
      + process(): PaymentResult
      + refund(amount): RefundResult
    }

    class EWalletPayment {
      - walletId: String
      + process(): PaymentResult
      + refund(amount): RefundResult
    }

    %% --- Логистика / Доставка ---
    class Delivery {
      - id: UUID
      - orderId: UUID
      - address: Address
      - status: DeliveryStatus
      - courierId: UUID
      - trackingNumber: String
      - providerId: UUID
      + ship(): boolean
      + track(): TrackingInfo
      + complete(): void
    }

    class Courier {
      - id: UUID
      - name: String
      - phone: String
      - vehicleInfo: String
      + assignDelivery(deliveryId): boolean
    }

    class ShippingProviderAdapter {
      <<interface>>
      + createShipment(delivery): ShipmentResponse
      + trackShipment(trackingNumber): TrackingInfo
    }

    %% --- Промо и скидки ---
    class PromoCode {
      - code: String
      - discountType: DiscountType
      - amount: Money | float
      - validFrom: DateTime
      - validUntil: DateTime
      - usageLimit: int
      + isValid(cart): boolean
      + apply(cart): Money
    }

    class Discount {
      - id: UUID
      - productId: UUID
      - percentage: float
      - fixedAmount: Money
      - validFrom: DateTime
      - validUntil: DateTime
    }

    %% --- Отзывы и рейтинг ---
    class Review {
      - id: UUID
      - productId: UUID
      - clientId: UUID
      - rating: int
      - comment: String
      - createdAt: DateTime
    }

    %% --- Лояльность и аудит ---
    class LoyaltyAccount {
      - clientId: UUID
      - points: int
      + addPoints(amount): void
      + redeemPoints(amount): boolean
    }

    class AuditLog {
      - id: UUID
      - adminId: UUID
      - action: String
      - timestamp: DateTime
      - details: String
    }

    %% --- Ассоциации / мультипликативности (комментарии) ---
    %% Клиент 1..* Заказы
    Client "1" o-- "*" Order : "places"
    %% Заказ 1 -- * OrderItem (каждый OrderItem -> один Product)
    Order "1" o-- "*" OrderItem : "contains"
    OrderItem "*" --> "1" Product : "refers"
    %% Товар принадлежит Категории
    Product "*" --> "1" Category : "in"
    %% Product <-> InventoryItem (много в складах)
    Product "1" o-- "*" InventoryItem : "stock entries"
    Warehouse "1" o-- "*" InventoryItem : "holds"
    %% Заказ 1 -- 1 Доставка
    Order "1" o-- "1" Delivery : "has"
    Delivery "1" --> "1" Courier : "handled by"
    %% Оплата связана с заказом
    Order "1" --> "1" Payment : "paid by"
    %% Отзывы: Client 1..* Review ; Product 1..* Review
    Client "1" o-- "*" Review : "writes"
    Product "1" o-- "*" Review : "receives"

    %% --- Взаимодействия с внешними API (адаптеры) ---
    class PaymentGatewayAdapter {
      <<interface>>
      + charge(paymentInfo): PaymentResult
      + refund(transactionId, amount): RefundResult
    }
    PaymentGatewayAdapter <.. CardPayment
    PaymentGatewayAdapter <.. EWalletPayment

    %% --- Доп: фабрика продуктов (обозначение) ---
    class ProductFactory {
      + createProduct(type, data): Product
    }
```
