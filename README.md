
## 📘 AlgaPosts

Sistema distribuído com dois microsserviços integrados via RabbitMQ. Permite criação e processamento assíncrono de **posts de texto**, com contagem de palavras e cálculo de valor estimado.

---

## 🧩 Arquitetura

```
[ Cliente ]
     │
     ▼
[ PostService - API REST ]
     │
     └──> Envia post para fila → 📨 `text-processor-service.post-processing.v1.q`
                                 ↓
                      [ TextProcessorService ]
                                 ↓
               📨 Envia resultado para fila → `post-service.post-processing-result.v1.q`
                                               ↓
                                   [ PostService atualiza post ]
```

---

## 📦 Microsserviços

### 1. 🧾 **PostService**

* Expõe API REST para criar e consultar posts
* Usa banco H2
* Envia novos posts para RabbitMQ
* Recebe resultados processados e atualiza os posts

#### Endpoints

| Método | URL                         | Descrição                               |
| ------ | --------------------------- | --------------------------------------- |
| `POST` | `/api/posts`                | Cria um post e envia para processamento |
| `GET`  | `/api/posts/{postId}`       | Retorna detalhes de um post             |
| `GET`  | `/api/posts?page=0&size=10` | Lista posts paginados                   |

#### DTOs

##### 📥 `PostInput`

```json
{
  "title": "string",
  "body": "string",
  "author": "string"
}
```

##### 📤 `PostOutput`

```json
{
  "id": "string",
  "title": "string",
  "body": "string",
  "author": "string",
  "wordCount": 123,
  "calculatedValue": 12.3
}
```

##### 📃 `PostSummaryOutput`

```json
{
  "id": "string",
  "title": "string",
  "summary": "string",
  "author": "string"
}
```

#### ✅ Validações

* `title`, `body` e `author` são obrigatórios (`@NotBlank`)
* `body` deve conter texto real
* `id` é UUID
* `summary` mostra as 3 primeiras linhas do `body`

---

### 2. ⚙️ **TextProcessorService**

* Consome mensagens com `postId` e `postBody`
* Calcula:

    * Número de palavras (`wordCount`)
    * Valor estimado (`calculatedValue = wordCount * 0.10`)
* Publica resultado para fila de retorno

#### 📩 Mensagem consumida

```json
{
  "postId": "string",
  "postBody": "string"
}
```

#### 📤 Mensagem produzida

```json
{
  "postId": "string",
  "wordCount": 123,
  "calculatedValue": 12.3
}
```

---

## 🐳 Como executar localmente

### ✅ Pré-requisitos

* Docker e Docker Compose
* Java 17+ (ou 21)
* Gradle ou Maven
* IDE (IntelliJ, VS Code, etc.)

### 🧪 Rodando os serviços

1. **Suba o RabbitMQ com Docker Compose**

```yaml
# docker-compose.yml
services:
  posts-rabbbitmq:
    image: rabbitmq:3-management
    restart: no
    ports:
      - 5672:5672
      - 15672:15672
    environment:
      RABBITMQ_DEFAULT_USER: rabbitmq
      RABBITMQ_DEFAULT_PASS: rabbitmq
    volumes:
      - posts-rabbitmq:/var/lib/rabbitmq/
volumes:
  posts-rabbitmq:
```

```bash
docker compose up -d
```

2. **Execute os microsserviços**

* Rode `PostServiceApplication` e `TextProcessorServiceApplication` com Spring Boot.

---

## 🧪 Como testar

### 1. Criar um post

```bash
curl -X POST http://localhost:8080/api/posts \
  -H "Content-Type: application/json" \
  -d '{
        "title": "Exemplo",
        "body": "Linha 1\nLinha 2\nLinha 3\nLinha 4\nLinha 5\nLinha 6",
        "author": "Usuário"
      }'
```

### 2. Listar posts com resumo

```bash
curl "http://localhost:8080/api/posts?page=0&size=5"
```

### 3. Consultar post por ID

```bash
curl http://localhost:8080/api/posts/{postId}
```

---

## 📊 RabbitMQ Management

Acesse o painel de filas em:
🔗 [http://localhost:15672](http://localhost:15672)
Login: `rabbitmq`
Senha: `rabbitmq`

---

## ✅ Tecnologias usadas

* Spring Boot 3
* RabbitMQ
* Spring AMQP
* Jakarta Bean Validation
* H2 Database
* Docker Compose
* Lombok

---
