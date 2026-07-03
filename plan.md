# LyricLab — Project Plan

## Šta gradimo i zašto

LyricLab je web aplikacija koja pomaže muzičarima da analiziraju i poboljšaju tekstove pesama koristeći AI.

Korisnik može:
- Uneti tekst pesme i dobiti AI analizu (struktura, emocija, snaga refrena, sugestije)
- Sačuvati istoriju analiza
- Porediti dve verzije iste pesme (A/B)
- Porediti svoju pesmu sa drugom pesmom

Zašto je ovo dobar portfolio projekat:
- Vidljivo korisno (svako razume šta radi)
- Pokazuje full-stack znanje: auth, baza, API, AI integracija
- Pokazuje sistem dizajn razmišljanje
- Live demo odmah impresionira na intervjuu

---

## Tech Stack

| Sloj | Tehnologija | Zašto |
|------|-------------|-------|
| Frontend | Next.js 14 App Router + TypeScript | Industrijski standard, SSR/SSG, file-based routing |
| Stilovi | Tailwind CSS | Brzo, utility-first, bez pisanja CSS fajlova |
| Backend | Next.js API Routes (Route Handlers) | Monorepo pristup, nema posebnog servera |
| Baza | PostgreSQL + Prisma ORM | Industrijski standard, type-safe upiti |
| Auth | NextAuth.js (email/password + Google OAuth) | Najlakši auth za Next.js |
| AI | Google Gemini 2.0 Flash API | Besplatno, moćno, lako za integraciju |
| Testovi | Vitest (unit/integration) + Playwright (e2e) | Moderni alati, traženi na intervjuima |
| Deploy | Vercel (frontend) + Supabase (PostgreSQL) | Besplatno, profesionalno |
| IDE AI | DeepSeek kroz Continue/Cline | Lokalni AI asistent za kod |

---

## Arhitektura (Sistem Dizajn)

```
[Browser]
    |
    | HTTP requests
    v
[Next.js App] ← vercel.com
    |
    |-- /app              ← Frontend (React komponente)
    |-- /app/api          ← Backend (API Routes)
    |
    |-- Prisma ORM
    |       |
    |       v
    |   [PostgreSQL] ← supabase.com
    |
    |-- Gemini API ← generativelanguage.googleapis.com
    |
    |-- NextAuth.js ← sesije i JWT tokeni
```

### Zašto ovakva arhitektura?

Next.js nam daje i frontend i backend u jednom projektu. Ovo se zove "monolith" pristup i idealan je za MVP. Kada projekat poraste, backend se može izdvojiti u poseban servis.

Prisma je "ORM" (Object Relational Mapper) — znači pišemo TypeScript kod, a Prisma ga prevodi u SQL. Nikada ne pišemo SQL ručno (osim kada učimo).

---

## Baza podataka (Data Model)

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

---

## Faze razvoja (5 dana)

### Dan 1 — Infrastruktura i Auth
- [ ] Next.js projekat setup (TypeScript, Tailwind, ESLint)
- [ ] PostgreSQL konekcija (Supabase besplatni tier)
- [ ] Prisma setup + migracije
- [ ] NextAuth.js (email/password registracija i login)
- [ ] Osnovna navigacija i layout

### Dan 2 — Core funkcionalnost (Songs + AI)
- [ ] CRUD za pesme (kreiranje, čitanje, brisanje)
- [ ] Gemini API integracija
- [ ] AI analiza teksta (struktura, emocija, snaga, sugestije)
- [ ] Prikaz rezultata analize

### Dan 3 — Istorija i Poređenje
- [ ] Stranica sa istorijom analiza
- [ ] A/B poređenje dve verzije pesme
- [ ] Poređenje dve različite pesme
- [ ] Čuvanje rezultata u bazi

### Dan 4 — UI/UX Polish + Testovi
- [ ] Finalni dizajn svih stranica
- [ ] Loading stanja, error handling, empty states
- [ ] Vitest unit testovi (helper funkcije, API logika)
- [ ] Playwright e2e testovi (login flow, analiza flow)

### Dan 5 — Deploy + Portfolio
- [ ] Vercel deploy
- [ ] Environment varijable na Vercelu
- [ ] README.md sa screenshotovima
- [ ] Portfolio stranica opis projekta
- [ ] Intervju priprema dokument

---

## API Endpoints

```
POST   /api/auth/register     ← registracija
POST   /api/auth/login        ← login (NextAuth)
GET    /api/songs             ← lista pesama korisnika
POST   /api/songs             ← nova pesma
DELETE /api/songs/:id         ← brisanje pesme
POST   /api/analyze           ← AI analiza jedne pesme
POST   /api/compare           ← AI poređenje dve pesme
GET    /api/analyses          ← istorija analiza
GET    /api/analyses/:id      ← jedna analiza
```

---

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

---

## KISS, DRY, SOLID, Clean Code pravila za ovaj projekat

**KISS** (Keep It Simple Stupid)
- Nema over-engineeringa. Ako može jednostavnije, bira se jednostavnije.
- Primer: ne pravimo mikroservise, koristimo monolith.

**DRY** (Don't Repeat Yourself)
- Svaka logika se piše jednom i reusuje.
- Primer: Gemini API poziv je u jednoj helper funkciji `/lib/gemini.ts`

**SOLID** (5 principa dobrog OOP dizajna)
- Single Responsibility: svaka funkcija/komponenta radi jednu stvar
- Open/Closed, Liskov, Interface Segregation, Dependency Inversion — učimo uz put

**Clean Code**
- Ime funkcije objašnjava šta radi: `analyzeWithGemini()` ne `doStuff()`
- Nema magic numbers/strings — sve u konstantama
- Komentari samo kada "zašto", ne "šta"

---

## Gemini AI — Šta tražimo od modela

### Analiza jedne pesme
```
Analiziraj ovaj tekst pesme:
- Žanr: [žanr]
- Tekst: [tekst]

Vrati JSON sa:
{
  "structure": "opis strukture (vers, refren, bridge...)",
  "emotion": "dominantna emocija i ton",
  "strengths": ["snaga 1", "snaga 2"],
  "weaknesses": ["slabost 1", "slabost 2"],
  "suggestions": ["sugestija 1", "sugestija 2"],
  "refrainScore": 8,  // 1-10
  "overallScore": 7   // 1-10
}
```

### Poređenje dve pesme
```
Poredi ova dva teksta pesama:
- Pesma A: [tekst]
- Pesma B: [tekst]

Vrati JSON sa:
{
  "songA": { ...analiza },
  "songB": { ...analiza },
  "comparison": "tekst poređenja",
  "winner": "A" | "B" | "tie",
  "reasoning": "zašto"
}
```

---

## Testovi

### Vitest (unit i integration)
- `analyzeWithGemini()` — mock Gemini, testiraj da li parsira JSON
- `compareWithGemini()` — isto
- API routes — testiraj response statuse
- Auth middleware — testiraj zaštitu ruta

### Playwright (e2e)
- Register → Login → Dodaj pesmu → Analiziraj → Vidi rezultat
- Login → Dodaj dve pesme → Poredi → Vidi rezultat
- Login → Istorija analiza

---

## Intervju priprema

Dokument `/interview-prep.md` se pravi posebno i sadrži:
- Sistem dizajn pitanja i odgovori za ovaj projekat
- "Walk me through your architecture" skript
- Česta React/Next.js pitanja
- Česta backend/API pitanja
- Česta baza podataka pitanja
- Šta bi radio drugačije (pokazuje zrelost)

---

## Slash komanda za novi chat

Kopiraj ovo na početak svakog novog chata:

```
/lyriclab

Radimo na LyricLab projektu. Plan je u plan.md.
Stack: Next.js 14, TypeScript, Tailwind, PostgreSQL, Prisma, NextAuth.js, Gemini 2.0 Flash API.
Pravila: KISS, DRY, SOLID, Clean Code. Testovi: Vitest + Playwright.
Ti si moj tutor, objašnjavaš sve kao malom detetu, bez povlađivanja.
Trenutna faza: [OVDE NAPIŠEŠ KOJA FAZA]
Poslednje što smo radili: [OVDE NAPIŠEŠ]
```

---

## Resursi za učenje (uz projekat)

- [Next.js dokumentacija](https://nextjs.org/docs) — App Router sekcija
- [Prisma dokumentacija](https://www.prisma.io/docs) — Getting Started
- [NextAuth.js dokumentacija](https://next-auth.js.org)
- [Gemini API dokumentacija](https://ai.google.dev/docs)
- [Vitest dokumentacija](https://vitest.dev)
- [Playwright dokumentacija](https://playwright.dev)
