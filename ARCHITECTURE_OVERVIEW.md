# LyricLab — Arhitektura

Ovde je potpuni opis kako je aplikacija sagrađena, od korisnika do baze podataka.

---

## Dijagram 1 — Visoki nivo (šta aplikacija radi)

![High Level Architecture](./ARCHITECTURE_1_HighLevel.svg)

### Objašnjenje

Korisnik (muzičar) koristi LyricLab aplikaciju kroz browser. Aplikacija ima pet glavnih funkcionalnosti:

1. **Registracija i login** — korisnik se registruje sa email-om ili Google account-om
2. **Dodavanje pesama** — unesi naslov, tekst pesme i žanr
3. **AI analiza** — aplikacija šalje pesmu Gemini API-ju i dobija analizu
4. **Poređenje pesama** — poredi dve verzije ili dve različite pesme
5. **Istorija analiza** — sve analize se čuvaju i mogu se pregledate

Gemini je eksterni servis koji radi analizu. PostgreSQL je baza gde se čuva sve trajno.

---

## Dijagram 2 — Tok podataka (šta se dešava korak po korak)

![Request Flow](./ARCHITECTURE_2_RequestFlow.svg)

### Step-by-step tok

Kada korisnik klikne "Analiziraj", evo šta se dešava:

1. **Korisnik klikće "Analiziraj"** 
   - React forma u browseru prikuplja tekst pesme
   - Forma šalje HTTP POST zahtev ka `/api/analyze`

2. **Middleware proverava autentifikaciju**
   - Svaki zahtev prvo ide kroz middleware koji proverava: "Da li je ovaj korisnik ulogovan?"
   - Ako nije ulogovan → redirect na `/login`
   - Ako jeste → nastavi dalje

3. **API Route prima zahtev**
   - `POST /api/analyze` ruta u `/app/api/analyze/route.ts` prima zahtev
   - Ruta validira da input nije null, nije preudalek, itd.

4. **Poziv Gemini API-ja**
   - Aplikacija šalje tekst pesme Gemini-ju preko HTTPS
   - Gemini analizira tekst i vraća JSON sa rezultatima
   - JSON sadrži: strukturu, emociju, snage, slabosti, score, itd.

5. **Čuvanje u PostgreSQL**
   - API ruta prima JSON od Gemini-ja
   - Prisma ORM upisuje rezultat u `Analysis` tabelu
   - Sada je rezultat trajno sačuvan

6. **Rezultat nazad korisniku**
   - API route vraća JSON korisniku (browser)
   - React komponenta prima rezultat u `state`
   - React re-renderuje stranica i korisnik vidi analizu

### Ključne odluke

- **Middleware prvo** — autentifikacija je prvi check. Ako padne, ništa se ne izvršava.
- **Validacija inputa** — pre nego što pošljemo Gemini-ju, proveravamo da input nije null/preudalek
- **Čuvanje u bazu POSLE analize** — ako Gemini padne, ne čuvamo ništa
- **Async/await** — API ruta čeka na Gemini odgovor (može trajati 1-2s), korisnik vidi spinner

---

## Dijagram 3 — Slojevi i komunikacija

![Layer Architecture](./ARCHITECTURE_3_Layers.svg)

### Presentation Layer — Browser

**Šta:** React komponente koje korisnik vidi i koristi

**Šta se dešava:**
- `SongForm.tsx` — forma za unos teksta pesme
- `layout.tsx` — zajednički layout za sve stranice (navbar, footer)
- `middleware.ts` — čuva sesiju, proverava auth na svakom zahtev

**Ne čuva se ništa trajno** — samo state u memoriji dok je stranica otvorena

---

### Application Layer — Next.js API Routes

**Šta:** Business logika — šta aplikacija zaista radi

**Šta se dešava:**
- `/api/analyze` — prima tekst pesme, poziva Gemini, čuva u bazu
- `/api/songs` — CRUD za pesme (create, read, delete)
- `/api/auth` — login, logout, register (NextAuth.js rukuje)

**Ključna pravila:**
- Svaka ruta je odvojena datoteka (`/api/songs/route.ts`, `/api/songs/[id]/route.ts`)
- Svaka ruta proverava auth pre nego što napravi nešto
- Svaka ruta validira input pre nego što ga obradi

---

### Data Layer — PostgreSQL

**Šta:** Trajno čuvanje podataka

**Šta se čuva:**
- `User` — korisnici (email, password hash, ime)
- `Song` — pesme koje je korisnik dodao
- `Analysis` — rezultati AI analiza (čuva se kao JSON)
- `Comparison` — rezultati poređenja

**Kako se čuva:**
- Prisma ORM — ne pišeš SQL ručno, pišeš TypeScript koji Prisma prevodi u SQL
- Migracije — automatski verzionisanje baze, bez downtime-a

---

### AI Layer — Gemini 2.0 Flash

**Šta:** Eksterni servis koji radi AI analizu

**Šta se dešava:**
- Aplikacija šalje HTTP zahtev sa tekstom pesme
- Gemini procesira tekst (to može trajati 1-2s)
- Gemini vraća JSON sa analizom

**Ključne odluke:**
- Ne instaliramo ništa — samo pozivamo API ključem
- Ako Gemini padne, aplikacija savetuje korisniku da pokuša kasnije
- Rezultat se čuva čak i ako je Gemini odgovor "loš" (može biti poučan)

---

## Komunikacija između slojeva

```
Browser (React)
    ↓ HTTP zahtev
Middleware (auth check)
    ↓ 
API Route (business logika)
    ├→ Gemini API (poziv sa tekstom)
    │    ← JSON odgovor
    ├→ PostgreSQL (Prisma upiti)
    │    ← Podaci iz baze
    ↓ HTTP odgovor
Browser (React re-renders)
```

---

## Šta je šta?

### Backend vs Frontend

**Frontend** = Browser
- React komponente
- Tailwind CSS stilizovanje
- React hooks za state management
- Ono što korisnik vidi

**Backend** = API Routes
- Business logika
- Autentifikacija
- Komunikacija sa bazom i Gemini-jem
- Ono što korisnik ne vidi

---

### Sinhronus vs Asinhronus

**Sinhrono** = jedan korak čeka drugi
```javascript
const result = fetch('/api/analyze')  // čeka se odgovor
console.log(result)                   // tek posle ispisuje
```

**Asinhronog** = ne čekamo
```javascript
const result = await fetch('/api/analyze')  // čeka se u async funkciji
// ostatak koda se pokreće tek posle
```

U LyricLab-u: API šalje zahtev Gemini-ju i **čeka** odgovor (sinhronog), ali korisnik ne čeka — vidi spinner dok se poziva dešava.

---

### State Machine — tri stanja u analizi

Svaki put kada korisnik klikne "Analiziraj", komponenta je u jednom od tri stanja:

```
┌─────────────────┐
│    LOADING      │  Čekamo Gemini odgovor, spinner je vidljiv
│  (spinning...)  │
└─────────────────┘
     ↓
┌─────────────────┐
│    SUCCESS      │  Dobili smo analizu, prikazuje se na ekranu
│  (view data)    │
└─────────────────┘
     ↓
┌─────────────────┐
│     ERROR       │  Gemini API padnuo, prikazuje se error poruka
│  (error msg)    │
└─────────────────┘
```

Korisnik može biti u jednom od ova tri stanja — nikad u više od jednog u isto vreme.

---

## Zaštita podataka

### Autentifikacija (Authentication) — Ko si ti?

NextAuth.js rukuje:
- Email + password registracijom
- Google OAuth login-om
- JWT tokenim (secret string koji se šalje sa svakim zahtevom)
- Session-om (privremenom autentifikacijom)

Bez autentifikacije: bilo ko može pristupiti `/api/songs` i videti tuđe pesme.

### Autorizacija (Authorization) — Šta smeš da vidiš?

Middleware proverava:
- Da li si ulogovan? (auth)
- Da li je pesma TVOJA? (autorizacija)
- Ako nije tvoja → 403 Forbidden

U kodu:
```typescript
const userId = session.user.id  // iz JWT tokena
const song = await prisma.song.findUnique({ where: { id: songId }})
if (song.userId !== userId) {
  throw new Error("Not authorized")  // pesma nije tvoja
}
```

---

## Skalabilnost — ako projekat poraste

Za MVP (sad), ovo je dovoljno:

```
[Frontend — Vercel]
        ↓
[API Routes — Vercel]
        ↓
[PostgreSQL — Supabase]
        ↓
[Gemini API — Google]
```

Ako budeš morao da skaliraš (1M korisnika):

```
[Frontend CDN — Vercel]
        ↓
[Load Balancer]
        ↓
[Multiple API Servers]
        ↓
[Database Cluster]
        ├→ Primary (write)
        └→ Replicas (read)
        ↓
[Gemini API — Google]
        ↓
[Redis Cache]
        ↓
[Message Queue (RabbitMQ)]
```

ALI: za intervju, MVP je dovoljno. Znalac će te pitati "kako bi to skalirao", a ti kažeš: "Izdvojio bih backend u odvojeni server, dodao bih Redis cache za česte analize, async queue za Gemini pozive..."

---

## Checklist — šta razumeš?

- [ ] Shvatam flow: Browser → Middleware → API → Gemini/DB → Browser
- [ ] Znam šta je Frontend, šta je Backend
- [ ] Razumem autentifikaciju vs autorizaciju
- [ ] Znam šta je state machine (loading, success, error)
- [ ] Mogu objasniti zašto su svi slojevi potrebni
- [ ] Znam kako bi to skalirao ako budeš morao

Ako nešto nije jasno, pitaj pre nego što kreneš da kodiraš!
