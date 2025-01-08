# ft8042

My fork of keyboard and mouse driver for phytium platform notebooks.

## How to build

```bash
make
```

If you prefer to build with LLVM (which is a must on
[eweOS](https://os.ewe.moe)), run

```bash
make LLVM=1 LLVM_IAS=1
```

Specify `KERNELDIR` to build for a kernel different from the current running
one.

## dkms build

***NOT MAINTAINED: I don't have a dkms-available system on hand.***

```bash
sudo cp ft8042 /usr/src/ft8042-0.1/
sudo dkms add -m ft8042
sudo dkms build -m ft8042 -v 0.1
sudo dkms install -m ft8042 -v 0.1
```

## Maintaining Status

I have a Phytium D2000 laptop
(`Greatwall GW-XXXXXX-XXX/GW-001N1C-FTF, BIOS KunLun BIOS V4.0 03/08/2021`)
for daily testing purpose, thus I'll keep the module compatible with the
latest stable kernel.

## Why not upstream it?

In one sentence: IT'S HORRIBLE.

For those who're interested in the details, or willing to take the upstream
work (a REAL HERO!),

- There is NO documentation.
- This out-of-tree module is apparently a modified version of i8042, which
  implies the controller is mostly compatible with i8042s you could find on
  x86 devices. This is confirmed by applying some modification to the mainline
  i8042 driver, but
- i8042 is old-x86 stuff, existing code doesn't have a good mechanism for
  dynamic detection. A new platform-specific header must be added to the i8042
  codebase.
- On x86 systems, i8042 controllers are usually attached to a LPC bus.
  Identifiers and comments in this driver, the shared IRQ line and disassembled
  DSDT are implying it's also true on these phytium platforms. The LPC bus is
  mapped as MMIO, which I hadn't come up with a good style to describe in
  kernel.
- The 8042 controller is flooding the host with thousands of interrupts. This
  could be reproduced with both this out-of-tree driver and a modified mainline
  kernel. I haven't figured out why.
- I don't know what the write to `LPC_BUS_BASE + 0x7fffff0` means. This may
  imply there're some secret bits for the LPC bus.

The (few) good news is,

- ACPI DSDT does describe the i8042 controller (`KBCI8042`) and LPC bus
  (`PHYT0007`). Comparison with this and the EC driver suggests it's correct.

## Useful Links

- [The EC driver](https://github.com/UEFICharlesZhang/gw-nb-ec)
