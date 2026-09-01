# How this plugin is built, and how to sell it in pieces

This is the guide that was missing: how the plugin works internally, and
— the part asked for specifically — how the module/add-on structure maps
to what you can charge for separately.

## 1. The core idea: one gate, two engines behind it

Everything fiscal goes through a single class, `RecordService`. It never
talks to AEAT or TicketBAI directly — it looks at `$issuer->regime` and
routes to one of two, completely separate engines:

- **AEAT** (`regime = aeat`): hash-chained, XAdES-signed in non-Verifactu
  mode, submitted synchronously-queued via `SubmissionDispatcher` →
  `SubmitRecordsJob`.
- **TicketBAI** (`regime = araba|bizkaia|gipuzkoa`): a *different* engine
  with its own chaining (by XAdES signature value, not hash), its own
  three territory endpoints, and fully synchronous registration (no queue
  at all — the accept/reject verdict is in the same call).

Both engines are wrapped behind their own gateway contract
(`AeatGateway`/`TicketBaiGateway`/`FaceGateway`/`Faceb2bGateway`) — that's
what makes the whole test suite fakeable without ever hitting a real
government endpoint.

Everything else in the plugin — documents, series, templates, reports,
repairs — is regime-agnostic by design: it never checks which engine an
issuer uses, it just calls `RecordService`/`DocumentGate` and lets those
decide. That's a deliberate boundary, not an accident: it's what let
TicketBAI get added as a second engine without rewriting the document
layer.

## 2. What's always on vs. what's opt-in

Three resources are **never toggleable** — they're the fiscal record
itself, and a host that installs this plugin gets them the moment the
plugin is registered, with nothing else enabled:

- Issuers (activation, certificates, API keys)
- Fiscal records, Submissions, System events, Audit trail
- The report drafts. Every page registers, and each hides itself from
  navigation when the install has no issuer it serves, so nobody ever
  reads another treasury's model numbers: the shared-numbering set (303,
  347, 349, 115) serves AEAT and the Basque territories alike, the state
  111 is AEAT-only against the Basque quarterly 110, the 347 hides for a
  Bizkaia-only install (the LROE replaces it), Navarra has its own foral
  set (F-69, F-50, foral 349, 715, 759), and the corporate-tax estimate
  sits alongside them (`RegimesPresent`).

Everything past that is a fluent opt-in on `VerifactuPlugin::make()`:

| Toggle | What it turns on |
| --- | --- |
| `->documents()` | The whole document lifecycle (quotes → invoices), templates, series |
| `->repairs()` | SAT repair orders |
| `->expenses()` | The received (purchase) side: supplier invoices, tickets and credit notes, OCR capture, attachment preview, and the LROE expenses book for Bizkaia |
| `->escPos()` | Thermal ticket printing |
| `->printAgent()` | Print Agent pairing + Windows installer generation |
| `->face()` | FACe (public-sector einvoicing) |
| `->faceb2b()` | FACeB2B (private large-subcontractor einvoicing) |
| `->ocr()` | OCR-assisted drafting from a photo or PDF |
| `->api()` | The API panel page (endpoints, keys); the sidecar itself is `VERIFACTU_API=true`, because it is reached without a panel |
| `->dashboardWidgets()` | Dashboard widgets, each active module contributing its own |
| `->mpdf()` / `->gotenbergPdf($url)` / `->pdfDriver(FQCN)` | The PDF engine. **Nothing is assumed**: with none of the three, `hasPdf()` is false and every PDF/email/ZIP action stays hidden |

And three settings that are not modules but identity/storage, also fluent
so a host never has to publish the config:

| Setting | What it carries |
| --- | --- |
| `->computerSystem(...)` | The implementer's identity for AEAT records (vendor, product, installation) |
| `->ticketBai(...)` | The implementer's identity for TicketBAI and the **software-guarantor licences per territory** |
| `->brandingDisk($disk, $visibility)` | Where template logos/watermark images live — uploads, previews, renderers and ESC/POS all use it |

Every fluent value is reconciled into `config('filament-verifactu.*')` in
`VerifactuPlugin::boot()`; `.env`/config are the fallback, fluent wins. Two
things cannot go through the toggles because they run without a panel: the
API sidecar (config) and the public routes for customer repair tracking and
the print agent (registered unconditionally).

## 3. The add-on question: what maps to what

Print Agent is a genuinely natural add-on to charge for separately: it's
the only toggle that ships an *installable Windows service* (the
installer/uninstaller `.bat` generation, NSSM, the pairing/online-status
UI) rather than just unlocking a code path — a physically separate thing a
till needs installed on it, per device.

**Whether `->face()`, `->faceb2b()`, `->ocr()` become their own paid tiers
on Anystack (vs. bundled into the base price) is a pricing call, not an
engineering one, and it's yours to make, not mine to invent.** What I can
tell you factually: each one is cleanly separable (its own toggle, its own
migrations, its own resources — turning one off doesn't touch the others),
so whatever split you land on, Anystack's per-product-tier model (see the
launch guide) can express it without any code changes — the toggles
already *are* the product boundaries.

## 4. Where each add-on's code actually lives (for a licensing/paywall gate, if you build one)

If you ever want a license check that disables a toggle server-side
(not just "the host chose not to call `->printAgent()`", but "the host
tried to and your license server said no") — this is not built, and
would need its own small gate inside `VerifactuPlugin::hasPrintAgent()`
etc. (each `has*()` method is already the single place that decides
whether a module is active — see `src/VerifactuPlugin.php`). That's the
one seam that would need work if you go down that road; nothing else in
the codebase assumes anything about licensing today.

## 4b. The received side

The purchase ledger lives in its own tables
(`verifactu_received_documents` + lines) and never touches the issued
gate: a received document is not numbered by us, not chained, not
converted and not printed as ours — the number and the date belong to the
supplier. What the two sides share is reused without inheritance: the
arithmetic (`DocumentTotals::computeLines()`, called through
`ReceivedTotals`) and the OCR.

`ExpenseBooking` is its gate — the mirror of `DocumentGate` minus
everything that is ours: booking freezes the amounts and stamps the
moment, and from there the drafts read them. `ExpenseLedger` is the only
door those drafts use: deductible input VAT per rate (never the whole
supported quota — the deductible share lives on the document, and a
reverse-charge purchase deducts nothing), purchases by third party,
intra-community acquisitions, and the withholding actually practised.
`ReceivedDocumentType::countsForTax()` is the line between what reaches a
report (purchase invoice, purchase ticket, supplier credit note) and what
is only supplier history (quotes, orders, delivery notes, proformas
received).

For Bizkaia, `LroeExpensesDriver` + `LroeExpensesMapper` submit the LROE's
received-invoices chapter (Modelo 140 for a self-employed issuer, 240 for
a company) through the same engine gateway — so both halves of Batuz
ship, which is why a Bizkaia issuer files no Modelo 347.

`CorporateTaxService` sits on top of both sides: invoiced income (from
whatever source that issuer's VAT draft already uses) less booked
purchases, with payroll, depreciation and fiscal adjustments declared by
the user. It is an estimate and a provisioning tool, never a filing.

## 5. The document layer

`DocumentGate::complete()` is the only way a document gets its number: it
takes the next slot of the series that matches issuer and type
(`SeriesService`, gapless, `{PREFIX}{YEAR}{STORE}{TERMINAL}`), computes the
totals in bcmath (`DocumentTotals`: line and general discounts, tax-inclusive
prices, equivalence surcharge, IRPF), freezes the document and its lines,
and — for a fiscal type on a sealed issuer — calls `RecordService` inside the
same transaction. Non-fiscal types (quote, order, delivery note, proforma)
complete with a number and chain nothing.

The flow is a forward-only graph (`DocumentConverter::EDGES`): quote → order
· delivery note · proforma · invoice; order → delivery note · proforma ·
invoice; delivery note → proforma · invoice; proforma → invoice. A ticket is
never a target (it is born at the POS). Conversions and merges copy every
commercial term (`DocumentConverter::lineAttributes()`), and a credit note
(`CreditNoteService`) inherits the invoice's own terms, so the credited
amount is the charged one. Repairs are a node of the same graph:
`RepairService::quote()` issues a Q document, `deliver()` opens the fiscal
paper descending from that quote (parent + source documents), and the device
serial travels as line traceability; `FiscalDocument::originRepairOrder()`
resolves the order from any of those documents.

Two JSON columns hold what is printed but never fiscal: `documents.shipping`
(carrier, tracking, packages, weight) and `document_lines.meta`
(serials, batch, expiry).

## 6. Printing: one HTML, two engines

`DocumentRenderer` resolves the template (`TemplateResolver`: issuer + type,
then issuer default, then global) and renders one Blade view per paper
(`documents.a4`, `documents.ticket`; `repairs.receipt`, `repairs.report`
for SAT). The view carries a `forPdf` flag: on screen it draws a sheet or a
roll on a grey backdrop; for PDF it emits the same markup with a `@page`
rule and two markers the drivers understand — `<meta name="verifactu-paper">`
(ticket width) and `<meta name="verifactu-watermark[-image]">` (text or
image with alpha, size and rotation) — plus a `.vf-page-footer` block.

`PdfDrivers::resolve()` returns the configured `PdfConverter`:

- `MpdfPdfConverter` (pure PHP): strips `@page`, moves the footer block
  into `SetHTMLFooter()`, stamps the watermark natively (text rotated;
  raster images cover-cropped, scaled and rotated through GD first; SVG
  natively, unrotated), and prints tickets on a roll-width page cut to the
  measured height (two passes).
- `GotenbergPdfConverter` (Chromium): `printBackground`, and `singlePage`
  + `paperWidth` for tickets. CSS `position: fixed` does the footer.

Every image (logo, watermark, QR) travels inside the HTML as a data URI or
inline SVG, through `BrandingDisk::dataUri()`, so no engine needs to reach
the host's disks by URL. `EscPosPrinter` reads the same template fields for
the thermal path. `tests/TemplateFidelityTest` walks every template field
of both models and checks it reaches the output.

## 7. Regime on paper

`QrRenderer` decides per record: AEAT VERI\*FACTU → QR + legend (+ CSV once
answered); AEAT non-Verifactu → QR alone; TicketBAI → the 39-character
identifier and the foral QR url stored at registration (never recomputed).
`QrUrlCheck` self-checks a URL against its record and the issuer environment
in both regimes. The API's `GET /records/{id}/qr` returns all of it (url,
legend or identifier, CSV, SVG, PNG, native ESC/POS block) so a POS prints
what the panel prints.

## 8. OCR

`OcrService` → `InvoiceExtractionAgent` (Laravel AI structured output, any
provider the host configured in `OcrSettings`). `OcrAttachment` reads the
real bytes of a Livewire temporary upload and sends PDFs as documents, images
as images. The result only becomes a draft when `OcrService::reconcile()`
finds that the read lines add up to the printed total (net + tax, or
tax-inclusive) in bcmath; otherwise it throws and nothing is created. Retries
cover provider overload/rate limits; the request budget is raised for the
call.

## 9. Licences and identity

The AEAT "sistema informático" identity (`computer_system.*`) and the
TicketBAI vendor identity (`tbai.developer_nif`, product) are installation-
level. The TicketBAI software-guarantor licences are too, one per territory
(`tbai.licenses.{araba,bizkaia,gipuzkoa}`): they belong to whoever
registered the software with each foral treasury, never to the customer, so
activation shows whether the territory licence is configured and asks for
nothing. `Issuer::tbai_license` overrides it for a host that is garante
itself. `TicketBaiDriver::vendor()` is the single resolution point.

## 10. Integrity checks

`ChainVerifier` verifies the AEAT chain twice per record — the link
(`previous_hash`) and the content (hash recomputed from the stored fields
through `RecordHydrator`, the same rehydration used before every
submission) — and reports `reason: link | content`. TicketBAI issuers get
a distinct check on stored signatures and identifiers. `AppendOnly` refuses
any Eloquent mutation of business fields; anything done straight in the
database is what the content check exists for.

## 11. Boundaries to respect

- The plugin and the AEAT engine share the PSR-4 root `Komma\Verifactu\`
  over two `src/` directories: never add `Models\Model`,
  `Models\ComputerSystem`, `Services\AeatClient` or `Exceptions\*` here.
- Regime-specific code lives only in the engines' adapters (`RecordService`
  and its TicketBAI branch, `TicketBaiDriver`/`Mapper`, `QrRenderer`,
  `QrUrlCheck`, `FacturaeInvoiceBuilder`); everything else stays
  regime-agnostic.
- Navarra is the fourth `FiscalRegime` today: full document lifecycle,
  foral report numbers (F-69, F-50, 349, 715, 759) computed from
  completed documents, activation locked and no records chained. Its
  NaTicket driver plugs in behind the same gate once a specification
  exists — the TicketBAI-only branches are guarded by
  `FiscalRegime::usesTicketBai()`, never by "not AEAT".
- Translations: six JSON catalogues (`en, es, gl, ca, eu, pt`) with the
  same key order; every new string goes to all six.