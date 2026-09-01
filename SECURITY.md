# Security Policy

## Supported versions

| Version | Supported |
| --- | --- |
| 1.x | ✅ |

Security fixes land on the latest 1.x release. Older tags are not patched.

## Reporting a vulnerability

Report privately, never in a public issue: open a
[security advisory](https://github.com/komma-softhouse/filament-verifactu/security/advisories/new)
on this repository, or write to **security@kommasofthouse.com**.

Please include the affected version, the steps to reproduce it, and what an
attacker could obtain or alter. A proof of concept helps, but a clear
description is enough.

What to expect:

- Acknowledgement within 3 working days.
- An assessment, with severity and a fix window, within 10 working days.
- Credit in the release notes when the fix ships, unless you prefer not to
  be named.

Please give us a reasonable window to release a fix before disclosing
publicly.

## Scope

This package builds and submits fiscal records to Spanish tax
administrations. Reports of the following are especially welcome:

- Anything that lets a chained fiscal record be altered, deleted or
  reordered outside the append-only guards.
- Anything that exposes certificates, private keys, certificate passwords
  or API keys, in storage, logs or responses.
- Authentication or authorisation bypasses on the on-prem API sidecar, or
  on the public routes (customer repair tracking, print agent pairing).
- Injection or path traversal reachable from uploaded documents, OCR input
  or template branding files.

Out of scope: findings that require an already-compromised host or
privileged access to the database, denial of service through resource
exhaustion, and vulnerabilities in the tax administrations' own services.