# LyricLab — Project Synopsis

## Šta pravimo?

**LyricLab** je web aplikacija koja pomaže muzičarima i tekstopiscima da analiziraju i poboljšaju tekstove pesama koristeći AI (Gemini 2.0 Flash).

## Problem koji rešavamo

Tekstopisac napravil pesmu i pita se:
- "Je li struktura dovoljno jaka?"
- "Refren je dovoljno zapamtljiv?"
- "Emocija se čuje kroz tekst?"
- "Šta bih mogao da poboljšam?"

**LyricLab odgovara** — kreira AI analizu koja daje:
- Strukturu pesme (verz, refren, bridge...)
- Emociju i ton
- Snage i slabosti
- Sugestije za poboljšanje
- Numerički score za refren (1-10) i pesmu ukupno (1-10)

## Šta korisnik može da radi?

### 1. Registracija i login
- Email + password ili Google OAuth
- NextAuth.js rukuje svim security detaljima

### 2. Dodavanje pesama
- Unese naslov, tekst, žanr
- Pesme se čuvaju u bazi (PostgreSQL)

### 3. AI analiza
- Klikne "Analiziraj"
- Gemini API procesira tekst i vraća JSON
- Rezultat se prikazuje na ekranu i čuva u istoriji

### 4. Poređenje pesama
- A/B poređenje: verzija A vs verzija B iste pesme
- Cross-song: "moja pesma vs svetska pesma" (da vidi razlike)
- Gemini generiše komparativnu analizu

### 5. Istorija analiza
- Sve analize se čuvaju u bazi
- Korisnik može da preugleda stare analize
- Može da eksportuje rezultate

---

## Tech Stack — Zašto ove tehnologije?

| Komponenta | Tehnologija | Razlog |
|---|---|---|
| Frontend | Next.js 14 App Router | SSR, file-based routing, server/client komponente |
| Tipizacija | TypeScript | Hvataj greške pre runtime-a |
| Stilovi | Tailwind CSS | Brzo, utility-first, zero CSS fajlova |
| Backend | Next.js API Routes | Monolith pristup, jedan deploy |
| Baza | PostgreSQL + Prisma | Type-safe upiti, migracije, skalabilnost |
| Auth | NextAuth.js | Industry standard, email + OAuth |
| AI | Gemini 2.0 Flash API | Besplatno, moćno, brzo |
| Testovi | Vitest + Playwright | Moderni, traženi na tržištu |
| Deploy | Vercel + Supabase | Besplatno, profesionalno, skalabilno |
| CI/CD | GitHub Actions | Automatski testovi i deploy |

---

## Portfolio vrednost

1. **Vidljivo korisno** — svako može da koristi i razume šta aplikacija radi
2. **Full-stack** — frontend, backend, baza, auth, AI, deploy — sve skupa
3. **Sistem dizajn** — koristiš KISS, DRY, SOLID, Clean Code principje
4. **Testovi** — pokazuješ da znaš kako da testiiraš kod
5. **Production-ready** — može se hitno poslati na intervjuu i pokrenuti

---

## Intervju talking points

Kada predstaviš ovaj projekat:

**"LyricLab je web aplikacija koju sam napravio za moje potrebe kao muzičara. Korisnik može da unese tekst pesme i dobije AI-generisanu analizu — strukturu, emociju, snage, slabosti i sugestije za poboljšanje.**

**Tehnički, to je monolith arhitektura sa:**
- **Frontend:** Next.js 14 App Router, React komponente, Tailwind CSS
- **Backend:** API Routes sa NextAuth.js autentifikacijom
- **Baza:** PostgreSQL na Supabase sa Prisma ORM
- **AI:** Gemini 2.0 Flash API za analizu tekstova
- **Testovi:** Vitest za unit/integration, Playwright za e2e
- **Deploy:** Vercel frontend, Supabase baza, GitHub Actions CI/CD

**Ključne odluke:**
- Koristio sam monolith umesto microservices jer je to najpraktičnije za MVP
- NextAuth.js jer se auth nikad ne improvizuje — koristim industry standard
- Gemini jer je besplatan i dostatan za analizu tekstova
- TypeScript svuda jer greške trebalo hvatati u razvoju, ne u produkciji

**Šta sam naučio:**
- Kako se auth pravi (JWT, sessions, OAuth flow)
- Kako se baza dizajnira i migrira bez downtime-a (Prisma migrations)
- Kako se AI API integriše i kako se rukuje greškama i timeout-ima
- Kako se testira full-stack aplikacija
- Kako se skališe od development u production (environment varijable, secrets, rate limiting)"

---

## Planirana skalabilnost

Ako projekat poraste:

**Arhitekturne izmene:**
- Backend izdvoji u Node.js/Express server
- Frontend ostaje na Vercelu
- Baza se horizontalno skalira (read replicas)
- Gemini API pozive prebaci u async queue (Bull/BullMQ)
- Dodaj Redis za caching

**ALI:** Za portfolio MVD je ovako — monolith je savršen.

---

## Success criteria

- [ ] Login radi (email + Google OAuth)
- [ ] Mogu dodavati pesme i čuvati u bazi
- [ ] Gemini analiza radi i prikazuje se
- [ ] Poređenje pesama radi
- [ ] Istorija analiza se čuva i prikazuje
- [ ] Testovi pokrivaju 70%+ koda
- [ ] GitHub Actions pokreće testove na push
- [ ] Aplikacija je live na Vercelu
- [ ] README je jasan za svakoga
- [ ] Portfolio stranica sa linkom je gotova

---

## Razlika od drugih portfoliolio projekata

❌ Todo lista — svi prave ovo
❌ Blog — svi prave ovo
❌ Chat aplikacija — svi prave ovo

✅ **LyricLab** — niko ne pravi ovo jer je specifičan za muzičare
✅ Realna primena — moraš ga zaista koristiti
✅ Domain znanje — razumeš šta musik treboji od analitike
✅ AI component — ne samo CRUD operacije

---

## Napomena za intervju

Nabiši ove tri rečenice i povlačiće se na njih:

1. "Monolith arhitekturu sam izabrao jer je to najpraktičnije za MVP, ali ako bi trebalo da skalira, znao bih tačno kako da izdvojim backend u microservice."

2. "Nisam koristio random library umesto NextAuth jer auth nikad ne improvizuješ — koristiš ono što je battle-tested."

3. "Svi testovi su automatski — na svakom push-u, GitHub Actions pokreće testove i ako nešto baca, deploy je blokiran. Ovo štiti production."

---

**Krećemo na Dan 1 — setup i PostgreSQL konekcija.**
