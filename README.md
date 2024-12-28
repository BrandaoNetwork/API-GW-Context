# API-GW-Context
# Monitorando o AWS API Gateway com New Relic

## Visão Geral
Este documento detalha como monitorar o **AWS API Gateway** utilizando o **New Relic**, aproveitando métricas, logs e traces para obter uma visão completa do desempenho, erros e tráfego das APIs.

---

## 1. Integração com o New Relic via AWS CloudWatch

### Passos:
1. **Habilitar métricas do API Gateway no CloudWatch:**
   - Ative o monitoramento detalhado no console do API Gateway.
   - Habilite logs de execução para capturar detalhes de cada requisição.

2. **Configurar a integração AWS-New Relic:**
   - Configure o **New Relic Infrastructure AWS Integration**.
   - Inclua as métricas do API Gateway, como:
     - `5XXError` (erros de servidor)
     - `4XXError` (erros do cliente)
     - `IntegrationLatency` (latência do backend)
     - `CacheHitCount` e `CacheMissCount` (uso de cache)
     - `Count` (total de requisições)

3. **Visualizar no New Relic:**
   - Crie painéis para monitorar métricas específicas do API Gateway.

---

## 2. Coleta de Logs do API Gateway

### Passos:
1. **Ativar logs no API Gateway:**
   - Configure logs de execução para capturar:
     - Corpo da solicitação (request body)
     - Corpo da resposta (response body)
     - Erros de integração
   - Envie os logs para o **CloudWatch Logs**.

2. **Enviar logs do CloudWatch para o New Relic:**
   - Configure um **AWS Lambda Function** para exportar os logs do CloudWatch para o New Relic.
   - Utilize o **New Relic Log Forwarder**.

3. **Monitorar logs no New Relic:**
   - Analise padrões de erros e latências.
   - Identifique problemas específicos de desempenho.

---

## 3. Monitoramento de Traces usando New Relic Distributed Tracing

### Passos:
1. **Integrar serviços downstream:**
   - Ative o **Tracing AWS X-Ray** no API Gateway.
   - Configure o X-Ray para enviar traces detalhados de cada requisição.

2. **Exportar traces para o New Relic:**
   - Configure a integração do New Relic com o **AWS X-Ray**.
   - Visualize os traces no New Relic, incluindo:
     - Latência do API Gateway
     - Tempo gasto em serviços backend (Lambda, DynamoDB, etc.)

---

## 4. Uso de Alertas no New Relic

Crie alertas para monitorar problemas em tempo real. Exemplos:

- **Taxa de erro alta:** Alerta se `5XXError` ou `4XXError` ultrapassar um limite definido.
- **Latência elevada:** Alerta se `Latency` exceder um valor.
- **Erros de integração:** Alerta se há falhas frequentes no backend.
- **Quotas atingidas:** Monitore `CacheHitCount` e `CacheMissCount`.

---

## 5. Painéis Customizados no New Relic

Crie painéis para visualizar dados relevantes em um único local. Exemplos:

### Visão Geral do Tráfego:
- Total de requisições
- Latência média
- Taxa de erro (4XX e 5XX)

### Desempenho do Backend:
- Tempo de resposta de serviços downstream

### Logs de API:
- Logs categorizados por status de resposta (`200`, `404`, `500`)
- Mensagens de erro específicas

---

## 6. Alternativa: Customização com New Relic One

Use o **New Relic One** para criar integrações customizadas. Exemplos:

- **Monitoramento de payloads específicos:** Capture dados de requisições (como parâmetros).
- **Monitoramento de usuários finais:** Combine com New Relic Browser ou Mobile para entender como os usuários interagem com as APIs.

---

## Resumo das Opções

| **Método**                      | **Recurso Monitorado**                       | **Quando Usar**                              |
|---------------------------------|---------------------------------------------|---------------------------------------------|
| Integração CloudWatch           | Métricas gerais (tráfego, erros, latência)   | Para métricas básicas e detalhadas          |
| Coleta de Logs                  | Logs de requisições/respostas               | Para análise de erros e depuração           |
| Distributed Tracing             | Traces de ponta a ponta                     | Para monitorar desempenho em microsserviços |
| Painéis e Alertas Customizados  | Combinação de métricas, logs e traces       | Para ter uma visão unificada                |

Com essas opções, o New Relic oferece uma visão robusta e completa do desempenho do AWS API Gateway, garantindo APIs confiáveis e de alto desempenho.
