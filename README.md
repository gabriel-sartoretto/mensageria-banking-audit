# banking-audit

Microsserviço de **auditoria**. Consome do RabbitMQ cada mudança de situação cadastral de agência e grava um registro
histórico no banco.

Faz parte do projeto [kafka-rabbitmq-banking](https://github.com/gabriel-sartoretto/kafka-rabbitmq-banking), junto com
[banking-validation](https://github.com/gabriel-sartoretto/mensageria-banking-validation) e
[banking-service](https://github.com/gabriel-sartoretto/mensageria-banking-service).

**Stack:** Java 21 · Quarkus 3.29 · Hibernate Reactive Panache · PostgreSQL · RabbitMQ

## O que ele faz

O `BankingAuditService` escuta o canal `notificacoes`:

| Configuração | Valor |
|---|---|
| Exchange | `notificacoes` (direct), publicado pelo banking-validation |
| Fila | `notificacao.audit` (ligada à routing key `agencia.change_status`) |
| Dead letter | DLQ criada automaticamente, routing key `agencia.change_status.dlq` |

Cada mensagem (JSON com `cnpj` e `situacaoCadastral`) vira uma linha na tabela `audit`:

```sql
audit(id serial, cnpj text, status text, dateTime timestamp default now())
```

O serviço não expõe endpoints de negócio. Ele só consome mensagens (porta HTTP 8282).

## Rodando localmente

Pré-requisito: o RabbitMQ rodando. Ele sobe pelo `docker-compose.yml` do
[banking-validation](https://github.com/gabriel-sartoretto/mensageria-banking-validation) (porta 5672).

```bash
docker compose up -d        # PostgreSQL na porta 5434, banco "audit"
./mvnw quarkus:dev
```

Para ver os registros: `docker exec -it postgres-db-alura-audit-container psql -U joao -d audit -c "select * from audit;"`

Variáveis de ambiente: `QUARKUS_DATASOURCE_HOST` / `_PORT` (padrão `localhost` / `5434`) e
`QUARKUS_DATASOURCE_USERNAME` / `_PASSWORD` (padrão `joao` / `joao`).

## Build

```bash
./mvnw package                                  # target/quarkus-app/quarkus-run.jar
./mvnw package -Dquarkus.package.jar.type=uber-jar
./mvnw package -Dnative                         # executável nativo (requer GraalVM)
```
