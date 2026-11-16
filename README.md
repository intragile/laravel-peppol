# Laravel Peppol

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-peppol/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-peppol/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-peppol.svg)](https://packagist.org/packages/intragile/laravel-peppol)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-peppol.svg)](https://packagist.org/packages/intragile/laravel-peppol)
[![License](https://img.shields.io/packagist/l/intragile/laravel-peppol.svg)](LICENSE)

Peppol for Laravel: send and receive Peppol BIS Billing 3.0 invoices through an access point, with queued sending, signed webhooks and validation before every send.

This is a thin metapackage: it has no code of its own and pulls in [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) together with the rule package this use case needs, so that you can find it. The full functionality is documented there.

[Magyar nyelvű leírás → README.hu.md](README.hu.md)

## Requirements

- PHP 8.3+ with `dom`, `libxml`, `mbstring`, `zlib`
- Laravel 12 or 13

## Quick start

```bash
composer require intragile/laravel-peppol
```

Install the Peppol BIS validation rules and give the default transport its credentials (Recommand shown; B2Brouter and e-invoice.be are configured the same way, see the full documentation):

```bash
php artisan einvoicing:install-rules
php artisan migrate   # create the two tables
```

```dotenv
EINVOICING_TRANSPORT=recommand
RECOMMAND_API_KEY_ID=key_...
RECOMMAND_API_SECRET=secret_...
RECOMMAND_COMPANY_ID=c_...
```

Check the installation:

```bash
php artisan einvoicing:doctor
```

Usage:

```php
use IntrAgile\EInvoicing\Laravel\Facades\EInvoicing;

$xml = file_get_contents('invoice.xml');   // Peppol BIS Billing 3.0 UBL

// Queued; validated before the send claims the row; one send, never two.
$message = EInvoicing::send($xml, reference: 'INV-2026-0001');

// Use a connection other than the default one:
EInvoicing::send($xml, reference: 'INV-2026-0002', transport: 'b2brouter');

$message->state;   // OutboundState: queued, sending, sent, delivered, ...
```

The invoice carries its own routing (sender, receiver, process) and must be Peppol BIS Billing 3.0 (or an XRechnung sent over Peppol); `send()` refuses anything else. A send whose outcome is unknown is never resent automatically. Receiving invoices (signed webhooks, or polling with `php artisan einvoicing:poll-inbound`) is covered in the [laravel-einvoicing documentation](https://github.com/intragile/laravel-einvoicing#receiving).

## Drivers

| Driver | Status |
|---|---|
| Recommand (`recommand`) | live-tested on the Peppol test network |
| B2Brouter (`b2brouter`) | live-tested against the provider sandbox (simulated network) |
| e-invoice.be (`einvoicebe`) | live-tested against the provider sandbox (simulated network) |

More providers are planned. A test against a provider sandbox that simulates the network proves
the provider's API contract, not delivery over Peppol; only Recommand can reach the Peppol test
network. Provider details are in the [full documentation](https://github.com/intragile/laravel-einvoicing#readme).


## What it pulls in

`intragile/laravel-einvoicing` (the Laravel bridge, MIT), which brings the engine `intragile/php-einvoicing` (MIT) and the rule packages `intragile/einvoicing-rules-en16931` (EUPL-1.2), `intragile/einvoicing-rules-xrechnung` (Apache-2.0) and `intragile/einvoicing-rules-facturx` (Apache-2.0).

The metapackage itself is MIT-licensed and ships no code; each pulled package keeps its own licence.

## Full documentation

Full documentation: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#readme)

## License

MIT. See [LICENSE](LICENSE).

---

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)
