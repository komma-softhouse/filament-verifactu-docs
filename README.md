# Filament Spanish fiscal compliance for Filament v5

<p align="center">
    <img
        src="assets/01.png"
        alt="Filament Spanish fiscal compliance for Filament v5"
        width="100%"
    >
</p>

Official documentation for
[Filament Verifactu](https://github.com/komma-softhouse/filament-verifactu),
a Filament v5 fiscal compliance engine for VERI*FACTU, TicketBAI,
FACe and FACeB2B, covering all four Spanish fiscal regimes — AEAT, the
three TicketBAI territories, and Navarra (foral tax models today,
NaTicket-ready for when its specification is published).

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
lifecycle, FACe/FACeB2B electronic invoicing, and fiscal reporting drafts
under both the state numbers (303, 347, 349, 111, 115) and the Navarra
foral ones (F-69, F-50, 349, 715, 759) — for Spanish businesses that need
their invoicing to comply with Real Decreto 1007/2023, Orden
HAC/1177/2024, (for the three Basque territories) their own TicketBAI
regulation, or (for Navarra) the Convenio Económico's own treasury, whose
antifraud system NaTicket joins as the fourth engine the day its
specification is published.

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
- **Models Navarra as a first-class regime**: neither VERI\*FACTU nor
  TicketBAI apply in the Comunidad Foral, and its own antifraud system
  (NaTicket) has no published specification yet — so a Navarrese issuer
  runs the entire document lifecycle with gapless numbering, prints,
  reports under the foral model numbers, and simply has no antifraud
  chain to activate until the NaTicket driver exists. The activation
  stays locked for this regime by design: nothing is faked, nothing is
  chained against an unpublished spec.
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
- **Prints**: A4 sheets and 80/58 mm roll tickets that are the exact
  layout designed in the template designer — every template field (logo,
  colours, texts, labels, watermark text or image with its geometry, legal
  legend, QR) reaches the paper, on screen and on PDF alike — through the
  PDF engine you choose (`->mpdf()`, a bundled pure-PHP engine with nothing
  to install, or `->gotenbergPdf()` for hosts with Docker; nothing is
  assumed), with automatic draft/copy watermarking, an in-panel preview of
  the printed document, and 14 kinds of ESC/POS thermal ticket — including a paired **Print Agent** add-on
  (Windows service installer generated from the panel) for hosts that can't
  talk to a receipt printer directly from the browser. Each ticket
  variant carries its own template message — the gift receipt's "no
  purchase value" line (printed bold, so a price-less print never passes
  for a sale), the return receipt's note and the gift voucher's terms —
  with sensible defaults when left blank. The tributary QR can
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
  Facturae stays inspectable pretty-printed in the panel — the delivered
  signed XML from the history, or an unsigned build right from the invoice. It also
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
  each of the 41 abilities switchable globally or per user.
- **Keeps the received side of the ledger too** (`->expenses()`): purchase
  invoices, purchase tickets and supplier credit notes, drafted by hand or
  captured with the same OCR, with the deductible VAT share, the expense
  category and the IRPF withheld. Booking freezes a purchase exactly as
  completing freezes a sale — and from that moment the tax drafts read it.
- **Submits both LROE chapters in Bizkaia**: TicketBAI sends what the
  business issues, and the expenses book (Modelo 140 for a self-employed
  issuer, 240 for a company) sends what it receives — which is what makes
  the Batuz obligation complete, and why a Bizkaia issuer files no 347.
- **Drafts Modelos 303, 347, 349 (real E/S classification from each
  record's own data), 111 and 115** for the accountant, each exportable
  as CSV — with input VAT, the purchase side of the 347, intra-EU
  acquisitions and the withholding payees all derived from those booked
  purchases instead of typed in.
- **Estimates the corporate income tax and its instalment** from invoiced
  income less booked purchases, on the 2026 rate scale, with payroll,
  depreciation and fiscal adjustments declared — a provisioning tool, not
  a filing. The Basque treasuries keep the same numbering, so a TicketBAI
  issuer's 303/347/349 drafts read its own chained foral records — with
  the quarterly 110 as their general payroll model (their 111 is the
  monthly large-company one) and no 347 in Bizkaia, where the LROE
  (Batuz) replaces it — the LROE covers income and expenses (models
  140/240): this plugin submits its issued-sales chapter, the expenses
  chapters live in your accounting. Navarra issuers get the same drafts
  under the
  foral numbers — F-69, F-50, the foral 349, 715 and 759 — read from the
  completed fiscal documents.

Every surface beyond the core fiscal resources ships **disabled by
default** — a host that registers only the plugin object gets the fiscal
chain and nothing else; every other module is a fluent opt-in.

## Screenshots

<p align="center">
    <img src="assets/02-documents.jpeg" alt="Invoices & documents — list with actions" width="100%">
</p>
<p align="center">
    <img src="assets/03-document-preview.jpeg" alt="Document preview — A4 with tributary QR and legend" width="49%">
    <img src="assets/04-ticket-preview.jpeg" alt="Ticket preview — 80 mm roll with QR" width="49%">
</p>
<p align="center">
    <img src="assets/05-template-designer.jpeg" alt="Template designer — live A4 preview" width="49%">
    <img src="assets/06-template-designer-ticket.jpeg" alt="Template designer — live thermal preview" width="49%">
</p>
<p align="center">
    <img src="assets/07-fiscal-records.jpeg" alt="Fiscal records — hash chain, statuses, QR self-check" width="49%">
    <img src="assets/08-qr-modal.jpeg" alt="Tributary QR modal — AEAT / TicketBAI" width="49%">
</p>
<p align="center">
    <img src="assets/09-activation.jpeg" alt="Sealed fiscal activation — regime and mode" width="49%">
    <img src="assets/10-repair-order.jpeg" alt="Repair order — SAT lifecycle, receipt and report" width="49%">
</p>
<p align="center">
    <img src="assets/11-face-history.jpeg" alt="FACe history — registry numbers and lifecycle" width="49%">
    <img src="assets/12-faceb2b.jpeg" alt="FACeB2B — the receiver-side lifecycle FACe alone lacks" width="49%">
</p>
<p align="center">
    <img src="assets/13-expenses.jpeg" alt="Expenses & received invoices — issuer, type, category and scan" width="100%">
</p>
<p align="center">
    <img src="assets/14-expense-form.jpeg" alt="Registering a supplier's paper — deductible share, category, withholding" width="49%">
    <img src="assets/15-expense-attachment.jpeg" alt="The supporting document, readable from the panel" width="49%">
</p>
<p align="center">
    <img src="assets/16-expense-ocr.jpeg" alt="Capturing an expense from a photo, reconciled against the printed total" width="49%">
    <img src="assets/17-lroe-expenses.jpeg" alt="Bizkaia's LROE expenses book — the other half of Batuz" width="49%">
</p>
<p align="center">
    <img src="assets/18-model-303.jpeg" alt="Modelo 303 — output VAT chained, input VAT derived from booked purchases" width="100%">
</p>
<p align="center">
    <img src="assets/19-model-347.jpeg" alt="Modelo 347 — both sides of the form: sales (B) and purchases (A)" width="49%">
    <img src="assets/20-withholding-prefill.jpeg" alt="Withholding drafts prefilled from booked purchases" width="49%">
</p>
<p align="center">
    <img src="assets/21-reports-navigation.jpeg" alt="Reports — state and foral sets side by side, each hidden without issuers of its regime" width="49%">
    <img src="assets/22-model-f69.jpeg" alt="F-69 — Navarra's quarterly VAT draft" width="49%">
</p>
<p align="center">
    <img src="assets/23-corporate-tax.jpeg" alt="Corporate income tax estimate and instalment provision" width="49%">
    <img src="assets/24-activation-navarra.jpeg" alt="Navarra listed but locked — NaTicket has no published specification" width="49%">
</p>

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

**3. Install:**

```bash
php artisan filament-verifactu:install
```

It publishes the config and offers to run the migrations (`php artisan
migrate` by hand works exactly the same).

Register the plugin on your panel provider:

```php
use Komma\Verifactu\VerifactuPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        ->plugin(
            VerifactuPlugin::make()
                // Who you are, as the implementer. Travels in every record,
                // AEAT and foral alike.
                ->computerSystem(
                    vendorName: 'Your Company SL',
                    vendorNif: 'B00000000',
                    name: 'Your ERP',
                    version: '1.0',
                )
                // TicketBAI identity and the software-guarantor licences
                // per territory: the vendor's, never the customer's.
                ->ticketBai(developerNif: 'B00000000', licenses: ['gipuzkoa' => 'TBAIGIP…'])

                // Surfaces. Every one ships off; enable what you sell.
                ->documents()          // quotes → invoices, series, templates
                ->repairs()            // SAT repair orders
                ->expenses()           // the received side: supplier invoices and expenses
                ->escPos()             // thermal ticket printing
                ->printAgent()         // cloud → till printing through the Windows agent
                ->face()               // FACe (public administrations)
                ->faceb2b()            // FACeB2B (large private subcontractors)
                ->ocr()                // draft a document or an expense from a photo
                ->api()                // the on-prem API page; the sidecar itself is VERIFACTU_API
                ->dashboardWidgets()

                // PDF engine: pick one, or PDF actions stay hidden.
                ->mpdf()               // bundled, pure PHP
                // ->gotenbergPdf('http://gotenberg:8')
                // ->pdfDriver(YourConverter::class)

                // Where template logos and watermark images live.
                ->brandingDisk('public'),

                // ->einvoicing() lands once the Orden Ministerial is published.
        );
}
```

Everything in that snippet can also be set through `.env` or the published
config — the fluent calls simply win when present, so a host never has to
publish the config to get going.

Publish and fill in the config:

```bash
php artisan vendor:publish --tag="filament-verifactu-config"
```

At minimum, set the `ComputerSystem` (implementer identity) variables before
the first submission — see [Configuration](#configuration).

## Configuration

| Toggle | What it enables |
| --- | --- |
| <code>->documents()</code> | Document, series-format and template resources, the document lifecycle |
| <code>->repairs()</code> | SAT repair order resource |
| <code>->expenses()</code> | The received side: supplier invoices, tickets and credit notes, OCR capture, attachment preview, and Bizkaia's LROE expenses book. Turns the tax drafts' input side on |
| <code>->mpdf()</code> | PDFs through the bundled pure-PHP engine — nothing to install, no Docker |
| <code>->gotenbergPdf($url)</code> | PDFs through a Gotenberg instance (Docker, best fidelity, same layout) |
| <code>->pdfDriver(Class::class)</code> | Your own <code>Printing\Contracts\PdfConverter</code> implementation |
| <code>->brandingDisk($disk, $visibility = null)</code> | The filesystem disk template logos and watermark images are stored on and read from (<code>public</code> unless told otherwise — any disk of your <code>config/filesystems.php</code>: local, s3, minio…) |
| <code>->computerSystem(...)</code> | The implementer identity for AEAT records (vendor name/NIF, product name/id/version, installation number) |
| <code>->ticketBai(...)</code> | The implementer identity for TicketBAI (developer NIF, product) and the **software-guarantor licences per territory** — the vendor's, never the customer's |
| <code>->escPos()</code> | Thermal ticket printing |
| <code>->printAgent()</code> | Print agent pairing page and installer generation |
| <code>->einvoicing()</code> | Pending BOE publication of the Orden Ministerial (Ley Crea y Crece / RD 238/2026) |
| <code>->face()</code> | FACe: the "Send to FACe" action, history, settings, generate and directory pages |
| <code>->faceb2b()</code> | FACeB2B: the "Send to FACeB2B" action and cancellation requests |
| <code>->ocr()</code> | The "Create from photo" OCR-assisted draft action and its settings page |
| <code>->api()</code> | The API panel page (endpoints, per-issuer bearer keys, first call); the sidecar itself is a config toggle — see below |
| <code>->dashboardWidgets()</code> | Dashboard widgets: fiscal stats overview + 30-day records chart |

The core fiscal resources (Issuers, Fiscal records, Submissions, System
events, Audit trail), the Certificates page and the report pages are
always registered — and each page hides from navigation when the install
has no issuer it serves, so nobody ever reads another treasury's model
numbers. The shared-numbering pages (303, 347, 349, 115) serve AEAT and
Basque issuers alike, each filing with its own Hacienda; the state 111
is AEAT-only and the Basque quarterly 110 is its own page; the 347 hides
for Bizkaia-only installs (the LROE replaces it); and the Navarra foral
set (F-69, F-50, foral 349, 715, 759) shows only with Navarrese
issuers. PDF generation is an opt-in like every other surface:
without `->mpdf()`, `->gotenbergPdf()` or `VERIFACTU_PDF_DRIVER`, the
PDF, email and ZIP actions stay hidden.

With `->dashboardWidgets()` (or `VERIFACTU_WIDGETS=true`) the panel
dashboard gains widgets — modular like the plugin itself, so every ACTIVE
module contributes its own: the fiscal pulse always (records today,
accepted this month with a 7-day sparkline, rejections and
awaiting-submission linking straight to the records list, plus a 30-day
line chart by outcome); with `->documents()`, the billing pulse (euros
billed today bcmath-summed with credit notes subtracting themselves,
documents completed, credit notes this month, open drafts); with
`->face()`, the public-sector view (registered this month, marked as paid
by the administration, rejections to resend); and with `->repairs()`, the
workshop counter (in progress, ready for pickup, delivered this month).
Everything aggregates across every issuer and auto-refreshes.

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
# equivalent of the ComputerSystem block above, plus the software-guarantor
# licences. The licences are issued BY EACH FORAL TREASURY TO THE SOFTWARE
# VENDOR when it registers as "software garante" — they are yours, not the
# customer's: an issuer is activated without typing any of them. (An
# issuer-level Issuer::tbai_license, when set, overrides the territory one
# — for a host that registered as garante itself.)
VERIFACTU_TBAI_DEVELOPER_NIF=
VERIFACTU_TBAI_SYSTEM_NAME="Komma Verifactu"
VERIFACTU_TBAI_SYSTEM_VERSION=1.0
VERIFACTU_TBAI_LICENSE_ARABA=
VERIFACTU_TBAI_LICENSE_BIZKAIA=
VERIFACTU_TBAI_LICENSE_GIPUZKOA=

# Template branding storage (logos, watermark images). One disk for uploads
# and for rendering; any disk of your config/filesystems.php. Filament v5
# uploads to the app's default disk (private `local` on a fresh Laravel)
# unless told otherwise — this setting is what keeps both sides aligned.
VERIFACTU_BRANDING_DISK=public
VERIFACTU_BRANDING_VISIBILITY=        # empty = the disk's own default; public | private

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

# PDF rendering — an explicit choice. mpdf: bundled pure-PHP engine, nothing
# to install. gotenberg: Chromium over HTTP (Docker), best fidelity. Or the
# FQCN of your own PdfConverter. Unset = PDF disabled (actions hidden).
VERIFACTU_PDF_DRIVER=mpdf
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
# provider works, the host supplies its own key in their own config/ai.php
# or stores it encrypted on that page)
VERIFACTU_OCR=true

# Documents, repairs, on-prem API sidecar
VERIFACTU_DOCUMENTS=true
VERIFACTU_REPAIRS=true
VERIFACTU_API=true
```

## Authorization

Every panel action — completing, voiding, sending to FACe, uploading
certificates, generating API keys, all 41 of them — is gated through
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
       // ... all 41 keys ship in the published config, grouped by domain
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
| <code>complete-document</code> | Completing a draft (numbering + chaining the fiscal record) |
| <code>convert-document</code> | Forward conversions (quote → order → … → invoice) |
| <code>create-from-photo</code> | OCR-assisted draft from a photo |
| <code>merge-into-invoice</code> | Merging delivery notes into one invoice (bulk) |
| <code>void-document</code> | Credit notes / cancellations |
| <code>delete-draft</code> | Deleting a draft or pre-billed document (single and bulk) — completed and voided documents can never be deleted, the model refuses regardless of the UI |
| <code>reprint-ticket</code> | Thermal reprint (the COPIA-marked duplicate) |
| <code>send-by-email</code> | Emailing the document PDF |
| <code>download-document</code> | PDF and HTML downloads of a document |
| **FACe / Facturae** | |
| <code>send-to-face</code> | Send to FACe action |
| <code>send-to-faceb2b</code> | Send to FACeB2B action |
| <code>download-facturae</code> | Signed/unsigned Facturae download from a document |
| <code>generate-facturae</code> | The "New Facturae" page action |
| <code>generate-facturae-ocr</code> | The "From PDF / photo (OCR)" standalone flow |
| <code>check-face-registration</code> | Polling FACe and persisting the tramitación status |
| <code>resend-to-face</code> | Resending a stored submission |
| <code>cancel-faceb2b</code> | FACeB2B cancellation requests |
| <code>download-xsig</code> | Downloading the stored signed <code>.xsig</code> |
| <code>view-face-xml</code> | The pretty-printed XML inspector |
| **Advisory-firm bulks** | |
| <code>download-pdf-zip</code> | ZIP of selected documents' PDFs |
| <code>download-facturae-zip</code> | ZIP of selected invoices as signed Facturae |
| **Issuers & fiscal lifecycle** | |
| <code>create-issuer</code> | Creating an issuer from auxiliary screens |
| <code>activate-fiscal</code> | The sealed, irreversible fiscal activation |
| <code>switch-to-verifactu</code> | The one-way non-Verifactu → VERI\*FACTU switch |
| <code>edit-tbai-settings</code> | TicketBAI territory settings (Bizkaia self-employed flag, advanced licence override) |
| <code>remit-on-demand</code> | The non-Verifactu voluntary/requirement remission |
| <code>upload-certificate</code> | Uploading an issuer certificate |
| <code>remove-certificate</code> | Removing an issuer certificate |
| <code>manage-social-collaborator</code> | The global gestoría certificate (upload and remove) |
| <code>generate-api-key</code> | Generating a bearer key |
| <code>revoke-api-key</code> | Revoking a bearer key |
| **Fiscal records** | |
| <code>resend-record</code> | Requeueing a rejected record (single and bulk) |
| <code>download-tbai-xsig</code> | Downloading the signed TicketBAI XML |
| <code>query-aeat</code> | The AEAT read-back (quick status and detailed modal) |
| <code>view-qr</code> | Viewing and downloading the tributary QR |
| **Templates & SAT** | |
| <code>quote-repair</code> | Issuing the repair quote (Q series) from the order's lines |
| <code>decline-repair-quote</code> | Closing a quoted order as declined, optionally charging the diagnostic fee |
| <code>transition-repair</code> | Moving a repair order through its lifecycle |
| <code>deliver-repair</code> | Delivering (which issues the fiscal document) |
| <code>print-repair-paper</code> | Deposit receipt and repair report printing |
| <code>send-repair-by-email</code> | Emailing either SAT paper |

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
- The document flow only moves forward: quote → order · delivery note ·
  proforma · invoice; order → delivery note · proforma · invoice; delivery
  note → proforma · invoice; proforma → invoice. A **ticket is never a
  conversion target** — it is born at the point of sale. Conversions and
  merges carry everything: general discount, withholding, price mode
  (tax-inclusive or not), the shipping block and, per line, discount, IRPF,
  surcharge, kitchen zone, traceability and the host product link; a
  credit note rectifies the invoice **at the invoice's own terms**, so the
  credited amount is the amount that was charged.
- Repairs (SAT) are a node of that same flow: an order can issue a repair
  quote (Q), and delivery opens the ticket/invoice **descending from that
  quote** (parent + source documents), with the device IMEI/serial as line
  traceability. Every document born from a repair shows its order.
- Documents carry a **shipping block** (carrier, tracking number, packages,
  weight) and **per-line traceability** (serial numbers, batch, expiry) —
  printed as written, never part of the fiscal record.
- The chain check (`verifactu:validate-chain`, `GET /chain/verify`)
  verifies both the **link** (each `previous_hash`) and the **content**
  (every AEAT record's hash recomputed from its stored fields with the AEAT
  formula); a record altered straight in the database with its hash left in
  place is reported as `reason: content`.
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
- Basque foral models keep the state numbering, filed with each foral
  Hacienda: 303 (e.g. Araba's Orden Foral of 2021 approving its own 303),
  349, and 115 (115-A in Araba). Their general payroll-withholding model
  is the quarterly 110 (Orden Foral 631/2014 in Gipuzkoa, 104/2014 in
  Araba); the foral 111 is the monthly large-company variant, out of
  scope like every monthly filer. In Bizkaia the LROE (Batuz) replaces
  the Modelo 347 — models 140/240 cover income AND expenses, and this
  plugin submits both chapters — so the 347 draft refuses and the page
  hides.
- Corporate income tax rates for periods started in 2026 (Ley 7/2024,
  art. 29.1 LIS and transitional provision 44ª): a two-band scale of 19%
  up to €50,000 and 21% beyond for net turnover under €1M, a flat 23%
  between €1M and €10M, the 25% general rate, and 15% for newly created
  entities. Instalments follow art. 40.2 LIS (18% of the quota); the
  optional art. 40.3 modality is out of scope.
- Navarra models (Convenio Económico, art. 34: VAT follows the same
  substantive rules as the state law): F-69 quarterly VAT (Orden Foral
  280/2006; monthly filers above €6,010,121.04 use F-66, out of scope
  like the state monthly 303) · F-50 annual third-party operations
  (Decreto Foral 69/2010, Orden Foral 177/2010 — same €3,005.06
  threshold) · its own 349 filing · 715 payroll/professional
  withholding · 759 urban-lease withholding (19%, quarterly; monthly
  filers use 760, out of scope).
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
- The **software-guarantor licence** each foral treasury issues to the
  registered software vendor → configured per territory at installation
  level (`VERIFACTU_TBAI_LICENSE_*` or `->ticketBai(licenses: [...])`);
  the customer activating an issuer never types it. `Issuer::tbai_license`
  remains as an advanced per-issuer override.

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
panel itself ships in Spanish, Galician, Catalan, Basque and Portuguese
(`resources/lang/{es,gl,ca,eu,pt}.json`) alongside English, and resolves
against your own application's `resources/lang/{locale}.json` too if you
add translations there. Language and regime are independent: a Catalan
business is common territory (AEAT / VERI\*FACTU) in Catalan; a Basque one
is TicketBAI in Basque or Spanish.

Every form field with rules validates the moment you leave it, with the
field's own messages — a wrong NIF is flagged before the submit.

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
// The software-guarantor licence is the VENDOR's, configured once per territory
// (VERIFACTU_TBAI_LICENSE_ARABA or ->ticketBai(licenses: [...])) — nothing per issuer.

// Navarra instead: set the regime at creation and do NOT activate —
// activateFiscal() refuses the regime until NaTicket publishes its
// specification. The issuer issues numbered documents without a chain,
// and its report drafts already take the foral numbers:
// $issuer = Issuer::create(['name' => 'Bodega Iruñea SL', 'nif' => 'B31000001', 'regime' => FiscalRegime::Navarra]);
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
GET /api/v1/verifactu/records/{id}/qr?size=300&module=5
Authorization: Bearer vf_...
```

Everything a POS needs to put the tributary QR on its own ticket, in
whichever form its printer consumes:

```json
{
  "invoice_number": "T2026-0001", "status": "accepted", "hash": "…",
  "url": "https://…",
  "regime": "aeat",
  "legend": "Factura verificable en la sede electrónica de la AEAT / VERI*FACTU",
  "ticketbai_identifier": null,
  "csv": "…",
  "svg": "<svg …>",
  "png_base64": "…",
  "escpos_base64": "…"
}
```

`legend` is present in VERI\*FACTU mode (AEAT non-Verifactu prints the QR
alone, as the Orden requires); for a foral issuer `legend` is null and
`ticketbai_identifier` carries the 39-character identifier that must be
printed on the invoice. `escpos_base64` is the native ESC/POS QR block,
sent as-is to the printer port; `png_base64` fits any other driver.

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

## Printed papers and templates

Document templates (A4 and thermal, per issuer or global — the most
specific wins: issuer + type, then issuer default, then global) and SAT
templates (deposit receipt and repair report) are designed live in the
panel, and **what the designer shows is exactly what prints**: every field
— logo, colours, header texts and HTML blocks, footer notes, column
labels, legal legend and QR toggles, the watermark (text or image, with
opacity, size and rotation), the SAT title, custody clause, warranty text
and signature/tracking toggles — reaches the on-screen preview, the PDF
and the ESC/POS ticket. The regime is respected on every output: AEAT
VERI\*FACTU prints the QR with its legend and CSV, AEAT non-Verifactu the
QR alone, TicketBAI the 39-character identifier and the foral QR.

Logos and watermark images are stored on the **branding disk**
(`->brandingDisk()` / `VERIFACTU_BRANDING_DISK`, `public` by default —
any disk of yours) and travel inside the rendered HTML as data URIs, so
neither PDF engine ever needs to reach your disks by URL.

PDFs are the document as previewed: an A4 sheet for invoices, quotes and
the rest, and a **roll-width page (80 or 58 mm, as tall as the content)**
for tickets. Two engines, one layout: `->mpdf()` (bundled, pure PHP) or
`->gotenbergPdf()` (Chromium, Docker). The only engine-specific limit:
mPDF does not rotate *image* watermarks (text watermarks rotate on both);
Gotenberg honours the configured rotation.

## OCR: drafting from a photo or PDF

With `->ocr()`, Invoices & documents gains **Create from photo**: upload a
photo or a PDF and the AI provider you picked in *OCR settings* transcribes
it into a pre-filled draft. Any [Laravel AI](https://laravel.com/docs/13.x/ai-sdk)
provider works (OpenAI, Anthropic, Gemini, …) — you choose the provider and
model there, and the key comes from your own `config/ai.php`/`.env` or is
stored encrypted on that page.

Fiscal data is never guessed. The reading is only turned into a draft when
the lines the model read add up to the total printed on the paper — as net
prices plus tax, or as tax-inclusive prices (receipts often print them
that way), within a cent per line of rounding. Otherwise nothing is
created and the error says what did not reconcile. PDFs are sent as
documents (not as images), transient provider conditions (overloaded, rate
limited) are retried, and a failure is always reported in the panel.
Gemini 2.0 Flash tends to answer "overloaded" at peak hours; if it keeps
failing, switch the model or the provider in *OCR settings*.

## Print Agent (thermal printing from the cloud)

Hosts on a local server usually print ESC/POS straight to the receipt
printer (IP:port, a Windows share or `/dev/usb/lp0`) — nothing to install.
Hosts that cannot reach the till from the browser (cloud installs, tablets)
enable `->printAgent()` and pair a small Windows service that sits next to
the printer:

1. In *Print agent*, pick the issuer and the printer address and download
   the installer. It is a `.bat` generated for that terminal, with a
   pairing token minted for the issuer.
2. Run it on the till PC. It installs `pos-agent.exe` as a Windows service
   (through `nssm.exe`), authenticates back to the panel and starts a
   heartbeat — the page shows the agent online.
3. From then on every ticket, kitchen order or Z report the panel prints is
   broadcast to the agent (`PrintJobDispatched` on a private channel), which
   pushes the raw ESC/POS bytes to the paired printer. *Test print* sends a
   real ticket with logo and cut.

The agent listens over websockets, so the host needs a **broadcasting
connection**: Laravel Reverb or Pusher configured under
`broadcasting.connections.pusher` (the installer preview on the page shows
the `PUSHER_HOST/PORT/SCHEME/KEY` it will write on the terminal — if they
are empty, broadcasting is not configured yet). The page's own *How does
it work?* and *Windows installation guide* walk the installer through it.

Both binaries ship inside the package (`downloads/windows/pos-agent.exe`
and `nssm.exe`) and are served from `/download/windows/{file}`; a host may
override either by placing its own copy under `public/download/windows/`.
The agent pairs against `/api/v1/printer-agent` (register · auth ·
heartbeat · test-print), routes registered unconditionally because they
are reached without a panel. The uninstaller is generated the same way.

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

The repair lifecycle now has its quote branch: from received or
diagnosing, **Issue quote** turns the order's parts/labour lines into a
completed Q-series document (numbered, never fiscalized) and parks the
order in "quoted"; accepting means moving it on to parts or repairing;
**Decline quote** closes it as rejected, returning the device unrepaired,
with an optional diagnostic fee fiscalized on the spot as a simplified
invoice through the single gate.

Every transition on a repair order (`RepairService::transition()`,
including the initial "received", "quoted"/"rejected", and the final
"delivered") fires
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

## Navarra: the fourth regime

The Comunidad Foral has its own treasury under the Convenio Económico:
neither VERI\*FACTU (common territory) nor TicketBAI (Araba, Bizkaia,
Gipuzkoa) applies to a business whose tax home is Navarra. Its own
antifraud system, **NaTicket**, is being developed by the Hacienda Foral
de Navarra under the 2025-2027 fraud plan — with **no technical
specification, no schemas, no endpoints and no mandatory calendar
published** at the time of this build.

This plugin models that reality instead of papering over it:

- `FiscalRegime::Navarra` is picked at issuer creation like any other
  regime. The issuer runs the **entire document lifecycle** — quotes,
  orders, delivery notes, proformas, invoices, tickets, credit notes, SAT
  repairs, gapless series numbering, templates, A4 and thermal printing,
  FACe/FACeB2B — exactly like every other issuer.
- **Fiscal activation stays locked** for this regime: `activateFiscal()`
  refuses it, the panel shows the option disabled with the reason, and
  `RecordService` refuses to chain even a force-sealed Navarra issuer.
  No record is ever invented against an unpublished spec.
- **Report drafts take the foral numbers**, computed from the completed
  fiscal documents (their numbers, customers and lines freeze at
  completion — the same immutable base the AEAT drafts get from the
  chained records; credit notes subtract through their negative lines,
  voided documents drop out):

  | State model | Navarra model | Draft source |
  |---|---|---|
  | 303 (quarterly VAT) | **F-69** (monthly F-66 out of scope, like the state monthly 303) | completed documents |
  | 347 (third parties) | **F-50** — same €3,005.06 threshold | completed documents |
  | 349 (intra-EU) | **349**, filed with the Hacienda Foral — lines flagged `E?` to confirm | completed documents |
  | 111 (payroll/fees withholding) | **715** | manual entries |
  | 115 (rent withholding) | **759** — urban leases, same 19% default | manual entries |

  Each foral model is its **own page** — nobody ever reads another
  treasury's numbers: a common-territory install sees only 303/347/349/
  111/115, a Navarrese install sees only F-69/F-50/349/715/759, and a
  mixed install sees both sets side by side, each with its issuer select
  filtered to its own regime.
- **When the Orden Foral lands**, NaTicket joins as the fourth driver
  behind the same `RecordService` gate the other engines share — the
  TicketBAI-only code paths are guarded by
  `FiscalRegime::usesTicketBai()`, never by "not AEAT", so nothing
  misroutes in the meantime.
- A Navarrese business also selling from a common-territory establishment
  models that as **two issuers** (one AEAT, one Navarra), same as any
  multi-administration business.

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

**Document module over the API** for external POS/ERP terminals that want
this plugin to number, fiscalize and render their documents (create a
draft with lines, complete it, fetch the PDF) — today the sidecar assumes
the terminal owns its numbering and only needs the fiscal record and the
QR.

**Received-invoices ledger** (purchase invoices, OCR-fed), so Modelo 303's
input VAT (soportado) can be derived instead of typed.

**SII (Suministro Inmediato de Información)**: near-real-time VAT ledger
submission for REDEME and large-company issuers — the AEAT flavour and the
foral ones that exist (Navarra runs its own SII; Bizkaia's ledger duty is
already Batuz/LROE, which this plugin submits today). The issued-invoices
ledger is derivable from the chained records now; the received side waits
on the received-invoices ledger above, so `->sii()` ships after it.

**NaTicket engine**: everything around it already ships — see
[Navarra: the fourth regime](#navarra-the-fourth-regime). The driver
itself joins behind the same `RecordService` gate the moment the Hacienda
Foral de Navarra publishes the technical specification; nothing is built
against an unpublished spec.

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

## Going to production

What the plugin cannot do for you and the installer must set up once:

1. **Choose the PDF engine** — `->mpdf()` (no infrastructure) or
   `->gotenbergPdf('http://gotenberg:8')` with a Gotenberg container
   reachable from PHP. Without one there are no PDFs, by design.
2. **Queue worker** — AEAT submissions run as queued jobs
   (`SubmitRecordsJob`, queue `VERIFACTU_QUEUE`, `verifactu` by default).
   A worker must be running: `php artisan queue:work --queue=verifactu`
   (or Horizon). TicketBAI does not use the queue.
3. **Scheduler** — for VERI\*FACTU issuers the continuous remission is
   `php artisan verifactu:send-pending`; schedule it every few minutes in
   the host's scheduler (`Schedule::command('verifactu:send-pending')->everyFiveMinutes()`)
   and make sure the cron that runs `schedule:run` exists. Non-Verifactu
   issuers never need this: they remit on demand from the panel.
4. **Disks** — certificates go to `VERIFACTU_CERTIFICATES_DISK` (a private
   disk; never `public`), template branding to `VERIFACTU_BRANDING_DISK`.
   Any disk of your `config/filesystems.php` works (local, S3…); the
   PHP process must be able to read and write both.
5. **Implementer identity** — `->computerSystem(...)` / `->ticketBai(...)`
   (or their env variables) with **your** company's data: it travels in
   every record and is what the responsible declaration names.
6. **Certificates** — one `.p12` per issuer (or the global
   social-collaborator one for advisory firms), uploaded from the panel;
   the passphrase is stored encrypted with the app key, so **back up
   `APP_KEY`**: without it neither passphrases nor stored API keys can be
   read.
7. **Backups** — the tables `verifactu_records`, `verifactu_submissions`
   and `verifactu_events` are append-only and legally must be kept (four
   years of prescription). Back them up with the rest of the database;
   never restore them partially or older than the rest — a restored chain
   that misses its tail is a broken chain.
8. **HTTPS** and a real `APP_URL` — the customer repair-tracking page, the
   print-agent endpoints and the API sidecar are public routes of the host.
   With `->printAgent()`, also a broadcasting connection (Reverb or Pusher)
   reachable from the tills: the agent receives its jobs over websockets.
9. **Run the homologation battery** before the first real submission (see
   Testing): it proves certificate, identity, chain, signature and QR
   against the sandbox with your own data.

## Testing

```bash
composer test
```

The suite ships with the package — 230 tests covering the chained hash
formula against the AEAT payload spec, record immutability, sealed
activation, gapless numbering, both remission modes and their guards, the
TicketBAI driver per territory (including Zuzendu and the Batuz
envelopes), Facturae validation, the FACe/FACeB2B lifecycles, the API
sidecar, document totals in bcmath, the printing pipeline, template
fidelity (every template field reaching the paper), field-complete
conversions and credit notes, chain content tampering, OCR reconciliation
and the repair lineage. Run it after installing to verify your environment.

Run the local homologation battery against a disposable sandbox issuer
before going live:

```bash
php artisan verifactu:query-records B12345678 --last=10   # ask the AEAT what it holds, hash-checked
php artisan verifactu:homologate                    # AEAT
php artisan verifactu:homologate --submit           # also attempts a real AEAT sandbox round-trip, if a certificate is present
php artisan verifactu:homologate-tbai                       # TicketBAI, Araba by default
php artisan verifactu:homologate-tbai --territory=bizkaia    # or --territory=gipuzkoa
```

## Gotchas worth knowing before you hit them

Hard-won, all of them from building this against real panels and real
printers:

- **Filament v5 uploads to the app's default disk** (`FILESYSTEM_DISK`,
  the private `local` on a fresh Laravel). Anything that later reads the
  file must use that same disk: branding through `BrandingDisk`,
  certificates through `certificates.disk`, expense attachments through
  the panel's disk.
- **Livewire temporary uploads**: `TemporaryUploadedFile` hands Symfony an
  empty tmpfile as pathname, so `getContent()` returns zero bytes. Read
  them with `$file->get()` — that is what `OcrAttachment` does.
- **Blade directives glued to a word** (`kg@endif`) are never compiled
  (`\B@`). Leave a space.
- **Filament rule closures** are evaluated with injection, so a
  Laravel-style `function (string $attribute, $value, Closure $fail)` must
  be wrapped: `->rule(fn (): Closure => function (...) {...})`.
- **Livewire cannot hold plain objects** as public properties — the report
  DTOs implement `Wireable` for exactly that reason.
- **A table `SelectFilter` wants a plain `value => label` array**, unlike a
  form `Select`, which takes the enum class itself.
- **Infolist action groups live in `Filament\Schemas\Components\Actions`**
  in v5; the infolists namespace no longer has them.
- **A toggleable column remembers the user's choice**, so a column that
  ships hidden stays hidden for anyone who already opened the page. If it
  must always be visible (the issuer on a multi-company table), do not make
  it toggleable.
- **mPDF**: no `position: fixed` (the footer goes through `SetHTMLFooter`,
  and any `@page` rule wipes it — the driver strips it first); no `div`
  with border or height inside table cells; raster watermarks are prepared
  with GD (cover, scale, rotate); SVG watermarks cannot be rotated.
- **Gotenberg**: send `printBackground`, and `singlePage` + `paperWidth`
  for the roll-width ticket page.
- **PHP's 30 s request budget** kills long OCR calls: the services raise
  it, pass an explicit timeout to Laravel AI and retry on provider
  overload.
- **`ActionGroup` visibility on view pages** is evaluated before the record
  is bound: take `?Model $record` and fall back to
  `$livewire->getRecord()`.
- **Customer tracking and print-agent routes register unconditionally**:
  they are reached without a panel, so fluent toggles cannot gate them.
- **Demo seeding is incremental** — `db:seed --class=VerifactuDemoSeeder`
  adds only what is missing. Never `migrate:fresh` a database that has
  been used: fiscal chains are append-only.
- **The plugin and the AEAT engine share the PSR-4 root
  `Komma\Verifactu\`** across two `src/` directories. Never add
  `Models\Model`, `Models\ComputerSystem`, `Services\AeatClient` or
  `Exceptions\*` to the plugin — they would shadow the engine silently.

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
