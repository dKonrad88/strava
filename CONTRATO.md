# Contrato de dados — dKonrad Run (Strava)

> **Para o HUB Pessoal.** Este app GRAVA aqui; o HUB só LÊ (read-only).
> Se este formato mudar, a dashboard do HUB quebra — então é **estável/versionado**.

- **Projeto Supabase:** `jlouesrrmqeauzlgvrpw` (suíte pessoal — mesmo login/origin dos outros apps)
- **App:** https://dkonrad88.github.io/strava/
- **Edge Function:** `strava-sync` (`verify_jwt` on) — faz OAuth, refresh de token e grava o estado.

## Tabela pública (o contrato) — `public.strava_state`
Padrão key/value idêntico a `foco_state` / `horas_state`.

| coluna | tipo | |
|---|---|---|
| `user_id` | uuid | FK `auth.users`, parte da PK |
| `key` | text | parte da PK |
| `value` | jsonb | o conteúdo |
| `updated_at` | timestamptz | default `now()` |

PK `(user_id, key)` · **RLS** `auth.uid() = user_id` (cada um só lê o próprio).

### Como o HUB lê (read-only)
```js
const { data } = await sb.from('strava_state').select('key,value');
const st = {}; data.forEach(r => st[r.key] = r.value);
// st.athlete, st.activities, st.totals, st.meta
```

## Chaves e formato do `value`

### `key = "meta"` — status de conexão/sync (use pra mostrar "conectado / última sync")
```json
{ "connected": true, "athlete_id": 17979924, "last_sync": "2026-06-29T15:00:00Z",
  "last_activity_at": "2026-06-27T05:28:09", "scope": "read,activity:read_all,profile:read_all", "error": null }
```

### `key = "athlete"`
```json
{ "strava_id": 17979924, "name": "Diego Konrad", "city": "Westfalia",
  "state": "Rio Grande do Sul", "country": "Brazil", "sex": "M", "weight": 75,
  "ftp": 167, "profile_pic": "https://...", "measurement": "meters",
  "updated_at": "2026-06-29T15:00:00Z" }
```

### `key = "totals"`
```json
{ "updated_at": "2026-06-29T15:00:00Z",
  "week":  { "runs": 2, "distance_km": 19.1, "time_s": 6369, "elev_m": 144, "gym": 3, "load": 273 },
  "month": { "runs": 12, "distance_km": 95.2, "time_s": 33000, "elev_m": 540 },
  "ytd":   { "runs": 66, "distance_km": 480.0, "time_s": 0, "elev_m": 0, "calories": 0 } }
```

### `key = "activities"`
```json
{ "synced_at": "2026-06-29T15:00:00Z", "count": 66, "items": [
  { "id": "19083212011", "name": "Longão de sábado", "type": "Run", "sport": "Run",
    "start": "2026-06-27T05:28:09", "distance_m": 12034.6, "moving_s": 4009, "elapsed_s": 4009,
    "elev_gain_m": 93, "pace_s_per_km": 333.1, "avg_speed_ms": 3.0, "avg_hr": 164.0,
    "max_hr": 185, "avg_cadence": 86.3, "calories": null, "relative_effort": 197,
    "kudos": 4, "pr_count": 0, "indoor": false }
] }
```
- `pace_s_per_km` e `avg_hr` = `null` quando não se aplica / não há sensor.
- `items` ordenado do mais recente pro mais antigo. MVP sincroniza até ~300 atividades recentes.
- `calories` costuma vir `null` no resumo do Strava (só no detalhe da atividade) — reservado pra fase futura.

## Tabela privada — `public.strava_tokens` (NÃO é contrato; NÃO ler do HUB)
Guarda `access_token` / `refresh_token` do Strava. **RLS sem policy de cliente**: só a Edge
Function (service_role) acessa. O front e o HUB **nunca** veem os tokens. Não dependa dela.

## Versão
**v1** (2026-06-29). Mudanças que removam/renomeiem chaves ou campos = **breaking** → avisar o HUB.
