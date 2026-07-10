# GUIA DE APIS - Conexões Necessárias
*Credenciais e configuração para o Bot do Telegram*

---

## 1. GOOGLE SHEETS API

**O quê:** Ler e escrever em planilha Google Sheets

**O que você vai fazer:**
- [ ] Abrir Google Cloud Console
- [ ] Criar Project: "Negulha Tattoo Bot"
- [ ] Ativar Google Sheets API
- [ ] Criar Service Account
- [ ] Gerar chave JSON
- [ ] Compartilhar planilha com email da Service Account

**Dados necessários:**
```
- Service Account Email: xxxxx@xxxxx.iam.gserviceaccount.com
- Private Key: (chave JSON inteira)
- Spreadsheet ID: (do link da planilha)
- Sheet Names: "Leads", "Agendamentos", "Histórico de Mensagens", "Estoque", etc
```

**Como obter o Spreadsheet ID:**
Seu link: `https://docs.google.com/spreadsheets/d/[ID_AQUI]/edit`
Copie o ID entre `/d/` e `/edit`

**Permissões necessárias:**
- `https://www.googleapis.com/auth/spreadsheets`
- `https://www.googleapis.com/auth/drive`

**Você vai passar pro bot:**
- Arquivo JSON completo da Service Account
- Spreadsheet ID
- Nomes exatos das abas

---

## 2. GOOGLE CALENDAR API

**O quê:** Ler e criar eventos no Google Calendar

**O que você vai fazer:**
- [ ] No mesmo Google Cloud Project (anterior)
- [ ] Ativar Google Calendar API
- [ ] Criar OAuth 2.0 Client (tipo: Desktop Application)
- [ ] Fazer primeira autenticação
- [ ] Obter Refresh Token

**Dados necessários:**
```
- Calendar ID: seu email Gmail (ou calendário específico)
- OAuth2 Client ID: xxxxx.apps.googleusercontent.com
- OAuth2 Client Secret: (chave secreta)
- Refresh Token: (obtido após autenticação)
```

**Permissões necessárias:**
- `https://www.googleapis.com/auth/calendar`

**Você vai passar pro bot:**
- Client ID
- Client Secret
- Refresh Token
- Calendar ID (provavelmente seu email)

---

## 3. GITHUB API

**O quê:** Fazer commits e atualizar continuity.md

**O que você vai fazer:**
- [ ] Criar Personal Access Token no GitHub
- [ ] Dar permissões: `repo` + `contents` + `workflow`
- [ ] Notar: seu username GitHub e nome do repo

**Dados necessários:**
```
- GitHub Username: seu username
- Repository Name: (ex: "negulha-tattoo" ou "studio-automation")
- Branch: "main" ou qual você usa
- Personal Access Token: ghp_xxxxxxxxxxxxx
```

**Arquivo a atualizar:**
- `continuity.md` (raiz do repo)
- Commits automáticos com: "[Data] — Bot — [Assunto]"

**Você vai passar pro bot:**
- GitHub Username
- Repo Name
- Personal Access Token
- Branch name

---

## 4. N8N API

**O quê:** Disparar workflows do n8n (opcional, pra depois)

**O que você já tem:**
- URL do n8n: https://n8n.theroninstituto.com.br (ou sua VPS)
- Webhook endpoints já criados

**Dados necessários:**
```
- N8N URL: https://seu-n8n.com
- N8N API Key: (gerar em Settings → API Keys)
- Webhook URLs: (dos workflows existentes)
```

**Você vai passar pro bot:**
- N8N URL
- N8N API Key
- Lista de Webhooks (com nomes):
  - `/negulha-atendimento`
  - Qualquer outro que existir

---

## 5. EVOLUTION API (JÁ CONFIGURADO)

**O quê:** Enviar/receber mensagens WhatsApp

**Você já tem:**
- Instância de Evolution na sua VPS
- 2 números conectados
- Webhooks apontando pro n8n

**Dados que você já sabe:**
```
- Evolution URL: (sua URL)
- API Key: (você já tem)
- Instance Name: "NGL" (ou qual você usa)
- Webhook para n8n: https://seu-n8n.com/webhook/negulha-atendimento
```

**O bot pode usar Evolution se precisar**, mas por enquanto o n8n já está intermediando.

---

## 6. TELEGRAM BOT API (PARA O BOT EXISTIR)

**O quê:** Bot responder no Telegram

**Você já tem:**
- Bot criado no BotFather (@BotFather)
- Bot Token
- Grupo criado com o bot

**Dados necessários:**
```
- Bot Token: xxxxx:AAAxxxx (you@BotFather)
- Group/Chat ID: (número do grupo onde o bot opera)
```

**O bot já está rodando**, então você só passa o Token pra referência.

---

## CHECKLIST DE CREDENCIAIS

Você vai precisar compilar e passar tudo isso:

```
[ ] GOOGLE SHEETS
    [ ] Service Account JSON (arquivo completo)
    [ ] Spreadsheet ID
    [ ] Sheet Names (confirmados)

[ ] GOOGLE CALENDAR
    [ ] OAuth2 Client ID
    [ ] OAuth2 Client Secret
    [ ] Refresh Token
    [ ] Calendar ID

[ ] GITHUB
    [ ] GitHub Username
    [ ] Repository Name
    [ ] Branch Name
    [ ] Personal Access Token

[ ] N8N
    [ ] N8N URL
    [ ] N8N API Key
    [ ] Webhook URLs (lista)

[ ] TELEGRAM
    [ ] Bot Token (referência)
    [ ] Group/Chat ID
```

---

## COMO GERAR CREDENCIAIS PASSO A PASSO

### **Google Cloud Console (Sheets + Calendar)**

1. Acesse: https://console.cloud.google.com
2. Crie novo projeto: "Negulha Tattoo"
3. Ative APIs:
   - Google Sheets API
   - Google Calendar API
4. Crie Service Account:
   - IAM & Admin → Service Accounts
   - New Service Account
   - Nome: "negulha-bot"
   - Role: Editor
   - Criar chave JSON
   - **Salve o arquivo JSON completo**
5. Compartilhe planilha:
   - Planilha Google Sheets
   - Compartilhar → colar email da Service Account
   - Dar acesso de Editor

### **GitHub**

1. Acesse: https://github.com/settings/tokens
2. Generate New Token → Generate New Token (classic)
3. Nome: "Negulha Bot"
4. Scopes: repo, workflow, contents
5. Gerar token
6. **Copie o token** (só aparece uma vez)

### **N8N**

1. Acesse: https://seu-n8n.com/settings
2. Vá em "API Keys"
3. Generate API Key
4. Copie

---

## ONDE VOCÊ VAI USAR ISSO

**Você vai fazer um arquivo chamado `CREDENCIAIS_BOT.env`** com:

```env
# Google Sheets
GOOGLE_SHEETS_SERVICE_ACCOUNT="{ JSON completo aqui }"
GOOGLE_SHEETS_SPREADSHEET_ID="xxxxx"

# Google Calendar
GOOGLE_CALENDAR_CLIENT_ID="xxxxx.apps.googleusercontent.com"
GOOGLE_CALENDAR_CLIENT_SECRET="xxxxx"
GOOGLE_CALENDAR_REFRESH_TOKEN="xxxxx"
GOOGLE_CALENDAR_ID="seu_email@gmail.com"

# GitHub
GITHUB_USERNAME="seu_username"
GITHUB_REPO="negulha-tattoo"
GITHUB_BRANCH="main"
GITHUB_TOKEN="ghp_xxxxx"

# N8N
N8N_URL="https://seu-n8n.com"
N8N_API_KEY="xxxxx"

# Telegram
TELEGRAM_BOT_TOKEN="xxxxx:AAAxxxx"
TELEGRAM_CHAT_ID="xxxxx"
```

**IMPORTANTE:** Nunca commitar esse arquivo (adicionar a .gitignore)

---

## PRÓXIMO PASSO

Documento 4 vai ser: INSTRUÇÕES_SETUP_BOT.md — passo a passo de como colocar tudo junto.
