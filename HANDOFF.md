# HANDOFF — dKonrad Run (continuação em outra máquina)

> Cole o conteúdo deste arquivo (ou peça pro Claude Code abri-lo) ao iniciar uma sessão
> nova noutra máquina. Ele dá todo o contexto pra continuar sem o histórico do chat.

## O QUE É
**dKonrad Run** — app mobile (single-file HTML/CSS/JS) que é **conector + dashboard do Strava**,
com indicadores de evolução na corrida. Faz parte da suíte pessoal do Diego (Foco, english-trainer,
Horas) integrada a um **HUB Pessoal**.
- App no ar: **https://dkonrad88.github.io/strava/**
- Mobile-first, tema escuro, navbar inferior de **6 abas**, moldura de celular no desktop.

## ONDE TUDO VIVE
- **App (PÚBLICO):** `github.com/dKonrad88/strava` — arquivo único **`index.html`** (todo o app).
  Publica via **GitHub Pages** em `dkonrad88.github.io/strava` (rebuild ~1 min após push).
- **Garmin sync (PRIVADO):** `github.com/dKonrad88/garmin-sync` — robô **Python + GitHub Actions**
  (cron diário ~06h BRT) que puxa recuperação do Garmin (não-oficial, via `garminconnect`).
- **Supabase (suíte pessoal):** id `jlouesrrmqeauzlgvrpw` · URL `https://jlouesrrmqeauzlgvrpw.supabase.co`
  - Tabelas (padrão key/value, RLS `auth.uid() = user_id`):
    - `strava_state` — **contrato** lido pelo app + HUB: chaves `athlete`, `activities`, `totals`, `meta`
    - `strava_tokens` — **PRIVADA**, sem policy de cliente (só Edge Function com service_role)
    - `garmin_state` — chaves `daily`, `fitness`, `trends`, `meta`
  - Edge Functions: **`strava-sync`** (`verify_jwt=true`: exchange/sync/disconnect/subscribe),
    **`strava-webhook`** (`verify_jwt=false`: Strava Push Subscription → tempo real).
  - Strava API app **client_id=261805**; o `client_secret` é secret no Supabase (`STRAVA_CLIENT_SECRET`).
- **Contrato de dados:** ver [`CONTRATO.md`](CONTRATO.md) (LEIA antes de mexer em dados).

## ARQUITETURA (regras de ouro)
- Login Supabase **compartilhado** na suíte (mesma origin `dkonrad88.github.io` + mesmo projeto/anon
  → mesmo usuário, sem novo cadastro). Só a **chave ANON** (pública, protegida por RLS) fica no front.
- **Nenhum segredo no HTML.** `client_secret` do Strava e credenciais do Garmin ficam **server-side**
  (secrets do Supabase / secrets do GitHub Actions).
- **Tempo real** = webhook do Strava chama a Edge Function que re-sincroniza sozinho.
- **Garmin** = robô diário não-oficial (Garmin não tem API aberta p/ pessoa física). Secrets no repo
  `garmin-sync`: `GARMIN_EMAIL`, `GARMIN_PASSWORD`, `SUPABASE_EMAIL`, `SUPABASE_PASSWORD`.

## O QUE JÁ ESTÁ PRONTO (6 abas)
- **Hoje** — coach do dia, KPIs dos últimos 7 dias, índice de carga, rumo aos 21k, resumo do histórico.
- **Evolução** — volume semanal, ritmo por corrida, IEA (eficiência aeróbica), cadência,
  **Forma (CTL/ATL/TSB)**, tabela mês a mês.
- **Treinos** — lista filtrável + detalhe + comparação automática por distância.
- **Metas** — **VDOT + paces de treino (Jack Daniels)**, previsões de prova (Riegel), mural de recordes.
- **Insights** — Termômetro de carga (ACWR), eficiência fácil, **polarização 80/20** (zona cinza),
  IEA, **desacoplamento aeróbico (Pa:HR)**.
- **Prontidão** — Garmin: FC repouso, HRV, sono, Body Battery, stress + **score de prontidão**.

## COMO FAZER MUDANÇAS
- **App:** editar o único `index.html`. Antes de commitar, validar que `{}`, `()`, `[]` e crases
  estão **balanceados** (sem Node/JS local, valida-se por contagem).
- **Backend:** via Supabase (MCP do Supabase, se conectado; senão CLI/dashboard). Edge functions
  com `deploy_edge_function`; schema com `apply_migration`; rodar `get_advisors` (security) após DDL.
- **Git:** usuário `Diego Konrad <konraddiego@gmail.com>`. Commits terminam com:
  `Co-Authored-By: Claude <noreply@anthropic.com>`. Só commitar/push quando o Diego pedir.

## CONTEXTO DO ATLETA (pra indicadores fazerem sentido)
Diego volta de pausa de 2 semanas, mirando uma **meia maratona (21k)**; maior longo = **16,1 km**.
Tendência a treinar demais na **"zona cinza" (Z3)** — por isso VDOT/paces e 80/20 importam.
**VDOT ~40.** Zonas reais (FC e ritmo) já estão no código.

## PRIMEIRAS AÇÕES NUMA SESSÃO NOVA
1. Clonar os repos (sugestão: `~/Documents/GitHub/`):
   ```
   gh repo clone dKonrad88/strava
   gh repo clone dKonrad88/garmin-sync
   ```
2. Ler `strava/CONTRATO.md` e passar o olho no `strava/index.html`.
3. Perguntar ao Diego o que ele quer melhorar (modo "melhorar aos poucos").

## PENDÊNCIAS / IDEIAS
- Melhorar a extração de **VO₂máx / Training Readiness** no `garmin-sync` (vieram vazios —
  depende do modelo do relógio).
- Seguir refinando indicadores conforme o Diego pedir.
