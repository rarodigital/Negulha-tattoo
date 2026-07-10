# continuity.md — Histórico de Sincronização (Negulha Tattoo)

## [10/07/2026] — Bot — Agente WhatsApp (n8n) no ar em MODO TESTE ✅

Recepção com IA funcionando ponta a ponta (workflow "Sendm", id 4kuw3Im6Z8X7SeO3):
- Webhook Evolution (path /webhook/sendm, formato LID: número real em data.key.senderPn)
- Cliente novo → cria lead na aba Leads + envia menu de boas-vindas
- Cliente recorrente → AI Agent (Groq llama-3.1-8b) + memória Postgres no Supabase (pooler IPv4)
- Grupos/status ignorados (filtro @g.us); fromMe ignorado
- Resposta enviada via Evolution (instância NGL)

Estado: ATIVO em modo teste (allowlist só 5521995816571). LLM = Groq (grátis; Gemini free tier estourou).

Pendências:
- Sair do modo teste (remover allowlist) p/ atender clientes reais
- Religar tools do agente: Notificar_Admin (Tool Workflow) + Ler_Agenda (service account HTTP)
- Testar rota do dono (!orcamento / !pago)
- Ligar/testar Fluxo B (resumo diário 23:55 -> Telegram grupo)
- Segurança: rotacionar GitHub PAT + senha Supabase

Infra/credenciais n8n: googleApi (service account) Sheets; Postgres pooler ssl (Supabase); Groq; Telegram.
Docs técnicos completos no workspace: carioquinha-dev/n8n-flows/README.md + build_flows.py.

## [10/07/2026] — Bot — Integrações conectadas ✅ (Sheets, Calendar, n8n, GitHub)

## [10/07/2026] — Bot — Sincronização inicial ✅
