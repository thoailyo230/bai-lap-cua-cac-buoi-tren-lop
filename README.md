%%{init: {'theme':'neutral','themeVariables':{'fontSize':'11px'}}}%%
classDiagram
    class User{+id +username +email +phone}
    class Product{+id +name +price +stock}
    class Cart{+id +items}
    class CartItem{+qty +price}
    class Order{+id +total +status +paymentMethod}
    class OrderItem{+qty +price}
    class ProductService
    class CartService
    class OrderService
    User "1" --> "1" Cart
    User "1" --> "*" Order
    Cart "1" --> "*" CartItem
    CartItem "*" --> "1" Product
    Order "1" --> "*" OrderItem
    OrderItem "*" --> "1" Product
    ProductService ..> Product
    CartService ..> Cart
    OrderService ..> Order
