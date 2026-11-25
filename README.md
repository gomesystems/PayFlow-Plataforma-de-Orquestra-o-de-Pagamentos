# PayFlow-Plataforma-de-Orquestra-o-de-Pagamentos
A ideia é demonstrar o desenvolvimento .NET, mas também arquitetura de software, escalabilidade, resiliência, mensageria, consistência financeira, segurança e observabilidade.

## Overview

PayFlow is a reference architecture for payment processing, focused on scalability, resilience, security, consistency and observability.

## Architecture & Technologies

- C# / .NET
- Domain-Driven Design (DDD)
- Clean Architecture
- Microservices
- RabbitMQ and Event-Driven Architecture
- SQL Server / PostgreSQL
- Redis
- Transactional Outbox and Inbox
- Idempotency
- Saga Pattern
- Docker and Kubernetes
- OpenTelemetry and Grafana
- SonarCloud and Checkmarx
- CI/CD with GitHub Actions

## Key Engineering Challenges

- Preventing duplicate payment processing.
- Handling distributed transactions and eventual consistency.
- Building resilient integrations with external payment providers.
- Maintaining financial consistency through double-entry bookkeeping.
- Designing for horizontal scalability.
- Implementing observability across distributed services.

## Project Purpose

This is an educational and architectural reference project. It uses simulated payment providers and does not process real financial transactions or cardholder data.

**Status:** Architecture design and implementation roadmap.




## 1. O problema que vamos resolver
Imagine uma empresa que precisa processar milhões de pagamentos por mês, utilizando diferentes provedores de pagamento.

## O sistema precisa:
- Receber pagamentos via cartão, PIX e outros meios.
- Evitar cobranças duplicadas.
- Processar pagamentos de forma assíncrona.
- Integrar com diferentes adquirentes e gateways.
- Controlar o ciclo de vida das transações.
- Garantir rastreabilidade financeira.
- Suportar indisponibilidade de serviços externos.
- Escalar componentes independentemente.
- Manter consistência entre pagamentos, transações e lançamentos financeiros.
Premissa de projeto: vamos desenhar uma plataforma preparada para 1.000 transações por segundo em picos, com possibilidade de expansão horizontal. Esse número será uma meta de dimensionamento, não uma capacidade já comprovada.

## 2. Arquitetura proposta
A arquitetura será baseada em microsserviços, DDD, Clean Architecture, mensageria e processamento orientado a eventos.

<img width="1219" height="835" alt="image" src="https://github.com/user-attachments/assets/ff98c383-54f7-433b-9cb8-f024f4d45060" />

## Diagrama
<img width="1602" height="1256" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/24606f46-d977-4c70-bf58-49e37460ff06" />

O diagrama representa a visão lógica. Na implementação, cada serviço terá limites de domínio, persistência e implantação próprios.


## 3. Tecnologias utilizadas
Camada	Tecnologia	Aplicação
Back-end	C# / .NET	APIs e serviços
Arquitetura	DDD + Clean Architecture	Organização e domínio
Comunicação	REST	Comunicação síncrona
Mensageria	RabbitMQ	Processamento assíncrono
Eventos	Kafka (evolução)	Distribuição de eventos em alto volume
Banco de dados	SQL Server / PostgreSQL	Persistência transacional
Cache	Redis	Idempotência e consultas rápidas
Containerização	Docker	Empacotamento
Orquestração	Kubernetes	Escalabilidade horizontal
Segurança	OAuth2 / JWT	Autenticação e autorização
Qualidade	SonarCloud	Qualidade de código
Segurança de código	Checkmarx	Análise estática de vulnerabilidades
Observabilidade	OpenTelemetry + Grafana	Métricas, logs e traces
Testes	xUnit	Testes automatizados
CI/CD	GitHub Actions	Build, testes e deploy

Não precisamos começar com todas essas tecnologias. RabbitMQ será o broker principal do MVP; Kafka ficará documentado como alternativa para cenários específicos de streaming e distribuição de eventos.


## 4. Microsserviços e responsabilidades 

1. API Gateway

Ponto de entrada, autenticação, rate limiting e roteamento.


2. Payment Service

Criação de pagamentos, validação, idempotência e controle de status.


3. Payment Router

Seleção do provedor, regras de roteamento e fallback.


4. Acquirer Integration

Adaptadores para comunicação com adquirentes e gateways externos.


5. Ledger Service

Registro financeiro imutável e partidas dobradas.


6. Notification Service

Notificações de aprovação, recusa e atualização de pagamento.


7. Reconciliation Service

Conciliação entre registros internos e retornos dos provedores.


## 5. Fluxo de processamento de um pagamento
Vamos considerar uma compra de R$ 150,00 no cartão.
1. Recebimento
   
   O cliente envia uma solicitação para POST /api/payments, com identificador de idempotência.
2. Validação
   
   O Payment Service valida os dados, o cliente, o valor e a chave de idempotência.
3. Registro inicial
   
   A transação é persistida com status Pending.
4. Publicação do evento
   
   O evento PaymentRequested é publicado usando Transactional Outbox.
5. Roteamento
   
   O Payment Router escolhe o provedor de pagamento conforme regras de negócio e disponibilidade.
6. Autorização
   
   O adaptador envia a solicitação ao provedor externo e recebe o resultado.
7. Atualização
   
   O pagamento é atualizado para Authorized ou Declined, considerando também retornos assíncronos.
8. Registro financeiro
   
   O Ledger Service registra os lançamentos correspondentes, sem alterar registros financeiros anteriores.
9. Notificação
   
   O cliente recebe o resultado do pagamento por webhook ou outro canal.
## 6. Como garantir escala e confiabilidade
Aqui está a parte mais importante do System Design: não basta criar microsserviços; precisamos resolver os problemas que aparecem quando o volume aumenta.
Problema	Solução arquitetural
Pagamento duplicado	Idempotency Key
Mensagem perdida	Transactional Outbox
Evento processado duas vezes	Inbox e consumidores idempotentes
Provedor indisponível	Retry, timeout e Circuit Breaker
Sobrecarga	Rate Limiting e Backpressure
Alto volume	Escalabilidade horizontal
Falha parcial	Saga e compensação
Divergência financeira	Ledger e conciliação
Consultas pesadas	Cache e read models
Dificuldade de diagnóstico	Logs, métricas e distributed tracing

Um ponto importante: não devemos prometer processamento exatamente uma vez entre serviços e provedores externos. A arquitetura deve trabalhar com entrega pelo menos uma vez, operações idempotentes e reconciliação para tratar incertezas.

## 7. Modelo de dados inicial
Entidades principais:
- Payment: identificação, valor, moeda e status.
- PaymentAttempt: tentativas de autorização.
- PaymentProvider: provedores integrados.
- IdempotencyRecord: controle de requisições repetidas.
- LedgerAccount: contas contábeis.
- LedgerTransaction: transações financeiras.
- LedgerEntry: débitos e créditos.
- OutboxMessage: eventos pendentes de publicação.
- WebhookDelivery: controle de notificações.
- ReconciliationRecord: registros de conciliação.

