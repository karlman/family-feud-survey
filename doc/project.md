# family-feud-survey

## Links
- **GitHub:** https://github.com/karlman/feud-survey

## Overview
A Next.js web app for crowdsourcing Family Feud survey answers. Admins create surveys, share QR codes, collect responses, and use Claude to tabulate and group similar answers. Results export as JSON that the game server (`family-feud-game`) can load directly.

## Stack
- **Next.js 14** (App Router, TypeScript, `output: 'standalone'`)
- **Prisma ORM** + **SQLite** (local dev) / persisted Azure Files (production)
- **Tailwind CSS**
- **Anthropic SDK** — Claude for AI tabulation (`claude-opus-4-5`)
- **jose** — JWT session auth
- **qrcode** — QR code generation for survey links

## Key Routes
| Route | Access | Purpose |
|-------|--------|---------|
| `/admin` | Auth | Survey dashboard |
| `/admin/surveys/new` | Auth | Create a survey |
| `/admin/surveys/[id]` | Auth | Edit, view QR code, lock/unlock |
| `/admin/surveys/[id]/results` | Auth | Response counts, AI tabulation, export |
| `/survey/[token]` | Public | Respondent-facing survey form |
| `/login` | Public | Admin login |

## Workflow
1. Admin creates a survey with questions
2. QR code generated → respondents scan and answer anonymously
3. Admin monitors response counts (target: ~100 per question)
4. Admin triggers AI tabulation → Claude groups similar answers and scores them
5. Results exported as JSON → loaded into `family-feud-game`

## Export Format
```json
{
  "title": "Survey Title",
  "rounds": [
    {
      "question": "Name something in a kitchen",
      "answers": [
        { "text": "Refrigerator", "points": 45 },
        { "text": "Stove", "points": 22 }
      ]
    }
  ]
}
```
Points are scaled so totals ≈ 100 per question.

## Auth
Simple JWT cookie auth. `ADMIN_PASSWORD` set via environment variable. Session valid 7 days.

## Production Deployment
- Hosted on **Azure App Service** via Docker container
- SQLite DB at `/home/data/feud.db` on persistent Azure Files storage
- `startup.sh` runs `prisma migrate deploy` on container start
- Deploy via Docker → Azure Container Registry → App Service

## Environment Variables
| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | `file:/home/data/feud.db` (prod) |
| `ADMIN_PASSWORD` | Admin login password |
| `SESSION_SECRET` | JWT signing secret |
| `ANTHROPIC_API_KEY` | Claude API access |
| `NEXT_PUBLIC_APP_URL` | Public base URL |

## Key Files
- `src/app/` — Next.js App Router pages and API routes
- `prisma/schema.prisma` — Database schema
- `Dockerfile` — Container build (targets `linux/amd64`)
- `startup.sh` — Container entrypoint (migrations + server start)

## Related Repos
- `family-feud-game` — Consumes the exported survey JSON for questions
