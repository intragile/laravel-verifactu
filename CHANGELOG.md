# Changelog

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project adheres to
[Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.0.0] - 2026-11-24

### Added

- A metapackage that requires `intragile/laravel-einvoicing` (`^1.3`) and `intragile/php-einvoicing` (`^1.3`), so that the Spanish VERI\*FACTU use case is found by its name. It ships no code.
- An install CI job on PHP 8.3, 8.4, 8.5 and 8.6 × Laravel 12 and 13 that installs the metapackage into a fresh Laravel application, installs the AEAT schemas and runs the doctor.
- Release label: Verifactu live-verified on the B2Brouter sandbox (registered; hash and QR cross-checked); direct AEAT submission verified offline only — it needs a Spanish NIF and a qualified certificate

---

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)
