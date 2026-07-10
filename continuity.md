# continuity.md — Histórico de Sincronização (Negulha Tattoo)

## [10/07/2026] — Bot — Integrações conectadas ✅✅✅

Todas as integrações testadas e funcionando:

**GitHub** ✅ commit via API (repo rarodigital/Negulha-tattoo)

**Google Sheets** ✅ (Service Account nglauto / negulha-asistente@nglauto.iam.gserviceaccount.com)
- Planilha "Planilha Base" — 7 abas: Leads, Agendamentos, Histórico de Mensagens, Estoque, Histórico de Postagens, Métricas, Financeiro
- Leitura confirmada (Leads e Financeiro)

**Google Calendar** ✅ social.adaltovieira@gmail.com (America/Sao_Paulo) — acesso de escrita liberado

**n8n** ✅ https://n8n.theroninstituto.com.br
- Workflow em uso: "Sendm" (id 4kuw3Im6Z8X7SeO3) — webhook POST /webhook/sendm
- ATENÇÃO: workflow está INATIVO; precisa ser ativado no n8n pra o webhook de produção responder

**Estado financeiro real (da planilha):** Caixa R$670,22 (não R$671)

**Próximos passos:**
- Ativar workflow Sendm no n8n (Adalto)
- Rodar os 4 testes de escrita do doc 5 (com confirmação do Adalto antes de escrever na planilha real)

**Status:** 🟢 Totalmente conectado (leitura). Escrita pronta, aguardando OK pra testar.

## [10/07/2026] — Bot — Sincronização inicial ✅
Contexto carregado dos docs 1, 2 e 5. Repo e token OK.
