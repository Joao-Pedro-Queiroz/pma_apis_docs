# Exchange API

A **Exchange API** fornece serviços de conversão de moedas para o domínio `store`.  
Ela permite consultar a taxa de câmbio entre duas moedas (`from_curr` → `to_curr`), aplicando automaticamente o spread configurado e vinculando a operação ao usuário autenticado.

!!! attention
    Para consumir a API é necessário estar autenticado (JWT).

---

## Visão geral

- **exchange-service**: microserviço implementado em **FastAPI (Python)**, responsável por consultar um provedor externo de câmbio e aplicar regras de spread.  
- Ele é acessado **via gateway-service**, que faz a validação JWT e injeta o `id-account` nos headers.  
- Diferente de outros domínios (como `order` ou `product`), não existe um repositório JPA nem uma interface REST separada em Java todo o serviço roda em Python com FastAPI.

``` mermaid
classDiagram
    class ExchangeService {
        +getExchange(from: String, to: String): QuoteOut
    }

    class QuoteOut {
        -Double sell
        -Double buy
        -String date
        -String idAccount
    }
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
        gateway e1@==> exchange:::red
        gateway --> product
        gateway --> order
        product --> db
        order --> db
        order --> product
    end
    exchange e3@==> 3partyapi:::green@{label: "3rd-party API"}
    internet e2@==>|request| gateway
    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }
    classDef red fill:#fcc
    classDef green fill:#cfc
    click product "#product-api" "Product API"
```

## Exchange-Service

``` tree
api/
    exchange-service/
        app/
            __init__.py
            main.py
            auth.py
            config.py
            models.py
            clients/
                __init__.py
                rates.py
        requirements.txt
        Dockerfile
```

??? info "Source"

    === "requirements.txt"

        ``` { .txt .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/requirements.txt"
        ```

    === "Dockerfile"

        ``` { .dockerfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/DockerFile"
        ```

    === "main.py"

        ``` { .python .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/app/main.py"
        ```

    === "auth.py"

        ``` { .python .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/app/auth.py"
        ```

    === "config.py"

        ``` { .python .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/app/config.py"
        ```

    === "models.py"

        ``` { .python .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/app/models.py"
        ```

    === "clients/rates.py"

        ``` { .python .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/exchange-service/refs/heads/main/app/clients/rates.py"
        ```