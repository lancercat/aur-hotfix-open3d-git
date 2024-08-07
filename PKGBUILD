# Maintainer: Pellegrino Prevete (tallero) <pellegrinoprevete@gmail.com>
# Contributor: JunYoung Gwak <aur@jgwak.com>
# Contributor: Dylon Edwards <deltaecho at archlinux dot us>

_py="python"
_pkg="open3d"
pkgbase="${_pkg}-git"
pkgname=(
  {,"${_py}-"}"${_pkg}-git"
)
pkgver=r4081.fcf98ee45
pkgrel=1
pkgdesc="A Modern Library for 3D Data Processing"
arch=('x86_64')
url="http://www.${_pkg}.org"
_url="https://github.com/isl-org/Open3D"
license=('MIT')
depends=(
  eigen
  glew
  glfw-x11
  jsoncpp
  libjpeg-turbo
  libpng
  mesa
  xorg-server-devel
)
optdepends=(
  'openmp: Multiprocess support'
  'pybind11: System pybind11 support'
  "${_py}: Python support"
  'jupyter-notebook: Jupyter notebook support'
)
makedepends=(
  cmake
  automake
  autoconf
  git
  "${_py}-setuptools"
)
source=(
  "${pkgbase}::git+${_url}.git"
  fixass.patch
  fix_3rdparty_path.patch
)
sha256sums=(
  'SKIP'
  'SKIP'
  '3bf6b79fd075b356a5c2d86a557e0bc6e6df0e84d53c2077d2c6685641838d81'
)

function pkgver() {
  cd "${pkgbase}"
  printf "r%s.%s" "$(git rev-list --count HEAD)" \
                  "$(git rev-parse --short HEAD)"
}

function prepare() {
    cd "${srcdir}/${pkgbase}"
    sed -i '/^CMAKE_ARGS.*/a -DCMAKE_INSTALL_DEFAULT_LIBDIR="lib"' \
           "3rdparty/libjpeg-turbo/libjpeg-turbo.cmake"
    patch --forward \
          --strip=1 \
          --input="${srcdir}/fixass.patch"
    git submodule update --init \
                         --recursive

    find . -name "CMakeLists.txt" -exec sed -i 's/-Werror//g' {} \;
    grep --files-with-matches -r "_FORTIFY_SOURCE" | xargs -I {} sed -i 's/_FORTIFY_SOURCE=[0-9]/""/g' {}
    mkdir build
}

function build() {
  local _cmake_opts=(
    -DCMAKE_INSTALL_PREFIX="${pkgdir}/usr"
    -DBUILD_SHARED_LIBS=ON
    -DCMAKE_BUILD_TYPE=Release
    -DBUILD_ISPC_MODULE=OFF
    -G "Unix Makefiles"
    -DCMAKE_INSTALL_PREFIX=/usr
    -DBUILD_SHARED_LIBS=ON
    -DCMAKE_VERBOSE_MAKEFILE=ON
    -DCMAKE_MODULE_PATH=/usr/lib/cmake/OpenVDB
    -DCMAKE_FIND_ROOT_PATH=/opt/intel/oneapi/tbb/latest/lib/cmake/tbb
    -DUSE_SYSTEM_ASSIMP=ON
    -DUSE_SYSTEM_CURL=ON
    -DUSE_SYSTEM_BLAS=OFF
    -DBUILD_SYCL_MODULE=OFF
    -DUSE_SYSTEM_EIGEN3=ON
    -DUSE_SYSTEM_EMBREE=ON
    -DUSE_SYSTEM_TBB=ON
    -DOPEN3D_USE_ONEAPI_PACKAGES=ON
    -DoneDPL_DIR=/opt/intel/oneapi/dpl/latest/lib/cmake/oneDPL
    -DMKL_DIR=/opt/intel/oneapi/mkl/latest/lib/cmake/mkl
    -DUSE_SYSTEM_FMT=ON
    -DUSE_SYSTEM_GLEW=ON
    -DUSE_SYSTEM_GLFW=ON
    -DUSE_SYSTEM_GOOGLETEST=ON
    -DUSE_SYSTEM_JPEG=ON
    -DUSE_SYSTEM_NANOFLANN=ON
    -DUSE_SYSTEM_PNG=ON
    -DUSE_SYSTEM_PYBIND11=ON
    -DUSE_SYSTEM_QHULLCPP=ON
    -DUSE_SYSTEM_VTK=ON
    -DUSE_SYSTEM_JSONCPP=OFF
    -DBUILD_LIBREALSENSE=ON
  )
  cd "${srcdir}/${pkgbase}/build"
  cmake .. "${_cmake_opts[@]}"
  make -j10
}

function package_open3d-git() {
  optdepends=(
    'openmp: Multiprocess support'
  )
  provides=(
    "${_pkg}=${pkgver}"
  )
  conflicts=(
    "${_pkg}"
  )
  cd "${srcdir}/${pkgbase}/build"
  make install
}

function package_python-open3d-git() {
  depends+=(
    "${_pkg}-git"
    "${_py}"
  )
  optdepends=(
    'jupyter-notebook: Jupyter notebook support'
    'openmp: Multiprocess support'
    'pybind11: System pybind11 support'
  )
  provides=(
    "${_py}-${_pkg}=${pkgver}"
    "${_py}-py3d=${pkgver}"
  )
  conflicts=(
      "${_py}-${_pkg}"
      "${_py}-py3d"
  )
  cd "${srcdir}/${pkgbase}/build"
  make "${_py}-package"
  cd "${srcdir}/${pkgbase}/build/lib/${_py}_package"
  "${_py}" setup.py install --root="${pkgdir}" \
                            --optimize=1
}
