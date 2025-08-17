# Disable RAM RGB (G.SKILL Trident Z RGB, DDR4)

> A quick note-to-future-me for turning off RAM LEDs from Linux using the `crgb.cc` approach.

---

## What this program does (summary of the script)

- Enumerates Linux I²C adapters under `/sys/bus/i2c/devices/`, resolves each adapter’s **PCI vendor/device** IDs, and opens the matching `/dev/i2c-*`.
- **Filters for AMD FCH SMBus** adapters (**vendor `0x1022`, device `0x790B`**).
- Probes **I²C address `0x77`** (typical ENE/Aura RAM LED controller on DDR4 G.SKILL Trident Z RGB).
- If the device at `0x77` ACKs, it performs ENE controller writes to switch LEDs **off** and **apply** the change.
- Uses Linux SMBus ioctls for safe access (no raw bit-banging).

> Result: RAM LEDs turn off at boot (you’ll re-run it via cron so it always applies).

---

## My local filenames & layout

- **Source file:** `disable_ram_rgb.cc`  
- **Executable:** `disable_ram_rgb`

> These replace the original author’s filenames; functionality stays the same.

---

## Dependencies (my install)

Debian/Ubuntu:
```bash
sudo apt update
sudo apt install -y g++ i2c-tools
```

---

## Build & run

```bash
# build
sudo g++ disable_ram_rgb.cc -o /root/disable_ram_rgb

# run once now (needs root to open /dev/i2c-*)
sudo /root/disable_ram_rgb
```

If it runs without error and the LEDs go dark, you’re set.

---

## Auto-run at boot (cron)

```bash
sudo crontab -e
# add:
@reboot /root/disable_ram_rgb
```

> Cron runs it at every boot so LEDs stay off without manual steps.

---

## Quick verification / sanity

List I²C buses:
```bash
i2cdetect -l
```

On systems like this one you’ll typically see an AMD PIIX4 SMBus (e.g. `i2c-13`). You can (read-only) probe a controller address if you need to troubleshoot:
```bash
sudo i2cdetect -y -r 13 0x70 0x77
# A hit at 0x77 means the ENE controller is visible on that bus.
```

---

## Notes & gotchas

- The program is deliberately conservative: it only acts on **AMD FCH SMBus** and **address 0x77**. If you move platforms (e.g., Intel) or your RAM maps the controller elsewhere, it’ll safely **do nothing** until you adapt the filter/addr.
- Running as **root** is required to access `/dev/i2c-*`.
- Some boards gate SMBus via ACPI; if you ever can’t see `0x77`, that’s likely why (not a program bug).
- This approach **does not** write a persistent LED profile into the module; using **cron @reboot** is the right pattern.

---

## Credits

- Based on **aditya-r-m’s** `crgb.cc` script (GitHub Gist).  
- Hardware: **G.SKILL TridentZ RGB Series 64GB (4×16GB) DDR4-3600, F4-3600C16Q-64GTZRC**.  
- Chipset path: **AMD FCH SMBus `[1022:790B]` → I²C `0x77` (ENE/Aura)**.

---

## TL;DR

1. Save the script as `disable_ram_rgb.cc`.  
2. `sudo apt install g++ i2c-tools`  
3. `sudo g++ disable_ram_rgb.cc -o /root/disable_ram_rgb`  
4. `sudo /root/disable_ram_rgb` (watch LEDs turn off)  
5. Add `@reboot /root/disable_ram_rgb` to **root crontab**.
