# Amlogic
# Kernel Source Maintainer: Tobetter
# Contributor: Spikerguy <shareahack@hotmail.com>

pkgbase=linux-odroid
_commit=73626777fec89467c4a1494858888b081b24b34e
_srcname=linux-${_commit}
_kernelname=${pkgbase#linux}
_desc="Kernel for Amlogic Devices"
pkgver=5.18.5
pkgrel=1
arch=('aarch64')
url="https://github.com/tobetter/linux/tree/odroid-5.18.y"
license=('GPL2')
makedepends=('xmlto' 'ccache' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'gcc' 'clang' 'llvm' 'llvm-libs')
options=('!strip')
replaces=('linux-vim')
source=("https://github.com/tobetter/linux/archive/${_commit}.tar.gz"
        'https://gitlab.manjaro.org/manjaro-arm/packages/core/linux-khadas/-/raw/master/0065-add-ugoos-device.patch'
        'config'
        'config.anbox'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook')
md5sums=('1ce3185ef67306feeee3886a432327b8'
         '1b92d7617e60d3c525a4b18ab4351185'
         '2d4dcd4e190dac6c1828d5d5771919c4'
         'b8d23199cfc2e9992c4a8644559bfea6'
         'fbb7f2695efe0c83265cad1c5e6f0a81'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         '3dc88030a8f2f5a5f97266d99b149f77')

prepare() {
    #sed -i s/'EXTRAVERSION = -rc7'/'EXTRAVERSION ='/ "${_srcname}"/Makefile
    cd "${srcdir}/${_srcname}"

    # add upstream patch
    #patch -Np1 -i "${srcdir}/patch-${pkgver}"
    patch -Np1 -i "${srcdir}/0065-add-ugoos-device.patch"

    # Manjaro-ARM patches
    # Bootsplash patches

    cat "${srcdir}/config" > ./.config
    # Add anbox config
    # from https://gitlab.manjaro.org/packages/core/linux59/-/blob/b2d83e6150ff07cad3ad0256530aba41b7f9fd16/PKGBUILD
    # line 132 - because changes in the configuration file will be erased..
    cat "${srcdir}/config.anbox" >> ./.config

    # add pkgrel to extraversion
    sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

    # don't run depmod on 'make install'. We'll do this ourselves in packaging
    sed -i '2iexit 0' scripts/depmod.sh
  
    #make menuconfig
	#
    #cp ./.config "${srcdir}/config"
}

build() {
  cd "${srcdir}/${_srcname}"

  # get kernel version
  make -j $(nproc --all) HOSTCC="ccache clang" HOSTCXX="ccache clang++" CC="ccache gcc" CXX="ccache g++" prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #$cp ./.config /var/tmp/${pkgbase}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  # build!
  unset LDFLAGS

  make -j $(nproc --all) HOSTCC="ccache clang" HOSTCXX="ccache clang++" CC="ccache gcc" CXX="ccache g++" ${MAKEFLAGS} Image Image.gz modules
  # Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
  make -j $(nproc --all) HOSTCC="ccache clang" HOSTCXX="ccache clang++" CC="ccache gcc" CXX="ccache g++" ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'kmod' 'mkinitcpio>=0.7' 'fbset' 'uboot-tools')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")
  conflicts=('linux')
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=${pkgname}.install

  cd "${srcdir}/${_srcname}"

  KARCH=arm64

  # get kernel version
  #_kernver="$pkgver-$pkgrel-MANJARO-ARM"
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make -j $(nproc --all) CC="ccache gcc" CXX="ccache g++" INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make -j $(nproc --all) CC="ccache gcc" CXX="ccache g++" INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs" dtbs_install
  cp arch/$KARCH/boot/Image{,.gz} "${pkgdir}/boot"

   # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"

}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')
  cd ${_srcname}
  local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

  install -Dt "${_builddir}" -m644 Makefile .config Module.symvers
  install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

  mkdir "${_builddir}/.tmp_versions"

  cp -t "${_builddir}" -a include scripts

  install -Dt "${_builddir}/arch/${KARCH}" -m644 arch/${KARCH}/Makefile
  install -Dt "${_builddir}/arch/${KARCH}/kernel" -m644 arch/${KARCH}/kernel/asm-offsets.s 
  #arch/$KARCH/kernel/module.lds

  cp -t "${_builddir}/arch/${KARCH}" -a arch/${KARCH}/include

  install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
  install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

  # http://bugs.archlinux.org/task/13146
  install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # http://bugs.archlinux.org/task/20402
  install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # add xfs and shmem for aufs building
  mkdir -p "${_builddir}"/{fs/xfs,mm}

  # copy in Kconfig files
  find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

  # remove unneeded architectures
  local _arch
  for _arch in "${_builddir}"/arch/*/; do
    [[ ${_arch} == */${KARCH}/ ]] && continue
    rm -r "${_arch}"
  done

  # remove files already in linux-docs package
  rm -r "${_builddir}/Documentation"

  # remove now broken symlinks
  find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

  # Fix permissions
  chmod -R u=rwX,go=rX "${_builddir}"

  # strip scripts directory
  local _binary _strip
  while read -rd '' _binary; do
    case "$(file -bi "${_binary}")" in
      *application/x-sharedlib*)  _strip="${STRIP_SHARED}"   ;; # Libraries (.so)
      *application/x-archive*)    _strip="${STRIP_STATIC}"   ;; # Libraries (.a)
      *application/x-executable*) _strip="${STRIP_BINARIES}" ;; # Binaries
      *) continue ;;
    esac
    /usr/bin/strip ${_strip} "${_binary}"
  done < <(find "${_builddir}/scripts" -type f -perm -u+w -print0 2>/dev/null)
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
 

