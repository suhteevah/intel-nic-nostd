# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-02

### Added

- Initial release extracted from ClaudioOS bare-metal kernel
- e1000 (82540EM/82545EM) support for QEMU emulated NICs
- e1000e / I219-V support (Skylake through Alder Lake)
- igc / I225-V / I226-V 2.5GbE support with family-specific quirks
- PCI vendor/device ID auto-detection via `NicVariant::from_pci_ids()`
- Full hardware initialization: reset, EEPROM MAC read, PHY auto-negotiation
- Legacy 256-entry RX/TX descriptor rings with 2048-byte DMA buffers
- Interrupt handling (link status, RX/TX completion, overrun)
- PHY management via MDIC (MII register access, link status detection)
- I225/I226 quirks: GIO master disable, packet buffer sizing, post-link tuning
- Comprehensive `log` crate integration at all severity levels
