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
- The five report drafts (303, 347, 349, 111, 115)

Everything past that is a fluent opt-in on `VerifactuPlugin::make()`:

| Toggle | What it turns on |
| --- | --- |
| `->documents()` | The whole document lifecycle (quotes → invoices), templates, series |
| `->repairs()` | SAT repair orders |
| `->escPos()` | Thermal ticket printing |
| `->printAgent()` | Print Agent pairing + Windows installer generation |
| `->face()` | FACe (public-sector einvoicing) |
| `->faceb2b()` | FACeB2B (private large-subcontractor einvoicing) |
| `->ocr()` | OCR-assisted drafting from a photo |
| `->api()` *(env-based, not fluent — see README)* | The on-prem API sidecar |

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
