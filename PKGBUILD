# Maintainer: Andzai
# Contributors: Charles Ghislain, Guillaume ALAUX, Daniel J Griffiths, Jason Chu, Geoffroy Carrier, Army, kfgz,
#               Thomas Dziedzic, Dan Serban, jjacky, EasySly

pkgname=oracle-jre8
_major=8
_minor=25
_build=b17
pkgver=${_major}u${_minor}
pkgrel=1
pkgdesc="Oracle Java Runtime Environment"
arch=('i686' 'x86_64')
url=http://www.oracle.com/technetwork/java/javase/downloads/index.html
license=('custom')
depends=('ca-certificates-java' 'desktop-file-utils' 'hicolor-icon-theme' 'java-runtime-common'
         'libxrender' 'libxtst' 'shared-mime-info' 'xdg-utils')
optdepends=('alsa-lib: for basic sound support'
            'gtk2: for Gtk+ look and feel (desktop)'
            'ttf-font: fonts')
provides=("java-runtime=$_major" "java-runtime-headless=$_major" "java-web-start=$_major"
          "java-runtime-jre=$_major" "java-runtime-headless-jre=$_major" "java-web-start-jre=$_major")

# Variables
DLAGENTS=('http::/usr/bin/curl -LC - -b oraclelicense=a -O')
_arch=x64
_arch2=amd64
if [[ $CARCH = i686 ]]; then
  _arch=i586
  _arch2=i386
fi
_jname=jre${_major}
_jvmdir=/usr/lib/jvm/java-$_major-$pkgname/jre

backup=("etc/java-$_jname/$_arch2/server/Xusage.txt"
        "etc/java-$_jname/$_arch2/jvm.cfg"
        "etc/java-$_jname/images/cursors/cursors.properties"
        "etc/java-$_jname/management/jmxremote.access"
        "etc/java-$_jname/management/jmxremote.password.template"
        "etc/java-$_jname/management/management.properties"
        "etc/java-$_jname/management/snmp.acl.template"
        "etc/java-$_jname/security/java.policy"
        "etc/java-$_jname/security/java.security"
        "etc/java-$_jname/security/javaws.policy"
        "etc/java-$_jname/calendars.properties"
        "etc/java-$_jname/content-types.properties"
        "etc/java-$_jname/flavormap.properties"
        "etc/java-$_jname/fontconfig.properties.src"
        "etc/java-$_jname/hijrah-config-umalqura.properties"
        "etc/java-$_jname/javafx.properties"
        "etc/java-$_jname/jvm.hprof.txt"
        "etc/java-$_jname/logging.properties"
        "etc/java-$_jname/net.properties"
        "etc/java-$_jname/psfont.properties.ja"
        "etc/java-$_jname/psfontj2d.properties"
        "etc/java-$_jname/sound.properties")
install=$pkgname.install
_jrename=jre
source=("http://download.oracle.com/otn-pub/java/jdk/$pkgver-$_build/$_jrename-$pkgver-linux-$_arch.tar.gz"
        "policytool-$pkgname.desktop")
md5sums=(`curl -sL ${url/te*}/webfolder/s/digest/${pkgver}checksum.html | grep -Po ">${source##*/}</td><td>\K[^<]*"`
         '5e725c3121159f837c5055399927a2a0') # policytool-$_jname.desktop
## Alternative mirror, if your local one is throttled:
#source[0]="http://ftp.wsisiz.edu.pl/pub/pc/pozyteczne%20oprogramowanie/java/$_jrename-$pkgver-linux-$_arch.gz"

package() {
  cd ${_jrename}1.${_major}.0_${_minor}

  msg2 "Creating directory structure"
  install -d "$pkgdir"/etc/.java/.systemPrefs
  install -d "$pkgdir"/usr/lib/jvm/java-$_major-$pkgname/jre/bin
  install -d "$pkgdir"/usr/lib/mozilla/plugins
  install -d "$pkgdir"/usr/share/licenses/java$_major-$pkgname

  msg2 "Removing redundancies"
  rm    lib/fontconfig.*.bfc
  rm    lib/fontconfig.*.properties.src
  rm    man/ja
  rm -r plugin/

  msg2 "Moving contents"
  mv * "$pkgdir"/$_jvmdir

  # Cd to the new playground
  cd "$pkgdir"/$_jvmdir

  msg2 "Fixing directory structure"
  # Suffix .desktops + icons (sun-java.png -> sun-java-$_jname.png)
  for i in $(find lib/desktop/ -type f); do
    rename -- "." "-$_jname." $i
  done

  # Move .desktops + icons to /usr/share
  mv lib/desktop/* "$pkgdir"/usr/share/
  install -m644 "$srcdir"/*.desktop "$pkgdir"/usr/share/applications/

  # Fix .desktop paths
  sed -e "s|Exec=|&$_jvmdir/bin/|" \
      -e "s|.png|-$_jname.png|" \
  -i "$pkgdir"/usr/share/applications/*

  # Move configs to /etc and link back to /usr: /usr/lib/jvm/java-$_jname/lib -> /etc
  for new_etc_path in ${backup[@]}; do
    # Old location
    old_usr_path="lib/${new_etc_path#*$_jname/}"

    # Move
    install -Dm644 "$old_usr_path" "$pkgdir/$new_etc_path"
    ln -sf "/$new_etc_path" "$old_usr_path"
  done

  # Link NPAPI plugin
  ln -sf $_jvmdir/lib/$_arch2/libnpjp2.so "$pkgdir"/usr/lib/mozilla/plugins/libnpjp2-$_jname.so

  # Replace JKS keystore with 'ca-certificates-java'
  ln -sf /etc/ssl/certs/java/cacerts lib/security/cacerts

  # Suffix man pages
  for i in $(find man/ -type f); do
    mv "${i}" "${i/.1}-${_jname}.1"
  done

  # Move man pages
  mv man/ja_JP.UTF-8/ man/ja
  mv man/ "$pkgdir"/usr/share

  # Move/link licenses
  mv COPYRIGHT LICENSE README *.txt "$pkgdir"/usr/share/licenses/java$_major-$pkgname/
  ln -sf /usr/share/licenses/java$_major-$pkgname/ "$pkgdir"/usr/share/licenses/$pkgname

  msg2 "Enabling copy+paste in unsigned applets"
  # Copy/paste from system clipboard to unsigned Java applets has been disabled since 6u24:
  # - https://blogs.oracle.com/kyle/entry/copy_and_paste_in_java
  # - http://slightlyrandombrokenthoughts.blogspot.com/2011/03/oracle-java-applet-clipboard-injection.html
  _line=$(awk '/permission/{a=NR}; END{print a}' "$pkgdir"/etc/java-$_jname/security/java.policy)
  sed "$_line a\\\\n \
        // (AUR) Allow unsigned applets to read system clipboard, see:\n \
        // - https://blogs.oracle.com/kyle/entry/copy_and_paste_in_java\n \
        // - http://slightlyrandombrokenthoughts.blogspot.com/2011/03/oracle-java-applet-clipboard-injection.html\n \
        permission java.awt.AWTPermission \"accessClipboard\";" \
  -i "$pkgdir"/etc/java-$_jname/security/java.policy
}
