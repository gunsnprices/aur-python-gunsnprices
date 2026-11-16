# python-gunsnprices (AUR package)

Arch User Repository (AUR) package for [`gunsnprices`](https://pypi.org/project/gunsnprices/), the
Python library and command-line tool that parses firearm and ammunition product titles into
brand, model, caliber, magazine capacity, barrel length and rounds per box.

- Upstream source: https://github.com/gunsnprices/gunsnprices-python
- Documentation: https://gunsnprices.com/developers
- AUR page (after the first upload): https://aur.archlinux.org/packages/python-gunsnprices

**This GitHub repository is a CI mirror.** The AUR git repository
(`ssh://aur@aur.archlinux.org/python-gunsnprices.git`) holds only `PKGBUILD`, `.SRCINFO` and
`LICENSE`. This repository has the same files plus a GitHub Actions workflow that builds and tests
them on Arch Linux. Status: not yet uploaded to the AUR. See [PUBLISHING.md](PUBLISHING.md).

## Install (once the package is on the AUR)

    git clone https://aur.archlinux.org/python-gunsnprices.git
    cd python-gunsnprices
    makepkg -si

Or with an AUR helper, for example `paru -S python-gunsnprices` or `yay -S python-gunsnprices`.

## Use

    $ gunsnprices "Glock 19 Gen 5 9mm 15rd"
    $ echo "Federal American Eagle 9mm 115gr FMJ 50rd" | gunsnprices --price 18.99

```python
from gunsnprices import parse_title

info = parse_title("Sig Sauer P365 XL 9mm 3.7in 12rd Pistol")
print(info.to_dict())
```

## Package

| Field | Value |
|---|---|
| `pkgname` | `python-gunsnprices` (Arch naming for a Python library) |
| `pkgver` | 0.1.2 |
| `arch` | `any` (pure Python) |
| `url` | https://gunsnprices.com/developers |
| `license` | `MIT AND CC-BY-4.0` (code MIT, bundled model and caliber data CC BY 4.0) |
| `depends` | `python` (no other runtime dependencies) |
| `makedepends` | `python-build`, `python-hatchling`, `python-installer`, `python-wheel` |
| `checkdepends` | `python-pytest` |
| `source` | GitHub tag `v0.1.2` of gunsnprices-python, with `sha256sums` and `b2sums` |

`build()` makes a wheel with `python -m build --wheel --no-isolation`. `check()` installs that
wheel into a temporary venv (`--system-site-packages`) and runs the upstream test suite (1,191
tests) against it. `package()` installs the wheel with `python -m installer` and copies `LICENSE`
and `LICENSE-DATA` to `/usr/share/licenses/python-gunsnprices/`.

## CI

[`.github/workflows/archlinux.yml`](.github/workflows/archlinux.yml) runs in the
`archlinux:latest` container on every push:

1. `makepkg --printsrcinfo` must match the committed `.SRCINFO`.
2. `namcap PKGBUILD` (the job fails on errors).
3. `makepkg --syncdeps --rmdeps --cleanbuild` as an unprivileged user. Only the declared
   dependencies are installed, and `check()` runs the test suite.
4. `namcap` on the built `.pkg.tar.zst`.
5. `pacman -U` of the package, then `gunsnprices --help`, `--version`, a firearm title and an
   ammunition title with `--price`, and an import of the module from `/usr/lib/python3.*`.

[`.github/workflows/arch-command.yml`](.github/workflows/arch-command.yml) is a manual helper
that runs one shell command in `archlinux:latest`. It is used to answer the `pacman` question on
the AUR registration form (see PUBLISHING.md).

## Licenses

The packaging files in this repository (`PKGBUILD`, `.SRCINFO`, workflows, documentation) are
under the [0BSD](LICENSE) license, as recommended by the AUR submission guidelines. The packaged
software is MIT (code) and CC BY 4.0 (model and caliber data, credit "GunsNPrices
(gunsnprices.com)"), as declared upstream.
