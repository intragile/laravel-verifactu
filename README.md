# Laravel VERI\*FACTU / Spanish billing records

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-verifactu/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-verifactu/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-verifactu.svg)](https://packagist.org/packages/intragile/laravel-verifactu)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-verifactu.svg)](https://packagist.org/packages/intragile/laravel-verifactu)
[![License](https://img.shields.io/packagist/l/intragile/laravel-verifactu.svg)](LICENSE)

Spanish **VERI\*FACTU** for Laravel: generate, hash, chain and submit the AEAT billing records (`RegistroAlta`, `RegistroAnulacion`) of your invoices from EN 16931 invoices, directly to the AEAT or through B2Brouter, with QR codes and chain verification.

> **Status:** not released yet: it needs `intragile/laravel-einvoicing` 1.3 and `intragile/php-einvoicing` 1.3.

> **Release label:** Verifactu live-verified on the B2Brouter sandbox (registered; hash and QR cross-checked); direct AEAT submission verified offline only — it needs a Spanish NIF and a qualified certificate

This is a thin metapackage: it has no code of its own and pulls in [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) and the engine [`intragile/php-einvoicing`](https://github.com/intragile/php-einvoicing), so that you can find the Spanish use case by its name. The full functionality is documented in the **Spanish VERI\*FACTU** section of the [laravel-einvoicing README](https://github.com/intragile/laravel-einvoicing#spanish-verifactu).

[Magyar nyelvű leírás → README.hu.md](README.hu.md)

## What the Spanish flow covers

- **Records:** a `RegistroAlta` for every invoice and a `RegistroAnulacion` for every cancellation, generated from an EN 16931 invoice of the engine's model, including corrective invoices and exempt lines.
- **Chain:** the SHA-256 `Huella` of every record, chained per issuer and installation in the database; a chain can be seeded from the records of another billing system, and `einvoicing:verifactu-verify-chain` checks it.
- **QR code:** the AEAT verification URL is returned as soon as the record is generated, so the invoice can be printed at once.
- **Direct AEAT submission:** queued, under the AEAT flow control, over mutual TLS with the issuer's qualified certificate.
- **B2Brouter:** B2Brouter can take the AEAT side instead, asynchronously; the record it returns is cross-checked against our own hash, chain and QR.
- **Doctor:** `einvoicing:doctor` checks the configuration, the schemas, the certificate and the chains.

## Scope: VERI\*FACTU mode only

Not covered, each deferred to a later release or connector:

- **non-VERI\*FACTU mode** (XAdES-signed local records, the event log and its chain, the 6-hour summary event, the chain self-check and the *requerimiento* endpoint);
- **TicketBAI** (the Basque foral territories) and **Navarra**, which run their own regimes;
- **Facturae 3.2.x / FACe** (invoices to the public sector);
- **SII**: SII taxpayers are outside VERI\*FACTU;
- **the Crea y Crece B2B e-invoicing mandate** (Real Decreto 238/2026, Orden HAC/1028/2026);
- **the paginated `ConsultaFactuSistemaFacturacion` export**: only a single-invoice lookup is available.

**Dates.** Under current law (Real Decreto-ley 15/2025) VERI\*FACTU applies from 1 January 2027 to corporate-tax payers and from 1 July 2027 to everyone else. On 5 October 2026 the Ministry announced a move of both to October 2028 (**announced, not enacted**: nothing had been published in the BOE as of 23 November 2026). Nothing in this package depends on which date applies.

## No certification, no declaration

This package is a component, not a certified billing system (SIF). The producer of a billing system signs a *declaración responsable* that the system meets the RRSIF (Real Decreto 1007/2023, Orden HAC/1177/2024). **IntrAgile does not act as producer, signs no declaration and ships no template for one.** The libraries implement record generation, the `Huella` and the chain, the record XML, the QR URL and the AEAT SOAP client. Your application must still guarantee:

- the indefectible coupling of every invoice and its record (no invoice without a record);
- that records are never deleted or altered;
- that records are sent immediately;
- the correction and cancellation flows;
- its own `SistemaInformatico` identity (the producer, the system and the installation);
- the declaration itself, naming this library and its version as a component.

## Requirements

- PHP 8.3+ with `dom`, `libxml`, `mbstring`, `openssl` and `zlib`
- Laravel 12 or 13
- For direct AEAT submission: the issuer's (or its representative's) qualified certificate, and a CA bundle with the FNMT root **AC RAIZ FNMT-RCM SERVIDORES SEGUROS**
- Optional: `chillerlan/php-qrcode` ^5 to render the QR code (`composer require chillerlan/php-qrcode`)

## Quick start

Once released:

```bash
composer require intragile/laravel-verifactu
php artisan vendor:publish --tag=einvoicing-config   # config/einvoicing.php
php artisan einvoicing:install-rules                 # the Peppol rules the doctor checks
php artisan einvoicing:verifactu-install-schemas     # downloads the AEAT schemas (never bundled)
php artisan migrate                                  # the chain, throttle and record tables
php artisan einvoicing:doctor
```

Environment keys (they fill `einvoicing.verifactu`):

```dotenv
VERIFACTU_ENVIRONMENT=test               # test (AEAT pre-production) or production; production only when set
VERIFACTU_DRIVER=aeat                    # aeat (direct submission) or b2brouter
VERIFACTU_ISSUER_NIF=B12345674           # a single issuer; config/einvoicing.php takes several
VERIFACTU_ISSUER_NAME="Mi Empresa S.L."

# The billing software that produces the records (SistemaInformatico): yours, never IntrAgile.
VERIFACTU_PRODUCER_NAME="Your Software S.L."
VERIFACTU_PRODUCER_NIF=A39200019
VERIFACTU_SYSTEM_NAME=YourInvoicing
VERIFACTU_SYSTEM_ID=01
VERIFACTU_SYSTEM_VERSION=2.4.1
VERIFACTU_INSTALLATION_NUMBER=0001
VERIFACTU_SYSTEM_VERIFACTU_ONLY=true
VERIFACTU_SYSTEM_MULTIPLE_TAXPAYERS_POSSIBLE=false
VERIFACTU_SYSTEM_MULTIPLE_TAXPAYERS_INDICATOR=false

# Direct AEAT submission (driver aeat)
VERIFACTU_AEAT_CERTIFICATE=/run/secrets/aeat.p12
VERIFACTU_AEAT_PASSWORD=...
# VERIFACTU_CA_BUNDLE=/etc/ssl/aeat-ca.pem   # must contain the FNMT root

# B2Brouter (driver b2brouter): the name of a b2brouter transport of einvoicing.transports
# VERIFACTU_B2BROUTER_TRANSPORT=b2brouter
```

Register an invoice and print its QR code at once; the record is submitted through the queue:

```php
use IntrAgile\EInvoicing\Laravel\Facades\EInvoicing;

$record = EInvoicing::verifactu()->register($invoice);   // an EN 16931 invoice of the core model

$record->qr_url;   // the AEAT verification URL to print on the invoice
```

Start with `VERIFACTU_ENVIRONMENT=test`: records go to the AEAT production service only when the environment is `production`. Cancelling and correcting, the queue and the flow control, B2Brouter mode, seeding a chain, the events, the commands and the doctor are described in the [Spanish VERI\*FACTU section](https://github.com/intragile/laravel-einvoicing#spanish-verifactu); the record mapping, the chain, the QR code and the transports in the [Spain country notes](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/SPAIN.md).

## How it differs

- **[`squareetlabs/laravel-verifactu`](https://packagist.org/packages/squareetlabs/laravel-verifactu)** is a Laravel package for VERI\*FACTU records (Laravel 10 to 12) that requires `ext-soap`.
- **[`aichadigital/lara-verifactu`](https://packagist.org/packages/aichadigital/lara-verifactu)** is a Laravel package for VERI\*FACTU (Laravel 12 and 13) that requires `ext-soap`.

This package differs in three ways:

- **An EN 16931 engine underneath:** the records are built from the same EN 16931 invoice model that `intragile/php-einvoicing` reads, writes and validates as UBL and CII and sends over Peppol, so one invoice object serves the other countries as well.
- **AEAT vectors as tests:** the hash reproduces the official AEAT test vectors, every generated record is checked against the official AEAT schemas, and records generated by B2Brouter in its sandbox are kept as fixtures and re-verified.
- **Two transports:** direct AEAT submission (mutual TLS, no `ext-soap`) and B2Brouter, behind one API, with an explicit cut-over command (`einvoicing:verifactu-close-chain`) when an issuer switches from one to the other.

Both pull in different code; this metapackage does not depend on either. The comparison reflects their Packagist metadata on 24 November 2026.

## What it pulls in

`intragile/laravel-einvoicing` (the Laravel bridge, MIT), which brings the engine `intragile/php-einvoicing` (MIT) and the rule packages `intragile/einvoicing-rules-en16931` (EUPL-1.2), `intragile/einvoicing-rules-xrechnung` (Apache-2.0) and `intragile/einvoicing-rules-facturx` (Apache-2.0). `intragile/php-einvoicing` is required directly as well. The AEAT schemas are downloaded at install time, never bundled.

The metapackage itself is MIT-licensed and ships no code; each pulled package keeps its own licence.

## Full documentation

Full documentation: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#spanish-verifactu) and the [Spain country notes](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/SPAIN.md).

## License

MIT. See [LICENSE](LICENSE).

---

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)
