# BLINDHASA — Armored-Vehicle Workshop Tracker

> One self-contained `index.html` (HTML + CSS + vanilla JS, no backend, no build step) that tracks every vehicle in an armoring workshop from **arrival to delivery**, with **named-person quality sign-offs** and an **immutable, dated audit trail**.

Open `index.html` in any browser, or drop it on any static host. That's the whole deployment.

---

## 1. What the product does (in one paragraph)

An armoring shop runs 5–40 vehicles at a time through the same seven stages: Intake → Teardown → Armor install → Reassembly → Paint & finish → Quality check → Delivery. Today most shops track this on a whiteboard, WhatsApp and Excel, which means nobody knows which vehicle is late, who verified the ballistic overlaps, or why a promised date moved. BLINDHASA replaces that with a **job board** that shows what's late / due / on schedule in real time, a **job record** for each vehicle with checklists and dated sign-offs by named people, and **hard rules in code** (e.g. a car can't be delivered until the QC inspector has signed under their own login).

---

## 2. Feature list

### Job board (home · `#/board`)
- Eyebrow `ARMORING LINE · PLANTA 1`, title, "Updated Thu, Nov 12, 4:05 PM", round **+** button (New job / Intake form).
- **Live stat tiles** (tappable filters): In production · On schedule (with %) · Due in 7 days (`through Nov 19`) · Behind schedule (red).
- **Stage filter chips** with live counts (All 8 / Intake / Teardown / …).
- **Grouped list**: `BEHIND SCHEDULE · 2` (red, warning icon) → `DUE THIS WEEK` → `ON SCHEDULE`.
- **Job card**: red accent bar when late, mono job ID, armor-level shield badge, `LATE` pill, black pill with the **initials of the last signer**, year/make, model, client, promised date, `6d late` / `in 3d`, **7-segment stage bar** (black = done, red = current & late, outlined = current & on time, gray = pending), current stage name.
- **Desktop**: 2–3 column card grid or **Kanban by stage** toggle; job detail opens in a **right-hand panel** next to the board.
- **Search overlay** (header icon) — by ID, model, client.

### Job detail (`#/job/BLH-2033`)
- Back link, header card with ID + armor badge + status pill, year, **model**, client, armor spec line, print button.
- Sub-cards **CURRENT STAGE** ("Delivery · Step 7 of 7 · awaiting sign-off") and **PROMISED DELIVERY** ("Fri, Nov 13 · 1 day left" — turns red when late).
- **Forecast ready date**, **Actual handover date** ("Not yet delivered") and **Promised date** — editable by authorized roles, every change **requires a reason** and is written to history. A forecast later than the promise is flagged inline.
- **Horizontal stepper**: numbered circles, black check when signed off, outlined current step; under each step `HI · Oct 30` (signer initials · date) or `Since Nov 10`.
- **Dark "FINAL QUALITY CHECK · PASSED" card** with the inspector's avatar, name, role, date/time and quoted note.
- **Checklist per stage** (38 items for QC) with "Completed by <name> · <date/time>", progress `38/38`, locked for signed-off / not-started stages, uncheck requires a reason.
- **Audit trail** (collapsible timeline, newest first): creations, checklist ticks, sign-offs, date changes, overrides, notes, delivery. Cannot be edited or deleted in the UI.
- **Notes** list + "Add note".
- **Sticky action bar**: `Note` + context-sensitive primary button — `Sign off <Stage>` or `Sign off handover` (GM only, only with a valid inspector QC sign-off). When blocked, the reason is shown inline and as a toast.
- **Print-friendly job record** (`@media print`): header, stepper, QC card, checklist, full history, sign-off table.

### Stage sign-off bottom sheet (mobile) / modal (desktop)
- Eyebrow `STAGE SIGN-OFF · BLH-2035`, title `Sign off Quality check`, description, black info card.
- **"Who verified it?"** 2×2 person cards + dashed **"Someone else…"** (name + role inputs).
- Optional note (prefilled for QC: "38/38 checklist items. Road + water test passed. Overlaps checked on all doors.").
- Confirmation checkbox: *"I confirm **Ricardo Soto** personally verified **Quality check** on BLH-2035 and it meets Blindhasa's standard."*
- Primary button `Sign off → Delivery` disabled until a verifier is chosen **and** the box is checked. Footer: *"Saved to the audit trail with the verifier's name and time (Nov 12, 2026)."*
- Incomplete checklist → managers see an **override reason** field (mandatory, logged); non-managers are blocked.
- QC stage → only the logged-in **QC inspector** can be selected; other users see a block message with a one-tap "Switch to Ricardo Soto".
- Delivery stage → handover sheet with actual handover date; GM only.

### Alerts (`#/alerts`)
- Generated live: **late**, **due ≤ 7 days**, **forecast later than promised**, **awaiting sign-off** (checklist complete or at Delivery).
- Unread red badge on the tab/sidebar, "Mark all read", tap → opens the job.

### Jobs (`#/jobs`)
- Searchable, filterable (stage, status, armor level, client) and sortable (promised, forecast, ID) table of **all** jobs including delivered.

### Login page (`#/login`) — the app is only reachable after signing in
- Minimal card: **Role** (select) · **Email** · **Password** · Sign in.
- **Demo accounts** row (SM · HI · RS · GM · OW): tap a chip to **autofill** role + email + password with that account's dummy credentials. The form still validates them (unknown email, wrong password, role that doesn't match the account).
- No role switching inside the app: to act as someone else you sign out and sign in with their account. The header avatar shows the signed-in user's initials; the session persists in the browser until Sign out.
- Deep links (`#/job/BLH-2033`) are remembered and opened right after sign-in.

### Account / Settings (avatar menu)
- Session card (email + permissions) and **Sign out**.
- **Run guided demo** — creates a vehicle and walks it Intake → … → QC → Delivery with a step card. When a step needs another role (QC inspector, GM) the demo **signs you out and opens the real login page prefilled** for that account; after you sign in the step continues automatically.
- **Export data** (JSON download) · **Import data** (JSON, validated: ids, stage indexes, checklists, histories; never adopts a session from the file) · **Reset demo data**.

### Platform
- Responsive: < 640 px single column + bottom tab bar; 640–1023 tablet; ≥ 1024 px left sidebar + grid + right detail panel.
- Hash routing (`#/board`, `#/jobs`, `#/alerts`, `#/job/BLH-2033`, `#/login`) — back button and shareable links work.
- `localStorage` persistence wrapped in try/catch with in-memory fallback.
- Accessibility: semantic HTML, visible focus states, ARIA labels on icon buttons, ≥ 44 px touch targets, modal focus trap, Esc to close, `prefers-reduced-motion` respected.
- Only external resource: Google Fonts (Inter + JetBrains Mono) with system fallbacks. All icons are inline SVG.

---

## 3. Business rules enforced in code

| # | Rule | Where |
|---|------|-------|
| 1 | **Delivered requires inspector sign-off first.** QC must be signed by a user with `isInspector`, recorded under that inspector's **own login** (`signedInAs === userId`). Non-inspectors cannot sign QC. | `qcSignoffValid()`, `signoffGate()`, `signOffStage()` |
| 2 | **Only the GM (`canDeliver`) completes handover**, which sets `actualHandoverDate` and marks the job Delivered. | `signoffGate()` |
| 3 | **No sign-off with an incomplete checklist** — unless a manager overrides with a mandatory reason (logged as `override`). QC cannot be overridden. | `signoffGate()`, `signOffStage()` |
| 4 | **Late** = promised date < today and not delivered. **Due in 7 days** = promised within today … today + 7. | `jobStatus()` |
| 5 | **Every change writes an immutable history entry** (checklist, dates, sign-offs, notes, stage moves, overrides). No edit/delete UI. | `log()` |
| 6 | **Forecast later than promised triggers an alert.** | `buildAlerts()` |

Role matrix (seeded users):

| User | Role | Sign stages | Sign QC | Handover | Edit dates | Override |
|------|------|:-:|:-:|:-:|:-:|:-:|
| SM Sofía Méndez | Production manager | ✓ | — | — | ✓ | ✓ |
| HI Héctor Ibarra | Shop manager | ✓ | — | — | ✓ | ✓ |
| RS Ricardo Soto | QC inspector | ✓ | **✓** | — | — | — |
| GM General Manager | GM | ✓ | — | **✓** | ✓ | ✓ |
| OW Owner | Owner | — (read + notes) | — | — | ✓ | — |

---

## 4. Data model (JS, persisted as JSON)

```js
state = {
  version: 1,
  users: [{ id, name, initials, role, canSignStages, isInspector, canDeliver, isManager, canEditDates }],
  jobs: [{
    id: 'BLH-2041', armorLevel: 'B6', year: 2026, make: 'Toyota', model: 'Land Cruiser 300',
    client: 'Grupo Altamira', armorSpec: 'High-power rifle (7.62×51 mm)',
    currentStageIndex: 2, assignedTeam, nextAction,
    promisedDate: '2026-11-06', forecastReadyDate: '2026-11-18', actualHandoverDate: null, delivered: false,
    createdAt, checklist: [{ id, stage, text, done, doneBy, doneAt }],
    stageSignoffs: { teardown: { userId, name, initials, role, at, note, signedInAs } },
    history: [{ id, at, userId, type: 'created|checklist|stage_signoff|date_change|override|note|delivered', text }],
    notes: [{ id, at, userId, text }]
  }],
  currentUserId: 'sm', readAlerts: ['due:BLH-2033'], seq: 2046, prefs: {}
}
```

Seeded: **8 active** vehicles across all 7 stages (BLH-2041 6 days late, BLH-2038 2 days late, BLH-2035 ready for QC sign-off, BLH-2033 awaiting handover, four on schedule incl. one due within 7 days) + **2 delivered**. App clock is fixed at **Thu Nov 12, 2026, 4:05 PM** (`CONFIG.today`) so the demo math matches the screenshots; change it to `new Date()` for live use.

---

## 5. How to use

1. Open `index.html` (double-click, or serve the folder). You land on the **login page**.
2. Tap a demo-account chip (e.g. **SM**) to autofill role, email and password → **Sign in**.
3. Tap a card on the Job board → job detail → tick checklist items → **Sign off <Stage>**.
4. Avatar (top-right) → **Sign out** to switch to another account, **Run guided demo** for the end-to-end walkthrough.
5. Avatar → **Export data** before clearing the browser; **Import data** to restore.

**Demo logins** (password for all: `demo1234`):

| Chip | Name | Role (select on login) | Email |
|---|---|---|---|
| SM | Sofía Méndez | Production manager | sofia@blindhasa.com |
| HI | Héctor Ibarra | Shop manager | hector@blindhasa.com |
| RS | Ricardo Soto | QC inspector | ricardo@blindhasa.com |
| GM | General Manager | GM | gm@blindhasa.com |
| OW | Owner | Owner | owner@blindhasa.com |

Credentials are dummy values stored on the user records in `SEED_USERS`; swap them for real authentication in the hosted tier.

**Quick rule demo:** sign in as HI → open BLH-2035 → "Sign off Quality check" → the sheet blocks you → "Sign out so Ricardo Soto can sign in" → login page comes up prefilled → sign in → you land back on BLH-2035 → sign → it moves to Delivery. Then open BLH-2033 as HI → handover button disabled ("Only the General Manager…") → sign out, sign in as GM → sign off handover → Delivered.

## 6. Deploy (static hosting, ~1 minute)

| Host | Steps |
|------|-------|
| **Netlify** | Drag the folder onto app.netlify.com/drop, or `netlify deploy --prod --dir .` |
| **GitHub Pages** | Push to a repo → Settings → Pages → Source: `main` / root → `https://<user>.github.io/<repo>/` |
| **Railway (static)** | New project → Deploy from repo → add a `Dockerfile` with `FROM nginx:alpine` + `COPY index.html /usr/share/nginx/html/`, or use the Static Site template pointing at the repo root |
| **Vercel / Cloudflare Pages** | Import repo, framework = "Other", output dir = `/` |
| **On-prem / LAN** | Copy to any web server or open from a shared drive — it runs offline |

No environment variables, no build, no database. Each browser keeps its own data in `localStorage`; use Export/Import to move data between devices (multi-user sync is the first paid-tier feature — see §8).

Customize in `CONFIG` (top of the `<script>`): `today`, `plant`, `demoPin`, `dueWindowDays`. Stages, checklists, users and seed jobs live right below it.

---

## 7. Why this sells — positioning

**Who buys:** owners / general managers of independent vehicle-armoring workshops (10–150 employees) and the production managers of armoring divisions inside dealership groups or security integrators.

**The pain they already feel (use their words):**
- "Which cars are late and *why*?" — the owner finds out from the angry client, not the board.
- "Who checked the overlaps on the rear doors?" — nobody can prove it when a warranty claim or an incident comes in.
- Promised dates move silently; deposits were taken against dates nobody tracks.
- Audits / certifications (ballistic standards, insurer requirements, government fleet tenders) ask for traceability that WhatsApp photos can't provide.
- Key people (the one QC guy) hold the process in their head; the shop can't scale to a second line.

**One-liner:** *"BLINDHASA shows you every vehicle's stage, who signed it off and when — so late cars, missing QC signatures and moved dates are visible before the client calls."*

**Why not generic tools (Trello, Monday, a dealership DMS):** none of them model *named, role-gated sign-offs*, ballistic checklists, or the "can't deliver without the inspector's own signature" rule. BLINDHASA ships that logic out of the box, on a phone, in a tactical UI the shop floor will actually open.

**Proof you can show in 3 minutes:** the guided demo on your phone across the table — create → sign off → inspector login blocks the manager → GM delivers → print the audit record.

### Packaging & pricing (starting points)

| Tier | What | Price idea |
|------|------|-----------|
| **Pilot** | This single-file app, branded, deployed to their subdomain, their stages/checklists/users seeded. 30 days. | US$ 1,500–3,000 one-off (credited to the annual plan) |
| **Workshop** | Hosted multi-user sync (Supabase/Firebase or a small API), real logins, photo attachments, client-facing status link, PDF handover certificate. | US$ 400–900 / month per plant |
| **Group** | Multiple plants, KPIs (cycle time per stage, late ratio, QC first-pass yield), insurer/auditor read-only access, SSO. | US$ 1,500–4,000 / month |

Anchor on money they already lose: one late B6 SUV ≈ 2–4 weeks of a US$ 60–120k job sitting unpaid; one failed audit ≈ a lost fleet tender. The tool costs less than a day of a delayed delivery.

---

## 8. Finding your ICP on LinkedIn

### Ideal Customer Profile
- **Company:** vehicle armoring / "blindaje de autos" / "armoured vehicles" / "ballistic protection" manufacturers and workshops, 10–200 employees, running their own production line (not pure resellers).
- **Geographies with the densest clusters:** Mexico (CDMX, Estado de México, Guadalajara, Monterrey, Puebla, Querétaro), Brazil (São Paulo — the biggest market globally), Colombia (Bogotá, Medellín), Argentina, UAE (Dubai/Ras Al Khaimah), Saudi Arabia, Turkey, South Africa (Johannesburg), India (Jamshedpur/Delhi NCR), Pakistan, Nigeria, USA (Texas, Florida, Virginia/DC), Canada (Toronto), Germany/Italy (OEM-adjacent).
- **Trigger signals:** hiring posts for "QC inspector", "production supervisor", "jefe de planta"; posts about a new plant / second line; certification news (VPAM, NIJ, CEN 1063, UL 752, ISO 9001); complaints about delivery delays in reviews; participation in trade shows.

### Buyer personas (titles to search)
| Persona | Role in the deal | Titles (EN / ES / PT) |
|---------|------------------|----------------------|
| **Economic buyer** | signs | Owner, Founder, CEO, Managing Director, General Manager / Director General, Gerente General / Diretor Geral |
| **Champion** | lives the pain daily | Production Manager, Plant Manager, Operations Manager, Head of Production / Gerente de Producción, Jefe de Planta, Gerente de Operaciones / Gerente de Produção |
| **Technical validator** | must love the sign-off flow | Quality Manager, QC Inspector, Quality Assurance Lead / Jefe de Calidad, Inspector de Calidad / Gerente de Qualidade |
| **Influencers** | ask for traceability | Sales Director (promises the dates), After-sales / Warranty Manager, Compliance / Certification Manager |

### Search recipes

**LinkedIn (free) — people search, boolean in the keyword box:**
```
(armoring OR armouring OR "armored vehicles" OR "armoured vehicles" OR blindaje OR blindados OR blindagem OR "ballistic protection")
AND ("production manager" OR "plant manager" OR "operations manager" OR "general manager" OR owner OR CEO OR "quality manager" OR "jefe de planta" OR "gerente de producción" OR "gerente de operaciones" OR "director general" OR "gerente de produção")
```
**Spanish-first variant:**
```
(blindaje OR blindajes OR "blindaje automotriz" OR "autos blindados" OR "vehículos blindados") AND (gerente OR director OR jefe OR dueño OR fundador OR calidad)
```
**Portuguese variant (Brazil):**
```
(blindagem OR "blindagem veicular" OR "carros blindados") AND (gerente OR diretor OR produção OR qualidade OR sócio)
```

**Sales Navigator filters (best ROI):**
- Account: Industry = *Motor Vehicle Manufacturing*, *Security & Investigations*, *Defense & Space*; Headcount 11–200; Keywords = `armoring OR armouring OR blindaje OR blindagem OR ballistic`.
- Lead: Seniority = Owner/Partner, CXO, VP, Director, Manager; Function = Operations, Quality Assurance, Engineering, Business Development; Years in current company ≥ 1.
- Save the account list, then use "Posted on LinkedIn in last 30 days" and "Changed jobs in last 90 days" as triggers.

**Where the ICP congregates (follow, comment, then connect):**
- Hashtags: `#armoredvehicles #armouredvehicles #blindaje #blindajeautomotriz #autosblindados #blindagem #carrosblindados #ballisticprotection #vpam #executiveprotection #securityvehicles`
- Trade shows (exhibitor lists = prospect lists, attendees post selfies = warm openers): **Expo Seguridad México** (CDMX), **LAAD** (Rio), **ISC Brasil**, **Milipol Paris**, **IDEX / ISNR Abu Dhabi**, **Intersec Dubai**, **GPEC Germany**, **Securika Moscow**, **ISC West** (Las Vegas), **DSEI London**.
- Groups: "Armored Vehicles & Ballistic Protection", "Executive Protection Professionals", "Blindaje y Seguridad Automotriz", "Security Industry Professionals", "Automotive Manufacturing Professionals".
- Adjacent lists: ballistic glass & steel suppliers' follower lists (their customers are your prospects), armoring associations (e.g. AMBA in Mexico, ABRABLIN in Brazil), insurer / fleet-security buyers who force traceability on their vendors.

**Build the list, in order:** 1) exhibitor lists from the shows above → 2) company pages → "People" tab filtered by the titles → 3) Sales Navigator saved search for look-alikes → 4) enrich with the company website (do they run their own plant? how many bays?) → 5) tag by country/language so every message goes out in the right language.

### Content that warms the list (post 2–3×/week, 60 days before/while outreaching)
- Short screen recordings of the phone UI: "Why a B6 can't be delivered until the inspector signs under his own login."
- "Whiteboard vs. job board" before/after.
- Checklists as lead magnets: "38-point final QC checklist for B6 SUVs (PDF)".
- Carousel: "5 signs your armoring shop will miss December deliveries."
- Tag ballistic-glass and steel suppliers; they reshare to exactly your audience.

---

## 9. Outreach messages (copy, adapt, send)

Rules: ≤ 300 characters in the connection note, no link in message 1, one question, mention something specific (plant, show, post, model they armor). Send from a profile whose headline says *"Production tracking for armoring workshops"*.

### Connection request (EN)
> Hi {{first}}, I follow {{company}}'s work on {{model / level, e.g. B6 Suburbans}}. I build a simple job board for armoring shops — every vehicle's stage, who signed off QC and when. Would love to connect and compare notes on how you track the line today.

### Connection request (ES)
> Hola {{nombre}}, sigo el trabajo de {{empresa}} en {{ej. blindaje nivel 5 de Suburban}}. Desarrollo un tablero de producción para talleres de blindaje: etapa de cada unidad, quién firmó calidad y cuándo. Me gustaría conectar e intercambiar ideas sobre cómo llevan hoy la línea.

### Connection request (PT)
> Olá {{nome}}, acompanho o trabalho da {{empresa}} em blindagem {{nível}}. Desenvolvo um quadro de produção para blindadoras: etapa de cada veículo, quem assinou a qualidade e quando. Gostaria de conectar e trocar ideias sobre como vocês controlam a linha hoje.

### Message 1 — after they accept (EN)
> Thanks for connecting, {{first}}. Quick context: most shops I talk to run production on a whiteboard + WhatsApp, and the owner only finds out a car is late when the client calls. I built BLINDHASA to fix exactly that: a phone-first job board where each vehicle shows its stage, the promised vs. forecast date, and a **named** sign-off per stage — the GM literally can't mark "delivered" until the QC inspector signs under his own login.
> If I send a 90-second video of it running on a phone, would you tell me what's wrong with it for {{company}}?

### Message 1 (ES)
> Gracias por conectar, {{nombre}}. Contexto rápido: la mayoría de los talleres que conozco controlan producción con pizarrón + WhatsApp, y el dueño se entera de que una unidad va tarde cuando llama el cliente. Construí BLINDHASA para eso: un tablero desde el celular donde cada unidad muestra su etapa, fecha prometida vs. pronóstico y una firma **con nombre** por etapa — el gerente no puede marcar "entregado" hasta que el inspector de calidad firme con su propio usuario.
> Si te mando un video de 90 segundos corriendo en el teléfono, ¿me dirías qué le falta para {{empresa}}?

### Message 2 — value nudge, 4–5 days later
> {{first}}, one thing that resonated with other plant managers: the audit trail. Every checklist tick, date change and sign-off is logged with person + time and can't be edited — handy when an insurer, a fleet client or a VPAM/NIJ auditor asks "who verified the overlaps on this door?".
> Does {{company}} get asked for that kind of traceability today?

### Message 3 — pilot offer, ~1 week later
> Offer, no strings: I set BLINDHASA up with {{company}}'s stages, checklists and people, running on your own subdomain, within 48 h. Your team uses it on their phones for 30 days. If it doesn't make late cars and missing QC signatures visible earlier, you delete it. 20-minute call to set it up — {{two time options}}?

### Message 4 — break-up
> {{first}}, I'll stop here so I don't become noise. If a delayed delivery or an audit request ever makes production visibility urgent, reply "board" and I'll send the demo link. Wishing {{company}} a strong quarter.

### Openers for warm triggers
- *Trade show:* "Saw {{company}} at Expo Seguridad — the {{vehicle}} on the stand was impressive. How many units a month does the plant turn right now?"
- *Hiring post:* "Noticed you're hiring a QC inspector. Curious — how do new inspectors today prove a stage was verified: paper traveler, WhatsApp photo, or something else?"
- *New plant / line:* "Congrats on the second line. When the volume doubles, what's the tool that keeps the owner from walking the floor to know what's late?"

### Objections you'll hear
| Objection | Answer |
|-----------|--------|
| "We use Excel / Trello / Monday." | "Great — keep it for planning. BLINDHASA is the floor-level record: who signed, when, and the rule that blocks delivery without QC. Can Trello refuse a 'delivered' card until the inspector logs in?" |
| "My guys won't use software." | "It's three taps on a phone: open car, tick item, sign off. In the pilot the QC inspector is usually the first fan — his name is finally on the record." |
| "We're small." | "Small is where one late car hurts most. The pilot is flat-fee and runs on a single page; no IT project." |
| "Security / data." | "Pilot runs on your subdomain or even offline in the shop; you own the export. Multi-user tier can be self-hosted." |

### Metrics to track weekly
Connection acceptance ≥ 35 % · reply to message 1 ≥ 15 % · video → call ≥ 40 % · call → pilot ≥ 30 % · pilot → paid ≥ 50 %. 150 well-targeted requests/month ≈ 50 connections ≈ 8 conversations ≈ 2–3 pilots.

---

## 10. Roadmap after the pilot

1. Hosted sync + real auth (per-user PIN/email, device sessions) — turns `signedInAs` into a true identity guarantee.
2. Photo evidence per checklist item; PDF handover certificate with QR link to the signed record.
3. Client-facing read-only status page per vehicle.
4. KPIs: cycle time per stage, late ratio, QC first-pass yield, rework reasons.
5. Multi-plant, insurer/auditor roles, SSO; Spanish/Portuguese UI.

---

## 11. File map

```
index.html   ← the whole app (CONFIG · SEED DATA · HELPERS · STATE/STORAGE · RULES · RENDER · SHEETS · DATA OPS · ROUTING · DEMO · EVENTS · INIT)
project.md   ← this document
```
