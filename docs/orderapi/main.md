# Order API

A **Order API** gerencia os pedidos do domínio `store`, permitindo criar e consultar ordens associadas ao usuário autenticado.  
Ela segue o padrão adotado no projeto: **interface** (`order`) e **service** (`order-service`) atrás do **gateway** e protegidos por **JWT**.

!!! info "Trusted layer e segurança"
    Toda requisição externa entra pelo **gateway**.  
    As rotas `/order/**` são **protegidas**: é obrigatório enviar `Authorization: Bearer <jwt>`.

---

## Visão geral

- **Interface (`order`)**: define o contrato (DTOs e Feign) consumido por outros módulos/fronts.  
- **Service (`order-service`)**: implementação REST, regras de negócio, persistência (JPA), e migrações (Flyway).  

``` mermaid
classDiagram
    namespace order {
        class OrderController {
            +create(OrderIn orderIn): OrderOut
            +findAll(): List<OrderOut>
            +findById(String id): OrderOut
        }

        class OrderIn {
            -List<OrderItemIn> items
        }

        class OrderItemIn {
            -String idProduct
            -int quantity
        }

        class OrderOut {
            -String id
            -String date
            -List<OrderItemOut> items
            -Double total
        }

        class OrderItemOut {
            -String id
            -ProductOut product
            -int quantity
            -Double total
        }
    }
    namespace order-service {
        class OrderResource {
            +create(OrderIn orderIn): OrderOut
            +findAll(): List<OrderOut>
            +findById(String id): OrderOut
        }

        class OrderService {
            +create(OrderIn orderIn): OrderOut
            +findAll(): List<OrderOut>
            +findById(String id): OrderOut
        }

        class OrderRepository {
            +save(Order order): Order
            +findAll(): List<Order>
            +findById(String id): Optional<Order>
        }

        class Order {
            -String id
            -String date
            -List~OrderItem~ items
        }

        class OrderItem {
            -String id
            -String idProduct
            -int quantity
            -Double total
        }

        class OrderModel {
            +toEntity(OrderIn orderIn): Order
            +toOut(Order order): OrderOut
        }
    }
    <<Interface>> OrderController
    OrderController ..> OrderIn
    OrderController ..> OrderOut

    <<Interface>> OrderRepository
    OrderController <|-- OrderResource
    OrderResource *-- OrderService
    OrderService *-- OrderRepository
    OrderService ..> Order
    OrderService ..> OrderModel
    OrderRepository ..> Order
    Order ..> OrderItem
    OrderOut ..> OrderItemOut
    OrderItemOut ..> ProductOut
```

## Estrutura da requisição

``` mermaid
flowchart LR
    subgraph api [Trusted Layer]
        direction TB
        gateway --> account
        gateway --> auth
        account --> db@{ shape: cyl, label: "Database" }
        auth --> account
        gateway --> product
        gateway e6@==> order:::red
        product --> db
        order e3@==> db
        order e4@==> product
    end
    internet e1@==>|request| gateway
    e1@{ animate: true }
    e3@{ animate: true }
    e4@{ animate: true }
    e6@{ animate: true }
    classDef red fill:#fcc
    click order "#order-api" "Order API"
```

## Order

``` tree
api/
    order/
        src/
            main/
                java/
                    store/
                        order/
                            OrderController.java
                            OrderIn.java
                            OrderOut.java
                            OrderItemIn.java
                            OrderItemOut.java
        pom.xml
        Jenkinsfile
```

??? info "Source"

    === "pom.xml"
        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/pom.xml"
        ```

    === "Jenkinsfile"
        ``` { .jenkinsfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/Jenkinsfile"
        ```

    === "OrderController.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/src/main/java/store/order/OrderController.java"
        ```

    === "OrderIn.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/src/main/java/store/order/OrderIn.java"
        ```

    === "OrderOut.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/src/main/java/store/order/OrderOut.java"
        ```

    === "OrderItemIn.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/src/main/java/store/order/OrderItemIn.java"
        ```

    === "OrderItemOut.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order/refs/heads/main/src/main/java/store/order/OrderItemOut.java"
        ```

<!-- termynal -->

``` { bash }
> mvn clean install
```

## Order-Service

``` tree
api/
    order-service/
        src/
            main/
                java/
                    store/
                        order/
                            Order.java
                            OrderItem.java
                            OrderApplication.java
                            OrderModel.java
                            OrderParser.java
                            OrderRepository.java
                            OrderResource.java
                            OrderService.java
                            FeignAuthInterceptor.java
                resources/
                    application.yaml
                    db/
                        migration/
                            V2025.08.29.001__create_schema.sql
                            V2025.08.29.002__create_table_order.sql
                            V2025.08.29.003__create_table_order_item.sql
        pom.xml
        Dockerfile
        Jenkinsfile
```

??? info "Source"

    === "pom.xml"
        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/pom.xml"
        ```

    === "Dockerfile"
        ``` { .dockerfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/DockerFile"
        ```

    === "Jenkinsfile"
        ``` { .jenkinsfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/Jenkinsfile"
        ```

    === "application.yaml"
        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/resources/application.yaml"
        ```

    === "Order.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/Order.java"
        ```

    === "OrderItem.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/OrderItem.java"
        ```

    === "OrderApplication.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/OrderApplication.java"
        ```

    === "OrderService.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/OrderService.java"
        ```

    === "OrderResource.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/OrderResource.java"
        ```

    === "OrderRepository.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/OrderRepository.java"

    === "FeignAuthInterceptor.java"
        ``` { .java .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/java/store/order/FeignAuthInterceptor.java"
        ```

    === "V2025.08.29.001__create_schema.sql"
        ``` { .sql .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/resources/db/migration/V2025.08.29.001__create_schema.sql"

    === "V2025.08.29.002__create_table_order.sql"
        ``` { .sql .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/resources/db/migration/V2025.08.29.002__create_table_order.sql"
        ```

    === "V2025.08.29.003__create_table_order_item.sql"
        ``` { .sql .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/order-service/refs/heads/main/src/main/resources/db/migration/V2025.08.29.003__create_table_order_item.sql"
        ```

<!-- termynal -->

``` { bash }
> mvn clean package spring-boot:run
```