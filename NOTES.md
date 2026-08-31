# filament-verifactu

VERI*FACTU / RRSIF compliant fiscal engine for Filament 5. Chained hash
records, AEAT submission (VERI*FACTU and on-demand), XAdES signing, the full
document lifecycle (quotes through invoices, credit notes, repairs), thermal
and A4 printing, FACe electronic invoicing, OCR-assisted drafting, an on-prem
API sidecar for third-party ERPs, and fiscal reporting drafts (Modelos 303,
347, 349, 111, 115).

## Installation

```php
use Komma\Verifactu\VerifactuPlugin;

$panel->plugin(
    VerifactuPlugin::make()
        ->computerSystem(vendorName: 'Komma SoftHouse', vendorNif: 'B75447854', name: 'Komma Fiscal', version: '1.0')
        ->ticketBai(developerNif: 'B75447854', licenses: ['gipuzkoa' => 'TBAIGIP…'])
        ->documents()
        ->repairs()
        ->mpdf()                     // or ->gotenbergPdf('http://gotenberg:8'); no engine = no PDF
        ->brandingDisk('public')
        ->escPos()
        ->face()
        ->ocr()
        ->api(),
);
```

Every surface ships disabled; the fluent calls above are additive. The core
fiscal resources (Issuers, Fiscal records, Submissions, System events, Audit
trail) and the five report pages register unconditionally — they are the
plugin's reason to exist. Documents, series formats, templates, and repair
orders register only when their corresponding toggle is enabled.

## Environment variables

```
# ComputerSystem identification (RRSIF art. 8.1/13 — the implementer's
# responsible declaration; required before the first submission)
VERIFACTU_VENDOR_NAME="Komma SoftHouse"
VERIFACTU_VENDOR_NIF=B75447854
VERIFACTU_SYSTEM_NAME="Komma Fiscal"
VERIFACTU_SYSTEM_ID=01
VERIFACTU_SYSTEM_VERSION=1.0
VERIFACTU_INSTALLATION_NUMBER=1

# Submission pipeline
VERIFACTU_QUEUE=verifactu
VERIFACTU_BATCH_SIZE=1000
VERIFACTU_CERTIFICATES_DISK=local
VERIFACTU_CERTIFICATES_DIR=verifactu/certificates

# Representative/collaborator header (requires AEAT form GENERALLEY58 filed
# by the taxpayer being represented)
VERIFACTU_REPRESENTATIVE_NAME=
VERIFACTU_REPRESENTATIVE_NIF=

# TicketBAI implementer identity and the software-guarantor licences — the
# vendor's (issued by each foral treasury to whoever registered the
# software), never the customer's. Issuer::tbai_license overrides per issuer.
VERIFACTU_TBAI_DEVELOPER_NIF=B75447854
VERIFACTU_TBAI_SYSTEM_NAME="Komma Verifactu"
VERIFACTU_TBAI_SYSTEM_VERSION=1.0
VERIFACTU_TBAI_LICENSE_ARABA=
VERIFACTU_TBAI_LICENSE_BIZKAIA=
VERIFACTU_TBAI_LICENSE_GIPUZKOA=

# Template branding storage (logos, watermark images): any disk of the host.
VERIFACTU_BRANDING_DISK=public
VERIFACTU_BRANDING_VISIBILITY=

# PDF rendering — explicit: mpdf (bundled, pure PHP) | gotenberg | FQCN.
# Unset = PDF disabled (actions hidden).
VERIFACTU_PDF_DRIVER=mpdf
GOTENBERG_URL=http://gotenberg:8

# ESC/POS printing, print agent
VERIFACTU_ESCPOS=true
VERIFACTU_PRINT_AGENT=false

# FACe (public-sector electronic invoicing) / FACeB2B — module toggles only;
# seller address and FACe defaults live in FaceSettings (settings page).
VERIFACTU_FACE=true
VERIFACTU_FACEB2B=false

# OCR — module toggle only; provider, model and key are set on OcrSettingsPage
# (any Laravel AI provider; the key comes from the host's config/ai.php/.env
# or is stored encrypted on that page). Gemini 2.0 Flash gets "overloaded"
# at peak hours (retried ×3); OpenAI worked first time in the demo.
VERIFACTU_OCR=true

# Documents, repairs, on-prem API sidecar
VERIFACTU_DOCUMENTS=true
VERIFACTU_REPAIRS=true
VERIFACTU_API=true
```

The API sidecar's activation (`VERIFACTU_API`) is read directly from config
at framework boot, independent of the panel plugin's `->api()` toggle — the
sidecar is meant for LAN-local POS/ERP terminals that never load a Filament
panel, so it cannot depend on a toggle that only resolves once a panel does.

## Usage

```php
use Komma\Verifactu\Facades\Verifactu;
use Komma\Verifactu\Enums\FiscalRegime;

$issuer = Issuer::create(['name' => 'My Company SL', 'nif' => 'B12345678']);
$issuer->activateFiscal(FiscalRegime::Aeat); // sealed, irreversible

// Fiscal document lifecycle (recommended entry point)
$document = FiscalDocument::create(['issuer_id' => $issuer->id, 'type' => DocumentType::Ticket, 'status' => DocumentStatus::Draft]);
$document->lines()->create(['description' => 'Coffee', 'quantity' => 1, 'unit_price' => '1.50', 'tax_rate' => '10.00']);
Verifactu::completeDocument($document); // numbers, freezes, chains the fiscal record

// Direct engine access for hosts that already own their invoicing model
Verifactu::register($issuer, $invoiceData);
Verifactu::cancel($issuer, $cancellationData);
Verifactu::submitPending($issuer);
Verifactu::verifyChain($issuer);

// FACe
Verifactu::sendToFace($document, $seller, $buyer, 'buzon@administracion.example');

// Reports
Verifactu::model303($issuer, new FiscalPeriod(2026, 3));
Verifactu::model347($issuer, 2026);
```

Or via the on-prem API sidecar (bearer key generated from the Issuer's
actions in the panel):

```
POST /api/v1/verifactu/records
Authorization: Bearer vf_...
{"invoice_number": "F2026-0001", "issued_on": "2026-08-27", "invoice_type": "F1", ...}

GET  /api/v1/verifactu/records/{id}/qr     # url, legend or TBAI identifier, CSV, SVG, PNG, ESC/POS block
```

## Architecture notes

- The fiscal engines are Komma's own packages on Packagist
  (`komma-softhouse/verifactu-engine`, `tbai-engine`, `facturae-engine`).
  `RecordService`, `RecordHydrator`, `EngineAeatGateway`,
  `ComputerSystemFactory`, `QrRenderer` and `RecordXmlExporter` are the only
  files that import the AEAT engine; `TicketBaiDriver`, `TicketBaiMapper`
  and `EngineTicketBaiGateway` the only ones that import the foral engine;
  `FacturaeInvoiceBuilder`, `EngineFaceGateway` and `EngineFaceb2bGateway`
  the only ones that import the Facturae engine.
- The plugin and the AEAT engine share the PSR-4 root `Komma\Verifactu\`
  over two `src/` directories. No collision today — never add
  `Models\Model`, `Models\ComputerSystem`, `Services\AeatClient` or
  `Exceptions\*` to the plugin; they would shadow the engine silently.
- `invoice_number` always stores series+number concatenated (e.g.
  `F2026-0001`), matching what AEAT expects as `NumSerieFactura`.
  `hashed_at` is always UTC; the chain hash payload uses `format('c')` on
  that UTC instant, per the exact formula of the AEAT engine.
- Series are resolved per issuer and document type with placeholders
  `{PREFIX}{YEAR}{STORE}{TERMINAL}` (default prefixes Q·PE·A·P·F·T·R);
  changing a format resolves to a different series string going forward —
  the previous series stays closed with its numbering intact.
- Foral issuers (Araba, Bizkaia, Gipuzkoa) go through `TicketBaiDriver`
  behind the same `RecordService` gate: chained by signature value (not
  hash), synchronous registration, per-territory endpoints, Batuz/LROE
  envelope for Bizkaia, Zuzendu for Araba/Gipuzkoa. Only non-Verifactu
  mode exists there. Navarra (NaTicket) is out of scope until the Hacienda
  Foral publishes a specification.
- The document flow is forward-only (`DocumentConverter::EDGES`), a ticket
  is never a conversion target, conversions/merges/credit notes carry every
  commercial term, and repairs are a node of the flow (OR → Q → delivery
  paper, device serial as line traceability). `ChainVerifier` checks link
  and content (hash recomputed via `RecordHydrator`).
- Printing: one HTML per paper (`documents.a4`, `documents.ticket`,
  `repairs.*`), rendered on screen or for PDF (`forPdf`), converted by the
  configured `PdfConverter` (mPDF or Gotenberg). Images travel as data URIs
  from `BrandingDisk`. Every template field reaches the paper —
  `tests/TemplateFidelityTest` enforces it.
- OCR: `OcrAttachment` reads the real bytes of Livewire uploads (PDF as
  document, image as image); `OcrService::reconcile()` refuses any reading
  whose lines do not add up to the printed total. Fiscal data is never
  guessed.
- Multi-tenant hosts: `verifactu_issuers.nif` is UNIQUE globally. Add a
  `tenant_id` column and a compound unique constraint in your own migration
  if you run this behind Stancl or similar.

## Legal references (verified against BOE-A-2023-24840, Real Decreto
1007/2023, consolidated text as of 03/12/2025)

- Art. 8.2.a — integrity/immutability duty → enforced by the `AppendOnly`
  trait and the sealed-issuer guard on `Issuer`.
- Art. 8.2.b — traceability/chaining duty → `RecordService`, `ChainVerifier`.
- Art. 8.3 (Reglamento) / Orden HAC/1177/2024 art. 9 — event log duty →
  `VerifactuEvent`, `EventLogger`.
- Art. 12 — hash and electronic signature duty → `SignatureService`.
- Art. 16.3 — VERI*FACTU systems are exempt from signing (hash only) →
  `SignatureService` is only invoked in non-Verifactu mode.
- Arts. 15–16 — voluntary remission and the VERI*FACTU system definition →
  `SubmissionContext`, `BatchService::submitVoluntarily/submitOnRequirement`.
- The QR + "Factura verificable en la sede electrónica de la AEAT" /
  "VERI*FACTU" legend requirement comes from RD 1619/2012 arts. 6.5/7.5, as
  introduced by RD 1007/2023's disposición final primera, referencing RRSIF
  arts. 15–16 → `QrRenderer`.
- Modelo 347 threshold: €3,005.06/year, unchanged for decades →
  `Model347Report::THRESHOLD`.
- Modelo 111/115 default rates verified current as of this build: 15%
  general professional fees, 7% new professionals (first 3 years), 19% rent
  withholding.
- Modelo 303's output VAT (devengado) is derived from chained sales
  records; input VAT (soportado) is not tracked by this plugin (it belongs
  to the purchase ledger) and is supplied manually before the result box is
  meaningful. Modelo 111/115 are manual-entry calculators for the same
  reason: this plugin has no visibility into payroll or rent payments the
  business makes.
- FACe response parsing (`EngineFaceGateway`) matches the FACe web-service
  response shape (`resultado.codigo` "0" = success; `factura.numeroRegistro`,
  `factura.organoGestor` on success); registration checks map the
  tramitación codes (1200/1300/2400 registered · 2500 paid · 2600/3100
  rejected).
- TicketBAI software-guarantor licences belong to the software vendor, one
  per territory (`tbai.licenses.*`); the customer never types one at
  activation. `Issuer::tbai_license` is an override.

## FACeB2B

The Facturae engine's FACeB2B client exposes a materially larger lifecycle
than FACe (invoice details, cancellation requests and their acceptance/
rejection, download and download confirmation, rejection, mark as paid,
signature validation, the official reason codes) since FACeB2B
counterparties manage invoice state themselves rather than the public
administration. `EngineFaceb2bGateway` wires all of it behind
`Faceb2bGateway`, on the same `FacturaeInvoiceBuilder` output; sending,
cancelling and status-checking are in the panel, the receiver-side
lifecycle is at the service/facade layer (see README Roadmap).

## Testing

`composer test` runs the full Pest suite — 203 tests (chain hashing against
the exact AEAT payload formula, immutability guards and content tampering,
sealed activation, submission flow with a fake gateway, certificate
handling, XAdES sign/verify, the document lifecycle and field-complete
conversions/credit notes, series numbering, hospitality and repair flows
with lineage, printing and template fidelity, PDF drivers, reports, FACe/
FACeB2B and TicketBAI with fake gateways, OCR with a fake agent and the
reconciliation gate, and the API sidecar). Needs ext-gd, bcmath, openssl,
sqlite3 and `mpdf/mpdf` installed. `php artisan verifactu:homologate` runs a local
battery (chain, signature, QR) against a disposable sandbox issuer before
going live, with `--submit` to also attempt a real AEAT sandbox round-trip
when a certificate is present.

## Translations

`resources/lang/{es,gl,ca,eu,pt}.json` cover every `__()` string in the
codebase (1,095 keys each, same order as `es.json`; `en.json` is the English
base and may lag — the key itself is the English text). Add every new
string to the five catalogues in the same turn.
They are registered both as the plugin's own JSON translation path
(`loadJsonTranslationsFrom`) and — per Laravel's standard convention for
bare-string translations — resolve against the host application's own
`resources/lang/{locale}.json` if present there too.

## Things that bit us (keep them in mind)

- **Filament v5 uploads to the app default disk** (`FILESYSTEM_DISK`,
  private `local` on a fresh Laravel). Anything that later reads a file must
  use the same disk: branding through `BrandingDisk`, certificates through
  `certificates.disk`.
- **Livewire temporary uploads**: `TemporaryUploadedFile` gives Symfony an
  empty tmpfile as pathname, so `getContent()` returns zero bytes. Read with
  `$file->get()` (`OcrAttachment`).
- **Blade directives glued to a word** (`kg@endif`) are not compiled
  (`\B@`). Leave a space.
- **Filament rule closures** are evaluated with injection: a Laravel-style
  `function (string $attribute, $value, Closure $fail)` must be wrapped,
  `->rule(fn (): Closure => function (...) {...})`.
- **Livewire cannot hold plain objects** as public properties: the report
  DTOs implement `Wireable`.
- **mPDF**: no `position: fixed` (footer via `SetHTMLFooter`, and any
  `@page` rule wipes it — the driver strips it first); no `div` with
  border/height inside table cells (SAT boxes are tables); raster watermark
  images are prepared with GD (cover, scale, rotate); SVG watermarks cannot
  be rotated.
- **Gotenberg**: send `printBackground`; `singlePage` + `paperWidth` for the
  roll-width ticket page.
- **PHP's 30 s request budget** kills long OCR calls: the services raise it,
  pass an explicit timeout to Laravel AI and retry `FailoverableException`.
- **ActionGroup visibility on view pages** runs before the record is bound:
  take `?Model $record` and fall back to `$livewire->getRecord()`.
- Customer tracking and print-agent routes are registered unconditionally:
  they are reached without a panel, so fluent toggles cannot gate them.
- Demo seeding is incremental (`db:seed --class=VerifactuDemoSeeder` adds
  what is missing); never `migrate:fresh` a database that has been used.