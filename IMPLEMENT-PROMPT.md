# RevDiagnostic v1 — Implementation Prompt

> Copia este prompt completo en el modelo económico junto con el archivo `index.html`.
> El modelo debe hacer los 4 cambios al JSX inline y entregarte el `index.html` modificado listo para subir a Vercel.

---

## Context

You are implementing 4 targeted fixes to a React web application embedded as **inline JSX** inside a single HTML file (`index.html`, ~1.7MB).

**How the file works:** The entire React app lives as a raw JSX string inside a `<script type="text/babel">` tag at line 182. There is no build step — the browser compiles it at runtime using Babel. You edit the JSX directly inside the file and save. No npm, no webpack, nothing else.

**The app:** A hotel revenue management diagnostic calculator in Spanish for Mexican hotel operators. Calculates KPIs (RevPAR, ADR, Occupancy, GOPPAR) and shows color-coded insights.

**Icons:** The app uses inline SVG icon components (`ArrowUp`, `TrendingUp`, `HelpCircle`, `CheckCircle`, etc.) defined near the top of the JSX. There is NO Lucide import. Do NOT try to use `<Check>` or any Lucide component — use the text character `✓` instead, or one of the defined SVG icons.

**Input fields (state object `{ rooms, occupied, roomRevenue, operatingCosts, marketAdr, marketOcc }`):**
- `rooms` — room-nights available (physical rooms × days). Example: 50 rooms × 30 days = 1,500. UNITLESS.
- `occupied` — room-nights sold. UNITLESS.
- `roomRevenue` — total room revenue for the period. MONETARY.
- `operatingCosts` — total operating costs. MONETARY.
- `marketAdr` — competitor market ADR. MONETARY.
- `marketOcc` — competitor market occupancy %. UNITLESS.

**Critical rule:** `rooms`, `occupied`, `marketOcc` are unitless. They must NEVER be converted when toggling currency. Only `roomRevenue`, `operatingCosts`, `marketAdr` are monetary values.

---

## Fix 1 — ALREADY DONE. DO NOT TOUCH.

The input auto-clear on focus is already implemented (`onFocus` handler in the `Field` component). Skip entirely.

---

## Fix 2 — Real Currency Conversion (estimated 20 min)

### Problem

When the user clicks the USD/MXN toggle, the app currently only changes the display symbol — it does NOT convert the numbers. A user entering $780,000 MXN and switching to USD still sees $780,000, which is wrong.

### Step 2a — Add the exchange rate constant

Find this line in the JSX:

```javascript
CURRENCIES = {
```

Add the following line **immediately before** it:

```javascript
const MXN_PER_USD = 17.5; // Update monthly — last updated 2026-06-17
```

### Step 2b — Add the conversion handler

Find this line inside the `App` component (there is exactly one occurrence):

```javascript
const set = (k) => (v) => setDraft(prev => ({ ...prev, [k]: v }));
```

Add the following block **immediately after** that line (one blank line between is fine):

```javascript
const MONETARY_FIELDS = ['roomRevenue', 'operatingCosts', 'marketAdr'];

const handleCurrencyChange = (newCurrency) => {
  if (newCurrency === currency) return;
  const convert = (amount) => newCurrency === 'USD'
    ? Math.round((amount / MXN_PER_USD) * 100) / 100
    : Math.round((amount * MXN_PER_USD) * 100) / 100;
  setDraft(prev => {
    const next = { ...prev };
    MONETARY_FIELDS.forEach(f => { next[f] = convert(prev[f]); });
    return next;
  });
  setInputs(prev => {
    const next = { ...prev };
    MONETARY_FIELDS.forEach(f => { next[f] = convert(prev[f]); });
    return next;
  });
  setCurrency(newCurrency);
};
```

### Step 2c — Wire the toggle button

Find the currency toggle button click handler (there is exactly one occurrence):

```javascript
onClick={() => setCurrency(c.code)}
```

Replace it with:

```javascript
onClick={() => handleCurrencyChange(c.code)}
```

### Verification for Fix 2

1. Enter `roomRevenue = 100000` in MXN mode
2. Click USD → `roomRevenue` becomes approximately `5714.29`
3. Click MXN again → `roomRevenue` returns to **exactly** `100000` (not 99999.96)
4. `rooms` must stay the same number through both toggles
5. `occupied` must stay the same number through both toggles
6. `marketOcc` must stay the same number through both toggles

---

## Fix 3 — Kit Express Upsell Section (estimated 35 min)

### Problem

After the user sees their diagnostic, there is no path to purchase the product that transforms their hotel. The consulting CTA (dark section at the very bottom) is already there — you are NOT replacing it. You are adding a NEW product section ABOVE it.

### Where to insert

Search for this exact comment in the JSX (it appears once):

```
{/* ---------- CTA ---------- */}
```

The full context around it looks like this:

```javascript
          {/* ---------- CTA ---------- */}
          <section className="print-avoid-break relative overflow-hidden bg-slate-900 rounded-2xl p-8 lg:p-12 text-white">
```

Insert the following JSX block **immediately before** `{/* ---------- CTA ---------- */}` — keep the existing CTA section completely intact below it:

```jsx
          {/* ---------- KIT EXPRESS UPSELL ---------- */}
          <section className="print-avoid-break bg-gradient-to-br from-emerald-50 to-teal-50 border border-emerald-200 rounded-2xl p-8 lg:p-10 no-print">
            <div className="max-w-3xl">
              <div className="flex items-center gap-2 text-[11px] font-semibold uppercase tracking-[0.18em] text-emerald-600 mb-3">
                <span className="w-1.5 h-1.5 rounded-full bg-emerald-500 inline-block"></span>
                Kit de Hotelería Express · $47 USD
              </div>
              <h2 className="text-2xl lg:text-3xl font-bold tracking-tight leading-tight mb-3 text-slate-900">
                ¿Quieres incrementar tu revenue un <span className="text-emerald-600">25%</span>, mejorar tu operación y activar tu canal directo?
              </h2>
              <p className="text-slate-600 text-base leading-relaxed mb-6 max-w-2xl">
                Armé el Kit de Hotelería Express con todo lo que un hotelero necesita para tomar el control de su revenue — sin contratar un consultor.
              </p>
              <ul className="space-y-2.5 mb-8">
                {[
                  'Mapa de 6 pasos para hoteleros independientes',
                  'Guía para implementar tu canal directo y reducir comisiones OTA',
                  'Manual Operativo Estándar (SOP) listo para usar',
                  'Esta calculadora RevDiagnostic (versión editable en Excel)',
                  'RevManagement Tool — tarifas dinámicas por BAR y temporada',
                ].map((item, i) => (
                  <li key={i} className="flex items-start gap-3 text-slate-700 text-sm">
                    <span className="w-5 h-5 rounded-full bg-emerald-500 text-white flex items-center justify-center flex-shrink-0 mt-0.5 text-[11px] font-bold leading-none">
                      ✓
                    </span>
                    {item}
                  </li>
                ))}
              </ul>
              <div className="flex flex-col sm:flex-row gap-3">
                <a
                  href="#"
                  className="inline-flex items-center justify-center gap-2 px-6 py-3.5 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold rounded-xl transition-colors text-sm shadow-lg shadow-emerald-500/20"
                >
                  Quiero el Kit Express — $47 →
                </a>
                <a
                  href="https://calendar.app.google/wa6RuUDBE1sCWCrEA"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="inline-flex items-center justify-center gap-2 px-6 py-3.5 bg-white hover:bg-slate-50 text-slate-700 font-semibold rounded-xl border border-slate-200 transition-colors text-sm"
                >
                  O agenda una llamada gratuita →
                  <span className="text-xs font-normal text-slate-400 ml-1">30 min · sin compromiso</span>
                </a>
              </div>
            </div>
          </section>

```

**Note:** The `href="#"` on the first button is intentional — the Kit Express landing page URL is not yet available. Leave it as `#`. Do NOT invent a URL.

### Verification for Fix 3

1. Run any diagnostic with valid inputs
2. Scroll to the bottom of the results
3. A green/teal section appears with "Kit de Hotelería Express · $47 USD" label, the 5 bullet points, and two buttons
4. That section appears **above** (before) the dark `bg-slate-900` consulting section
5. The calendar button opens `https://calendar.app.google/wa6RuUDBE1sCWCrEA` in a new tab
6. On mobile (narrow viewport): both buttons stack vertically

---

## Fix 4 — GOPPAR Status Threshold (estimated 5 min)

### Problem

The current GOPPAR status logic uses a hardcoded dollar amount that makes no sense:

```javascript
gopparStatus = kpi.goppar >= 60 ? 'good' : (kpi.goppar >= 30 ? 'warn' : 'bad');
```

`60` is neither a meaningful MXN nor USD threshold. The fix: use the **GOP Margin percentage** (`gopMargin`) already calculated by the app, with standard hotel industry benchmarks (30% / 15%).

### The fix — replace one line with one line

Search for this exact line (it appears once):

```javascript
gopparStatus = kpi.goppar >= 60 ? 'good' : (kpi.goppar >= 30 ? 'warn' : 'bad');
```

Replace it with:

```javascript
gopparStatus = kpi.gopMargin >= 30 ? 'good' : (kpi.gopMargin >= 15 ? 'warn' : 'bad');
```

**Why this works:** `gopMargin` is already calculated as `((roomRevenue - operatingCosts) / roomRevenue) × 100`. It's a percentage, so it's currency-agnostic and scale-agnostic — a 30% GOP margin is healthy whether the hotel is in MXN or USD, and whether it has 20 or 200 rooms.

**Thresholds:**
- ≥ 30% → green (good): hotel covers all costs and generates meaningful surplus
- 15–29% → yellow (warn): operating but thin margins, no cushion for surprises
- < 15% → red (bad): critical, almost nothing left after operating expenses

### Verification for Fix 4

Use these inputs to force specific gopMargin values:

**Test A — gopMargin = 35.9% → should be GREEN:**
- rooms=1500, occupied=975, roomRevenue=780000, operatingCosts=500000
- gopMargin = (780000-500000)/780000 × 100 = 35.9% → green ✓

**Test B — gopMargin = 20% → should be YELLOW:**
- rooms=1500, occupied=975, roomRevenue=780000, operatingCosts=624000
- gopMargin = (780000-624000)/780000 × 100 = 20% → warn ✓

**Test C — gopMargin = 8% → should be RED:**
- rooms=1500, occupied=975, roomRevenue=780000, operatingCosts=717600
- gopMargin = (780000-717600)/780000 × 100 = 8% → bad ✓

Also verify: after toggling currency, the goppar STATUS updates correctly (because gopMargin recalculates based on the converted values).

---

## Fix 5 — Update "Costos Operativos" Field Help Text (estimated 5 min)

### Problem

The current help text for `operatingCosts` says "departamento de habitaciones" and excludes some costs (marketing, admin). The app now uses TOTAL operating costs, and users need to know what to include — plus a disclaimer so they don't think we made an accounting error.

### The fix

Find this exact attribute block in the `operatingCosts` Field component:

```
helpTitle="Costos operativos del departamento de habitaciones"
                helpBody="Suma de costos directos asociados a operar habitaciones: housekeeping, lavandería, amenities, mantenimiento de habitaciones, comisiones OTA, energía, agua. NO incluye costos fijos generales (renta, deuda, marketing corporativo)."
                helpHow="Estado de resultados → línea 'Rooms Department Expenses'. Si no lo tienes desglosado, suma nóminas de housekeeping + lavandería + amenities + comisiones OTA del periodo."
```

Replace those three attributes with:

```
helpTitle="¿Qué incluir en Costos Operativos?"
                helpBody="Suma TODO lo que el hotel gasta para operar: sueldos del personal completo (housekeeping, recepción, gerencia), comisiones de OTA (Booking, Expedia, etc.), marketing y publicidad, servicios públicos (luz, agua, gas), suministros de habitación y lavandería. NO incluyas impuestos, depreciación, pago de deuda ni renta del inmueble."
                helpHow="Cómo obtenerlo: estado de resultados → suma todas las líneas de gasto operativo antes de EBITDA. Sin desglose: nómina total + comisiones OTA + facturas de servicios del periodo. · Nota: sí, las comisiones OTA y sueldos de gerencia son técnicamente 'gastos administrativos' en contabilidad formal — los incluimos aquí a propósito para no pedirte 5 estados de cuenta distintos. Lo importante: usa siempre la misma definición para que tu diagnóstico sea comparable mes a mes."
```

### Verification for Fix 5

1. Open the tool
2. Hover over (or tap) the `?` icon next to "Costos operativos"
3. A popup appears with title "¿Qué incluir en Costos Operativos?"
4. The main body lists all cost types to include (staff, OTAs, marketing, utilities)
5. At the bottom of the popup, in a teal box, the note explains why OTAs/admin are included here

---

## Final Smoke Test — Run Before Saving

**Test inputs (MXN mode):**
- rooms: `1500`
- occupied: `975`
- roomRevenue: `780000`
- operatingCosts: `500000`
- marketAdr: `850`
- marketOcc: `70`

**Expected outputs:**
| KPI | Expected value |
|---|---|
| RevPAR | $520 MXN |
| ADR | $800 MXN |
| Occupancy | 65.0% |
| GOPPAR | $186.67 MXN |
| GOP Margin | 35.9% → status GREEN ✓ |

**Currency toggle test:**
1. With the inputs above entered, switch to USD
2. `roomRevenue` becomes `~44,571` USD
3. `operatingCosts` becomes `~28,571` USD
4. `marketAdr` becomes `~48.57` USD
5. `rooms` stays `1500` (unchanged)
6. `occupied` stays `975` (unchanged)
7. `marketOcc` stays `70` (unchanged)
8. Switch back to MXN → `roomRevenue` returns to **exactly** `780000`

**Upsell section:**
- Scroll to bottom of results → green/teal Kit Express section appears ABOVE the dark consulting section

**Help text:**
- Hover `?` on Costos Operativos → shows new title and disclaimer note

If all tests pass: save `index.html`. Do not make any other changes.

---

## Deploy to Vercel

1. Go to vercel.com → click **Add New → Project** → choose **"Deploy without Git"** or drag-and-drop
2. Drop the `index.html` file → Framework: **Other** → click Deploy
3. Copy the `*.vercel.app` URL — the tool is live immediately, share this while DNS propagates
4. **Add custom domain** `revdiagnostic.host-feast.online`:
   - Vercel dashboard → your project → **Settings → Domains** → Add domain → type `revdiagnostic.host-feast.online`
   - Vercel shows you a **CNAME record** to add (value will be something like `cname.vercel-dns.com`)
   - Go to your DNS registrar (where `host-feast.online` is registered) → DNS settings → add:
     - **Type:** CNAME
     - **Name:** `revdiagnostic`
     - **Value:** (the value Vercel showed you)
   - Save. Wait 5–60 minutes. Vercel auto-provisions HTTPS once DNS resolves.

---

## What NOT to do

- Do NOT rewrite the JSX structure or move components around
- Do NOT add a build step, package.json, or any npm packages
- Do NOT change any KPI formula (RevPAR, ADR, occupancy, goppar, gopMargin) — they are correct
- Do NOT convert `rooms`, `occupied`, or `marketOcc` when the currency toggles — they are unitless
- Do NOT remove the existing dark consulting CTA section — the Kit Express section goes ABOVE it
- Do NOT use `<Check>` or any Lucide component — use the text `✓` character instead
- Do NOT add code comments explaining what the code does

---

## Summary of all changes (5 edits in 4 places)

| # | What to find | What to change |
|---|---|---|
| 2a | Line above `CURRENCIES = {` | Add `const MXN_PER_USD = 17.5;` before it |
| 2b | `const set = (k) =>` | Add `MONETARY_FIELDS` + `handleCurrencyChange` after it |
| 2c | `onClick={() => setCurrency(c.code)}` | Replace with `onClick={() => handleCurrencyChange(c.code)}` |
| 3 | `{/* ---------- CTA ---------- */}` line | Insert Kit Express `<section>` block before it |
| 4 | `gopparStatus = kpi.goppar >= 60` | Replace with `gopparStatus = kpi.gopMargin >= 30 ? 'good' : (kpi.gopMargin >= 15 ? 'warn' : 'bad');` |
| 5 | `helpTitle="Costos operativos del departamento` | Replace helpTitle + helpBody + helpHow with updated versions |
