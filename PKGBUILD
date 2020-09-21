# Maintainer: Felix Schindler <aur at felixschindler dot net>
# Contributor: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>

pkgname=('llvm8' 'llvm8-libs' 'llvm8-ocaml')
pkgver=8.0.1
pkgrel=1
_ocaml_ver=4.11.0
arch=('x86_64')
url="https://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('cmake' 'gcc8' 'ninja' 'libffi' 'libedit' 'ncurses' 'libxml2'
             "ocaml=$_ocaml_ver" 'ocaml-ctypes' 'ocaml-findlib'
             'python-sphinx' 'python-recommonmark')
options=('staticlibs')
source=(https://github.com/llvm/llvm-project/releases/download/llvmorg-$pkgver/llvm-$pkgver.src.tar.xz
        llvm-config.h)
sha256sums=('44787a6d02f7140f145e2250d56c9f849334e11f9ae379827510ed72f12b75e7'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')

prepare() {
  cd "$srcdir/llvm-$pkgver.src"
  mkdir build
}

build() {
  cd "$srcdir/llvm-$pkgver.src/build"

  export CC=cc-8
  export CXX=c++-8

  cmake .. -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DLLVM_HOST_TRIPLE=$CHOST \
    -DLLVM_BUILD_LLVM_DYLIB=ON \
    -DLLVM_LINK_LLVM_DYLIB=ON \
    -DLLVM_INSTALL_UTILS=ON \
    -DLLVM_ENABLE_RTTI=ON \
    -DLLVM_ENABLE_FFI=ON \
    -DLLVM_BUILD_TESTS=ON \
    -DLLVM_BUILD_DOCS=ON \
    -DLLVM_ENABLE_SPHINX=ON \
    -DLLVM_ENABLE_DOXYGEN=OFF \
    -DSPHINX_WARNINGS_AS_ERRORS=OFF \
    -DFFI_INCLUDE_DIR=$(pkg-config --variable=includedir libffi) \
    -DLLVM_BINUTILS_INCDIR=/usr/include
  ninja all ocaml_doc
}

check() {
  cd "$srcdir/llvm-$pkgver.src/build"
  ninja check
}

package_llvm8() {
  _pkgname=llvm
  pkgdesc="Collection of modular and reusable compiler and toolchain technologies (8.x)"
  depends=('llvm8-libs' 'perl')
  optdepends=('python-setuptools: for using lit (LLVM Integrated Tester)')
  provides=("llvm=$pkgver")
  conflicts=('llvm')

  cd "$srcdir/llvm-$pkgver.src/build"

  DESTDIR="$pkgdir" ninja install

  # Include lit for running lit-based tests in other projects
  pushd ../utils/lit
  python3 setup.py install --root="$pkgdir" -O1
  popd

  # Remove documentation sources
  rm -r "$pkgdir"/usr/share/doc/$_pkgname/html/{_sources,.buildinfo}

  # The runtime libraries go into llvm-libs
  mv -f "$pkgdir"/usr/lib/lib{LLVM,LTO}*.so* "$srcdir"
  mv -f "$pkgdir"/usr/lib/LLVMgold.so "$srcdir"

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/ocaml.{lib,doc}
  mv "$pkgdir/usr/lib/ocaml" "$srcdir/ocaml.lib"
  mv "$pkgdir/usr/share/doc/$_pkgname/ocaml-html" "$srcdir/ocaml.doc"

  if [[ $CARCH == x86_64 ]]; then
    # Needed for multilib (https://bugs.archlinux.org/task/29951)
    # Header stub is taken from Fedora
    mv "$pkgdir/usr/include/llvm/Config/llvm-config"{,-64}.h
    cp "$srcdir/llvm-config.h" "$pkgdir/usr/include/llvm/Config/llvm-config.h"
  fi

  install -Dm644 ../LICENSE.TXT "$pkgdir/usr/share/licenses/$_pkgname/LICENSE"
}

package_llvm8-libs() {
  _pkgname=llvm-libs
  pkgdesc="LLVM runtime libraries (8.x)"
  depends=('gcc-libs' 'zlib' 'libffi' 'libedit' 'ncurses' 'libxml2')
  provides=("llvm-libs=$pkgver")
  conflicts=('llvm-libs')

  install -d "$pkgdir/usr/lib"
  cp -P \
    "$srcdir"/lib{LLVM,LTO}*.so* \
    "$srcdir"/LLVMgold.so \
    "$pkgdir/usr/lib/"

  # Symlink LLVMgold.so from /usr/lib/bfd-plugins
  # https://bugs.archlinux.org/task/28479
  install -d "$pkgdir/usr/lib/bfd-plugins"
  ln -s ../LLVMgold.so "$pkgdir/usr/lib/bfd-plugins/LLVMgold.so"

  install -Dm644 "$srcdir/llvm-$pkgver.src/LICENSE.TXT" \
    "$pkgdir/usr/share/licenses/$_pkgname/LICENSE"
}

package_llvm8-ocaml() {
  _pkgname=llvm-ocaml
  pkgdesc="OCaml bindings for LLVM (8.x)"
  depends=('llvm8' "ocaml=$_ocaml_ver" 'ocaml-ctypes')
  provides=("llvm-ocaml=$pkgver")
  conflicts=('llvm-ocaml')

  install -d "$pkgdir"/{usr/lib,usr/share/doc/$_pkgname}
  cp -a "$srcdir/ocaml.lib" "$pkgdir/usr/lib/ocaml"
  cp -a "$srcdir/ocaml.doc" "$pkgdir/usr/share/doc/$_pkgname/html"

  install -Dm644 "$srcdir/llvm-$pkgver.src/LICENSE.TXT" \
    "$pkgdir/usr/share/licenses/$_pkgname/LICENSE"
}

# vim:set ts=2 sw=2 et:
