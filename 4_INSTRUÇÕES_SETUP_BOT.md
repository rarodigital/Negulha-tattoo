# INSTRUÇÕES DE SETUP - Bot Telegram Operacional
*Passo a passo para deixar o bot pronto*

---

## PRÉ-REQUISITOS

- [ ] VPS com Node.js instalado (você já tem)
- [ ] Bot Telegram criado (você já tem)
- [ ] Grupo Telegram criado (você já tem)
- [ ] Credenciais compiladas (ver GUIA_DE_APIS.md)
- [ ] GitHub repo com continuity.md
- [ ] Google Sheets pronta
- [ ] Google Calendar ativo

---

## PASSO 1: ESTRUTURAR CREDENCIAIS

**Crie arquivo `.env` na pasta do bot:**

```
# Copie do GUIA_DE_APIS.md e preencha tudo
GOOGLE_SHEETS_SERVICE_ACCOUNT="{JSON aqui}"
GOOGLE_SHEETS_SPREADSHEET_ID="xxxx"
GOOGLE_CALENDAR_CLIENT_ID="xxxx"
... (resto)
```

**Nunca comitar esse arquivo:**
```bash
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add .env to gitignore"
```

---

## PASSO 2: INSTALAR DEPENDÊNCIAS

```bash
cd /caminho/do/seu/bot
npm install

# Dependências necessárias:
# - google-auth-library
# - googleapis
# - octokit (GitHub API)
# - axios (HTTP requests)
# - dotenv (carregar .env)
# - telegraf (Telegram bot)
```

---

## PASSO 3: ESTRUTURA DE PASTAS

```
negulha-bot/
├── .env (credenciais — NÃO COMITAR)
├── .gitignore
├── index.js (bot principal)
├── commands/
│   ├── leads.js (gerenciar leads)
│   ├── agendamentos.js (gerenciar agendamentos)
│   ├── financeiro.js (rastrear caixa)
│   ├── estoque.js (monitorar estoque)
│   └── postagens.js (trackear conteúdo)
├── integrations/
│   ├── googleSheets.js (API Sheets)
│   ├── googleCalendar.js (API Calendar)
│   ├── github.js (API GitHub)
│   └── n8n.js (API n8n)
├── utils/
│   ├── parser.js (interpretar mensagens naturais)
│   ├── formatter.js (formatar respostas)
│   └── logger.js (registrar ações)
├── docs/
│   ├── continuity.md (histórico no GitHub)
│   └── prompts.md (instruções de IA)
└── package.json
```

---

## PASSO 4: ARQUIVO PRINCIPAL (index.js)

Template básico:

```javascript
const { Telegraf } = require('telegraf');
require('dotenv').config();

const bot = new Telegraf(process.env.TELEGRAM_BOT_TOKEN);

// Importar integrações
const sheets = require('./integrations/googleSheets');
const calendar = require('./integrations/googleCalendar');
const github = require('./integrations/github');
const parser = require('./utils/parser');

// Comando: /start
bot.start((ctx) => {
  ctx.reply(`
Bem-vindo ao Bot Negulha Tattoo! 🔥
    
Aqui você gerencia tudo sobre seu estúdio:
- /leads — Ver leads em qualificação
- /agendamentos — Ver agendamentos confirmados
- /financeiro — Ver caixa e movimentação
- /estoque — Ver status de estoque
- /postagens — Trackear postagens
- /help — Ver todos comandos
  `);
});

// Comando: /leads
bot.command('leads', async (ctx) => {
  const leads = await sheets.read('Leads');
  ctx.reply(`📋 LEADS ATUAIS:\n${formatLeads(leads)}`);
});

// Comando: /agendamentos
bot.command('agendamentos', async (ctx) => {
  const agend = await sheets.read('Agendamentos');
  ctx.reply(`📅 AGENDAMENTOS:\n${formatAgend(agend)}`);
});

// Mensagem natural (principal)
bot.on('text', async (ctx) => {
  const mensagem = ctx.message.text;
  
  // Parser reconhece intenção
  const intencao = parser.parse(mensagem);
  // Ex: intencao = { tipo: "confirmacao", cliente: "Casal", data: "12/07" }
  
  // Executa ação
  const resultado = await executeAcao(intencao);
  
  // Responde ao usuário
  ctx.reply(resultado.resposta);
  
  // Registra no GitHub
  if (resultado.importante) {
    await github.commit(`[${new Date().toLocaleDateString()}] — Bot — ${intencao.tipo}`, resultado.dados);
  }
});

// Iniciar bot
bot.launch();
```

---

## PASSO 5: MÓDULO PARSER (Reconhecer Intenções)

`utils/parser.js`:

```javascript
function parse(mensagem) {
  const msg = mensagem.toLowerCase();
  
  // Confirmação de agendamento
  if (msg.includes('confirmou') || msg.includes('confirmado')) {
    return {
      tipo: 'confirmacao_agendamento',
      chave: 'confirmacao',
      exemplos: [
        'Casal confirmou pra 12 de julho às 14h',
        'Letícia confirmou pra 18 de julho',
        'Kayke confirmou presença'
      ]
    };
  }
  
  // Orçamento enviado
  if (msg.includes('orçamento') || msg.includes('valor')) {
    return {
      tipo: 'orcamento_enviado',
      chave: 'orcamento'
    };
  }
  
  // Tattoo realizada
  if (msg.includes('pronta') || msg.includes('finalizada') || msg.includes('realizado')) {
    return {
      tipo: 'tattoo_realizada',
      chave: 'realizado'
    };
  }
  
  // Financeiro
  if (msg.includes('cobrei') || msg.includes('paguei')) {
    return {
      tipo: 'movimentacao_financeira',
      chave: 'financeiro'
    };
  }
  
  // Estoque
  if (msg.includes('falta') || msg.includes('acabou') || msg.includes('estoque')) {
    return {
      tipo: 'alerta_estoque',
      chave: 'estoque'
    };
  }
  
  // Postagem
  if (msg.includes('postar') || msg.includes('vou fazer video')) {
    return {
      tipo: 'planejamento_postagem',
      chave: 'postagem'
    };
  }
  
  // Query
  if (msg.includes('quantos') || msg.includes('qual') || msg.includes('como')) {
    return {
      tipo: 'query_dados',
      chave: 'query'
    };
  }
  
  return { tipo: 'desconhecido' };
}

module.exports = { parse };
```

---

## PASSO 6: MÓDULO GOOGLE SHEETS (Ler/Escrever)

`integrations/googleSheets.js`:

```javascript
const { google } = require('googleapis');
const fs = require('fs');

const auth = new google.auth.GoogleAuth({
  keyFile: '/path/to/service-account.json', // Ou carregar do .env
  scopes: ['https://www.googleapis.com/auth/spreadsheets'],
});

const sheets = google.sheets({ version: 'v4', auth });

async function read(sheetName) {
  const result = await sheets.spreadsheets.values.get({
    spreadsheetId: process.env.GOOGLE_SHEETS_SPREADSHEET_ID,
    range: `'${sheetName}'!A:Z`, // Tudo
  });
  return result.data.values || [];
}

async function append(sheetName, values) {
  const result = await sheets.spreadsheets.values.append({
    spreadsheetId: process.env.GOOGLE_SHEETS_SPREADSHEET_ID,
    range: `'${sheetName}'!A:Z`,
    valueInputOption: 'USER_ENTERED',
    resource: { values: [values] },
  });
  return result;
}

async function update(sheetName, rowIndex, values) {
  const result = await sheets.spreadsheets.values.update({
    spreadsheetId: process.env.GOOGLE_SHEETS_SPREADSHEET_ID,
    range: `'${sheetName}'!A${rowIndex}:Z${rowIndex}`,
    valueInputOption: 'USER_ENTERED',
    resource: { values: [values] },
  });
  return result;
}

module.exports = { read, append, update };
```

---

## PASSO 7: MÓDULO GITHUB (Fazer Commits)

`integrations/github.js`:

```javascript
const { Octokit } = require('octokit');

const octokit = new Octokit({
  auth: process.env.GITHUB_TOKEN,
});

async function commit(mensagem, dados) {
  try {
    // Ler arquivo atual
    const file = await octokit.rest.repos.getContent({
      owner: process.env.GITHUB_USERNAME,
      repo: process.env.GITHUB_REPO,
      path: 'continuity.md',
    });

    // Parse conteúdo
    const currentContent = Buffer.from(file.data.content, 'base64').toString();
    
    // Adicionar nova entrada no topo
    const novaEntrada = `\n## ${new Date().toLocaleDateString()} — Bot — ${mensagem}\n**Dados:** ${JSON.stringify(dados)}\n---`;
    const novoContent = novaEntrada + '\n' + currentContent;

    // Fazer commit
    await octokit.rest.repos.createOrUpdateFileContents({
      owner: process.env.GITHUB_USERNAME,
      repo: process.env.GITHUB_REPO,
      path: 'continuity.md',
      message: `Bot: ${mensagem}`,
      content: Buffer.from(novoContent).toString('base64'),
      sha: file.data.sha,
    });

    console.log('✅ Commit feito:', mensagem);
  } catch (error) {
    console.error('❌ Erro ao fazer commit:', error);
  }
}

module.exports = { commit };
```

---

## PASSO 8: MÓDULO GOOGLE CALENDAR (Criar Eventos)

`integrations/googleCalendar.js`:

```javascript
const { google } = require('googleapis');

// OAuth2 (você vai autenticar uma vez)
const oauth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CALENDAR_CLIENT_ID,
  process.env.GOOGLE_CALENDAR_CLIENT_SECRET,
  'http://localhost:3000/oauth2callback'
);

oauth2Client.setCredentials({
  refresh_token: process.env.GOOGLE_CALENDAR_REFRESH_TOKEN,
});

const calendar = google.calendar({ version: 'v3', auth: oauth2Client });

async function criarEvento(titulo, dataInicio, dataFim, descricao) {
  const event = {
    summary: titulo,
    description: descricao,
    start: { dateTime: dataInicio, timeZone: 'America/Sao_Paulo' },
    end: { dateTime: dataFim, timeZone: 'America/Sao_Paulo' },
  };

  const result = await calendar.events.insert({
    calendarId: process.env.GOOGLE_CALENDAR_ID,
    resource: event,
  });

  return result.data;
}

module.exports = { criarEvento };
```

---

## PASSO 9: TESTAR BOT

```bash
# Terminal 1: Rodar bot
npm start

# Terminal 2: Testar
# No Telegram, no grupo:
/start
/leads
/agendamentos

# Teste natural:
"Casal confirmou pra 12 de julho às 14h"
# Bot deve: atualizar Sheets + Calendar + fazer commit
```

---

## PASSO 10: DEPLOY NA VPS

```bash
# Fazer deploy com PM2 (mantém bot rodando 24/7)
npm install -g pm2

pm2 start index.js --name "negulha-bot"
pm2 save
pm2 startup

# Ver status
pm2 status
```

---

## CHECKLIST FINAL

- [ ] `.env` criado com todas credenciais
- [ ] `package.json` com dependências
- [ ] `index.js` criado e rodando
- [ ] Módulos de integração criados
- [ ] Parser reconhecendo intenções
- [ ] Testes básicos passando
- [ ] Bot respondendo no Telegram
- [ ] Commitando no GitHub
- [ ] Atualizando Google Sheets
- [ ] Criando eventos no Calendar

---

## PRÓXIMO PASSO

Documento 5: PROMPT_SISTEMA_BOT.md — instruções que você vai dar AO bot quando criar/usar.
