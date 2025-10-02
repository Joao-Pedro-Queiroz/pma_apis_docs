# Product API

A **Product API** implementa o CRUD de produtos do domínio `store`, seguindo o mesmo padrão adotado no projeto: **interface** (`product`) e **service** (`product-service`) por trás do **gateway** e protegido por **JWT**.

!!! info "Trusted layer e segurança"
    Toda requisição externa entra pelo **gateway**.  
    As rotas `/product/**` são **protegidas**: é obrigatório enviar `Authorization: Bearer <jwt>`.

---

## 1) Visão geral

- **Interface (`product`)**: define o contrato (DTOs e Feign) consumido por outros módulos/fronts.
- **Service (`product-service`)**: implementação REST, regras de negócio, persistência (JPA), e migrações (Flyway).

``` mermaid
flowchart LR
    subgraph api [Trusted Layer]
        direction TB
        gateway --> account
        gateway --> auth
        account --> db@{ shape: cyl, label: "Database" }
        auth --> account
        gateway e5@==> product:::red
        gateway e6@==> order
        product e2@==> db
        order e3@==> db
        order e4@==> product
    end
    internet e1@==>|request| gateway
    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }
    e4@{ animate: true }
    e5@{ animate: true }
    e6@{ animate: true }
    classDef red fill:#fcc
    click product "#product-api" "Product API"
```