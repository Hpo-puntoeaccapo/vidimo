# Vidimo

**Native PHP validation of eIDAS electronic signatures against the EU Trusted Lists.**

Vidimo tells you whether a digitally signed document is legally valid. Give it a signed `.p7m` file, a signed PDF or a signed XML file and it checks that:

- the document has not been changed since it was signed;
- the signing certificate was issued by a provider on the EU Trusted Lists, and whether the signature is qualified;
- the certificate was neither expired nor revoked when the signature was made;
- when the signature was made, using any timestamp it carries.

The answer is **valid**, **invalid** or **undetermined**, following the validation process of ETSI EN 319 102-1. It comes with a standard ETSI validation report and a short summary anyone can read. When in doubt, Vidimo never says "valid".

Everything runs on your own server, in plain PHP. You don't need Java or an external service, and your documents never leave your server. The only outgoing requests are the standard revocation checks (OCSP/CRL) to the certificate issuers.

> **Status: planned.** Vidimo is at the proposal stage. The examples below show the intended API and may change.

## Why

PHP runs most of the web, including a lot of software for public administrations, document management and e-invoicing. Today that software usually calls `openssl_pkcs7_verify()`, which only confirms the maths of a signature. It cannot tell whether the signature is qualified, whether the certificate had been revoked, or when the document was signed. The only complete open-source validator is the European Commission's [DSS](https://github.com/esig/dss), and it needs Java.

Vidimo brings the same standards-based validation to PHP. Every release is checked against DSS on a shared set of test documents.

## Scope

| Format | Covered |
|---|---|
| CAdES (`.p7m`, detached `.p7s`), including nested envelopes | baseline B and T (with timestamp) |
| PAdES (signed PDF), including incremental updates and shadow-attack detection | baseline B, T, LT and LTA |
| XAdES (signed XML), enveloped and detached | baseline B |
| EU List of Trusted Lists and national Trusted Lists | download, signature check, cache, qualification at signing time |
| Revocation and time | OCSP, CRL, RFC 3161 timestamps |

JAdES, ASiC containers and long-term XAdES are not covered for now.

## Intended usage

```php
use Vidimo\Validator;

$result = Validator::create()->validate('contract.pdf.p7m');

$result->indication();   // VALID, INVALID or UNDETERMINED
$result->summary();      // "Qualified signature by Mario Rossi, made on 12 March 2026 at 10:41"
$result->etsiReport();   // ETSI TS 119 102-2 validation report (XML)
$result->content();      // the original document, taken out of its envelope
```

From the command line:

```sh
vidimo check contract.pdf.p7m
vidimo check --report report.xml signed.pdf
vidimo trusted-lists:update
```

There will be packages for Laravel and Symfony, and a prototype integration with [LibreSign](https://github.com/LibreSign/libresign) for Nextcloud.

## Roadmap

1. `.p7m` validation against the Trusted Lists, with revocation checks
2. Signed PDFs, including long-term signatures
3. Signed XML and standard ETSI reports
4. Automatic comparison with DSS, fuzzing and a security review
5. Laravel and Symfony packages, LibreSign integration, documentation, version 1.0

## Standards

- ETSI EN 319 102-1: validation of AdES digital signatures
- ETSI EN 319 122, 319 132 and 319 142: CAdES, XAdES and PAdES
- ETSI TS 119 102-2: validation report
- ETSI TS 119 612 and TS 119 615: Trusted Lists and their use
- RFC 5652 (CMS), RFC 3161 (timestamps), RFC 6960 (OCSP), RFC 5280 (certificates and CRLs)
- Implementing Regulation (EU) 2025/1945: validation of qualified electronic signatures

## Licence

[EUPL-1.2](https://interoperable-europe.ec.europa.eu/collection/eupl/eupl-text-eupl-12)

## Who

Vidimo is developed by [HPO Software House](https://puntoeaccapo.com), Naples, Italy.
