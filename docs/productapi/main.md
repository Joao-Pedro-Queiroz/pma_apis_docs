# Product API

A **Product API** implementa o CRUD de produtos do domínio `store`, seguindo o mesmo padrão adotado no projeto: **interface** (`product`) e **service** (`product-service`) por trás do **gateway** e protegido por **JWT**.

!!! info "Trusted layer e segurança"
    Toda requisição externa entra pelo **gateway**.  
    As rotas `/product/**` são **protegidas**: é obrigatório enviar `Authorization: Bearer <jwt>`.

---

## 1) Visão geral

- **Interface (`product`)**: define o contrato (DTOs e Feign) consumido por outros módulos/fronts.
- **Service (`product-service`)**: implementação REST, regras de negócio, persistência (JPA), e migrações (Flyway).

```mermaid
flowchart LR
    A[internet] -->|request| G[gateway]
    G -->|/auth/*| AUTH[auth-service]
    G -->|/product/*| PROD[product-service]
    AUTH --> ACC[account-service]
    PROD --> DB[(PostgreSQL)]