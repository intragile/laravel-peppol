# Laravel Peppol

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-peppol/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-peppol/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-peppol.svg)](https://packagist.org/packages/intragile/laravel-peppol)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-peppol.svg)](https://packagist.org/packages/intragile/laravel-peppol)
[![License](https://img.shields.io/packagist/l/intragile/laravel-peppol.svg)](LICENSE)

Peppol Laravelhez: Peppol BIS Billing 3.0 számlák küldése és fogadása access pointon keresztül, sorba állított küldéssel, aláírt webhookokkal és validációval minden küldés előtt.

Ez egy vékony metacsomag: nincs saját kódja, csak a [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) csomagot húzza be, hogy könnyen megtaláld; a szabálycsomagokat az integráció hozza. A teljes funkcionalitás ott van dokumentálva.

[English documentation → README.md](README.md)

## Követelmények

- PHP 8.3+ a `dom`, `libxml`, `mbstring` és `zlib` kiterjesztéssel
- Laravel 12 vagy 13

## Telepítés

```bash
composer require intragile/laravel-peppol
```

Telepítsd a Peppol BIS validációs szabályokat, és add meg az alapértelmezett transport hozzáférési adatait (a Recommand látható; a B2Brouter és az e-invoice.be ugyanígy állítható be, lásd a teljes dokumentációt):

```bash
php artisan einvoicing:install-rules
php artisan migrate   # létrehozza a két táblát
```

```dotenv
EINVOICING_TRANSPORT=recommand
RECOMMAND_API_KEY_ID=key_...
RECOMMAND_API_SECRET=secret_...
RECOMMAND_COMPANY_ID=c_...
```

Ellenőrzés:

```bash
php artisan einvoicing:doctor
```

Használat:

```php
use IntrAgile\EInvoicing\Laravel\Facades\EInvoicing;

$xml = file_get_contents('invoice.xml');   // Peppol BIS Billing 3.0 UBL

// Queued; validated before the send claims the row; one send, never two.
$message = EInvoicing::send($xml, reference: 'INV-2026-0001');

// Use a connection other than the default one:
EInvoicing::send($xml, reference: 'INV-2026-0002', transport: 'b2brouter');

$message->state;   // OutboundState: queued, sending, sent, delivered, ...
```

A számla maga hordozza az útválasztást (feladó, címzett, folyamat), és Peppol BIS Billing 3.0 (vagy Peppolon küldött XRechnung) kell legyen; a `send()` minden mást elutasít. Egy ismeretlen kimenetelű küldést a csomag soha nem küld újra magától. A számlák fogadása (aláírt webhookok, vagy lekérdezés a `php artisan einvoicing:poll-inbound` paranccsal) a [laravel-einvoicing dokumentációjában](https://github.com/intragile/laravel-einvoicing#fogadás) van leírva.

## Driverek

| Driver | Állapot |
|---|---|
| Recommand (`recommand`) | élesben tesztelve a Peppol teszthálózaton |
| B2Brouter (`b2brouter`) | élesben tesztelve a szolgáltatói sandbox ellen (szimulált hálózat) |
| e-invoice.be (`einvoicebe`) | élesben tesztelve a szolgáltatói sandbox ellen (szimulált hálózat) |

További szolgáltatók tervezettek. A szimulált hálózatú szolgáltatói sandbox elleni teszt a
szolgáltató API-szerződését bizonyítja, nem a Peppolon történő kézbesítést; a Peppol
teszthálózatot csak a Recommand éri el. A szolgáltatók részletei a [teljes
dokumentációban](https://github.com/intragile/laravel-einvoicing#readme) vannak.


## Amit behúz

`intragile/laravel-einvoicing` (a Laravel-integráció, MIT), amely hozza a `intragile/php-einvoicing` motort (MIT) és a szabálycsomagokat: `intragile/einvoicing-rules-en16931` (EUPL-1.2), `intragile/einvoicing-rules-xrechnung` (Apache-2.0) és `intragile/einvoicing-rules-facturx` (Apache-2.0).

A metacsomag maga MIT licencű és nem szállít kódot; minden behúzott csomag a saját licencét tartja meg.

## Teljes dokumentáció

Teljes dokumentáció: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#readme)

## Licenc

MIT. Lásd: [LICENSE](LICENSE).

---

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)
