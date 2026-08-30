# Filament Spanish fiscal compliance for Filament v5

<p align="center">
    <img
        src="/assets/01.jpg"
        alt="Filament Spanish fiscal compliance for Filament v5"
        width="49%"
    >
    <img
        src="/assets/certificados-resource.jpeg"
        alt="Certificates management — per-issuer and global social-collaborator certificates"
        width="49%"
    >
</p>

Official documentation for
[Filament Verifactu](https://github.com/komma-softhouse/filament-verifactu),
a Filament v5 fiscal compliance engine for VERI*FACTU, TicketBAI,
FACe and FACeB2B.

<p>
    <a href="https://github.com/komma-softhouse/filament-verifactu">
        Plugin repository
    </a>
    ·
    <a href="https://packagist.org/packages/komma-softhouse/filament-verifactu">
        Packagist
    </a>
</p>

VERI\*FACTU / RRSIF and TicketBAI compliant fiscal engine for
[Filament](https://filamentphp.com) 5.x: chained fiscal records, AEAT and
foral (Araba/Bizkaia/Gipuzkoa) submission, XAdES signing, a full document
lifecycle, FACe/FACeB2B electronic invoicing, and fiscal reporting drafts —
for Spanish businesses that need their invoicing to comply with Real
Decreto 1007/2023, Orden HAC/1177/2024 and (for the three Basque
territories) their own TicketBAI regulation.

[![Latest Version on Packagist](https://img.shields.io/packagist/v/komma-softhouse/filament-verifactu.svg?style=flat-square)](https://packagist.org/packages/komma-softhouse/filament-verifactu)
[![Tests](https://img.shields.io/github/actions/workflow/status/komma-softhouse/filament-verifactu/tests.yml?branch=main&label=tests&style=flat-square)](https://github.com/komma-softhouse/filament-verifactu/actions/workflows/tests.yml)
[![Total Downloads](https://img.shields.io/packagist/dt/komma-softhouse/filament-verifactu.svg?style=flat-square)](https://packagist.org/packages/komma-softhouse/filament-verifactu)

## What this plugin does

- **Chains and hashes fiscal records** exactly as RRSIF requires, with a
  single gate (`RecordService`) that locks the issuer, links the new record
  to the previous hash, validates it against the AEAT engine, and persists
  it — impossible to fork, impossible to edit afterwards.
- **Submits to the AEAT** in both remission modes: continuous (VERI\*FACTU)
  and on-demand (signed XML batches, answering a requirement or opened as a
  voluntary remission whenever the issuer decides — a dedicated panel
  action carries the legal context), respecting the AEAT's flow-control
  window. The continuous pipeline refuses non-Verifactu issuers by design:
  their records only ever leave on demand.
- **Reads back from the AEAT** (`ConsultaFactuSistemaFacturacion`): asks
  the office what IT holds for a record — state, presentation data, and
  the hash it stores, cross-checked against our own chain — and translates
  every rejection through the official 239-code error catalog, with its
  severity (whole-submission / this-record / accepted-but-fix).
- **Submits to TicketBAI** (Araba, Bizkaia, Gipuzkoa) through the exact same
  `RecordService` gate — a completely separate engine under the hood, with
  its own signature-value chaining, registration, cancellation, corrective
  (rectificativa) invoices, and Zuzendu subsanación (Araba/Gipuzkoa; Bizkaia
  does not implement it). Bizkaia submissions are automatically wrapped in
  the correct Batuz/LROE envelope (Modelo 140 autónomo or 240 sociedad).
- **Signs with XAdES-BES** when an AEAT issuer operates in non-Verifactu
  mode (VERI\*FACTU systems are exempt from signing per art. 16.3 — hash
  only); TicketBAI records are always signed, per its own regulation.
  Certificates live per issuer, with a **global social-collaborator
  certificate** (the advisory-firm scenario: one gestoría certificate
  signing for every client issuer that has none of its own) as automatic
  fallback across all four signers.
- **Runs the whole document lifecycle**: quotes → orders → delivery notes →
  proformas → invoices/tickets, a forward-only conversion graph (including
  merging several delivery notes into one invoice), credit notes with
  per-line returns, per-line and general discounts, IRPF withholding,
  hospitality pre-bills and kitchen routing, and SAT repair orders (with a
  customer-facing tracking QR) that fiscalize on delivery.
- **Prints**: A4 with automatic draft/copy watermarking, and 14 kinds of
  ESC/POS thermal ticket — including a paired **Print Agent** add-on
  (Windows service installer generated from the panel) for hosts that can't
  talk to a receipt printer directly from the browser. The tributary QR can
  also be inspected on screen, with a live self-check of its URL against
  the record and the issuer environment (sandbox/production drift is
  flagged before it ever reaches paper).
- **Sends to FACe** (public-sector electronic invoicing) and **FACeB2B**
  (Ley 25/2013's separate, already-active extension to large private
  subcontractors of public administrations — full lifecycle, both as sender
  and as receiver: send, cancel, download, reject, mark as paid, resolve
  cancellations, validate signatures), with the full compliance pack a real
  town hall validated: three DIR3 centres, Ref. Receptor contract
  reference, issuer IBAN as transfer account, due date, optional embedded
  invoice PDF, TSA timestamping, and a pre-flight validator. Facturae can
  also be generated **signed or unsigned** for external inspection, an
  existing paper/PDF invoice can be transcribed by OCR into a standalone
  Facturae (amounts copied literally, never recomputed), and every
  delivered XML stays inspectable pretty-printed from the history. It also
  offers OCR-assisted drafting from a photo via
  [Laravel AI](https://laravel.com/docs/13.x/ai-sdk) — the host picks any
  provider they already hold a key for, never locked to one.
- **Exposes an on-prem API sidecar** so external ERPs/POS terminals can
  register records and fetch a QR synchronously, authenticated per issuer,
  with signed webhooks on every accepted/rejected submission — and an API
  panel page listing every endpoint, managing per-issuer bearer keys and
  showing a truthful first call.
- **Ships advisory-firm bulk flows**: select a period's documents and
  download one ZIP of PDFs or one ZIP of signed Facturae, and requeue
  rejected records in bulk.
- **Gates every panel action behind a configurable authorization system**
  (see [Authorization](#authorization)) — everything enabled by default,
  each of the 39 abilities switchable globally or per user.
- **Drafts Modelos 303, 347, 349 (real E/S classification from each
  record's own data), 111 and 115** for the accountant, each exportable
  as CSV.

Every surface beyond the core fiscal resources ships **disabled by
default** — a host that registers only the plugin object gets the fiscal
chain and nothing else; every other module is a fluent opt-in.

## Requirements

- PHP >= 8.4
- Laravel >= 13.0
- Filament >= 5.0

## Installation

This is a commercial package distributed through
[Anystack](https://anystack.sh) — it is not on public Packagist. Your
license key is what authenticates the download.

**1. Add the private repository** to your project's `composer.json`:

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://filament-verifactu.composer.sh"
        }
    ]
}
```

**2. Require the package:**

```bash
composer require komma-softhouse/filament-verifactu
```

Composer will prompt for authentication against
`filament-verifactu.composer.sh`:

- **Username**: the email address your license is registered to.
- **Password**: your license key. If your license policy requires a
  fingerprint, append it to the key separated by a colon —
  `your-license-key:your-domain.com` — using the fingerprint you entered
  when activating the license.

Answer yes when Composer offers to store the credentials (they go in
`auth.json` — add that file to `.gitignore`). For CI, configure the same
credentials as http-basic auth from your secrets instead:

```bash
composer config http-basic.filament-verifactu.composer.sh your@email.com YOUR-LICENSE-KEY
```

**3. Migrate:**

```bash
php artisan migrate
```

Register the plugin on your panel provider:

```php
use Komma\Verifactu\VerifactuPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        ->plugin(
            VerifactuPlugin::make()
                ->documents()
                ->repairs()
                ->gotenbergPdf('http://gotenberg:8')
                ->escPos()
                ->face()
                //->einvoincing() See Configuration below for details.
                ->ocr()
                ->api(),
        );
}
```

Publish and fill in the config:

```bash
php artisan vendor:publish --tag="filament-verifactu-config"
```

At minimum, set the `ComputerSystem` (implementer identity) variables before
the first submission — see [Configuration](#configuration).

## Configuration

| Toggle | What it enables |
| --- | --- |
| `->documents()` | Document, series-format and template resources, the document lifecycle |
| `->repairs()` | SAT repair order resource |
| `->gotenbergPdf($url)` | A4 PDF rendering via a Gotenberg instance |
| `->escPos()` | Thermal ticket printing |
| `->printAgent()` | Print agent pairing page and installer generation |
| `->einvoincing()` | Pending BOE publication of the Orden Ministerial (Ley Crea y Crece / RD 238/2026) |
| `->face()` | FACe: the "Send to FACe" action, history, settings, generate and directory pages |
| `->faceb2b()` | FACeB2B: the "Send to FACeB2B" action and cancellation requests |
| `->ocr()` | The "Create from photo" OCR-assisted draft action and its settings page |
| `->api()` | *(informational — see below)* |

The core fiscal resources (Issuers, Fiscal records, Submissions, System
events, Audit trail), the Certificates and API panel pages, and the five
report pages are always registered.

The **API sidecar** is activated independently, via config/env
(`VERIFACTU_API=true`), not through the fluent `->api()` call: it is meant
for LAN-local ERPs and POS terminals that never load a Filament panel, so it
cannot depend on a toggle that only resolves once a panel does.

See the published `config/filament-verifactu.php` for every environment
variable. The full list:

```
# ComputerSystem identification (RRSIF art. 8.1/13 — the implementer's
# responsible declaration; required before the first submission).
# Use YOUR OWN company's data here — you are the implementer.
VERIFACTU_VENDOR_NAME="Your Company SL"
VERIFACTU_VENDOR_NIF=B00000000
VERIFACTU_SYSTEM_NAME="Komma Fiscal"
VERIFACTU_SYSTEM_ID=01
VERIFACTU_SYSTEM_VERSION=1.0
VERIFACTU_INSTALLATION_NUMBER=1

# TicketBAI implementer identity (Araba/Bizkaia/Gipuzkoa only) — the foral
# equivalent of the ComputerSystem block above. The per-issuer software-
# guarantor license itself is NOT here: it's set per issuer from the
# Verifactu settings page (Issuer::tbai_license).
VERIFACTU_TBAI_DEVELOPER_NIF=
VERIFACTU_TBAI_SYSTEM_NAME="Komma Verifactu"
VERIFACTU_TBAI_SYSTEM_VERSION=1.0

# Submission pipeline
VERIFACTU_QUEUE=verifactu
VERIFACTU_BATCH_SIZE=1000
VERIFACTU_CERTIFICATES_DISK=local
VERIFACTU_CERTIFICATES_DIR=verifactu/certificates

# Representative/collaborator header (requires AEAT form GENERALLEY58 filed
# by the taxpayer being represented)
VERIFACTU_REPRESENTATIVE_NAME=
VERIFACTU_REPRESENTATIVE_NIF=

# AEAT read-back (ConsultaFactuSistemaFacturacion) — defaults follow the
# official WSDL; override only if the AEAT moves the service.
VERIFACTU_AEAT_QUERY_SANDBOX=https://prewww1.aeat.es
VERIFACTU_AEAT_QUERY_PRODUCTION=https://www1.agenciatributaria.gob.es
VERIFACTU_AEAT_QUERY_PATH=/wlpl/TIKE-CONT/ws/SistemaFacturacion/ConsultaSOAP

# PDF rendering
VERIFACTU_PDF_DRIVER=gotenberg
GOTENBERG_URL=http://gotenberg:8

# ESC/POS printing, print agent
VERIFACTU_ESCPOS=true
VERIFACTU_PRINT_AGENT=false

# FACe (public-sector) / FACeB2B (large private subcontractors, Ley 25/2013)
# — module toggles only. Seller address, FACe defaults and OCR provider
# are NOT env vars: they're set from their own settings pages
# (FacturaeSettingsPage → FaceSettings, OcrSettingsPage → OcrSettings,
# both spatie/laravel-settings), editable without touching .env or deploying.
VERIFACTU_FACE=true
VERIFACTU_FACEB2B=false

# Optional TSA timestamping of the Facturae XAdES signature
VERIFACTU_FACE_TSA_URL=
VERIFACTU_FACE_TSA_USER=
VERIFACTU_FACE_TSA_PASSWORD=

# OCR-assisted drafting (module toggle only — the provider itself, model
# and enabled flag are all set from OcrSettingsPage; any Laravel AI
# provider works, the host supplies its own key in their own config/ai.php)
VERIFACTU_OCR=true

# Documents, repairs, on-prem API sidecar
VERIFACTU_DOCUMENTS=true
VERIFACTU_REPAIRS=true
VERIFACTU_API=true
```

## Authorization

Every panel action — completing, voiding, sending to FACe, uploading
certificates, generating API keys, all 44 of them — is gated through
`VerifactuGate`, resolved in three layers, **most specific first and
permissive by default** (a drop-in install shows everything):

1. **A programmatic resolver**, when the host registered one — the escape
   hatch for centralized custom logic. Return `null` to fall through to
   the next layer:

   ```php
   use Komma\Verifactu\Support\VerifactuGate;

   // e.g. in a service provider's boot():
   VerifactuGate::resolveUsing(function (string $ability): ?bool {
       if (str_starts_with($ability, 'download-')) {
           return auth()->user()?->can('export-fiscal-data');
       }

       return null; // everything else falls through
   });
   ```

2. **A Laravel Gate** named `verifactu.{ability}` — per-user control that
   plays naturally with policies and spatie/laravel-permission. A defined
   Gate always wins over the config switch:

   ```php
   use Illuminate\Support\Facades\Gate;

   Gate::define('verifactu.void-document', fn ($user) => $user->hasRole('manager'));
   Gate::define('verifactu.upload-certificate', fn ($user) => $user->hasPermissionTo('manage certificates'));
   ```

3. **The config switch** — `config('filament-verifactu.permissions.{ability}')`,
   the global on/off layer, `true` when absent. Flip one to `false` to
   hide that action for everyone:

   ```php
   // config/filament-verifactu.php
   'permissions' => [
       'void-document' => false,   // nobody voids from the panel
       'query-aeat' => true,
       // ... all 39 keys ship in the published config, grouped by domain
   ],
   ```

An unauthorized action is **hidden** (Filament's native behavior): a
cashier who cannot void documents never sees the button. The
"How does it work?" help modals are deliberately ungated — read-only
guidance stays visible to everyone.

### The full ability list

| Ability | Gates |
| --- | --- |
| **Documents** | |
| `complete-document` | Completing a draft (numbering + chaining the fiscal record) |
| `convert-document` | Forward conversions (quote → order → … → invoice) |
| `create-from-photo` | OCR-assisted draft from a photo |
| `merge-into-invoice` | Merging delivery notes into one invoice (bulk) |
| `void-document` | Credit notes / cancellations |
| `reprint-ticket` | Thermal reprint (the COPIA-marked duplicate) |
| `send-by-email` | Emailing the document PDF |
| `download-document` | PDF and HTML downloads of a document |
| **FACe / Facturae** | |
| `send-to-face` | Send to FACe action |
| `send-to-faceb2b` | Send to FACeB2B action |
| `download-facturae` | Signed/unsigned Facturae download from a document |
| `generate-facturae` | The "New Facturae" page action |
| `generate-facturae-ocr` | The "From PDF / photo (OCR)" standalone flow |
| `check-face-registration` | Polling FACe and persisting the tramitación status |
| `resend-to-face` | Resending a stored submission |
| `cancel-faceb2b` | FACeB2B cancellation requests |
| `download-xsig` | Downloading the stored signed `.xsig` |
| `view-face-xml` | The pretty-printed XML inspector |
| **Advisory-firm bulks** | |
| `download-pdf-zip` | ZIP of selected documents' PDFs |
| `download-facturae-zip` | ZIP of selected invoices as signed Facturae |
| **Issuers & fiscal lifecycle** | |
| `create-issuer` | Creating an issuer from auxiliary screens |
| `activate-fiscal` | The sealed, irreversible fiscal activation |
| `switch-to-verifactu` | The one-way non-Verifactu → VERI\*FACTU switch |
| `edit-tbai-settings` | TicketBAI license and territory settings |
| `remit-on-demand` | The non-Verifactu voluntary/requirement remission |
| `upload-certificate` | Uploading an issuer certificate |
| `remove-certificate` | Removing an issuer certificate |
| `manage-social-collaborator` | The global gestoría certificate (upload and remove) |
| `generate-api-key` | Generating a bearer key |
| `revoke-api-key` | Revoking a bearer key |
| **Fiscal records** | |
| `resend-record` | Requeueing a rejected record (single and bulk) |
| `download-tbai-xsig` | Downloading the signed TicketBAI XML |
| `query-aeat` | The AEAT read-back (quick status and detailed modal) |
| `view-qr` | Viewing and downloading the tributary QR |
| **Templates & SAT** | |
| `manage-templates` | Opening the template designer |
| `transition-repair` | Moving a repair order through its lifecycle |
| `deliver-repair` | Delivering (which issues the fiscal document) |
| `print-repair-paper` | Deposit receipt and repair report printing |
| `send-repair-by-email` | Emailing either SAT paper |

Groupings are deliberate: receipt and report share `print-repair-paper`,
the quick and detailed AEAT queries share `query-aeat`, viewing and
downloading the QR share `view-qr`, PDF and HTML downloads share
`download-document`, and the bulk requeue falls under `resend-record` —
same power, different scale.

Note the boundary: this system gates **panel actions**. Resource-level
CRUD (viewing lists, editing an issuer's plain fields) follows Filament's
standard model policies, exactly as in any Filament app; and the API
sidecar authenticates by bearer key per issuer, independent of panel
permissions.

## Architecture notes

- `invoice_number` always stores series+number concatenated (e.g.
  `F2026-0001`), matching what AEAT expects as `NumSerieFactura`.
  `hashed_at` is always UTC; the chain hash payload uses `format('c')` on
  that UTC instant, per the exact formula the AEAT engine requires.
- Series are resolved per issuer and document type with placeholders
  `{PREFIX}{YEAR}{STORE}{TERMINAL}` (default prefixes Q·PE·A·P·F·T·R);
  changing a format resolves to a different series string going forward —
  the previous series stays closed with its numbering intact.
- The Facturae seller block resolves **per issuer**: NIF and legal name
  always come from the issuer; its fiscal address wins when filled, the
  installation-wide one from the Facturae settings is the fallback — the
  advisory-firm scenario (many issuers, one installation) depends on it.
- **Multi-tenant hosts, read this first:** `verifactu_issuers.nif` is
  UNIQUE globally across the whole database. If you run this behind a
  multi-tenant package (Stancl or similar), add a `tenant_id` column and a
  compound unique constraint (`tenant_id` + `nif`) in your own migration
  before going live — two different tenants both legitimately having their
  own `B12345678`-style issuer will otherwise collide.

## Legal references

Verified against BOE-A-2023-24840, Real Decreto 1007/2023, consolidated
text as of 03/12/2025:

- Art. 8.2.a — integrity/immutability duty → enforced by the append-only
  guard and the sealed-issuer guard on `Issuer`.
- Art. 8.2.b — traceability/chaining duty → `RecordService`,
  `ChainVerifier`.
- Art. 8.3 (Reglamento) / Orden HAC/1177/2024 art. 9 — event log duty →
  the system events log.
- Art. 12 — hash and electronic signature duty → `SignatureService`.
- Art. 16.3 — VERI\*FACTU systems are exempt from signing (hash only) →
  `SignatureService` is only invoked in non-Verifactu mode.
- Arts. 15–16 — voluntary remission and the VERI\*FACTU system definition →
  `SubmissionContext`, on-demand submission (requirement/voluntary) and the
  "Remit on demand" panel action.
- The QR + "Factura verificable en la sede electrónica de la AEAT" /
  "VERI\*FACTU" legend requirement comes from RD 1619/2012 arts. 6.5/7.5, as
  introduced by RD 1007/2023's disposición final primera, referencing RRSIF
  arts. 15–16 → `QrRenderer`, plus a self-check of the generated QR URL
  (parameters against the record, environment against the issuer) wired
  into the QR view modal and the homologation battery.
- **RD 1619/2012 (Reglamento de facturación) is the backbone of the whole
  document layer**: art. 6 (full invoice content — why an F1 requires a
  named recipient with NIF, enforced by `DocumentGate`); art. 7
  (simplified invoices/F2 and their thresholds — ≤400€ VAT included in
  general, or ≤3.000€ in the traditionally ticket-authorised retail
  sectors); art. 13 (recapitulative invoices — the legal figure behind
  merging several delivery notes for the same customer into one invoice);
  art. 14 (duplicates must be marked as such — the "COPIA" watermark and
  the print-copy audit trail); art. 15 (corrective invoices — mandatory
  when the original breaches art. 6/7 requirements or quotas were wrong —
  `CreditNoteService`, R1-R5).
- Modelo 349's legal basis: arts. 78–81 of the VAT Reglamento (RD
  1624/1992) regulate the recapitulative declaration of intra-community
  operations; the modelo itself is approved by Orden EHA/769/2010 → the
  per-line E/S operation-key classification in the 349 draft.
- Modelo 347 threshold: €3,005.06/year, unchanged for decades.
- Modelo 111/115 default rates verified current as of this build: 15%
  general professional fees, 7% new professionals (first 3 years), 19%
  rent withholding.
- Modelo 303's output VAT (devengado) is derived from chained sales
  records; input VAT (soportado) is not tracked by this plugin (it belongs
  to the purchase ledger) and is supplied manually before the result box
  is meaningful.
- FACe response parsing matches the documented `FaceClient` response shape
  (`resultado.codigo` "0" = success; `factura.numeroRegistro`,
  `factura.organoGestor` on success), and registration checks map the
  tramitación codes (1200/1300/2400 registered · 2500 paid · 2600/3100
  rejected) onto the submission's lifecycle status.
- AEAT rejection codes are translated through the official
  `errores.properties` catalog (239 codes) with the catalog's own three
  severities: whole-submission rejection, per-record rejection, and
  accepted-but-must-fix.

TicketBAI (each territory regulates it through its own foral instruments;
the obligation itself is anchored identically in each territory's IRPF and
Sociedades Normas Forales):

- **Gipuzkoa** — Norma Foral 3/2020 (the TicketBAI obligation) and Decreto
  Foral 32/2020, de 22 de diciembre (its development reglamento) →
  registration/cancellation via the Gipuzkoa endpoint, Zuzendu supported.
- **Araba** — Norma Foral 13/2021, de 21 de abril (the obligation, BOTHA)
  and its development regulations → registration/cancellation via the
  Araba endpoint, Zuzendu supported.
- **Bizkaia** — Norma Foral 5/2020, de 15 de julio (the Batuz system:
  TicketBAI + the LROE ledger + administration-drafted returns) and Norma
  Foral 8/2023, de 22 de noviembre (definitive implantation ordering) →
  every Bizkaia submission is wrapped in its Batuz/LROE envelope (Modelo
  140 for autónomos, 240 for sociedades) automatically; Zuzendu is not
  part of Bizkaia's system (its endpoint rejects it — see Scope).
- The per-territory **XAdES signing policies** (policy identifier +
  digest each foral treasury publishes — e.g. Bizkaia's
  `batuz.eus/.../especificaciones_de_la_firma_electronica_v1_1.pdf`,
  Gipuzkoa's `gipuzkoa.eus/ticketbai/sinadura`) → applied automatically
  per issuer regime when signing; TicketBAI records are always signed,
  unlike VERI\*FACTU's art. 16.3 exemption.
- The **TBAI identifier + QR** on every invoice and the **chaining by the
  previous invoice's own signature value** (not by hash, as AEAT chains) →
  `QrRenderer` and the TicketBAI fingerprint in `RecordService`.
- The **software-guarantor license** each foral treasury issues per
  registered software → stored per issuer (`Issuer::tbai_license`),
  editable from the Verifactu settings page.

Facturae / FACe / FACeB2B:

- **Ley 25/2013, de 27 de diciembre** (impulso de la factura electrónica y
  creación del registro contable de facturas en el Sector Público) — the
  foundational law: art. 4 defines who must invoice public administrations
  electronically; its **disposición adicional segunda** is what fixes the
  structured **Facturae 3.2** format itself → `FacturaeInvoiceBuilder`
  (builds 3.2.2).
- **Orden HAP/1074/2014, de 24 de junio** — the technical and functional
  conditions of the Punto General de Entrada de Facturas Electrónicas
  (FACe as the AGE's PGEFe); it also anchors the **advanced electronic
  signature** requirement on the invoice (per RD 1619/2012 art. 10.1.a) →
  the XAdES signing step in `FacturaeInvoiceBuilder`. Signing is optional
  only for local inspection downloads (an unsigned `.xml` to validate
  externally); every submission to FACe is signed, always — the builder
  throws rather than ever exporting a silently-unsigned submission.
- **Orden PRE/2971/2007, de 5 de octubre** — the original order
  establishing the facturae XML format for invoices addressed to the AGE.
- FACe routes every invoice by **DIR3 codes** (órgano gestor / oficina
  contable / unidad tramitadora) — all **three centres** travel on every
  submission, gestor/tramitador falling back to the accounting office
  (the single-code town-hall case) → the DIR3 directory the plugin
  remembers per (issuer, recipient) and its management screen.
- **Ley 9/2017, de 8 de noviembre (LCSP), disposición adicional trigésima
  segunda, apartado 3** — creates the **FACeB2B** registry: subcontractors
  in the art. 4.1 (Ley 25/2013) cases must invoice their main contractor
  electronically through it when the invoice exceeds €5,000 → the whole
  `->faceb2b()` module (send, cancel, and the receiver-side lifecycle),
  routed by DIRe codes rather than DIR3. Note RD 238/2026's transitional
  regime: FACeB2B will eventually be replaced by the new general B2B
  e-invoicing system (24-month adaptation window from the pending Orden
  Ministerial) — which is exactly what the Roadmap's `->einvoicing()`
  toggle is reserved for.

## Dark mode and translations

Every screen this plugin adds is dark-mode aware out of the box — hooks
into Filament's own light/dark toggle with no configuration needed. The
panel itself ships in Spanish, Galician and Portuguese
(`resources/lang/{es,gl,pt}.json`) alongside English, and resolves
against your own application's `resources/lang/{locale}.json` too if you
add translations there.

## Usage

```php
use Komma\Verifactu\Facades\Verifactu;
use Komma\Verifactu\Enums\FiscalRegime;
use Komma\Verifactu\Models\Issuer;
use Komma\Verifactu\Models\FiscalDocument;

$issuer = Issuer::create(['name' => 'My Company SL', 'nif' => 'B12345678']);
$issuer->activateFiscal(FiscalRegime::Aeat); // sealed, irreversible

$document = FiscalDocument::create([
    'issuer_id' => $issuer->id,
    'type' => DocumentType::Ticket,
    'status' => DocumentStatus::Draft,
]);
$document->lines()->create(['description' => 'Coffee', 'quantity' => 1, 'unit_price' => '1.50', 'tax_rate' => '10.00']);

Verifactu::completeDocument($document); // numbers, freezes, chains the fiscal record
Verifactu::submitPending($issuer);
Verifactu::verifyChain($issuer);
```

Hosts with their own invoicing model can skip the document layer entirely
and call the fiscal engine directly:

```php
Verifactu::register($issuer, $invoiceData);   // InvoiceData DTO
Verifactu::cancel($issuer, $cancellationData);
```

## Minimal integration: bolting compliance onto existing host software

This is the scenario for a third-party developer who installs this plugin
into a shop or bar's **existing** POS/ERP — one that has no VERI\*FACTU or
TicketBAI compliance at all — and only needs two things: **register the
sale** and **get its QR**. None of `->documents()`, `->repairs()`,
`->escPos()`, FACe or OCR are required for this — only the core fiscal
resources, which are always registered.

### 1. One-time setup per issuer (regime picked once, sealed forever)

```php
use Komma\Verifactu\Enums\FiscalRegime;
use Komma\Verifactu\Enums\FiscalMode;
use Komma\Verifactu\Models\Issuer;

$issuer = Issuer::create(['name' => 'Bar Rosalía SL', 'nif' => 'B12345678']);

// AEAT / common territory:
$issuer->activateFiscal(FiscalRegime::Aeat, FiscalMode::NonVerifactu);

// Araba, Bizkaia or Gipuzkoa (TicketBAI) instead — VERI*FACTU mode does
// not exist there, only NonVerifactu is valid:
// $issuer->activateFiscal(FiscalRegime::Araba, FiscalMode::NonVerifactu);
// $issuer->update(['tbai_license' => 'the software-guarantor license the foral treasury issued you']);
```

Upload the issuer's certificate once (`CertificateService::store()`, the
"Upload certificate" action on `VerifactuSettingsPage`, or the Certificates
page — all three manage the same certificate). The service itself has no
Filament dependency at all:

```php
use Komma\Verifactu\Enums\CertificateHolderType;
use Komma\Verifactu\Services\CertificateService;

app(CertificateService::class)->store(
    $issuer,
    file_get_contents('/path/to/certificate.p12'),
    'the certificate passphrase',
    CertificateHolderType::Obligado, // or Representative / SocialCollaborator
);
```

An advisory firm (gestoría) managing many client issuers can instead
upload **one global social-collaborator certificate** — it signs and
submits for every issuer that has no certificate of its own, across
VERI\*FACTU, TicketBAI and Facturae alike:

```php
app(CertificateService::class)->storeSocialCollaborator(
    file_get_contents('/path/to/gestoria.p12'),
    'the passphrase',
);
```

Resolution is always issuer-first: an issuer's own certificate wins; the
social-collaborator one is the fallback; with neither, nothing signs.

### 2. Per sale: register, then print/display the QR

```php
use Komma\Verifactu\Facades\Verifactu;
use Komma\Verifactu\Data\{InvoiceData, BreakdownData, RecipientData};

$record = Verifactu::register($issuer, new InvoiceData(
    invoiceNumber: 'T2026-0001',       // the host's own series+number
    issuedOn: new DateTimeImmutable(),
    invoiceType: 'F2',                 // F2: simplified (ticket) — the common POS case
    description: 'Coffee and toast',
    breakdown: [
        new BreakdownData(
            taxType: '01', regimeType: '01', operationType: 'S1',
            baseAmount: '4.50', taxRate: '10.00', taxAmount: '0.45',
        ),
    ],
    totalTaxAmount: '0.45',
    totalAmount: '4.95',
));

// Same call for BOTH regimes — RecordService routes to AEAT or TicketBAI
// internally from $issuer->regime. Only difference: a named recipient on
// a TicketBAI issuer needs postalCode/address (AEAT/VERI*FACTU do not):
// recipients: [new RecipientData('Customer SL', 'B87654321', postalCode: '01001', address: 'Calle Mayor 1')]

$qrSvg = Verifactu::qrSvg($record);       // print/display straight away
$qrUrl = Verifactu::qrUrl($record);       // or just the URL, to build your own QR
```

**This is not the whole picture for an AEAT issuer.** `register()` only
chains and hashes the record locally (and, in non-Verifactu mode, signs it
XAdES) — it does **not** submit to the AEAT by itself, in either
remission mode. What actually sends depends on the issuer's mode:

- **VERI\*FACTU mode** — continuous cadence: schedule
  `php artisan verifactu:send-pending` every few minutes; it queues every
  pending record automatically.
- **Non-Verifactu mode** — records stay signed and local, and leave
  **only on demand, whenever the issuer decides**: a voluntary remission
  (no requirement needed — just pick the period end date) or an answer to
  an AEAT requirement. Both from the "Remit on demand" action on the
  Issuers list, or `php artisan verifactu:submit-batch` /
  `BatchService::submitVoluntarily()` / `submitOnRequirement()`
  programmatically. The continuous `verifactu:send-pending` pipeline
  refuses these issuers by design — a non-Verifactu issuer can never be
  drip-submitted by accident.

Skip all of this only for a **TicketBAI** issuer — `register()` there is
fully synchronous end-to-end, nothing else to run.

That is the entire integration surface: two calls (three if AEAT
VERI\*FACTU mode, for the submission cron), no Filament UI required on the
host's side at all (though `php artisan migrate` still needs to have run,
and the host process needs the certificate on disk/its configured disk,
and — for AEAT — a queue worker running).

### 3. Or integrate from outside PHP entirely: the on-prem API sidecar

For host software that isn't Laravel (or isn't PHP) at all, the same two
steps go over HTTP instead — enable it with `VERIFACTU_API=true` and
generate a bearer key per issuer from `VerifactuSettingsPage` or the API
panel page.

The same operational note from step 2 applies here too, just moved to the
*server* side: for an AEAT VERI\*FACTU issuer, whatever Laravel app is
actually running this plugin (even if it only ever receives calls from
the API, never loads its panel for anything else) still needs
`php artisan verifactu:send-pending` scheduled — the client calling the
API is never responsible for that, only for calling `store()`/`cancel()`.

```
POST /api/v1/verifactu/records
Authorization: Bearer vf_...
Content-Type: application/json

{
  "invoice_number": "T2026-0001",
  "issued_on": "2026-08-27",
  "invoice_type": "F2",
  "description": "Coffee and toast",
  "total_tax_amount": "0.45",
  "total_amount": "4.95",
  "breakdown": [
    {"tax_type": "01", "regime_type": "01", "operation_type": "S1", "base_amount": "4.50", "tax_rate": "10.00", "tax_amount": "0.45"}
  ],
  "recipients": [
    {"name": "Customer SL", "nif": "B87654321", "postal_code": "01001", "address": "Calle Mayor 1"}
  ]
}
```

`recipients` (and its `postal_code`/`address`, required only for a
TicketBAI issuer) is optional and omittable for a plain ticket with no
named buyer. The response includes the QR URL directly. To check whether
an AEAT submission actually resolved (without setting up a webhook
receiver — see below for that), poll the same id later:

```
GET /api/v1/verifactu/records/{id}
Authorization: Bearer vf_...
```

```json
{"id": 1, "status": "accepted", "error_code": null, "error_message": null, "qr_url": "https://..."}
```

```
GET /api/v1/verifactu/records/{id}/qr
Authorization: Bearer vf_...
```

For a TicketBAI issuer specifically, correcting an already-registered
invoice's data (Zuzendu — Araba/Gipuzkoa only, Bizkaia does not implement
it) goes through its own endpoint rather than a new corrective invoice:

```
POST /api/v1/verifactu/records/correct
Authorization: Bearer vf_...
Content-Type: application/json

{
  "original_invoice_number": "T2026-0001",
  "invoice_number": "T2026-0001",
  "issued_on": "2026-08-27",
  "invoice_type": "F2",
  "description": "Corrected description or amounts",
  "total_tax_amount": "0.45",
  "total_amount": "4.95",
  "breakdown": [
    {"tax_type": "01", "regime_type": "01", "operation_type": "S1", "base_amount": "4.50", "tax_rate": "10.00", "tax_amount": "0.45"}
  ]
}
```

Corrective (rectificativa/abono) invoices — a return or a billing error,
the single most common thing a shop or bar needs beyond a plain sale — go
through the exact same endpoint: set `invoice_type` to `R1`-`R5`,
`corrective_type` to `S` (substitution) or `I` (differences, the common
case for a partial return), and `corrected_invoices` to the original
sale's `invoice_number`/`issued_on`:

```json
{
  "invoice_number": "T2026-0002",
  "issued_on": "2026-08-27",
  "invoice_type": "R5",
  "description": "Return: cold coffee",
  "corrective_type": "I",
  "corrected_invoices": [{"invoice_number": "T2026-0001", "issued_on": "2026-08-27"}],
  "total_tax_amount": "-0.45",
  "total_amount": "-4.95",
  "breakdown": [
    {"tax_type": "01", "regime_type": "01", "operation_type": "S1", "base_amount": "-4.50", "tax_rate": "10.00", "tax_amount": "-0.45"}
  ]
}
```

Bearer keys are generated per issuer from `VerifactuSettingsPage` or the
API panel page.

A host that built its own `FiscalDocument` (again, plain Eloquent — create
the `FiscalDocument`/`DocumentLine` rows and call
`app(Komma\Verifactu\Documents\Services\DocumentGate::class)->complete($document)`
directly, no Filament UI required) can send it to FACe or FACeB2B the same
way. The `seller_*` fields are optional — the issuer's fiscal address and
the Facturae settings resolve them exactly as the panel does; send them
only to override. The three DIR3 centres, the contract reference and the
embedded-PDF flag travel too:

```
POST /api/v1/verifactu/face/send        (FACe — needs dir3_code + recipient_email)
POST /api/v1/verifactu/faceb2b/send     (FACeB2B — routes by buyer_nif, no DIR3)
Authorization: Bearer vf_...
Content-Type: application/json

{
  "document_id": 42,
  "buyer_name": "Concello de Ribeira", "buyer_nif": "P1507400H",
  "recipient_email": "notificaciones@ribeira.gal",
  "dir3_code": "L01150737",
  "dir3_gestor": null, "dir3_tramitador": null,
  "contract_reference": "EXP 2026/0042",
  "attach_pdf": true
}
```

## FACe and FACeB2B lifecycle

Beyond sending, this plugin keeps a full FACe history: **download**
re-streams the already-signed `.xsig` stored on the submission; **resend**
reuses that same stored XML (never rebuilds or re-signs it); **view XML**
shows the exact delivered Facturae pretty-printed; and **check
registration** polls FACe, maps its tramitación codes (registered · paid ·
rejected) and persists the new status on the submission — a town hall
marking the invoice paid shows up in the history. Every submission runs a
**pre-flight validator** first (totals against the engine's own math,
IBAN mod-97, the three DIR3 centres, duplicate contract references).
The DIR3 codes used for a registered send are remembered per (issuer,
recipient NIF) and pre-fill the next invoice to the same public body —
manageable from its own directory screen in the panel.

Two more Facturae flows live next to the send action: **generate without
submitting** (signed `.xsig`, or an unsigned `.xml` to inspect or run
through an external validator first), and **from PDF/photo (OCR)** — an
invoice that already exists on paper is transcribed literally (amounts
copied verbatim, never recomputed) into a standalone Facturae, downloaded
or delivered straight to FACe.

FACeB2B counterparties manage invoice state themselves rather than the
public administration, so it exposes a materially larger lifecycle than
FACe: sending, cancelling, and status-checking are wired into the panel
today; the fuller receiver-side lifecycle (download, reject, mark as
paid, resolve a sender's cancellation, validate signatures) is
implemented at the service/facade layer with no dedicated panel screen
yet — see Roadmap.

### 4. Getting notified without polling: webhooks (AEAT only)

For an **AEAT** issuer specifically, submission is asynchronous — `store()`
only chains and queues the record; the actual accept/reject verdict
arrives later, when `verifactu:send-pending` runs. Set `webhook_url` (and
optionally `webhook_secret`) on the `Issuer` — from `IssuerForm` in the
panel, or `$issuer->update(['webhook_url' => ..., 'webhook_secret' => ...])`
directly — and this plugin `POST`s the verdict there itself instead of
making the host poll:

```json
{
  "event": "submission.accepted",
  "issuer_nif": "B12345678",
  "submission_id": 42,
  "status": "accepted",
  "csv": "...",
  "records_count": 3,
  "error": null,
  "occurred_at": "2026-08-27T10:00:00+00:00"
}
```

If `webhook_secret` is set, the body is signed with HMAC-SHA256 in the
`X-Verifactu-Signature` header — verify it before trusting the payload.

**TicketBAI has no webhook of its own, and does not need one**: unlike
AEAT, `register()`/`correctTbaiInvoice()` there are fully synchronous —
the accepted/rejected verdict is already in the same response `store()`
gives back, so there is nothing later to be notified about.

## Notifying customers on repair status changes

Every transition on a repair order (`RepairService::transition()`,
including the initial "received" and the final "delivered") fires
`Komma\Verifactu\Documents\Events\RepairStatusChanged` — listen for it to
send an SMS/email/WhatsApp yourself; this plugin only builds the tracking
page behind each order's own QR (`RepairOrder::tracking_token`), it never
sends anything on its own:

```php
use Komma\Verifactu\Documents\Events\RepairStatusChanged;

Event::listen(function (RepairStatusChanged $event) {
    // $event->repairOrder, $event->from, $event->to (both RepairStatus enums)
    $phone = $event->repairOrder->customer['phone'] ?? null; // 'customer' is a plain JSON column — whatever shape the host stored
    if ($phone) {
        Notification::route('vonage', $phone)->notify(new RepairUpdated($event->repairOrder));
    }
});
```

## Roadmap

**B2B electronic invoicing** (Ley 18/2022 "Crea y Crece", RD 238/2026): the
Reglamento is already published (BOE, 31 March 2026) and mandates EN 16931 /
UBL for structured B2B invoices, but the Orden Ministerial fixing the
technical detail of Hacienda's public solution is still pending its
definitive BOE publication (public consultation closed 8 May 2026, in force
expected 1 October 2026 — the date that starts the 12/24-month countdown to
mandatory adoption in October 2027, >€8M turnover, and October 2028 for
everyone else). `->einvoicing()` will ship once that Order is published and
its technical annex is stable enough to build against. This is unrelated to
FACeB2B (Ley 25/2013), already fully supported today.

**FACeB2B received-invoices inbox**: sending, cancelling and status-checking
are built and wired into the panel; the full receiver-side lifecycle
(download, reject, mark as paid, resolve a sender's cancellation, validate
signatures) is implemented at the service/facade layer but has no dedicated
panel screen yet for browsing invoices this issuer has received.

## Scope

TicketBAI's Zuzendu (subsanación) is not implemented for Bizkaia — its own
`Endpoint` class rejects it outright; that territory's corrections go
through Batuz/LROE's own modification operations instead, which this
package does not build. TicketBAI's breakdown only covers goods/services
sold nationally or abroad as classified by each line's own AEAT-style
operation code (S1/S2/E1-E6/N1-N2) — foreign lines default to the "service"
category absent a line-level goods/services signal in `BreakdownData`, the
same honest limitation Modelo 349's classification already documents.
Modelo 111 and 115 are manual entry calculators: this plugin fiscalizes
sales and has no visibility into payroll or rent payments the business
makes.

## Architecture and add-on pricing

See [ARCHITECTURE.md](ARCHITECTURE.md) for how the plugin is built
internally (the single-gate/two-engines design) and — the part that
matters for selling this — exactly which toggles map to which add-ons,
and what pricing was actually settled on versus what's still an open
decision.

## Testing

```bash
composer test
```

The suite ships with the package — 177 tests covering the chained hash
formula against the AEAT payload spec, record immutability, sealed
activation, gapless numbering, both remission modes and their guards, the
TicketBAI driver per territory (including Zuzendu and the Batuz
envelopes), Facturae validation, the FACe/FACeB2B lifecycles, the API
sidecar, document totals in bcmath, and the printing pipeline. Run it
after installing to verify your environment.

Run the local homologation battery against a disposable sandbox issuer
before going live:

```bash
php artisan verifactu:homologate                    # AEAT
php artisan verifactu:homologate --submit           # also attempts a real AEAT sandbox round-trip, if a certificate is present
php artisan verifactu:homologate-tbai                       # TicketBAI, Araba by default
php artisan verifactu:homologate-tbai --territory=bizkaia    # or --territory=gipuzkoa
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has
changed recently.

## Contributing

Please see [CONTRIBUTING](.github/CONTRIBUTING.md) for details.

## Security Vulnerabilities

Please review [our security policy](.github/SECURITY.md) on how to report
security vulnerabilities.

## Credits

- [Elias Olivtradet](https://github.com/edeoliv)
- [Komma SoftHouse](https://github.com/komma-softhouse)
- [All Contributors](../../contributors)

## License

This is commercial software. Please see [License File](LICENSE.md) for the
full terms.
