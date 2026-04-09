# intel-nic-nostd

[![Crates.io](https://img.shields.io/crates/v/intel-nic-nostd.svg)](https://crates.io/crates/intel-nic-nostd)
[![Documentation](https://docs.rs/intel-nic-nostd/badge.svg)](https://docs.rs/intel-nic-nostd)
[![License](https://img.shields.io/crates/l/intel-nic-nostd.svg)](https://github.com/suhteevah/intel-nic-nostd)

A `#![no_std]` Intel Ethernet NIC driver for bare-metal Rust, supporting the e1000, e1000e (I219-V), and igc (I225-V/I226-V) families.

## Supported Hardware

| Family | Devices | Speed |
|--------|---------|-------|
| **e1000** | 82540EM, 82545EM | 1 GbE |
| **e1000e** | I219-V, I219-LM (Skylake through Alder Lake) | 1 GbE |
| **igc** | I225-V, I225-LM, I226-V, I226-LM, I226-IT | 2.5 GbE |

The driver auto-detects the NIC variant from PCI vendor/device IDs and applies family-specific quirks (GIO master disable for I225, EEPROM access differences, PHY initialization, packet buffer sizing).

## Features

- **Pure `no_std` + `alloc`** -- no OS dependencies, no `std` required
- **Single driver struct** (`E1000`) for all three families with variant-based dispatch
- **Full initialization sequence** -- software reset, EEPROM MAC read with RAL/RAH fallback, PHY auto-negotiation, descriptor ring setup
- **Legacy descriptor format** -- 256-entry RX/TX rings with 2048-byte DMA buffers
- **Interrupt handling** -- link status change, RX/TX completion, receiver overrun
- **PHY management** -- MII register access via MDIC, auto-negotiation, link status detection
- **Extensive logging** via the `log` crate at trace/debug/info/warn/error levels

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
intel-nic-nostd = "0.1"
```

Initialize from your PCI enumeration:

```rust,ignore
use intel_nic_nostd::{E1000, NicVariant};

// Detect NIC from PCI config space
let variant = NicVariant::from_pci_ids(vendor_id, device_id)
    .expect("unsupported NIC");

// Provide a virtual-to-physical address translation function
fn virt_to_phys(virt: usize) -> u64 {
    // Your page table translation here
    virt as u64
}

// Initialize the NIC (bar0 must be MMIO-mapped, bus mastering enabled)
let mut nic = unsafe {
    E1000::init(bar0_ptr, irq_line, variant, virt_to_phys)
}.expect("NIC init failed");

// Transmit a frame
nic.transmit(&ethernet_frame).unwrap();

// Receive a frame
let mut buf = [0u8; 2048];
if let Ok(Some(len)) = nic.receive(&mut buf) {
    // Process buf[..len]
}

// Handle interrupt (call from your IRQ handler)
let icr = nic.handle_interrupt();
```

## Requirements

- A global allocator (the driver heap-allocates descriptor rings and DMA buffers)
- A virtual-to-physical address translation function
- PCI bus mastering must be enabled before calling `E1000::init`
- The `log` crate facade must be initialized for diagnostic output

## Architecture

```
intel-nic-nostd
  lib.rs         -- NicVariant enum, error types, re-exports
  regs.rs        -- MMIO register offsets, bit definitions, read/write helpers
  descriptors.rs -- RX/TX descriptor structs, ring buffer management
  phy.rs         -- PHY (MII) register access, auto-negotiation, link detection
  e1000.rs       -- Core driver: init, transmit, receive, interrupt handling
  i225.rs        -- I225-V/I226-V specific quirks and register offsets
```

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
- MIT License ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)

at your option.

---

---

---

---

---

## Support This Project

If you find this project useful, consider buying me a coffee! Your support helps me keep building and sharing open-source tools.

[![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg?logo=paypal)](https://www.paypal.me/baal_hosting)

**PayPal:** [baal_hosting@live.com](https://paypal.me/baal_hosting)

Every donation, no matter how small, is greatly appreciated and motivates continued development. Thank you!
