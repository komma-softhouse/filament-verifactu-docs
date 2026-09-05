# Changelog

All notable changes to `filament-verifactu` are documented here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this
project adheres to [Semantic Versioning](https://semver.org/).

## [1.3.5] - 2026-09-05
 
### Changed
- The Composer archive no longer ships the Windows print-agent binaries,
  screenshots, tests or CI (`.gitattributes` export-ignore): 17 MB → 1 MB,
  and a package Anystack's Composer endpoint can serve.
- The print-agent installer downloads `pos-agent.exe` and `nssm.exe` from
  `VERIFACTU_AGENT_BINARIES_URL` (default: the host's
  `public/downloads/windows/marfil-printer-agent/`); `/download/windows/{file}`
  redirects there for installers generated earlier.

## [1.3.1] - 2026-09-05

### Added
- `VERIFACTU_MIGRATIONS=false` (config `migrations.enabled`): keeps the
  package's migrations out of the host's central `migrate`, for multi-tenant
  hosts with a database per tenant that run them through their tenant
  migrator instead.

## [1.3.0] - 2026-09-05
 
### Added
- Host hooks: `Verifactu::beforeIssuing()` — a veto with the host's own
  reason before completing, chaining, cancelling, booking or activating —
  and domain events after commit (`IssuerActivated`, `DocumentCompleted`,
  `DocumentVoided`, `RecordRegistered`, `RecordCancelled`, `ExpenseBooked`,
  `SubmissionResolved`). The API answers a veto with 402 `denied_by_host`.
- API sidecar: the issuer behind the key (read, edit plain fields, sealed
  activation, certificate upload/read/remove) and the document layer (list,
  create with lines, complete, convert, credit note whole or per line,
  void, PDF).
- Recurring documents: templates that issue drafts (or completed, chained
  documents) on a weekly/monthly/quarterly/yearly schedule, with
  `verifactu:recurring:run` and "Run now" / "Generate now".
- `->catalog()`: a minimal product & service catalogue that prefills
  document lines, global or per issuer.
- Aging & balances report: receivables and payables by days past due and
  per party, with CSV.
- `verifactu:selftest {issuer}`: one real one-cent ticket against the
  sandbox with the issuer's certificate; refuses production.
- Presentation files in the AEAT record layout (BOE) for AEAT issuers, from
  their report pages: 347 and 349 (500-position records), 303, 111 and 115
  (tagged self-assessment layout, 2026 designs). The 347 draft now carries
  the quarterly split and the party's postal code.
### Changed
- Row actions grouped behind a "More" menu with sections on the six large
  tables (documents, received documents, repairs, records, FACe history,
  fiscal calendar): two or three primary buttons stay visible.

## [1.2.2] - 2026-09-04

### Changed
- The licence check's test-suite exclusion detects the test runner itself
  (its classes loaded), not `APP_ENV`: an environment variable is a
  setting, and a setting that switches the check off would be no check at
  all.
- README: the banner is hidden on the Filament directory
  (`filament-hidden`), screenshots render at one width, the engineering
  notes moved to NOTES.md, the licensing paragraph no longer describes the
  exclusion mechanism, and the FACe example names a fictitious
  administration.

## [1.2.0] - 2026-09-04

### Added
- Fiscal calendar: every issuer's obligations under its treasury's models,
  with statutory deadlines, an overdue badge, "Fill in" opening the exact
  draft, the treasury portal, and filed / omitted states with the receipt
  reference (`verifactu_filings`).
- "Registered in the SII" at fiscal activation: an AEAT issuer sealed as
  outside the RRSIF (art. 3.2 RD 1007/2023) — numbered invoices, no chain,
  no tributary QR, drafts read from the completed documents. Refused for
  foral issuers and for VERI*FACTU mode.
- Collections on sales: collected / pending / overdue against the due
  date, payment method, "Mark as collected" and "Mark as not collected",
  filters, and the Collected / Pending / Overdue cards above the documents
  list. Purchases gain a due date and a payment method.
- Chart of accounts seeded from the PGC Pymes, an income account per sale
  and an expense account per purchase (category defaults, remembered per
  counterparty NIF), and a yearly accounting export CSV.
- Financial summary widget on the dashboard: income, booked purchases,
  result, VAT to settle, IRPF to settle — year to date, every issuer.
- `mark-collected` ability; demo seeder blocks for the chart, collections,
  an SII-exempt issuer and the calendar.

### Fixed
- Fourth-quarter deadline of the AEAT withholding models (111, 115) is
  20 January, not 30.

## [1.1.1] - 2026-09-03

### Changed
- Provenance notes in 21 docblocks rewritten without naming other
  products; comments only.
- README links to the architecture, changelog, security policy and
  licence files are absolute to the public docs repository, so they
  resolve from the Filament plugin directory as well as from GitHub.
- The licence check reports release `1.1.1` in its validation scope.

## [1.1.0] - 2026-09-03

### Added
- Runtime licence check against Anystack (`LicenseGuard`): the key is read
  from Composer's `auth.json` — nothing to configure — and validated daily
  with the host of `APP_URL` as fingerprint; verdict persisted in
  `verifactu_license_state`, banners at 30/15/7 days before expiry, a
  15-day grace period once the licence is not in force, and only then a
  stop on issuing — completing, chaining, submitting, FACe, OCR and
  booking, from the panel, the facade and the API sidecar alike. Reading,
  exporting, downloading and chain verification never stop. A licence
  server outage keeps the last known verdict. `verifactu:license` shows
  and refreshes the verdict.

### Fixed
- `AeatQueryService` posted to a `/ConsultaSOAP` path that does not exist
  at the AEAT; the query operation is served by the same `VerifactuSOAP`
  endpoint as the submissions.
- `verifactu:homologate`: nine-character sandbox NIF, no recipient on the
  F2, a recipient on the R1, and child rows removed before the disposable
  issuer.
- Fiscal records table gains a copyable, searchable CSV column (TBAI
  identifier for foral records).

## [1.0.0] - 2026-09-03

First release: everything below ships together; nothing was published before.

### Added
- Navarra as the fourth fiscal regime: full document lifecycle for
  Navarrese issuers with the antifraud activation locked (NaTicket has no
  published specification yet, so nothing is chained), report drafts under
  the foral numbers as their own pages — F-69 quarterly VAT, F-50
  third-party operations (same €3,005.06 threshold), the foral 349, 715
  and 759 — computed from the completed fiscal documents, with each
  treasury's report set hidden from navigation when the install has no
  issuer of its regime, and TicketBAI-only code paths guarded by
  `FiscalRegime::usesTicketBai()` instead of "not AEAT".
- Basque foral reports: the 303, 347, 349 and 115 drafts serve TicketBAI
  issuers from their own chained foral records (same model numbers, each
  filing with its own Hacienda), the quarterly 110 joins as the Basque
  general payroll page (the foral 111 is monthly/large-company, out of
  scope), and Bizkaia gets no 347 — the LROE (Batuz, models 140/240)
  replaces it and this plugin submits both of its chapters, so the draft
  refuses and the page hides for Bizkaia-only installs.
- Chained fiscal records (registration and cancellation) against the AEAT
  VERI*FACTU / RRSIF hash chain, with the sealed two-step fiscal activation
  flow and forward-only mode switching.
- Submission pipeline: per-issuer batching, AEAT flow-control aware queued
  job, certificate management, and the on-demand (non-Verifactu) channel
  with requirement/voluntary-remission context.
- XAdES-BES signing and standalone record export for non-Verifactu issuers.
- Tributary QR generation (URL, SVG, ESC/POS) for both remission modes.
- Full document lifecycle: quotes, orders, delivery notes, proformas,
  invoices, simplified tickets and credit notes, with a single completion
  gate, a forward-only conversion DAG, per-line returns, hospitality
  pre-bills and kitchen routing, and SAT repair orders.
- Gapless per-issuer, per-type series numbering with configurable formats.
- Document templates with a dual (A4 + thermal ticket) live designer, A4
  rendering with automatic draft/copy watermarking, and a 14-kind ESC/POS
  printer.
- FACe electronic invoicing (send, cancel, DIR3 administration lookup) and
  OCR-assisted document drafting.
- An on-prem API sidecar for external ERPs/POS terminals, authenticated by
  per-issuer bearer keys.
- Draft fiscal reports: Modelo 303 (VAT), 347 (third-party operations), 349
  (intra-community operations), 111 and 115 (withholding), each exportable
  as CSV.
- The received side of the ledger (`->expenses()`, off by default): purchase
  invoices, purchase tickets and supplier credit notes on their own tables —
  never numbered, chained or converted — with OCR capture, attachment
  preview and download, a deductible-VAT share kept separate from the
  expense, expense categories, purchase type and withholding. Booking
  freezes a purchase the way completing freezes a sale.
- The tax drafts stop being half-blind: deductible input VAT feeds the
  303/F-69 with the manual figure demoted to an adjustment, the 347/F-50
  gains its purchase side (clave A), the 349 its intra-community
  acquisitions, and the 110/111/715 and 115/759 prefill their payees from
  booked purchases.
- Bizkaia's LROE received-invoices chapter (Modelo 140 for a self-employed
  issuer, 240 for a company) submitted through the engine, so both halves of
  Batuz ship.
- Corporate income tax estimate and art. 40.2 LIS instalment provisioning on
  the 2026 rate scale (Ley 7/2024): invoiced income less booked purchases,
  with payroll, depreciation and fiscal adjustments declared by the user — a
  provisioning tool, never a filing.
- English, Spanish, Galician, Catalan, Basque and Portuguese translations
  (1,268 keys each).
- A local homologation battery (`verifactu:homologate`) exercising the full
  chain, signature and QR before going live.
- **TicketBAI** (Araba, Bizkaia, Gipuzkoa) over `komma-softhouse/tbai-engine`: a
  second, complete fiscal engine behind the same `RecordService` gate as
  AEAT — registration, cancellation, corrective (rectificativa) invoices,
  Zuzendu subsanación (Araba/Gipuzkoa only — Bizkaia's own `Endpoint`
  rejects it), national and foreign breakdown categories, and automatic
  Batuz/LROE envelope selection (Modelo 140 autónomo vs 240 sociedad) for
  Bizkaia. Chained by XAdES signature value, not hash. A dedicated
  `verifactu:homologate-tbai` battery, per-territory.
- **FACeB2B** (Ley 25/2013's already-active extension to large private
  subcontractors of public administrations, distinct from the still-pending
  Crea y Crece B2B mandate): full lifecycle, all 13 `Faceb2bTrait` methods
  — send, request/accept/reject cancellation, check status, list
  registered invoices and pending cancellations, download, confirm
  download, reject, mark as paid, validate signature, and look up the real
  business codes (never free-text guesses) for cancellation/rejection
  reasons.
- Per-line and general document discounts, and IRPF withholding on
  invoices (mapped to Facturae's `addWithholding()`), shown on the A4
  template, the ESC/POS ticket, the document view and the documents table.
- Merging several completed delivery notes/orders into a single invoice
  draft (`Verifactu::mergeDocuments()`), from a bulk table action.
- Print Agent add-on: pairing, Windows installer/uninstaller generation
  (NSSM service), broadcast-based ticket dispatch, and online/offline
  status — the pattern validated in production on real tills.
- OCR rewritten on [Laravel AI](https://laravel.com/docs/13.x/ai-sdk) with
  structured output — the provider is never hardcoded; the host picks any
  provider they already hold a key for from `OcrSettingsPage`.
- Dedicated `FacturaeSettingsPage` and `GenerarFacturaePage` (generate +
  history in one screen), both backed by real, persisted settings
  (`FaceSettings`, `OcrSettings` — `spatie/laravel-settings`).
- Repair orders: a customer-facing tracking QR (distinct from the tributary
  QR) and a `RepairStatusChanged` event on every transition.
- Bulk CSV/XLSX export on the fiscal records and system events tables, and
  CSV export on every report page.
- `verifactu:submit-batch` command for on-demand (requirement/voluntary)
  submissions from the terminal.
- `Verifactu::zReport()`, `xReport()` and `movements()` — host-supplied
  shift-close, reading and sales-movement thermal reports, with an
  `AuditEntry` on every Z close.
- A `GiftVoucher` ESC/POS ticket kind (stored-value voucher), distinct from
  the gift receipt, and reprint tickets always carry a "COPY" mark.
- Modelo 349 classifies each line as goods (E) or services (S) from the
  chained record's own operationType (E5/N2), instead of a fixed default.
- A consolidated `VerifactuSettingsPage` per issuer (activation, certificate,
  mode, environment, API key, TicketBAI license/self-employed flag —
  editable after activation, unlike the other sealed fields).
- Signed webhooks (`X-Verifactu-Signature`, HMAC-SHA256) fired when a
  submission resolves as accepted or rejected.
- FACe history (`FaceSubmissionResource`): re-download the stored `.xsig`,
  resend without re-signing, and check whether a registry number is still
  known to FACe. TicketBAI records get the same `.xsig` re-download from
  their own record view.
- A per-issuer FACe directory: DIR3 codes are remembered after a registered
  send and pre-fill the next submission to the same recipient.
- **PDF engines as an explicit opt-in**: `->mpdf()` (bundled pure-PHP
  engine, nothing to install) or `->gotenbergPdf()` (Chromium, Docker) —
  same layout on both; without one, PDF actions stay hidden. Ticket PDFs
  print on their 80/58 mm roll page, as tall as the content.
- **Template fidelity**: A4, ticket and SAT papers mirror the designers
  field by field (logo, colours, texts, labels, watermark text/image with
  geometry, legend, QR, SAT title/clauses/toggles), on screen and on PDF,
  regime-aware (VERI\*FACTU legend, AEAT non-Verifactu QR only, TicketBAI
  identifier). In-panel previews for fiscal documents and SAT papers.
- **Branding disk** (`->brandingDisk()`, `VERIFACTU_BRANDING_DISK`): one
  configurable disk for template logos and watermark images, shared by
  uploads, previews, renderers and ESC/POS — any disk of the host.
- **Implementer identity from the panel provider**: `->computerSystem(...)`
  (AEAT) and `->ticketBai(...)` (foral developer NIF, product and the
  software-guarantor licences per territory — the vendor's, never the
  customer's; activation never asks for one).
- **OCR hardening**: any Laravel AI provider, PDFs sent as documents,
  literal-transcription instructions, retries on provider overload, and a
  bcmath reconciliation gate — no draft unless the read lines add up to
  the printed total.
- Tributary QR on documents (table and view), regime-aware, and on foral
  records; `GET /records/{id}/qr` returns number, status, hash, legend or
  TBAI identifier, CSV, SVG, PNG and the native ESC/POS QR block.
- Forward-only document flow (a ticket is never a conversion target),
  toggle-button type pickers, deletable drafts (`delete-draft`), shipping
  block and per-line traceability, field-complete conversions/merges/credit
  notes, repairs as a node of the flow (delivery descending from the repair
  quote, device serial as traceability, "Repair (SAT)" section).
- Chain verification of link and content (`reason: link | content`),
  Wireable report DTOs, user names in the audit trail, live validation on
  every validated form field (`->validatesLive()`).

[1.3.5]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.3.5
[1.3.1]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.3.1
[1.3.0]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.3.0
[1.2.2]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.2.2
[1.2.0]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.2.0
[1.1.1]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.1.1
[1.1.0]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.1.0
[1.0.0]: https://github.com/komma-softhouse/filament-verifactu/releases/tag/v1.0.0
