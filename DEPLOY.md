# Deploy rápido (Render + Vercel + Docker)

## Render (Backend)
1. Suba a pasta `backend/` para um repositório no GitHub.
2. No Render.com → New → Web Service → conecte seu repo.
3. Build: `npm install` · Start: `node src/index.js`
4. Vars necessárias:
   - `PORT` (4000)
   - `JWT_SECRET`
   - `DATABASE_FILE` (ex: `/data/data.sqlite`)
   - `ORIGIN` (URL do frontend ex.: `https://seu-frontend.vercel.app`)
   - (opcional) `SEED=true` na primeira execução
   - (opcional) `WHATSAPP_*`, `SMTP_*`
5. Adicione um **Persistent Disk** (Render) montado em `/data` para o SQLite.

## Vercel (Frontend)
1. Suba `frontend/` no GitHub.
2. Em Vercel → New Project → selecione `frontend`.
3. Adicione env `VITE_API_URL` apontando para o backend (Render).
4. Deploy.

## Docker (local ou VPS)
```bash
docker compose up --build
```
- Frontend: http://localhost:5173
- Backend: http://localhost:4000

> Produção: troque `ORIGIN` no backend para a URL do frontend e **remova** `SEED=true`.
