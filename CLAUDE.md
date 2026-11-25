# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

This is a single-file PWA. All HTML, CSS, and JavaScript live in `index.html`. There is no build step, no bundler, no package manager — just static files served directly.

**Backend: Supabase**
- URL and publishable key are hardcoded at the top of the `<script>` block in `index.html`
- Supabase is used for auth (email/password + Google OAuth) and as the database
- The Gemini API key is stored in the Supabase `config` table (`key='gemini_key'`) and cached in `localStorage`

**AI Chat: Google Gemini**
- Uses `gemini-pro-latest` via the REST API with function calling
- The `registrar_gasto` function declaration triggers a confirmation UI before saving to Supabase
- Chat history is persisted per context (group or personal) in `localStorage` (last 40 messages)

## Database schema (inferred)

| Table | Key columns |
|---|---|
| `profiles` | `id`, `nombre` |
| `gastos` | `user_id`, `grupo_id`, `concepto`, `importe`, `moneda`, `quien_paga`, `metodo`, `categoria`, `fecha`, `notas`, `es_personal` |
| `grupos` | `id`, `nombre`, `descripcion`, `moneda`, `owner_id` |
| `grupo_miembros` | `grupo_id`, `user_id`, `rol` |
| `gasto_participantes` | `gasto_id`, `user_id`, `porcentaje` |
| `invitaciones` | `id`, `grupo_id`, `created_by`, `token`, `used_at` |
| `config` | `key`, `value` |

## Screen & navigation model

Screens are fixed-position `<div class="screen">` elements. Navigation works by toggling `.hidden` and `.slide-left` CSS classes — no router. The active screen is always fully visible; others are translated off-screen.

Screens: `loading` → `auth` → `home` → `grupo` / `personal` / `profile`

Drawers (bottom sheets) are shown by adding `.show` to `.drawer` and `.drawer-bg`. Two drawers exist: `drawer-gasto` (add expense) and `drawer-grupo` (create group).

## Global state

All mutable state is held in module-level `let` variables:
- `user`, `profile` — current session
- `gastos`, `grupos` — cached lists
- `currentGrupo`, `currentGrupoMiembros` — active group context
- `selQuien`, `selPara`, `selCat`, `selMet` — drawer form selections
- `addingGastoGrupoId`, `addingPersonal` — context for which screen triggered the drawer
- `chatMsgs`, `pendingArgs` — AI chat state

## Running locally

Open `index.html` directly in a browser or serve it with any static file server, e.g.:

```
npx serve .
# or
python -m http.server
```

The service worker (`sw.js`) caches `index.html` and `manifest.json` under cache key `gastos-app-v4`. Hard-reload or bump the cache version when testing SW changes.

## Key constants

- `CATS` — expense categories (emoji + label strings)
- `METS` — payment methods
- `COLORS` — avatar color palette (assigned deterministically from `nombre.charCodeAt(0)`)
