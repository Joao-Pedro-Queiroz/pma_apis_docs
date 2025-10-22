# Kubernetes Deployment – Store Project

O ambiente **Kubernetes (K8s)** do domínio `store` é responsável por **orquestrar, escalar e manter os microserviços** do projeto em execução contínua e resiliente.  
Os manifests seguem um padrão único de **modularização**, dividindo os componentes de **banco de dados**, **serviços de aplicação** e **gateway de entrada**.

---

## Estrutura e Padrão de Deploy

A infraestrutura do `store` é organizada em **múltiplos deployments e services**, um para cada componente do sistema.

1) **Banco de dados (PostgreSQL)**  
   - Mantido em um único Deployment.  
   - Configuração via **ConfigMap** e **Secret**.  
   - Exposto internamente por `ClusterIP`.  

2) **Serviços de domínio** (`account-service`, `auth-service`, `product-service`, `order-service`)  
   - Deployments independentes.  
   - Cada um exposto via `Service` tipo `ClusterIP`.  
   - Comunicação via DNS interno do cluster (`service-name.namespace.svc.cluster.local`).  

3) **Gateway-Service**  
   - Exposto via **LoadBalancer** (AWS NLB ou ALB).  
   - Responsável por receber tráfego externo e redirecionar requisições internas.

---

## Diagrama Geral

```mermaid
flowchart TB
    subgraph AWS_EKS["EKS Cluster (namespace: store)"]
        DB[(Postgres Service\nClusterIP)]
        GW["Gateway (LoadBalancer)"]
        ACC["Account-Service"]
        AUTH["Auth-Service"]
        ORD["Order-Service"]
        PROD["Product-Service"]
        EXC["Exchange-Service"]

        GW --> AUTH
        GW --> ACC
        GW --> ORD
        GW --> PROD
        GW --> EXC
        ACC --> DB
        ORD --> DB
        PROD --> DB
    end

    EXT[Internet Client]
    EXT --> GW
```

---

## Localização dos manifests

Todos os arquivos de configuração do Kubernetes estão organizados por serviço, dentro da pasta `k8s` de cada módulo:

| Componente         | Documentação                                       |
|--------------------|----------------------------------------------------|
| Account-Service    | [Account API](../accountapi/main.md)               |
| Auth-Service       | [Auth API](../authapi/main.md)                     |
| Gateway-Service    | [Gateway API](../gatewayapi/main.md)               |
| Product-Service    | [Product API](../productapi/main.md)               |
| Order-Service      | [Order API](../orderapi/main.md)                   |
---

## Aplicando os manifests

```bash
# Subir todos os componentes
kubectl apply -f k8s/postgres/
kubectl apply -f k8s/gateway-service/
kubectl apply -f k8s/account-service/
kubectl apply -f k8s/auth-service/
kubectl apply -f k8s/product-service/
kubectl apply -f k8s/order-service/

# Verificar status dos pods e services
kubectl get pods,svc -n store
```

---

## Notas operacionais

- **Namespace:** `store` (pode ser criado via `kubectl create ns store`).  
- **Banco de dados:** volume persistente opcional a ser configurado no futuro.  
- **Gateway:** exposto publicamente via **AWS Load Balancer** (NLB por padrão).  
- **Rede interna:** comunicação entre microserviços via DNS do cluster (`service-name`).  
- **Ambiente:** compatível com **EKS**, **Minikube**, ou **Kind** (ambiente local).  

---

✅ *Este documento confirma que todos os manifests Kubernetes necessários estão padronizados, versionados e prontos para deploy no cluster AWS EKS, garantindo isolamento por serviço e integração completa via o Gateway.*
