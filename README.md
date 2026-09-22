# tuxedo-drivers-cctl-mirror

Kernel driver sources for **[cctl](https://github.com/bhusann/cctl)** — the Linux
control CLI (profiles, fans, keyboard backlight, battery thresholds, GPU MUX) for the
**Colorful Evol P15**, a Clevo/TUXEDO-based laptop.

This repository is the **single source of truth** for the driver stack `cctl` manages:
a readable source tree plus the pre-packed archive `cctl` installs.

## Contents

| Path | What it is |
|---|---|
| [`drivers/`](./drivers) | Readable driver sources: `clevo_acpi`, `tuxedo_keyboard`, `tuxedo_io` (+ install script [`drivers/driverinstall.sh`](./drivers/driverinstall.sh)) |
| [`drivers.tar.gz`](./drivers.tar.gz) | The same tree as a deterministic tarball — this is the file `cctl` downloads and installs |

Raw download URL (used by `cctl drivers-install`):

```
https://raw.githubusercontent.com/bhusann/tuxedo-drivers-cctl-mirror/main/drivers.tar.gz
```

Every GitHub **release of cctl** attaches this same `drivers.tar.gz` (fetched straight
from here, never rebuilt), so binary + matching drivers are always available from one
place: https://github.com/bhusann/cctl/releases

## How `cctl` consumes this repo

`cctl drivers-install` downloads the tarball (or uses one placed beside the binary),
verifies it against a **sha256 baked into the cctl binary** *before* extraction, and
then runs the DKMS install. A tarball that does not match the baked hash is refused.

Because of that pin, whenever anything under `drivers/` changes here:

1. Re-pack with the exact deterministic flags
   (`tar --sort=name --owner=0 --group=0 --numeric-owner --mtime="UTC 2026-09-22"`
   + the documented excludes) so the same tree always yields the same bytes,
2. update `DRIVERS_SHA256` and bump `CCTL_MICROVERSION` in cctl,
3. push this repo and cut a new cctl release (the release workflow fails loudly if
   the mirror tarball and the baked hash drift apart).

## Usage

### Via cctl (recommended)

```bash
sudo cctl drivers-install     # resolves/verifies sources itself
sudo cctl drivers-install     # interactive menu also offers uninstall
```

### Direct (bypass cctl)

```bash
git clone https://github.com/bhusann/tuxedo-drivers-cctl-mirror
cd tuxedo-drivers-cctl-mirror
sudo drivers/driverinstall.sh --install      # DKMS install + modprobe config
sudo drivers/driverinstall.sh --uninstall    # unload modules + DKMS remove
drivers/driverinstall.sh --status            # check state (no root needed)
```

## Credits & License

The driver code is derived from the **[TUXEDO Linux driver project](https://github.com/tuxedocomputers/tuxedo-drivers)**:
Copyright (c) 2018–2026 TUXEDO Computers GmbH and contributors. This mirror packages
those drivers — curated for the Colorful Evol P15 — alongside the `cctl` tool.

The driver code in this repository is licensed under
**[GPL-2.0-or-later](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)** — see
[`drivers/LICENSE`](./drivers/LICENSE) for details. The MIT license that covers the
`cctl` tool itself does **not** apply to the files under `drivers/`.
