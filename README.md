# CRM Backend (Express + SQLite + JWT)

## Rodar local
```bash
cd backend
cp .env.example .env
npm install
npm run dev
```
API em `http://localhost:$PORT` (default 4000).

Usuário seed:
- email: `admin@hsgrowth.com`
- senha: `admin123`

## Endpoints principais
- `POST /api/auth/login`
- `GET /api/clients` | `POST /api/clients` | `PUT /api/clients/:id` | `DELETE /api/clients/:id`
- `GET /api/deals` | `POST /api/deals` | `PUT /api/deals/:id` | `DELETE /api/deals/:id`
- `GET /api/clients/:id/interactions` | `POST /api/clients/:id/interactions`

## WhatsApp (Zenvia - opcional)
Configure no `.env`:
```
WHATSAPP_PROVIDER=zenvia
WHATSAPP_BASE=https://api.zenvia.com/v2
WHATSAPP_API_KEY=<sua_api_key>
WHATSAPP_FROM=<seu_numero_remetente>
```
Enviar mensagem (autenticado):
- `POST /api/whatsapp/send` body: `{ "to": "55DDDNYYYYYYY", "message": "Olá!" }`

### Provedores suportados
- `zenvia` (exemplo implementado)
- `wati` (exemplo implementado)
- `gupshup` (exemplo implementado)
Defina `WHATSAPP_PROVIDER` no `.env`.

## Roles e permissões
- `admin`: acesso total
- `manager`: pode gerenciar tarefas e dados (exclui tasks), ver relatórios
- `seller`: acesso básico de CRM (listar/criar/editar onde aplicável)

## Tasks / Agenda
- CRUD de tarefas em `/api/tasks`
- Rodar lembretes: `POST /api/tasks/run-reminders` (manual)


## E-mail (SMTP) para lembretes
Configure no `.env` as variáveis SMTP_* (SendGrid/Mailgun/Outro). Use `SMTP_TEST_TO` para testes rápidos.
