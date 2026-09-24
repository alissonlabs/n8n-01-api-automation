# Etapa 5 — Consolidação dos dados

Depois de realizar a requisição para a API externa, o workflow possui duas fontes de informação:

1. Os dados enviados originalmente pelo Webhook;
2. Os dados retornados pela API externa.

O HTTP Request passa a fornecer os dados da API no `$json` atual. Porém, os dados originais do Webhook não estão mais diretamente disponíveis nesse `$json`.

Para recuperar essas informações, o n8n permite acessar diretamente outro node através do seu nome.

Foi utilizado:

```javascript
const apiData = $input.first().json;
const webhookData = $('Webhook').first().json.body;
```

Nesse código:

* `$input.first().json` representa os dados recebidos do node anterior, neste caso o HTTP Request;
* `$('Webhook')` referencia diretamente o node chamado `Webhook`;
* `.first()` seleciona o primeiro item produzido pelo node;
* `.json.body` acessa o payload originalmente recebido pelo Webhook.

---

## Consolidação

Os dados das duas fontes são então combinados em um novo objeto:

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

O resultado final passa a reunir informações provenientes dos dois sistemas:

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

### Por que essa etapa é importante?

Em integrações reais, frequentemente precisamos:

* receber dados de um sistema;
* consultar outro sistema;
* cruzar as informações;
* transformar os dados;
* produzir uma nova estrutura para o próximo processo.

Neste workflow, o n8n está fazendo exatamente esse tipo de integração.

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
