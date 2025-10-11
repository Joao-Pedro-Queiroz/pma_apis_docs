# Auth API

A **Auth API** é responsável pela autenticação de usuários e geração de tokens JWT utilizados por todos os demais microserviços do domínio `store`.  
Ela valida credenciais, emite tokens de acesso e garante a integridade das comunicações dentro da arquitetura de microsserviços.

!!! info "Trusted layer e segurança"
    O acesso à Auth API é realizado via **Gateway**.  
    Após o login, o **JWT** é retornado ao cliente e utilizado nas demais requisições aos serviços protegidos (`account`, `order`, `product`, `exchange`).

---

## Visão geral

- **Service (`auth-service`)**: Implementado em **Spring Boot (Java)**.  
  Expõe endpoints públicos para autenticação e registro de usuários.  
  Integra-se com o `account-service` para validação de usuários e gera tokens JWT assinados com a chave definida em `JWT_SECRET_KEY`.

- **Fluxo de autenticação**  
  1. O cliente envia `email` e `password` para `/auth/login`.  
  2. O Auth API valida as credenciais no `account-service`.  
  3. Em caso de sucesso, é gerado e retornado o `token JWT`.  
  4. O `gateway-service` utiliza esse token para validar requisições futuras e injetar o `id-account` nos headers.

``` mermaid
classDiagram
    namespace auth {
        class AuthController {
            +register(RegisterIn RegisterIn): TokenOut
            +login(LoginIn loginIn): TokenOut
        }
        class RegisterIn {
            -String name
            -String email
            -String password
        }
        class LoginIn {
            -String name
            -String email
        }
        class TokenOut {
            -String token
        }
        class SolveOut {
            -String idAccount
        }
    }
    namespace auth-service {
        class AuthResource {
            +register(RegisterIn RegisterIn) TokenOut
            +login(LoginIn loginIn) TokenOut
        }
        class AuthService {
            +register(Register) Regiter
            +login(LoginIn loginIn) String
        }
        class Register {
            -String id
            -String name
            -String email
            -String password
        }
    }
    <<Interface>> AuthController
    AuthController ..> RegisterIn
    AuthController ..> LoginIn
    AuthController ..> TokenOut

    AuthController <|-- AuthResource
    AuthResource *-- AuthService
    AuthService ..> Register
```

## Estrutura da requisição

``` mermaid
flowchart LR
    subgraph api [Trusted Layer]
        direction TB
        gateway --> account
        gateway --> others
        gateway e4@==> auth:::red
        auth e2@==> account
        account --> db@{ shape: cyl, label: "Database" }
        others --> db
    end
    internet e1@==>|request| gateway:::orange
    e1@{ animate: true }
    e2@{ animate: true }
    e4@{ animate: true }
    classDef red fill:#fcc
    classDef orange fill:#FCBE3E
```

## Auth

``` tree
api
    auth
        src
            main
                java
                    store
                        auth
                            AuthController.java
                            LoginIn.java
                            RegisterIn.java
                            TokenOut.java
        pom.xml
        Jenkinsfile
```

??? info "Source"

    === "pom.xml"

        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth/refs/heads/main/pom.xml"
        ```

    === "Jenkinsfile"

        ``` { .jenkinsfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth/refs/heads/main/Jenkinsfile"
        ```

    === "AuthController.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth/refs/heads/main/src/main/java/store/auth/AuthController.java"
        ```

    === "LoginIn.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth/refs/heads/main/src/main/java/store/auth/LoginIn.java"
        ```

    === "RegisterIn.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth/refs/heads/main/src/main/java/store/auth/RegisterIn.java"
        ```

    === "TokenOut.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth/refs/heads/main/src/main/java/store/auth/TokenOut.java"
        ```

<!-- termynal -->

``` { bash }
> mvn clean install
```

## Auth-Service

``` tree
api
    auth-service
        src
            main
                java
                    store
                        auth
                            AuthApplication.java
                            AuthResource.java
                            AuthService.java
                            JwtService.java
            resources
                application.yaml
        pom.xml
        Dockerfile
        Jenkinsfile
```

??? info "Source"

    === "pom.xml"

        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/pom.xml"
        ```

    === "Dockerfile"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/Dockerfile"
        ```

    === "Jenkinsfile"

        ``` { .jenkinsfile .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/Jenkinsfile"
        ```

    === "application.yaml"

        ``` { .yaml .copy .select linenums="1" }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/src/main/resources/application.yaml"
        ```

    === "AuthApplication.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/src/main/java/store/auth/AuthApplication.java"
        ```

    === "AuthResource.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/src/main/java/store/auth/AuthResource.java"
        ```

    === "AuthService.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/src/main/java/store/auth/AuthService.java"
        ```

    === "JwtService.java"

        ``` { .java .copy .select linenums='1' }
        --8<-- "https://raw.githubusercontent.com/Joao-Pedro-Queiroz/auth-service/refs/heads/main/src/main/java/store/auth/JwtService.java"
        ```

<!-- termynal -->

``` { bash }
> mvn clean package spring-boot:run
```