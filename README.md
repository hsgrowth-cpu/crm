# HS CRM — Kommo-style (v5)
Recursos: Kanban, Inbox WhatsApp com **sincronia**, **retries/backoff**, **templates Meta**, **SLA horário comercial**, **respostas rápidas**, **menu interativo**, **multi-número**, PWA, Reports, Seeds.

## Instalação
```bash
pnpm install
cp .env.example .env.local
pnpm prisma generate
pnpm prisma migrate dev --name init
pnpm prisma db seed
pnpm dev
```

## Cron (Vercel)
- POST `/api/cron/wa-retry` a cada 5 min.

## WhatsApp
- Envio texto: `POST /api/messages/whatsapp/send` (aceita `phoneNumberId`/`accessToken`).
- Envio template: `POST /api/messages/whatsapp/template`.
- Envio menu (interactive list): `POST /api/messages/whatsapp/interactive`.
- Webhook: `POST /api/webhooks/whatsapp` (assinar messages + message status).
- Catálogo Meta: `GET /api/whatsapp/templates`.
- Read receipts: `POST /api/messages/whatsapp/read`.

## SLA
Calculado apenas no horário 09:00–18:00 (America/Manaus).

MIT © 2025


## Novidades (v6)
- **Retries distribuídos** com Upstash Queue (producer no webhook; consumer em `/api/cron/worker`).
- **Roteamento por skill/turno**: `/api/routing/assign` usa `User.skills[]` + `Team.shifts` (ex.: [{"day":1,"start":"09:00","end":"18:00"}]).
- **SLA configurável por equipe**: `Team.businessStartHour`, `Team.businessEndHour`, `Team.timezone`.
- **Quick Replies com tags e variáveis**: `QuickReply.tags[]` e envio por tag com `POST /api/messages/whatsapp/quick` (body `{ to, tag, vars, contactId }`).
- **Relatórios avançados**: `GET /api/reports/advanced` — previsão ponderada (valor × probabilidade) e volume de inbound por hora.
- **Cron worker**: agende POST em `/api/cron/worker` (1–2 min) e `/api/cron/wa-retry` (5 min).

### Exemplo de Quick Reply com variáveis
- Salve uma resposta com body: `Olá {{nome}}, sua proposta {{numero}} foi enviada.` e tag `proposta`.
- Envie:
```bash
curl -X POST "$BASE/api/messages/whatsapp/quick" -H "Content-Type: application/json" -d '{
  "to":"55DDDNUMERO",
  "tag":"proposta",
  "vars": { "nome":"Hallysson", "numero":"#123" },
  "contactId":"..."
}'
```



## v7 — Omnichannel + Meta Ads
- **Canais adicionados**: Instagram DM, Facebook Messenger, TikTok (DM *best-effort*), além do WhatsApp.
- **Webhooks**: 
  - IG: `/api/webhooks/instagram`
  - FB: `/api/webhooks/facebook`
  - TikTok: `/api/webhooks/tiktok`
- **Envio**:
  - IG: `POST /api/messages/instagram/send`
  - FB: `POST /api/messages/facebook/send`
  - TikTok: `POST /api/messages/tiktok/send` (*pode exigir permissões específicas*)
- **Inbox**: seletor de canal (WhatsApp/Instagram/Facebook/TikTok) para enviar do CRM.
- **Meta Ads Manager**:
  - Campanhas: `GET /api/ads/meta/campaigns`
  - Insights: `GET /api/ads/meta/insights`
  - UI: página **/ads** com gráfico de gasto 7d e lista de campanhas.
- **ENV**: `META_APP_ID`, `META_APP_SECRET`, `META_ACCESS_TOKEN`, `META_AD_ACCOUNT_ID`, `IG_BUSINESS_ID`, `FB_PAGE_ID`, `TIKTOK_APP_ID`, `TIKTOK_APP_SECRET`, `TIKTOK_ACCESS_TOKEN`.

### Passos para configurar os webhooks (Meta)
1. No app Meta, habilite Webhooks para **Instagram** e **Page** (Messenger).  
2. Aponte para `https://SEU_DOMINIO/api/webhooks/instagram` e `/api/webhooks/facebook`.  
3. Use o **verify token** de sua escolha (o GET já responde com `hub.challenge`).  
4. Conceda permissões exigidas e associe a **FB Page** e **IG Business** corretas.

### Observação sobre TikTok
- O envio de DM pode depender de permissões da conta Business; o endpoint de envio está como **best-effort**.


## v8 — Editor Visual de Automations • Inbox Omnichannel Unificada • Ads Holístico • RBAC • Auditoria
- **Automations (Editor Visual)**: CRUD em `/automations/editor` e rotas `/api/automations/flows`. Nó demo: `TRIGGER:message_received` → `ACTION:send_whatsapp_text`.
- **Inbox Unificada**: threads por contato somando todos os canais em `/inbox/unified`.
- **Ads Holístico**: Meta + Google + TikTok — página `/ads/overview` (comparação de gasto por dia).
- **RBAC granular**: checagem por recurso com `x-role` (ADMIN/MANAGER/SELLER/VIEWER) e **auditoria** persistida (`AuditLog`). Use Auth.js em produção.
- **API Guard + Audit**: wrapper de exemplo para rotas protegidas; logs em `AuditLog`.

### Rodar
```bash
pnpm install
pnpm prisma generate
pnpm prisma migrate dev --name v8_upgrade
pnpm dev
```

MIT © 2025


## v9 — Execução real de Automations • SLA por canal • Transferência • OAuth placeholders • ACL por objeto • Export Audit
- **Engine de Automations** com **IF / THEN / WAIT / BRANCH** e **scheduler** (rota `POST /api/cron/scheduler`).
- **SLA por canal** (cálculo respeitando horário comercial por equipe).
- **Transferência de atendimento**: `POST /api/inbox/transfer`.
- **OAuth placeholders** para Google/TikTok Ads (`/api/oauth/google`, `/api/oauth/tiktok`).
- **RBAC por objeto** com tabela **Acl** (`canRead/canWrite` por recurso `contacts:ID`, `deals:ID`, etc.).
- **Export de auditoria** em CSV: `GET /api/audit/export`.

### Rodar
```bash
pnpm install
pnpm prisma generate
pnpm prisma migrate dev --name v9_upgrade
pnpm dev
```

### Cron
- Agende `POST /api/cron/scheduler` (1–5 min) para executar nós **WAIT**.


## v10 — Nós avançados, Branch múltiplo, Versionamento, Relatórios e OAuth callbacks
**Novos nós de automations**
- `ACTION:send_email_resend` — envia e-mail via Resend.
- `ACTION:create_activity` — cria tarefa/compromisso.
- `ACTION:create_deal` — cria negócio e injeta `dealId` no contexto.
- `ACTION:move_stage` — move estágio do negócio.
- `ACTION:update_field` — atualiza campo em `contact`/`deal`.
- `BRANCH` — múltiplos caminhos condicionais por `expr` + `defaultNext`.

**Versionamento de fluxos**
- `POST /api/automations/flows/:id/versions` (snapshot)
- `GET /api/automations/flows/:id/versions`
- `POST /api/automations/flows/:id/restore`

**Relatórios de automations**
- `GET /api/automations/reports` — total/ok/erros e contagem por nó.

**OAuth callbacks (exemplo)**
- `GET /api/oauth/google/callback`
- `GET /api/oauth/tiktok/callback`
(Salva tokens em `OAuthToken` — ajuste troca de `code` → tokens reais.)

### Rodar upgrade
```bash
pnpm install
pnpm prisma generate
pnpm prisma migrate dev --name v10_upgrade
pnpm dev
```

MIT © 2025


## v11 — Nós WA template/mídia • Import CSV • Proposta (HTML) • Canvas DnD • KPIs • Playbooks
- **WhatsApp**: `sendWhatsAppText`, `sendWhatsAppTemplate`, `sendWhatsAppMedia` (Graph v20).
- **Proposta**: `POST /api/proposals/send` → gera **HTML de proposta** e envia via Resend.
- **Import CSV**: `POST /api/import/contacts` (texto CSV) com colunas `name,email,phone,tags`.
- **Engine**: novos nós `send_whatsapp_template`, `send_whatsapp_media`, `create_contact`, `send_proposal_email`.
- **Canvas visual**: `/automations/canvas` (drag/zoom básico, export de nós).
- **KPIs**: `/api/automations/reports/kpi` (conversões e erros básicos).
- **Playbooks**: `GET /api/automations/playbooks` e exemplos em `docs/playbooks/*.json`.

### Observações
- O envio de mídia/templated WhatsApp requer **templates aprovados** e permissões do número de telefone (Cloud API). Preencha `WHATSAPP_*` no `.env`.
- Para converter a proposta em **PDF real**, conecte um serviço de render (ex.: Puppeteer/Chromium via serverless functions) — a v11 envia HTML pronto por e-mail.


## v12 — PDF real, Canvas avançado, Import CSV (mapeamento), Template Manager e KPIs com gráficos
### Novidades
- **PDF real** com `puppeteer-core` + `@sparticuz/chromium`:
  - `POST /api/utils/pdf` — recebe `{ html }` e retorna o PDF.
  - `POST /api/proposals/send-pdf` — gera PDF da proposta e envia por e-mail (Resend) como **anexo**.
  - Ative com `PDF_ENABLE=true` e defina `BASE_URL` para a função chamar a si mesma.
- **Canvas avançado** (`/automations/canvas-advanced`) com arestas visuais e botão **Salvar grafo** → cria um flow no backend.
- **Import CSV (UI)** em `/import/contacts` — mapeie colunas e importe pela API existente.
- **Template Manager** (`/templates` + `/api/templates`) para **WhatsApp Template** e **E-mail** (HTML).
- **KPIs de Automations com gráficos** em `/reports/automations` (Chart.js) com **funil** e contadores.

### Instalação de pacotes (se necessário)
Veja `docs/v12-packages.txt` e adicione ao `package.json`:
- `puppeteer-core`
- `@sparticuz/chromium`

### Observações
- Em Vercel, mantenha `runtime = 'nodejs'` para rotas que usam PDF.
- WhatsApp Template requer modelos aprovados na **Meta**.
- Para PDF com fontes personalizadas, injete via `<style>`/`@font-face` no HTML.


## v12 — PDF real • Canvas avançado • Import/Mapeamento CSV • Template Manager • KPIs de Automations
**PDF real de Propostas**
- `POST /api/render/pdf` (serverless **Node runtime**) usa **puppeteer-core + chrome-aws-lambda**.
- `POST /api/proposals/send-pdf` — renderiza o HTML da proposta em **PDF** e envia por e-mail (Resend) com anexo.

**Canvas avançado**
- `/automations/canvas-advanced` — arraste nós, conecte arestas visualmente e **salve grafo** no fluxo.

**Importador CSV com mapeamento visual**
- Página `/import/contacts` para **validar e importar** CSV com campos mapeáveis.

**Template Manager**
- Página `/templates` para criar/editar **templates WhatsApp e E-mail** com variáveis e **prévia**.

**Relatórios**
- Página `/reports/automations` com gráficos de **KPIs** (total/ok/erros/conversões) e **execuções por nó**.

### Dependências
- Adicionadas: `puppeteer-core` e `chrome-aws-lambda` no `package.json`.
- **Vercel**: mantenha este endpoint como função **Node.js** (não Edge).

### Como rodar upgrade
```bash
pnpm install
pnpm prisma generate
pnpm prisma migrate dev --name v12_upgrade
pnpm dev
```

MIT © 2025

## v13 — Templates persistentes & aprovação • Autolayout+Validação • Import Lote c/ UNDO • Temas de Proposta • E‑sign
**Templates persistentes**
- Rotas: `GET/POST /api/templates`, `POST /api/templates/:id/approve`.
- Marcação de **aprovado** no CRM (a aprovação oficial de **templates WhatsApp** ocorre no **Meta Business**).

**Canvas — Autolayout e Validação**
- `GET /api/automations/flows/:id/validate` (TRIGGER/END, nós isolados, **ciclos**).
- `POST /api/automations/flows/:id/autolayout` → posições sugeridas por **camadas (topológico)**.

**Importador em lote + UNDO**
- `POST /api/import/contacts/batch` → cria **ImportJob** com `report.errors` e **UndoLog** por item criado.
- `POST /api/import/jobs/:id/undo` → reverte contatos importados (marca job como `undone`).

**Propostas com temas + E‑sign**
- `POST /api/proposals/themes` e `GET /api/proposals/themes` (salva **CSS** e **logo**).
- `renderProposalHTML()` aceita `theme` (CSS/logo).
- **Assinatura eletrônica simples**:
  - `POST /api/sign/request` → gera link único `/sign/{token}`
  - `GET/POST /api/sign/{token}` e página `/sign/[token]` com botão **Assinar**.

### Upgrade
```bash
pnpm install
pnpm prisma generate
pnpm prisma migrate dev --name v13_upgrade
pnpm dev
```

MIT © 2025
