# LyricLab — Claude Session Start

> **Kopiraj ovo na početak chata sa Claude-om.**
> Claude će automatski pročitati `AGENTS.md` (preko `CLAUDE.md` reference) i `plan.md`.

---

## Kontekst

Radimo na **LyricLab** projektu — web aplikaciji koja pomaže muzičarima da analiziraju i poboljšaju tekstove pesama koristeći AI (Gemini 2.0 Flash API).

## Trenutni status

**Faza:** Dan 1 — Infrastruktura i Auth (u toku)
**Poslednje urađeno:** Next.js boilerplate kreiran, osnovni fajlovi postavljeni
**Sledeći korak:** PostgreSQL konekcija (Supabase) + Prisma setup

## Šta je urađeno do sada

- [x] Next.js 16.2.10 projekat kreiran (TypeScript, Tailwind CSS v4, ESLint)
- [x] Boilerplate očišćen (default SVG ikonice obrisane, page.tsx i layout.tsx prepisani za LyricLab)
- [x] README.md ažuriran za LyricLab
- [x] `.clinerules` i `system-prompt.md` ažurirani (pravila za Cline i Continue)
- [x] `AGENTS.md` ažuriran (pravila za DeepSeek)
- [x] `MASTER_PROMPT.md` obrisan (sadržaj prebačen u `.clinerules`)
- [x] Verzija usaglašena: plan je pisan za Next.js 14, ali je instaliran Next.js 16 — odlučeno da se nastavi sa 16

## Šta NEDOSTAJE (prvi koraci)

1. PostgreSQL baza (Supabase) — treba account + connection string
2. Prisma ORM — `prisma init`, schema, migracija
3. NextAuth.js — email/password + Google OAuth
4. Osnovna navigacija i layout (navbar, footer, auth guard)

## Tech Stack (striktno)

| Sloj | Tehnologija |
|------|-------------|
| Frontend/Backend | Next.js 16 App Router |
| Jezik | TypeScript (stroga tipizacija) |
| Stilovi | Tailwind CSS v4 |
| Globalni State | Zustand |
| Baza | PostgreSQL (Supabase) + Prisma ORM |
| Autentifikacija | NextAuth.js (JWT, sesije, middleware) |
| AI | Google Gemini 2.0 Flash API |
| Testovi | Vitest + Playwright |
| Deploy | Vercel + Supabase |

## Arhitektura

```
[Browser]
    ↓ HTTP zahtev
[Middleware] — proverava autentifikaciju
    ↓
[API Route] — business logika
    ├→ Gemini API (AI analiza)
    ├→ Prisma ORM → PostgreSQL (Supabase)
    ↓ HTTP odgovor
[Browser] — React re-render
```

## Data Model

```prisma
model User {
  id        String    @id @default(cuid())
  email     String    @unique
  name      String?
  password  String?   // null ako koristi Google OAuth
  createdAt DateTime  @default(now())
  analyses  Analysis[]
  songs     Song[]
}

model Song {
  id        String    @id @default(cuid())
  title     String
  lyrics    String    @db.Text
  genre     String?
  userId    String
  user      User      @relation(fields: [userId], references: [id])
  createdAt DateTime  @default(now())
  analyses  Analysis[]
}

model Analysis {
  id          String   @id @default(cuid())
  songId      String
  song        Song     @relation(fields: [songId], references: [id])
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  result      Json     // AI odgovor u JSON formatu
  createdAt   DateTime @default(now())
}

model Comparison {
  id          String   @id @default(cuid())
  songAId     String
  songBId     String
  userId      String
  result      Json
  createdAt   DateTime @default(now())
}
```

## API Endpoints

```
POST   /api/auth/register       ← registracija
POST   /api/auth/login          ← login (NextAuth)
GET    /api/songs               ← lista pesama korisnika
POST   /api/songs               ← nova pesma
DELETE /api/songs/:id           ← brisanje pesme
POST   /api/analyze             ← AI analiza jedne pesme
POST   /api/compare             ← AI poređenje dve pesme
GET    /api/analyses            ← istorija analiza
GET    /api/analyses/:id        ← jedna analiza
```

## Stranice (Routes)

```
/                    ← Landing page (nije zaštićena)
/login               ← Login forma
/register            ← Registracija forma
/dashboard           ← Lista pesama korisnika (zaštićena)
/songs/new           ← Unos nove pesme (zaštićena)
/songs/:id           ← Detalji pesme + analiza (zaštićena)
/compare             ← Odabir dve pesme za poređenje (zaštićena)
/history             ← Istorija svih analiza (zaštićena)
```

## Pravila rada

1. Idemo strogo po planu iz `plan.md`
2. Na kraju svakog odgovora postavi checkpoint pitanje
3. Nikada ne trči napred — čekaj potvrdu pre sledećeg koraka
4. Obavezno koristi 3 stanja: LOADING, SUCCESS, ERROR
5. Service Layer Pattern: CRUD metode u `services/` fajlovima, ne u komponentama
6. Autentifikacija u middleware-u, autorizacija na nivou reda u bazi

## Reference dokumenti

- `plan.md` — plan razvoja po danima
- `ARCHITECTURE_OVERVIEW.md` — detaljna arhitektura
- `SYNOPSIS.md` — šta pravimo i zašto
- `AGENTS.md` — pravila za AI asistente
- `.clinerules` — sistemska pravila (za Cline)
- `system-prompt.md` — sistemska pravila (za Continue)
