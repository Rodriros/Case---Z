Este guia simula como você pode usar o **n8n** como orquestrador central para conectar a **Zoe** (Zenvia NLU), **Calendly**, **Salesforce** e a **Gemini API**, utilizando o **Postman** para validar cada etapa sem precisar de acesso imediato ao ambiente de produção da Zoe.

Siga o passo a passo abaixo:

---

### 🚀 Demonstração Prática: Resolução de Problemas Zoe NLU + n8n

Este documento descreve as ações e payloads necessários para resolver os problemas de "Amnésia", "Ponto Cego no CRM" e "Fadiga de Qualificação".

#### 1. Arquitetura da Solução

* **Zoe (Zenvia NLU):** Interface de conversação.
* **n8n:** Orquestrador de fluxo de dados (Webhooks e Lógica).
* **Postman:** Simulador de interações para testes de desenvolvedor.
* **Ferramentas Externas:** Calendly (Agendamento), Salesforce (CRM), Gemini API (Inteligência).

---

#### 2. Resolução do Problema A: Fim da "Amnésia" da Zoe

**Cenário:** O usuário agenda no Calendly, mas a Zoe não sabe disso e continua perguntando se ele quer agendar.
**Ação:** Criar um Webhook no n8n que recebe a confirmação do Calendly e atualiza o contexto da Zoe.

**Teste no Postman (Simulando Webhook do Calendly):**

* **Método:** `POST`
* **URL:** `URL_WEBHOOK_N8N_PROBLEMA_A`
* **Body (JSON):**

```json

 {
  "event": "invite.sent",
  "payload": {
    "email": "cliente@exemplo.com",
    "name": "Rodrigo Rosales",
    "phone": "5583991289129",
    "scheduled_event": {
      "start_time": "2026-05-15T13:00:00Z"
    },
    "tracking": {
      "utm_source": "zoe_chatbot",
      "session_id": "00Q8W00000XyZ123" 
    }
  }
}

```

**O que o n8n faz:**

1. Recebe o agendamento.
2. Faz uma chamada via `HTTP Request` para a API da Zenvia NLU (`POST /v1/assistants/{id}/context`) para setar uma variável `$agendamento_concluido = true`.
3. No Builder da Zoe, o próximo nó terá uma condição de entrada: `if $agendamento_concluido == true` -> "Opa, vi que você agendou! Vamos continuar?".

---

#### 3. Resolução do Problema B: Notificações no Salesforce

**Cenário:** O agendamento ocorre, mas o vendedor não recebe o alerta no CRM ou e-mail.
**Ação:** O n8n utiliza o dado do Calendly para buscar o Lead no Salesforce e disparar uma notificação via Apps Script (ou direto via API).

**Fluxo no n8n:**

1. **Nó Salesforce:** Procura Lead por E-mail.
2. **Nó Salesforce:** Cria uma "Task" ou "Event" vinculada ao Lead com os detalhes da reunião.
3. **Nó Apps Script / HTTP:** Envia um comando para o Google Apps Script para disparar um e-mail personalizado ao Vendedor.

**Teste no Postman (Simulando comando para o Apps Script):**

* **Método:** `POST`
* **URL:** `URL_GOOGLE_APPS_SCRIPT_DEPLOY`
* **Body (JSON):**

```json
{
  "vendedor_email": "vendedor.zenvia@exemplo.com",
  "lead_nome": "João Silva",
  "data_reuniao": "25/11 às 14h",
  "link_salesforce": "https://na1.salesforce.com/00Q8W00000XyZ123"
}

```

---

#### 4. Resolução do Problema C: Scoring Automático com Gemini API

**Cenário:** Qualificação longa e cansativa. A Zoe deve ouvir o cliente e classificar o Lead instantaneamente.
**Ação:** Enviar a frase de "dor" do cliente para o Gemini via n8n e devolver o score para o Salesforce.

**Teste no Postman (Simulando Zoe enviando texto para o n8n):**

* **Método:** `POST`
* **URL:** `URL_WEBHOOK_N8N_PROBLEMA_C`
* **Body (JSON):**

```json
{
  "atendimento_id": "999888",
  "lead_email": "cliente@exemplo.com",
  "userInput": "Estou precisando de uma solução de NLU urgente porque meu volume de chamados no suporte subiu 200% este mês e minha equipe não dá conta."
}

```

**Configuração do Prompt no Nó do Gemini (n8n):**

> "Você é um especialista em qualificação de leads B2B. Analise a frase do cliente e dê um score de 0 a 100 baseado em urgência e fit técnico. Responda apenas em JSON: { 'score': X, 'resumo': '...', 'categoria': 'HOT/WARM/COLD' }"

**Retorno esperado do Gemini para o Salesforce:**

```json
{
  "score": 95,
  "resumo": "Alta urgência devido a aumento de 200% no volume de chamados.",
  "categoria": "HOT"
}

```

*O n8n então faz um `PATCH` no Lead do Salesforce atualizando o campo `Lead_Score__c`.*

---

### Como testar sem o Builder da Zoe:

1. **No n8n:** Crie um workflow com 3 nós de `Webhook (Incoming)`.
2. **Integrações:** Adicione os nós de `Salesforce` e `Google Gemini` (usando sua API Key do Google AI Studio).
3. **No Postman:** Crie uma coleção chamada "Roleplay Zenvia" e salve os 3 exemplos de `POST` acima.
4. **Execução:** Dê o "Listen" no n8n e dispare o Postman. Você verá os dados fluindo em tempo real e os registros sendo criados no CRM e as notas sendo geradas pela IA.

---

### Resumo para o seu Roleplay:

"Como não temos acesso ao builder hoje, construí um **Middleware no n8n** que desacopla a lógica de negócio do chatbot. A Zoe foca na conversa, enquanto o n8n orquestra o **Calendly** para matar a 'amnésia', o **Salesforce** para dar visibilidade ao comercial e o **Gemini** para qualificar o lead sem fricção. Posso provar o funcionamento agora via Postman."
