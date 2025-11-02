# Guia Rápido: CRM no Bubble (No‑Code)

## 1) Estrutura de dados (Data Types)
Crie os tipos:
- **User** (padrão do Bubble) — campos: `name (text)`, `role (text: admin/seller)`
- **Client** — campos: `name (text)`, `email (text)`, `phone (text)`, `company (text)`, `status (text: lead/proposal/won/lost)`, `created_at (date)`
- **Deal** — campos: `client (Client)`, `value (number)`, `stage (text: new/proposal/closing/won/lost)`, `expected_close (date)`, `created_at (date)`
- **Interaction** — campos: `client (Client)`, `user (User)`, `type (text)`, `notes (text)`, `date (date)`

## 2) Páginas
- **login**: Inputs de email/senha + workflow "Log the user in".
- **dashboard**: 3 cards com contagens (Do Search for Clients:count; Deals:count; Deals filtradas por stage=won:sum value).
- **clients**: Repeating Group (tipo Client). Form lateral para criar/editar.
- **deals**: Repeating Group (tipo Deal) com dropdown para `stage` e input para `value` e `expected_close`.

## 3) Workflows essenciais
- Botão **Salvar Cliente** → Action: Create a new Client (campos dos inputs).
- Botão **Salvar Negócio** → Action: Create a new Deal.
- Quando selecionar cliente na tabela → Action: Display data (Group com detalhes + RG de Interactions).
- Botão **Adicionar Interação** → Action: Create a new Interaction (client=current, user=Current User, date=Current date/time).

## 4) Regras de Privacidade (Privacy Rules)
- **Client**: Logged in users can read; Only creator or admin can modify.
- **Deal**: Logged in users can read; Only creator or admin can modify.
- **Interaction**: Logged in users can read; Only creator ou admin pode criar/editar.
- Use um campo no User `role` para permitir admin (When Current User's role is 'admin').

## 5) UI rápida (estilo HS Growth)
- Header laranja com logo, menu lateral escuro, cards com números.
- Ícones: usar plugin "Feather Icons".
- Botões principais laranja (#FF6600), fundos escuros (#0a0a0a / #161616) e textos brancos.

## 6) Automação (opcional)
- Plugin WhatsApp via API (ex: Zenvia) → Chamar API Connector com endpoint `sendMessage`.
- Agendar lembretes: "Schedule API Workflow" para follow‑ups (envio de e-mail / WhatsApp).

## 7) Deploy
- Crie versão live (Deploy Development to Live).
- Defina Roles e Privacy antes de abrir para equipe.
