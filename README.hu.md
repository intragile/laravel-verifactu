# Laravel VERI\*FACTU / spanyol számlázási rekordok

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-verifactu/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-verifactu/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-verifactu.svg)](https://packagist.org/packages/intragile/laravel-verifactu)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-verifactu.svg)](https://packagist.org/packages/intragile/laravel-verifactu)
[![License](https://img.shields.io/packagist/l/intragile/laravel-verifactu.svg)](LICENSE)

A spanyol **VERI\*FACTU** Laravelhez: a számláid AEAT-nyilvántartási rekordjainak (`RegistroAlta`, `RegistroAnulacion`) előállítása EN 16931 szerinti számlákból, hash-elése, láncba fűzése és beküldése közvetlenül az AEAT-nak vagy a B2Brouteren keresztül, QR-kóddal és láncellenőrzéssel.

> **Állapot:** még nincs kiadva: az `intragile/laravel-einvoicing` 1.3-at és az `intragile/php-einvoicing` 1.3-at igényli.

> **Release label:** Verifactu live-verified on the B2Brouter sandbox (registered; hash and QR cross-checked); direct AEAT submission verified offline only — it needs a Spanish NIF and a qualified certificate
>
> Magyarul: a VERI\*FACTU élesben ellenőrzött a B2Brouter sandboxán (regisztrált rekordok; a hash és a QR keresztellenőrizve); a közvetlen AEAT-beküldés csak offline ellenőrzött, mert ahhoz spanyol adószám (NIF) és minősített tanúsítvány kell.

Ez egy vékony metacsomag: nincs saját kódja, csak a [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) csomagot és az [`intragile/php-einvoicing`](https://github.com/intragile/php-einvoicing) motort húzza be, hogy a spanyol használati eset név szerint megtalálható legyen. A teljes funkcionalitás a [laravel-einvoicing README](https://github.com/intragile/laravel-einvoicing#spanish-verifactu) **Spanish VERI\*FACTU** szakaszában van dokumentálva (magyarul: [Spanyol VERI\*FACTU](https://github.com/intragile/laravel-einvoicing/blob/main/README.hu.md#spanyol-verifactu)).

[English documentation → README.md](README.md)

## Amit a spanyol folyamat lefed

- **Rekordok:** minden számlához egy `RegistroAlta`, a motor modelljének EN 16931 szerinti számlájából előállítva, a helyesbítő számlákat és az adómentes tételeket is beleértve. A közvetlen AEAT driverrel minden sztornóhoz egy `RegistroAnulacion`, minden javításhoz egy új regisztráció; B2Brouter módban a bridge nem sztornóz és nem javít (a `cancel()` és a `correct()` kivételt dob), a sztornókat a B2Brouterben kell kezelni.
- **Lánc:** minden rekord SHA-256 `Huella`-ja, kibocsátónként és telepítésenként láncba fűzve az adatbázisban; a lánc egy másik számlázó rendszer rekordjaiból is folytatható, és az `einvoicing:verifactu-verify-chain` ellenőrzi.
- **QR-kód:** a közvetlen AEAT driverrel az AEAT ellenőrző URL-je a rekord előállításakor azonnal visszajön, így a számla rögtön nyomtatható. B2Brouter módban a rekord függőben van, és a `qr_url` üres, amíg a B2Brouter regisztrált rekordja be nem kapcsolódik; ekkor a sorban ott a B2Brouter által nyomtatott QR URL és a hash, és elsül a `VerifactuRecordRegistered` esemény.
- **Közvetlen AEAT-beküldés:** sorba állítva, az AEAT forgalomszabályozását betartva, kölcsönös TLS-sel, a kibocsátó minősített tanúsítványával.
- **B2Brouter:** az AEAT felőli oldalt a B2Brouter is átveheti, aszinkron módon; az általa visszaadott rekordot a saját hash-ünkkel, láncunkkal és QR-kódunkkal vetjük össze.
- **Doctor:** az `einvoicing:doctor` ellenőrzi a beállításokat, a sémákat, a tanúsítványt és a láncokat.

## Hatókör: csak VERI\*FACTU mód

Nem része, mindegyik egy későbbi kiadásra vagy connectorra marad:

- **a nem VERI\*FACTU mód** (XAdES-aláírású helyi rekordok, az eseménynapló és annak lánca, a 6 óránkénti összesítő esemény, a lánc önellenőrzése és a *requerimiento* végpont);
- **a TicketBAI** (a baszk foral területek) és **Navarra**, amelyeknek saját rendszerük van;
- **a Facturae 3.2.x / FACe** (számlák a közszférának);
- **az SII**: az SII hatálya alá tartozó adózók kívül esnek a VERI\*FACTU-n;
- **a Crea y Crece B2B e-számlázási kötelezettség** (Real Decreto 238/2026, Orden HAC/1028/2026);
- **a lapozós `ConsultaFactuSistemaFacturacion` export**: csak egyetlen számla lekérdezése érhető el.

**Határidők.** A hatályos szabály (Real Decreto-ley 15/2025) szerint a VERI\*FACTU 2027. január 1-jétől a társasági adó alanyaira, 2027. július 1-jétől mindenki másra vonatkozik. 2026. október 5-én a minisztérium bejelentette, hogy mindkettőt 2028 októberére tolja (**bejelentve, nem kihirdetve**: 2026. november 23-ig a BOE-ben semmi nem jelent meg). A csomag működése nem függ attól, melyik dátum lesz érvényes.

## Nincs tanúsítás, nincs nyilatkozat

Ez a csomag egy komponens, nem tanúsított számlázó rendszer (SIF). A számlázó rendszer gyártója *declaración responsable*-t ír alá arról, hogy a rendszer megfelel az RRSIF-nek (Real Decreto 1007/2023, Orden HAC/1177/2024). **Az IntrAgile nem lép fel gyártóként, nem ír alá nyilatkozatot, és nem ad hozzá sablont.** A könyvtárak a rekordok előállítását, a `Huella`-t és a láncot, a rekord XML-t, a QR URL-t és az AEAT SOAP klienst valósítják meg. Az alkalmazásodnak továbbra is garantálnia kell:

- minden számla és rekordja elválaszthatatlan összekapcsolását (nincs számla rekord nélkül);
- hogy a rekordokat soha nem törlik és nem módosítják;
- hogy a rekordokat azonnal elküldik;
- a helyesbítési és sztornófolyamatokat;
- a saját `SistemaInformatico` azonosságát (a gyártó, a rendszer és a telepítés);
- magát a nyilatkozatot, amely ezt a könyvtárat és verzióját komponensként megnevezi.

## Követelmények

- PHP 8.3+ a `dom`, `libxml`, `mbstring`, `openssl` és `zlib` kiterjesztéssel
- Laravel 12 vagy 13
- Közvetlen AEAT-beküldéshez: a kibocsátó (vagy képviselője) minősített tanúsítványa, és egy CA-csomag, amely tartalmazza az **AC RAIZ FNMT-RCM SERVIDORES SEGUROS** FNMT gyökértanúsítványt
- Opcionális: `chillerlan/php-qrcode` ^5 (`composer require chillerlan/php-qrcode`) a QR-kód kirajzolásához, B2Brouter módban pedig a B2Brouter QR-képének visszafejtéséhez a keresztellenőrzés 5. lépésében; a visszafejtéshez `ext-gd` vagy `ext-imagick` kell, ezek nélkül a QR URL-t a paraméterei alapján vetjük össze

## Telepítés

A kiadás után:

```bash
composer require intragile/laravel-verifactu
php artisan vendor:publish --tag=einvoicing-config   # config/einvoicing.php
php artisan einvoicing:install-rules                 # a Peppol szabályok, amelyeket a doctor ellenőriz
php artisan einvoicing:verifactu-install-schemas     # letölti az AEAT sémákat (sosem csomagoljuk be)
php artisan migrate                                  # a lánc-, forgalomszabályozási és rekordtáblák
php artisan einvoicing:doctor
```

Környezeti kulcsok (az `einvoicing.verifactu` beállítást töltik ki):

```dotenv
VERIFACTU_ENVIRONMENT=test               # test (AEAT előéles) vagy production; production csak ha így állítod
VERIFACTU_DRIVER=aeat                    # aeat (közvetlen beküldés) vagy b2brouter
VERIFACTU_ISSUER_NIF=B12345674           # egyetlen kibocsátó; többet a config/einvoicing.php fogad
VERIFACTU_ISSUER_NAME="Mi Empresa S.L."

# A rekordokat előállító számlázó szoftver (SistemaInformatico): a tiéd, sosem az IntrAgile.
VERIFACTU_PRODUCER_NAME="Your Software S.L."
VERIFACTU_PRODUCER_NIF=A39200019
VERIFACTU_SYSTEM_NAME=YourInvoicing
VERIFACTU_SYSTEM_ID=01
VERIFACTU_SYSTEM_VERSION=2.4.1
VERIFACTU_INSTALLATION_NUMBER=0001
VERIFACTU_SYSTEM_VERIFACTU_ONLY=true
VERIFACTU_SYSTEM_MULTIPLE_TAXPAYERS_POSSIBLE=false
VERIFACTU_SYSTEM_MULTIPLE_TAXPAYERS_INDICATOR=false

# Közvetlen AEAT-beküldés (aeat driver)
VERIFACTU_AEAT_CERTIFICATE=/run/secrets/aeat.p12
VERIFACTU_AEAT_PASSWORD=...
# VERIFACTU_CA_BUNDLE=/etc/ssl/aeat-ca.pem   # tartalmaznia kell az FNMT gyökértanúsítványt

# B2Brouter (b2brouter driver): az einvoicing.transports egyik b2brouter transportjának neve
# VERIFACTU_B2BROUTER_TRANSPORT=b2brouter
```

Számla regisztrálása és – a közvetlen AEAT driverrel – QR-kódjának azonnali nyomtatása; a rekord a queue-n át megy be:

```php
use IntrAgile\EInvoicing\Laravel\Facades\EInvoicing;

$record = EInvoicing::verifactu()->register($invoice);   // a core modell EN 16931 szerinti számlája

$record->qr_url;   // AEAT driver: az AEAT ellenőrző URL-je, amelyet a számlára kell nyomtatni
                   // B2Brouter mód: null a VerifactuRecordRegistered eseményig
```

Kezdd a `VERIFACTU_ENVIRONMENT=test` beállítással: a rekordok csak `production` környezetben mennek az AEAT éles szolgáltatásába. A sztornózást és a helyesbítést (csak a közvetlen AEAT driverrel), a queue-t és a forgalomszabályozást, a B2Brouter módot, a lánc folytatását, az eseményeket, a parancsokat és a doctort a [Spanish VERI\*FACTU szakasz](https://github.com/intragile/laravel-einvoicing#spanish-verifactu) írja le; a rekordleképezést, a láncot, a QR-kódot és a transportokat a [spanyol országjegyzet](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/SPAIN.md) (angolul).

## Miben más

- **[`squareetlabs/laravel-verifactu`](https://packagist.org/packages/squareetlabs/laravel-verifactu)** Laravel csomag VERI\*FACTU rekordokhoz (Laravel 10–12), amely `ext-soap`-ot igényel.
- **[`aichadigital/lara-verifactu`](https://packagist.org/packages/aichadigital/lara-verifactu)** Laravel csomag a VERI\*FACTU-hoz (Laravel 12 és 13), amely `ext-soap`-ot igényel.

Ez a csomag három dologban más:

- **EN 16931 motor alatta:** a rekordok ugyanabból az EN 16931 számlamodellből készülnek, amelyet az `intragile/php-einvoicing` UBL és CII formátumban olvas, ír és validál, és Peppolon küld, így egyetlen számlaobjektum a többi országot is kiszolgálja.
- **AEAT tesztvektorok tesztként:** a hash reprodukálja az AEAT hivatalos tesztvektorait, minden előállított rekordot a hivatalos AEAT sémák ellen ellenőrzünk, és a B2Brouter sandboxában előállított rekordok fixture-ként megmaradnak és újra ellenőrződnek.
- **Két transport:** közvetlen AEAT-beküldés (kölcsönös TLS, `ext-soap` nélkül) és B2Brouter, egyetlen API mögött, kifejezett átállási paranccsal (`einvoicing:verifactu-close-chain`), ha egy kibocsátó egyikről a másikra vált.

A két csomag más kódot hoz; ez a metacsomag egyiktől sem függ. Az összevetés a 2026. november 24-i Packagist-adataikon alapul.

## Amit behúz

`intragile/laravel-einvoicing` (a Laravel-integráció, MIT), amely hozza a `intragile/php-einvoicing` motort (MIT) és a szabálycsomagokat: `intragile/einvoicing-rules-en16931` (EUPL-1.2), `intragile/einvoicing-rules-xrechnung` (Apache-2.0) és `intragile/einvoicing-rules-facturx` (Apache-2.0). Az `intragile/php-einvoicing` közvetlenül is követelmény. Az AEAT sémák telepítéskor töltődnek le, sosem csomagoljuk be őket.

A metacsomag maga MIT licencű és nem szállít kódot; minden behúzott csomag a saját licencét tartja meg.

## Teljes dokumentáció

Teljes dokumentáció: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#spanish-verifactu) és a [spanyol országjegyzet](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/SPAIN.md).

## Licenc

MIT. Lásd: [LICENSE](LICENSE).

---

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)
