# Maintainer: GunsNPrices <app at gunsnprices dot com>

pkgname=python-gunsnprices
_repo=gunsnprices-python
pkgver=0.1.2
pkgrel=1
pkgdesc='Parse firearm and ammunition product titles into brand, model, caliber and capacity'
arch=('any')
url='https://gunsnprices.com/developers'
license=('MIT AND CC-BY-4.0')
depends=('python')
makedepends=(
  'python-build'
  'python-hatchling'
  'python-installer'
  'python-wheel'
)
checkdepends=('python-pytest')
source=("$_repo-$pkgver.tar.gz::https://github.com/gunsnprices/$_repo/archive/refs/tags/v$pkgver.tar.gz")
sha256sums=('45b67122801c5c168caa9f41b93405f30cf7b8922533f1a3045965f19c53ebf0')
b2sums=('caf735459a4ab0bbc0ecf4774572cc7566186c4e20e1f1630a15688f7e357dd5766b50b29b37287687d47971b9834cde1b8279d05ac02816aa0df389cb5ce590')

build() {
  cd "$_repo-$pkgver"
  python -m build --wheel --no-isolation
}

check() {
  cd "$_repo-$pkgver"
  # src layout: run the test suite against the built wheel
  rm -rf test-env
  python -m venv --system-site-packages test-env
  test-env/bin/python -m installer dist/*.whl
  test-env/bin/python -P -m pytest
}

package() {
  cd "$_repo-$pkgver"
  python -m installer --destdir="$pkgdir" dist/*.whl
  # LICENSE: MIT (code); LICENSE-DATA: CC BY 4.0 notice for the model and caliber data
  install -Dm644 LICENSE LICENSE-DATA -t "$pkgdir/usr/share/licenses/$pkgname/"
}
