# Exchange API

A **Exchange API** fornece serviços de conversão de moedas para o domínio `store`.  
Ela permite consultar a taxa de câmbio entre duas moedas (`from_curr` → `to_curr`), aplicando automaticamente o spread configurado e vinculando a operação ao usuário autenticado.

!!! info "Trusted layer e segurança"
    Toda requisição externa entra pelo **gateway**.  
    As rotas `/exchange/**` são **protegidas**: é obrigatório enviar `Authorization: Bearer <jwt>`.

---

## Visão geral

- **Service (`exchange-service`)**: Microserviço em **FastAPI (Python)** que consulta um **provedor externo** de câmbio (HTTP), aplica um **spread** configurável e retorna as cotações.

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