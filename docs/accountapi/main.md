# Account API

A **Account API** é responsável pela gestão das contas de usuário no domínio `store`.  
Ela realiza operações de cadastro, consulta, atualização e exclusão de contas, servindo como base para autenticação e relacionamento entre os demais serviços (auth, order, product, etc.).

!!! info "Trusted layer e segurança"
    Toda requisição externa entra pelo **gateway**.  
    As rotas `/qccount/**` são **protegidas**: é obrigatório enviar `Authorization: Bearer <jwt>`.

---

## Visão geral

- **Interface (`account`)**: define o contrato (DTOs e Feign) consumido por outros módulos/fronts.  
- **Service (`account-service`)**: implementação REST, regras de negócio, persistência (JPA), e migrações (Flyway).  

``` mermaid
classDiagram
    namespace account {
        class AccountController {
            +create(AccountIn accountIn): AccountOut
            +delete(String id): void
            +findAll(): List<AccountOut>
            +findById(String id): AccountOut
        }
        class AccountIn {
            -String name
            -String email
            -String password
        }
        class AccountOut {
            -String id
            -String name
            -String email
        }
    }
    namespace account-service {
        class AccountResource {
            +create(AccountIn accountIn): AccountOut
            +delete(String id): void
            +findAll(): List<AccountOut>
            +findById(String id): AccountOut
        }
        class AccountService {
            +create(AccountIn accountIn): AccountOut
            +delete(String id): void
            +findAll(): List<AccountOut>
            +findById(String id): AccountOut
        }
        class AccountRepository {
            +create(AccountIn accountIn): AccountOut
            +delete(String id): void
            +findAll(): List<AccountOut>
            +findById(String id): AccountOut
        }
        class Account {
            -String id
            -String name
            -String email
            -String password
            -String sha256
        }
        class AccountModel {
            +create(AccountIn accountIn): AccountOut
            +delete(String id): void
            +findAll(): List<AccountOut>
            +findById(String id): AccountOut
        }
    }
    <<Interface>> AccountController
    AccountController ..> AccountIn
    AccountController ..> AccountOut

    <<Interface>> AccountRepository
    AccountController <|-- AccountResource
    AccountResource *-- AccountService
    AccountService *-- AccountRepository
    AccountService ..> Account
    AccountService ..> AccountModel
    AccountRepository ..> AccountModel
```

## Estrutura da requisição

``` mermaid
flowchart LR
    subgraph api [Trusted Layer]
        direction TB
        account e3@==> db@{ shape: cyl, label: "Database" }
    end
    internet e1@==>|request| account:::red
    e1@{ animate: true }
    e3@{ animate: true }
    classDef red fill:#fcc
```

## Account

``` tree
api/
    account/
        src/
            main/
                java/
                    store/
                        account/
                            AccountController.java
                            AccountIn.java
                            AccountOut.java
        pom.xml
        Jenkinsfile
```

??? info "Source"

    === "pom.xml"

        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account/refs/heads/main/pom.xml"
        ```

    === "Jenkinsfile"

        ``` { .jenkinsfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account/refs/heads/main/Jenkinsfile"
        ```

    === "AccountController"

        ``` { .java title='AccountController.java' .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account/refs/heads/main/src/main/java/store/account/AccountController.java"
        ```

    === "AccountIn"

        ``` { .java title='AccountIn.java' .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account/refs/heads/main/src/main/java/store/account/AccountIn.java"
        ```

    === "AccountOut"

        ``` { .java title='AccountOut.java' .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account/refs/heads/main/src/main/java/store/account/AccountOut.java"
        ```

<!-- termynal -->

``` { bash }
> mvn clean install
```

## Account-Service

``` tree
api/
    account-service/
        src/
            main/
                java/
                    store/
                        account/
                            Account.java
                            AccountApplication.java
                            AccountModel.java
                            AccountParser.java
                            AccountRepository.java
                            AccountResource.java
                            AccountService.java
                resources/
                    application.yaml
                    db/
                        migration/
                            V2025.08.29.001__create_schema.sql
                            V2025.08.29.002__create_table_account.sql
                            V2025.09.02.001__create_index_email.sql
        pom.xml
        Dockerfile
        Jenkinsfile
```

??? info "Source"

    === "pom.xml"

        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/pom.xml"
        ```

    === "Dockerfile"

        ``` { .dockerfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/DockerFile"
        ```

    === "Jenkinsfile"

        ``` { .jenkinsfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/Jenkinsfile"
        ```

    === "application.yaml"

        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/resources/application.yaml"
        ```

    === "Account.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/Account.java"
        ```

    === "AccountApplication.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/AccountApplication.java"
        ```

    === "AccountModel.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/AccountModel.java"
        ```

    === "AccountParser.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/AccountParser.java"
        ```

    === "AccountRepository.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/AccountRepository.java"
        ```

    === "AccountResource.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/AccountResource.java"
        ```

    === "AccountService.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/java/store/account/AccountService.java"
        ```

    === "V2025.08.29.001__create_schema.sql"

        ``` { .sql .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/resources/db/migration/V2025.08.29.001__create_schema.sql"
        ```

    === "V2025.08.29.002__create_table_account.sql"

        ``` { .sql .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/resources/db/migration/V2025.08.29.002__create_table_account.sql"
        ```

    === "V2025.09.02.001__create_index_email.sql"

        ``` { .sql .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/account-service/refs/heads/main/src/main/resources/db/migration/V2025.09.02.001__create_index_email.sql"
        ```

<!-- termynal -->

``` { bash }
> mvn clean package spring-boot:run
```