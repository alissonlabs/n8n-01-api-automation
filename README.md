# n8n API Automation - Por Alisson Prado - Simulando o N8n Em máquina local após teste com VPS

Projeto desenvolvido para demonstrar conhecimentos práticos em automação de APIs utilizando n8n.

## Objetivo

Construir um workflow capaz de receber dados através de um Webhook, processar as informações e realizar uma requisição para uma API externa.

## Tecnologias

- n8n
- Docker
- REST API
- Webhooks
- HTTP
- JSON
- JavaScript
- Git/GitHub

## Arquitetura

Webhook → Processamento → API externa → Resposta

## Status

🚧 Em desenvolvimento 

## Simulação de Teste  Simulando APP Externo pelo Terminal 
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