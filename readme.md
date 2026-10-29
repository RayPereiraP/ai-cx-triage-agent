# ⚡ AI CX Incident Triage & Sentiment Engine

Pipeline automatizado de classificação, auditoria de sentimento e mitigação proativa de incidentes operacionais para ecossistemas de Delivery / CX utilizando **n8n** e **LLMs com Structured Outputs**.

---

## 🎯 Problema de Negócio

Em operações de grande escala (como iFood), milhares de feedbacks e avaliações são gerados por minuto. A triagem manual de reclamações críticas apresenta gargalos clássicos:
1. **Tempo de Resposta Elevado (SLA):** Casos graves (intoxicação alimentar, cancelamentos abusivos, atrasos extremos) demoram horas para alcançar a equipe de resolução.
2. **Custo Operacional de N1:** Analistas gastam mais de 60% da jornada categorizando chamados repetitivos em vez de resolver dores complexas.
3. **Falta de Padronização:** Erros humanos na taxonomia de dados prejudicam a identificação de causas-raiz em restaurantes e hubs parceiros.

### 📊 Impacto Estimado & ROI

| Métrica | Processo Manual Tradicional | Com o AI Triage Agent | Ganho Operacional |
| :--- | :--- | :--- | :--- |
| **Tempo de Triagem / Classificação** | ~3 a 5 minutos por ticket | < 2 segundos (tempo de inferência) | **98.8% de redução** |
| **Identificação de Incidentes Críticos** | Reativa (fila FIFO de chamados) | Proativa (desvio imediato para canal crítico) | **Alerta em tempo real** |
| **Horas Economizadas** | 100 tickets = ~6.5 horas humanas | 100 tickets = automação contínua | **~130h/mês por analista** |

---

## 🏗️ Arquitetura do Sistema
[Inbound Webhook / Payload]
│
▼
[Pre-processador & Anonimização PII (LGPD)]
│
▼
[LLM Engine (Structured JSON Output / Temperature 0)]
├── Classificação Taxonômica
├── Nível de Urgência (1 a 5)
└── Avaliação de Risco de Churn
│
▼
[Roteamento Condicional]
├── Se Urgência >= 4 ou Risco Alto ──► [Alerta Prioritário no Slack / Ops]
└── Se Normal ────────────────────────► [Armazenamento / Log Operacional]
│
▼
[Resposta HTTP 200 com Metadados Analisados]

---

## 🛠️ Tecnologias Utilizadas

* **Orquestrador:** n8n (Community / Cloud)
* **Inteligência Artificial:** OpenAI API (`gpt-4o-mini`) com *Strict JSON Schema*
* **Linguagem de Apoio:** JavaScript (nós de código para validação e regex PII)
* **Comunicação Operacional:** Webhooks REST & Slack Incoming Webhooks

---

## 🚀 Como Executar o Projeto

### 1. Pré-requisitos
* Instância do n8n instalada (Docker local, Cloud ou n8n Desktop).
* Chave de API da OpenAI (`OPENAI_API_KEY`).
* Webhook do Slack (ou Discord/Telegram).

### 2. Configuração do Workflow
1. No n8n, crie um novo workflow.
2. Abra o menu superior direito (`...`), selecione **Import from File** e carregue o arquivo `workflows/workflow.json`.
3. Configure as variáveis de ambiente no seu ambiente ou substitua diretamente nos nós correspondentes:
   * Chave da OpenAI no nó `LLM Structured Analysis`.
   * Webhook URL no nó `Send Slack Alert`.
4. Ative o fluxo (*Active: True*).

### 3. Teste Manual (cURL)

Envie uma requisição de teste para o Webhook gerado:

```bash
curl -X POST "http://localhost:5678/webhook/ticket-feedback" \
     -H "Content-Type: application/json" \
     -d '{
       "ticket_id": "TCK-9921",
       "rating": 1,
       "comment": "Meu pedido atrasou mais de 1h30, a comida chegou completamente fria e o entregador não respondeu no chat. Meu telefone é 11999998888."
     }'