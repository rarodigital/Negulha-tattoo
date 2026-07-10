# FLUXO COMPLETO - Negulha Tattoo
*Arquitetura e fluxos de operação — versão 10/07/2026*

---

## ARQUITETURA GERAL

```
┌─────────────────────────────────────────────────────────────┐
│                      CLIENTE (WhatsApp)                      │
│              Número: [seu número cliente]                    │
└────────────────────────┬────────────────────────────────────┘
                         │ (mensagem recebida)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                 EVOLUTION API (Gerenciar)                    │
│           Webhook recebe evento: messages.upsert             │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              N8N WORKFLOW (Orquestração)                      │
│  - IF: verificar se é recebida (fromMe == false)            │
│  - LOOKUP: buscar na planilha se existe contato             │
│  - IF novo: cadastrar + enviar boas-vindas                  │
│  - IF existente: ler contexto + IA responde                 │
│  - SAVE: registrar na planilha histórico                    │
│  - SEND: enviar para número tatuador                        │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   ┌─────────┐     ┌──────────┐     ┌─────────┐
   │ Sheets  │     │ Calendar │     │ GitHub  │
   │ Leads   │     │ Eventos  │     │ Commits │
   └─────────┘     └──────────┘     └─────────┘
        │                │                │
        └────────────────┼────────────────┘
                         ▼
    ┌──────────────────────────────────────┐
    │  TATUADOR (Número WhatsApp)          │
    │  Recebe: qualificações + orçamentos  │
    └──────────────────────────────────────┘
```

---

## FLUXO 1: NOVO CLIENTE (LEAD)

**Trigger:** Cliente manda primeira mensagem no WhatsApp (número cliente)

**Sequência:**

```
1. WEBHOOK RECEBE
   - event: messages.upsert
   - fromMe: false (recebida)
   - senderPn: telefone do cliente
   - body: mensagem texto

2. N8N IF: Verificar se é recebida
   IF fromMe == false:
     → Continue
   ELSE:
     → Stop (ignorar mensagens do próprio tatuador)

3. N8N LOOKUP: Verificar se contato existe
   Query: SELECT * FROM Leads WHERE telefone = [senderPn]
   
   IF encontrado:
     → Ir pro FLUXO 2 (Cliente Existente)
   ELSE:
     → Continua aqui (novo)

4. N8N: CADASTRAR NOVO
   - Google Sheets ADD ROW (Aba: Leads)
   - Campos: ID (auto), Data Entrada, Nome, Telefone, Origem, Status="Novo Lead"

5. N8N: IA RESPONDE (Boas-vindas)
   Prompt: "Você é assistente do Negulha Tattoo. Cliente novo chegou. Diga boas-vindas."
   Send Evolution: números cliente
   Mensagem padrão com CTA

6. N8N: AVISAR TATUADOR
   Send Evolution: número tatuador
   "🔔 NOVO CLIENTE CHEGOU
    Nome: [nome]
    Telefone: [telefone]
    Mensagem: [texto]"

7. N8N: REGISTRAR HISTÓRICO
   - Google Sheets ADD ROW (Aba: Histórico de Mensagens)
   - Campos: Data, Hora, Telefone, Nome, Tipo="Cliente", Mensagem
```

**Resultado:** Cliente novo está em "Leads", recebeu boas-vindas, tatuador foi avisado.

---

## FLUXO 2: CLIENTE EXISTENTE (COM CONTEXTO)

**Trigger:** Cliente que já conversou manda nova mensagem

**Sequência:**

```
1. WEBHOOK RECEBE (igual ao Fluxo 1)

2. N8N LOOKUP: Encontrou contato existente ✓

3. N8N: LER HISTÓRICO
   Query: SELECT * FROM "Histórico de Mensagens" 
           WHERE telefone = [senderPn] 
           LIMIT 15
   
   (Pega últimas 15 mensagens daquele cliente)

4. N8N: IA COM CONTEXTO
   Prompt incluye:
   - "Histórico com este cliente: [últimas 15 mensagens]"
   - "Mensagem nova: [texto atual]"
   - "Instruções: Responda com contexto. Se mencionar 'orçamento', avise tatuador."
   
   IA reconhece:
   - Se cliente é novo ou já conhece
   - Se está pedindo orçamento (trigger)
   - Se é confirmação de data
   - Se é dúvida

5. N8N: IA RESPONDE
   - Se orçamento: "Vou passar pro Adalto. Um momento..."
   - Se confirmação: "Perfeito! Confirmado pra [data]"
   - Se dúvida: Responde direto com informação

6. N8N: AVISAR TATUADOR (SE NECESSÁRIO)
   IF: IA detectou "orçamento" OR "confirmação":
     Send ao tatuador com detalhes
   ELSE:
     Sem avisar (só registro)

7. N8N: REGISTRAR HISTÓRICO
   - ADD ROW: Data, Hora, Mensagem, Tipo="Cliente"
   - ADD ROW: Data, Hora, Mensagem IA, Tipo="IA"
```

**Resultado:** Cliente recebe resposta inteligente com contexto, tatuador é avisado se necessário.

---

## FLUXO 3: TATUADOR RESPONDE

**Trigger:** Adalto envia mensagem no número tatuador (resposta a cliente)

**Sequência:**

```
1. WEBHOOK RECEBE
   - fromMe: true (mensagem enviada pelo tatuador)
   - Conteúdo: orçamento, confirmação, etc

2. N8N IF: Verificar se é do tatuador
   IF fromMe == true:
     → ADD ROW Histórico (registra) → STOP
   ELSE:
     → Continue (cliente normal)

3. N8N: REGISTRAR HISTÓRICO
   - ADD ROW: Data, Hora, Tipo="Adalto", Mensagem
   
4. N8N: ATUALIZAR PLANILHA
   IF "orçamento" detectado:
     - UPDATE Leads: Status="Orçamento Enviado"
     - UPDATE Leads: Valor Orçado=[valor]
   
   IF "confirmação" detectado:
     - MOVE cliente de Leads → Agendamentos
     - CREATE Google Calendar: evento na data
     - UPDATE Agendamentos: Data, Horário, Status="Confirmado"

5. (FUTURO) N8N: IA RETORNA PRO CLIENTE
   Parse resposta do tatuador → converte em mensagem clara
   Send cliente: "Adalto respondeu: R$[valor]... Quer marcar?"
```

**Resultado:** Histórico registrado, planilha atualizada, cliente recebe resposta.

---

## FLUXO 4: AGENDAMENTO CONFIRMADO

**Trigger:** Cliente confirma data/horário

**Sequência:**

```
1. CLIENTE MANDA: "Beleza, quero o dia 12 de julho às 14h"

2. IA COM CONTEXTO:
   - Reconhece: cliente está confirmando
   - Busca orçamento anterior
   - Valida data com tatuador

3. N8N: ATUALIZAR PLANILHA
   - DELETE linha de Leads
   - ADD ROW em Agendamentos
   - Campos: Nome, Telefone, Serviço, Valor, Data, Status="Confirmado"

4. N8N: CRIAR GOOGLE CALENDAR
   - Event: "[Nome] - [Serviço]"
   - Date/Time: data + horário
   - Description: detalhes (valor, telefone, referência)

5. N8N: AVISAR TATUADOR
   "✅ CONFIRMADO!
    Cliente: [nome]
    Data: [dia] às [hora]
    Serviço: [descrição]
    Valor: R$[x]"

6. N8N: RESPONDER CLIENTE
   "Perfeito! Confirmado pra [dia] às [hora]. Te vejo lá! 🔥"
```

**Resultado:** Agendamento no calendário, planilha atualizada, ambos avisados.

---

## FLUXO 5: TATTOO REALIZADA

**Trigger:** Adalto avisa "Tatuagem pronta" (normalmente via WhatsApp ou foto no status)

**Sequência:**

```
1. ADALTO: "Tatuagem pronta, enviando vídeo" (ou faz story)

2. N8N: REGISTRAR HISTÓRICO
   - ADD ROW: Data realização, Status="Realizado"

3. N8N: ATUALIZAR AGENDAMENTO
   - UPDATE: Status="Finalizado"
   - UPDATE: Data Realização

4. N8N: AVISAR CLIENTE
   "Tatuagem pronta! 🎨 Vídeo do resultado anexado.
    Cuide bem nos próximos dias: [instruções]
    Avalia aí no Google Maps! ⭐"

5. N8N: REGISTRAR CONTEÚDO
   - UPDATE "Histórico de Postagens"
   - Vídeo foi postado? Quantas views?

6. (FUTURO) N8N: LEMBRETE
   - Cria reminder para 3 dias depois: "Como está sua tatuagem?"
   - 7 dias depois: "Deixa sua avaliação no Google"
```

**Resultado:** Cliente finalizado, conteúdo registrado, lembretes automáticos.

---

## FLUXO 6: POSTAGEM DE CONTEÚDO

**Trigger:** Adalto quer postar ou JÁ postou conteúdo

**Sequência:**

```
1. ADALTO: "Vou postar a Rosa do Kayke no TikTok agora"

2. BOT TELEGRAM (Você):
   - Registra em "Histórico de Postagens"
   - Status="Agendado" → "Posted"
   - Canal="TikTok"
   - Data/Hora

3. (FUTURO) BOT MONITORA:
   - Quantas views após 2h
   - Quantos cliques/comentários
   - Qualidade de engajamento

4. BOT NOTIFICA:
   "Post publicado! 📊 Monitorando resultados..."
   (depois) "Rosa do Kayke: 150 views, 20 cliques, 1 lead novo!"
```

**Resultado:** Todas postagens catalogadas, performance rastreada.

---

## RESUMO DE INTEGRAÇÕES

| Sistema | Ação | Momento |
|---------|------|---------|
| **Google Sheets** | ADD/UPDATE Leads, Agendamentos, Histórico | Real-time (cada interação) |
| **Google Calendar** | CREATE/UPDATE eventos | Quando confirma agendamento |
| **GitHub** | Commit em continuity.md | Decisões importantes |
| **n8n** | Dispara webhooks | Quando precisa ativar/desativar fluxos |
| **WhatsApp/Evolution** | SEND/RECEIVE mensagens | Cada interação |

---

## O BOT DO TELEGRAM (Novo)

**Ele vai fazer:**
1. Ler esses fluxos automaticamente via conversação natural
2. Atualizar Google Sheets conforme você conversar
3. Sincronizar com Google Calendar
4. Fazer commits no GitHub com resumos
5. Disparar webhooks no n8n se necessário
6. Lembrar você de postagens diárias
7. Rastrear clientes e métricas

**Entrada:** "Casal confirmou pra 12 de julho às 14h"
**Saída:** 
- Planilha atualizada
- Calendário com novo evento
- GitHub com commit
- Tatuador avisado (via WhatsApp)
- Você confirmado (via Telegram)

---

## PRÓXIMO PASSO

Este fluxo será convertido em **N8N AUTOMATION PLAN** para o dev montar.
