# Documentação dos Microserviços

## Equipe de desenvolvedores

- João Pedro Queiroz Viana

## Estrutura atual dos microserviços

``` mermaid
flowchart LR
    subgraph api [Subnet API]
        direction TB
        gateway --> account
        gateway --> auth:::red
        gateway --> product
        gateway --> order
        gateway --> exchange
        auth --> account
        order --> product
        account --> db@{ shape: cyl, label: "Database" }
        product --> db
        order --> db
    end
    exchange e3@==> 3partyapi:::green@{label: "3rd-party API"}
    internet e2@==> |request| gateway:::orange
    e2@{ animate: true }
    e3@{ animate: true }
    classDef green fill:#cfc
    classDef orange fill:#FCBE3E
```

## Repositórios

Principal: 
[https://github.com/Joao-Pedro-Queiroz/pma.25.2](https://github.com/Joao-Pedro-Queiroz/pma.25.2)

| Microservice | Interface | Implementation |
|-|-|-|
| Account | [account](https://github.com/Joao-Pedro-Queiroz/account) | [account-service](https://github.com/Joao-Pedro-Queiroz/account-service) |
| Auth | [auth](https://github.com/Joao-Pedro-Queiroz/auth) | [auth-service](https://github.com/Joao-Pedro-Queiroz/auth-service) |
| Gateway |  | [gateway-service](https://github.com/Joao-Pedro-Queiroz/gateway-service) |
| Product | [product](https://github.com/Joao-Pedro-Queiroz/product) | [product-service](https://github.com/Joao-Pedro-Queiroz/product-service) |
| Order | [order](https://github.com/Joao-Pedro-Queiroz/order) | [order-service](https://github.com/Joao-Pedro-Queiroz/order-service) |