# n8n API Automation

Projeto desenvolvido para demonstrar conhecimentos práticos em automação de APIs utilizando **n8n**, Webhooks, JavaScript, REST APIs e integração entre diferentes sistemas.

O projeto está sendo construído de forma incremental, com cada etapa documentada e versionada utilizando Git e GitHub.

## Objetivo

Construir uma automação capaz de:

1. Receber dados através de um Webhook;
2. Processar e transformar as informações;
3. Gerar informações utilizadas dinamicamente no fluxo;
4. Realizar uma requisição para uma API externa;
5. Receber os dados retornados pela API;
6. Consolidar informações provenientes de diferentes fontes.

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

## Arquitetura atual

```text
Webhook
   ↓
Code
   ↓
HTTP Request
   ↓
Code
   ↓
Resultado consolidado
```

---

# Etapa 1 — Webhook

O primeiro componente do workflow é um **Webhook**.

Um webhook permite que um sistema externo envie uma requisição HTTP para uma URL quando determinado evento acontece.

Neste projeto, o Webhook está configurado para receber:

```text
POST /api-automation
```

Durante o desenvolvimento, utilizamos a **Test URL** disponibilizada pelo n8n.

## Exemplo de requisição

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

O Webhook recebe os dados enviados no corpo da requisição.

---

# O que é um Payload?

Payload é o conjunto de dados transportado por uma requisição.

Neste projeto, o payload enviado pelo cliente é:

```json
{
  "name": "Alisson",
  "email": "alisson@example.com",
  "message": "Meu primeiro teste com n8n"
}
```

A requisição HTTP contém outras informações, como método, URL e headers. O payload representa os dados que queremos transportar e processar.

---

# Etapa 2 — Processamento com Code

Depois do Webhook, utilizamos um node **Code**.

O objetivo é acessar os dados recebidos, transformá-los e adicionar novas informações ao fluxo.

Durante o desenvolvimento, identificamos que os dados enviados pelo Webhook ficam dentro da propriedade `body`.

A estrutura recebida pelo n8n é semelhante a:

```json
{
  "body": {
    "name": "Alisson",
    "email": "alisson@example.com",
    "message": "Meu primeiro teste com n8n"
  }
}
```

Por isso, utilizamos:

```javascript
const data = $input.first().json.body;
```

O workflow então transforma os dados:

```javascript
return [
  {
    json: {
      name: data.name,
      email: data.email,
      message: data.message,
      userId: 1,
      processed: true,
      processedAt: new Date().toISOString()
    }
  }
];
```

O resultado passa a conter:

```json
{
  "name": "Alisson",
  "email": "alisson@example.com",
  "message": "Meu primeiro teste com n8n",
  "userId": 1,
  "processed": true,
  "processedAt": "2026-..."
}
```

---

# Etapa 3 — Expression

Uma das funcionalidades importantes do n8n são as **Expressions**.

Elas permitem utilizar dinamicamente dados produzidos por nodes anteriores.

Neste projeto, o campo:

```json
"userId": 1
```

é utilizado posteriormente pelo node HTTP Request.

Em vez de deixar o número `1` fixo na URL:

```text
https://jsonplaceholder.typicode.com/users/1
```

utilizamos:

```text
https://jsonplaceholder.typicode.com/users/{{ $json.userId }}
```

Dessa forma, o n8n utiliza o valor existente no campo `userId`.

Por exemplo:

```text
userId = 1
```

gera:

```text
/users/1
```

Enquanto:

```text
userId = 10
```

geraria:

```text
/users/10
```

Isso torna o workflow **dinâmico**, evitando valores fixos desnecessários.

---

# Etapa 4 — HTTP Request

O próximo node é o **HTTP Request**.

Ele é utilizado para realizar uma chamada para uma API externa.

Configuração atual:

```text
Method: GET
```

URL:

```text
https://jsonplaceholder.typicode.com/users/{{ $json.userId }}
```

O JSONPlaceholder é utilizado neste projeto como uma API pública de testes.

Quando `userId` possui o valor `1`, o n8n realiza uma requisição equivalente a:

```text
GET /users/1
```

A API retorna os dados em formato JSON.

Um exemplo simplificado da resposta:

```json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "company": {
    "name": "Romaguera-Crona"
  }
}
```

---

# Etapa 5 — Consolidação dos dados

Depois da requisição para a API externa, o workflow possui duas fontes de informação:

1. Os dados enviados originalmente pelo Webhook;
2. Os dados retornados pela API externa.

O HTTP Request passa a fornecer os dados da API no `$json` atual.

Por isso, para recuperar também os dados originais do Webhook, utilizamos uma referência direta ao node `Webhook`.

```javascript
const apiData = $input.first().json;
const webhookData = $('Webhook').first().json.body;
```

Neste código:

* `$input.first().json` representa os dados recebidos do node anterior, neste caso o HTTP Request;
* `$('Webhook')` referencia diretamente o node chamado `Webhook`;
* `.first()` seleciona o primeiro item produzido pelo node;
* `.json.body` acessa o payload originalmente recebido pelo Webhook.

## Consolidação

Os dados das duas fontes são então combinados:

```javascript
return [
  {
    json: {
      cliente: webhookData.name,
      email: webhookData.email,
      mensagem: webhookData.message,

      usuario_api: apiData.name,
      empresa: apiData.company.name,

      processado: true,
      processadoAt: new Date().toISOString()
    }
  }
];
```

O resultado final reúne informações provenientes dos dois sistemas:

```json
{
  "cliente": "Alisson",
  "email": "alisson@example.com",
  "mensagem": "Meu primeiro teste com n8n",
  "usuario_api": "Leanne Graham",
  "empresa": "Romaguera-Crona",
  "processado": true,
  "processadoAt": "2026-..."
}
```

## Por que essa etapa é importante?

Em integrações reais, frequentemente precisamos:

* receber dados de um sistema;
* consultar outro sistema;
* cruzar as informações;
* transformar os dados;
* produzir uma nova estrutura para o próximo processo.

Neste workflow, o n8n está realizando exatamente esse tipo de integração.

A arquitetura passou a ser:

```text
Sistema externo
      ↓
   Webhook
      ↓
Processamento
      ↓
  HTTP Request
      ↓
   API externa
      ↓
Consolidação
      ↓
 Resultado final
```

---

# Conceitos praticados

Este workflow permite praticar conceitos fundamentais de integração e automação.

### Webhook

Recebimento de eventos através de HTTP.

### Payload

Dados transportados pela requisição.

### JSON

Formato utilizado para representar e transportar os dados.

### JavaScript

Processamento e transformação dos dados dentro do n8n.

### Expressions

Utilização dinâmica de informações produzidas por outros nodes.

### REST API

Comunicação com sistemas externos através de HTTP.

### HTTP Request

Consumo de uma API externa.

### Integração

Combinação de dados provenientes de diferentes sistemas.

### Versionamento

Controle das alterações do workflow utilizando Git e GitHub.

---

# Versionamento

O workflow é versionado utilizando Git.

Cada evolução significativa do projeto é registrada através de commits.

Exemplo:

```bash
git status
git add .
git commit -m "feat: add dynamic API request"
git push
```

O objetivo é manter um histórico das alterações e demonstrar uma prática comum em projetos profissionais de desenvolvimento de software.

O workflow exportado pelo n8n está disponível em:

```text
workflows/01-api-automation.json
```

---

# Ambiente de desenvolvimento

O n8n é executado localmente utilizando Docker.

A arquitetura do ambiente utiliza:

```text
Docker
 ├── n8n
 └── PostgreSQL
```

O PostgreSQL é utilizado como banco de dados do n8n.

---

# Próximas etapas

O workflow será evoluído gradualmente para demonstrar cenários mais próximos de aplicações reais.

* [ ] Validar os dados recebidos pelo Webhook;
* [ ] Implementar tratamento de dados obrigatórios;
* [ ] Adicionar tratamento de erros da API;
* [ ] Tratar respostas HTTP inválidas;
* [ ] Adicionar tratamento de timeout ou falha de comunicação;
* [ ] Adicionar condições ao fluxo;
* [ ] Persistir dados no PostgreSQL;
* [ ] Adicionar logs;
* [ ] Criar exemplos de entradas e saídas;
* [ ] Criar uma automação com aplicação prática de negócio.

---

# Status

🚧 Em desenvolvimento

O projeto está sendo construído incrementalmente, com foco em demonstrar conceitos de:

* Integração de APIs;
* Automação de processos;
* Processamento de dados;
* JavaScript;
* Webhooks;
* REST APIs;
* Boas práticas de desenvolvimento;
* Versionamento com Git e GitHub.
