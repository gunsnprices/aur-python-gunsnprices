# Publishing `python-gunsnprices` to the AUR

Status (2026-11-16): `PKGBUILD`, `.SRCINFO` and `LICENSE` are ready, and CI is green on
`archlinux:latest` (see "What CI verified" below). Nothing has been pushed to the AUR. No AUR
account exists yet.

Name check on 2026-11-16: `https://aur.archlinux.org/rpc/v5/info?arg[]=python-gunsnprices`
returned `"resultcount":0`, so the name is free. The official repositories have no package
matching `gunsnprices`.

## Rules that apply

| Rule | Source |
|---|---|
| A Python library is named `python-<module>`, all lowercase. | [Python package guidelines](https://manual.archlinux.page/package-guidelines/python/) |
| Build a wheel with `python -m build --wheel --no-isolation` and install it with `python -m installer --destdir="$pkgdir"`. `makedepends` = `python-build python-installer python-wheel` plus the project's build backend (`build-backend = "hatchling.build"` here, so `python-hatchling`). | same |
| Upstream source tarballs are the default (RFC 0020), not PyPI sdists. Here: GitHub tag `v0.1.2`, which contains the tests. | same, "Source" |
| Tests that need the installed package run in a `--system-site-packages` venv with `python -P -m pytest`. | same, "Running tests with an installed package" |
| `url` is "the URL of the official site of the software being packaged". The AUR page shows it as "Upstream URL". https://gunsnprices.com/developers documents this package (`pip install gunsnprices`, PyPI and GitHub links), so it is used instead of the site's home page. | [PKGBUILD#url](https://wiki.archlinux.org/title/PKGBUILD#url) |
| `license` uses SPDX identifiers. A combination is one quoted SPDX expression (`'MIT AND CC-BY-4.0'`). MIT is a license family: its text must be installed to `/usr/share/licenses/$pkgname/`. | [PKGBUILD#license](https://wiki.archlinux.org/title/PKGBUILD#license) |
| Not in the official repositories, no duplicate in the AUR, supports `x86_64` (`arch=('any')` does), no `-git`/`-bin` suffix for a versioned source build. | [AUR submission guidelines](https://wiki.archlinux.org/title/AUR_submission_guidelines#Rules_of_submission) |
| A `# Maintainer: Name <address at domain dot tld>` comment at the top of the `PKGBUILD`. | same |
| Add a `LICENSE` file for the package sources; 0BSD is encouraged (RFC 0040). | same |
| Every commit must include `.SRCINFO` (`makepkg --printsrcinfo > .SRCINFO`). Only the `master` branch is accepted. The git hook also rejects commits that contain subdirectories, so `.github/` and the docs of this repo are not uploaded. | same, "Publishing new package content" |
| Commits are authored with the git name and email, and they are hard to change after the push. Set them per repository. | same |

## 1. Register an AUR account (owner, once)

1. Open https://aur.archlinux.org/register.
2. Fill in:
   - Username: for example `gunsnprices`
   - Email address: `app@gunsnprices.com`. Tick "Hide Email Address" if it should not be shown
     on the profile.
   - SSH Public Key: paste the key from step 2 here, or add it later under "My Account".
3. The form asks a verification question that has to be answered on Arch Linux, for example
   "What is the output of `LC_ALL=C pacman -V|sed -r 's#[0-9]+#NN#g'|md5sum|cut -c1-6`?"
   (the number changes). Without an Arch machine, use this repository's helper workflow while the
   form stays open. Paste the command exactly as shown on the form, with its number, in place of
   the example:

       export GH_TOKEN="$(cat ~/.gunsnprices-secrets/github_token.txt)"
       R=gunsnprices/aur-python-gunsnprices
       gh workflow run arch-command.yml -R $R \
         -f command="LC_ALL=C pacman -V|sed -r 's#[0-9]+#NN#g'|md5sum|cut -c1-6"
       sleep 10
       RUN=$(gh run list -R $R -w arch-command.yml -L 1 --json databaseId -q '.[0].databaseId')
       gh run watch "$RUN" -R $R --exit-status > /dev/null
       gh run view "$RUN" -R $R --log | grep 'Run the command' | tail -n 1 | awk '{print $NF}'
       unset GH_TOKEN

   The last command prints the 6-character answer (a run takes about 40 seconds). The same
   works on github.com: Actions, "Run a command on Arch Linux", "Run workflow". The answer is the
   last line of the "Run the command" step.
4. Submit. The AUR sends an email to the address with a link to set the password.

## 2. SSH key (done on this Mac)

A dedicated key pair was generated for the AUR only (ed25519, no passphrase):

- private key `~/.ssh/aur_gunsnprices` (mode 600, never share it)
- public key `~/.ssh/aur_gunsnprices.pub`, fingerprint
  `SHA256:IFZX093dVHcCqV5bbL696Th3IbdmRGmufr2+3G7igjE`

Public key text for the AUR form ("SSH Public Key" field, at registration or under "My Account"
followed by "Update" with the current password):

    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMbRzxmFe+Hl346zlrc9nm5BNPesDYTOb+pN+5czgt/v gunsnprices-aur

To revoke it, delete the line from "My Account" and remove both files.

## 3. SSH config (done on this Mac)

Appended to `~/.ssh/config` (backup: `~/.ssh/config.bak-20261116`). There was no AUR entry
before.

    Host aur.archlinux.org
        User aur
        IdentityFile ~/.ssh/aur_gunsnprices
        IdentitiesOnly yes

## 4. First connection: check the host key

    ssh aur@aur.archlinux.org help

On the first connection ssh shows the server's key fingerprint. Answer `yes` only if it is one of
the fingerprints listed on https://aur.archlinux.org (home page, "SSH Key Fingerprints"). On
2026-11-16 the page listed:

    Ed25519  SHA256:RFzBCUItH9LZS0cKB5UE6ceAYhBD5C8GeOBip8Z11+4
    ECDSA    SHA256:uTa/0PndEgPZTf76e1DFqXKJEXKsn7m9ivhLQtzGOCI
    RSA      SHA256:5s5cIyReIfNNVGRFdDbe3hdYiI5OelHGpw2rOUud3Q8

If the key is registered, the command prints the list of AUR commands. "Permission denied
(publickey)" means the public key is not saved in the account yet.

## 5. Clone, copy, commit, push

Check that the latest commit on `main` of this repository has a green "Arch Linux" run first:
https://github.com/gunsnprices/aur-python-gunsnprices/actions

    mkdir -p ~/gunsnprices-oss/aur-remote && cd ~/gunsnprices-oss/aur-remote
    git -c init.defaultBranch=master clone ssh://aur@aur.archlinux.org/python-gunsnprices.git
    # expected: "warning: You appear to have cloned an empty repository."
    cd python-gunsnprices
    git config user.name "GunsNPrices"
    git config user.email "app@gunsnprices.com"
    cp ~/gunsnprices-oss/aur-python-gunsnprices/PKGBUILD \
       ~/gunsnprices-oss/aur-python-gunsnprices/.SRCINFO \
       ~/gunsnprices-oss/aur-python-gunsnprices/LICENSE .
    git add PKGBUILD .SRCINFO LICENSE
    git status                     # exactly these three files, no subdirectories
    git commit -m "Initial upload: python-gunsnprices 0.1.2"
    git push origin master

The `-c init.defaultBranch=master` matters: the AUR accepts only `master`.

## 6. Check the result

- https://aur.archlinux.org/packages/python-gunsnprices shows version 0.1.2-1, "Upstream URL:
  https://gunsnprices.com/developers", licenses `MIT AND CC-BY-4.0` and the maintainer.
- `curl -s 'https://aur.archlinux.org/rpc/v5/info?arg[]=python-gunsnprices'` returns
  `"resultcount":1`.
- Optional, to help search on the AUR:

      ssh aur@aur.archlinux.org set-keywords python-gunsnprices firearms guns ammunition ammo caliber parser

## 7. Updating to a new upstream release

1. In `PKGBUILD` of this repository: set `pkgver=X.Y.Z` and `pkgrel=1`.
2. New checksums of `https://github.com/gunsnprices/gunsnprices-python/archive/refs/tags/vX.Y.Z.tar.gz`:

       curl -sSfLo /tmp/gnp.tar.gz https://github.com/gunsnprices/gunsnprices-python/archive/refs/tags/vX.Y.Z.tar.gz
       shasum -a 256 /tmp/gnp.tar.gz
       python3 -c 'import hashlib,sys; print(hashlib.blake2b(open(sys.argv[1],"rb").read()).hexdigest())' /tmp/gnp.tar.gz

   (On Arch: `updpkgsums` from `pacman-contrib`.)
3. Update `.SRCINFO` (on Arch: `makepkg --printsrcinfo > .SRCINFO`; on the Mac edit the version,
   source and checksum lines by hand. CI prints a diff if anything differs).
4. Push to GitHub and wait for a green run. Then copy `PKGBUILD` and `.SRCINFO` into the AUR
   clone and commit (`Update to X.Y.Z`), then push.
5. If only the packaging changes (same `pkgver`), increase `pkgrel` instead. Typo fixes need no
   bump.

## What CI verified

See the "Arch Linux" workflow runs. The job summary of each run lists the namcap output and the
pytest result.
