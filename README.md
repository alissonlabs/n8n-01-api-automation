# n8n API Automation

Projeto desenvolvido para demonstrar conhecimentos práticos em automação de APIs utilizando **n8n**, Webhooks, JavaScript, REST APIs e integração entre diferentes sistemas.

O projeto está sendo construído de forma incremental, com cada etapa documentada e versionada utilizando Git e GitHub.

---

## Objetivo

Construir uma automação capaz de:

1. Receber dados através de um Webhook;
2. Processar e transformar esses dados;
3. Gerar informações utilizadas dinamicamente no fluxo;
4. Realizar uma requisição para uma API externa;
5. Receber e trabalhar com os dados retornados pela API;
6. Evoluir posteriormente para uma integração mais próxima de um cenário real de negócio.

---

## Tecnologias

* n8n
* Docker
* PostgreSQL
* REST API
* Webhooks
* HTTP
* JSON
* JavaScript
* Git
* GitHub

---

## Arquitetura atual

```text
Webhook
   ↓
Code
   ↓
HTTP Request
   ↓
API externa
   ↓
Dados retornados
```

---

# Etapa 1 — Webhook

O primeiro componente do workflow é um **Webhook**.

Um webhook permite que um sistema externo envie uma requisição HTTP para uma URL quando determinado evento acontece.

Neste projeto, o Webhook está configurado para receber requisições:

```text
POST /api-automation
```

Durante o desenvolvimento, o n8n disponibiliza uma **Test URL**, utilizada enquanto o workflow está aguardando um evento de teste.

### Exemplo de requisição

O teste foi realizado utilizando PowerShell:

```powershell
$body = @{
    name = "Alisson"
    email = "alisson@example.com"
    message = "Meu primeiro teste com n8n"
} | ConvertTo-Json

Invoke-RestMethod `
    -Uri "http://localhost:5678/webhook-test/api-automation" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

---

# O que é um Payload?

Payload é o conjunto de dados transportado por uma requisição.

Neste exemplo, o payload enviado pelo cliente é:

```json
{
  "name": "Alisson",
  "email": "alisson@example.
```
