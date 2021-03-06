# Kernel Build in OpenWrt Deep Dive

## Sequence

![Generic Package Infrastructures](images/buildroot_generic_package_steps.png)

### Download

```
/home/robbie/GitHub/widora/scripts/download.pl "/home/robbie/GitHub/widora/dl" "linux-3.18.29.tar.xz" "b25737a0bc98e80d12200de93f239c28" "" "@KERNEL/linux/kernel/v3.x"
```

### Extract

```
xzcat /home/robbie/GitHub/widora/dl/linux-3.18.29.tar.xz | tar -C /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2 -xf -
```

### Patch

```
rm -rf /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/patches; mkdir -p /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/patches
cp -fpR "/home/robbie/GitHub/widora/target/linux/generic/files"/. /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/
find /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/ -name \*.rej -or -name \*.orig | xargs -r rm -f
Applying /home/robbie/GitHub/widora/target/linux/generic/patches-3.18/000-keep_initrafs_the_default.patch using plaintext: 
patching file init/do_mounts.c

Applying /home/robbie/GitHub/widora/target/linux/generic/patches-3.18/020-ssb_update.patch using plaintext: 
patching file drivers/ssb/pcihost_wrapper.c
patching file drivers/ssb/driver_pcicore.c
patching file drivers/ssb/main.c

...
...
```

### Configure

### Build

### Install

## Makefile Analysis

```
include/download.mk
include/quilt.mk
include/kernel-build.mk
include/kernel-defaults.mk
include/kernel-version.mk
include/kernel.mk
target/linux/<target>
```

### Download

```
include/download.mk

define DownloadMethod/default
	$(SCRIPT_DIR)/download.pl "$(DL_DIR)" "$(FILE)" "$(MD5SUM)" "$(URL_FILE)" $(foreach url,$(URL),"$(url)")
endef
```

### Extract

```
include/kernel-defaults.mk

ifeq ($(strip $(CONFIG_EXTERNAL_KERNEL_TREE)),"")
	ifeq ($(strip $(CONFIG_KERNEL_GIT_CLONE_URI)),"")
		define Kernel/Prepare/Default
		xzcat $(DL_DIR)/$(LINUX_SOURCE) | $(TAR) -C $(KERNEL_BUILD_DIR) $(TAR_OPTIONS)  # -> extract
		$(Kernel/Patch)                                                                 # -> patch
		touch $(LINUX_DIR)/.quilt_used
		endef
	else
		define Kernel/Prepare/Default
		git clone $(KERNEL_GIT_OPTS) $(CONFIG_KERNEL_GIT_CLONE_URI) $(LINUX_DIR)
		endef
	endif
else
	define Kernel/Prepare/Default
	mkdir -p $(KERNEL_BUILD_DIR)
	if [ -d $(LINUX_DIR) ]; then \
		rmdir $(LINUX_DIR); \
	fi
	ln -s $(CONFIG_EXTERNAL_KERNEL_TREE) $(LINUX_DIR)
	endef
endif
```

### Patch

```
include/quilt.mk

kernel_files=$(foreach fdir,$(GENERIC_FILES_DIR) $(FILES_DIR),$(fdir)/.)
define Kernel/Patch/Default
	rm -rf $(PKG_BUILD_DIR)/patches; mkdir -p $(PKG_BUILD_DIR)/patches
	$(if $(kernel_files),$(CP) $(kernel_files) $(LINUX_DIR)/)
	find $(LINUX_DIR)/ -name \*.rej -or -name \*.orig | $(XARGS) rm -f
	$(call PatchDir,$(PKG_BUILD_DIR),$(GENERIC_PATCH_DIR),generic/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR),platform/)
endef
```

### Configure

### Build

### Install

## Full Log Example

```
/home/robbie/GitHub/widora/scripts/download.pl "/home/robbie/GitHub/widora/dl" "linux-3.18.29.tar.xz" "b25737a0bc98e80d12200de93f239c28" "" "@KERNEL/linux/kernel/v3.x"
xzcat /home/robbie/GitHub/widora/dl/linux-3.18.29.tar.xz | tar -C /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2 -xf -
rm -rf /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/patches; mkdir -p /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/patches
cp -fpR "/home/robbie/GitHub/widora/target/linux/generic/files"/. /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/
find /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/ -name \*.rej -or -name \*.orig | xargs -r rm -f
Applying /home/robbie/GitHub/widora/target/linux/generic/patches-3.18/000-keep_initrafs_the_default.patch using plaintext: 
patching file init/do_mounts.c

Applying /home/robbie/GitHub/widora/target/linux/generic/patches-3.18/020-ssb_update.patch using plaintext: 
patching file drivers/ssb/pcihost_wrapper.c
patching file drivers/ssb/driver_pcicore.c
patching file drivers/ssb/main.c

...
...

touch /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/.quilt_used
ln -sf linux-3.18.29 /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux
/home/robbie/GitHub/widora/staging_dir/host/bin/sed -i -e 's/@expr length/@-expr length/' /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/Makefile
touch /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/.prepared
make[3]: Leaving directory `/home/robbie/GitHub/widora/toolchain/kernel-headers'
make[3]: Entering directory `/home/robbie/GitHub/widora/toolchain/kernel-headers'
env
LC_PAPER=zh_CN.UTF-8
XDG_VTNR=7
SCAN_COOKIE=26073
LC_ADDRESS=zh_CN.UTF-8
XDG_SESSION_ID=c2
LC_MONETARY=zh_CN.UTF-8
XDG_GREETER_DATA_DIR=/var/lib/lightdm-data/robbie
M4=/home/robbie/GitHub/widora/staging_dir/host/bin/m4
SELINUX_INIT=YES
SESSION=xubuntu
GPG_AGENT_INFO=/run/user/1000/keyring-qB69ZW/gpg:0:1
GLADE_PIXMAP_PATH=:
TERM=screen
SH_FUNC=. /home/robbie/GitHub/widora/include/shell.sh;
XDG_MENU_PREFIX=xfce-
SHELL=/usr/bin/env bash
MAKEFLAGS=
LC_NUMERIC=zh_CN.UTF-8
WINDOWID=60817412
UPSTART_SESSION=unix:abstract=/com/ubuntu/upstart-session/1000/1570
GNOME_KEYRING_CONTROL=/run/user/1000/keyring-qB69ZW
GIT_CONFIG_PARAMETERS='core.autocrlf=false'
LC_ALL=C
ZSH=/home/robbie/.oh-my-zsh
RELEASE=Chaos Calmer
TARGET_CC_NOCACHE=mipsel-openwrt-linux-uclibc-gcc
ACLOCAL_INCLUDE=
USER=robbie
MAKEOVERRIDES=${-*-command-variables-*-}
LC_TELEPHONE=zh_CN.UTF-8
LD_LIBRARY_PATH=/home/robbie/GitHub/widora/staging_dir/host/lib
TMP_DIR=/home/robbie/GitHub/widora/tmp
CCACHE_DIR=/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/ccache
HOSTCC_WRAPPER=cc
GLADE_MODULE_PATH=:
XDG_SESSION_PATH=/org/freedesktop/DisplayManager/Session0
XDG_SEAT_PATH=/org/freedesktop/DisplayManager/Seat0
TERMCAP=SC|screen|VT 100/ANSI X3.64 virtual terminal:\
	:DO=\E[%dB:LE=\E[%dD:RI=\E[%dC:UP=\E[%dA:bs:bt=\E[Z:\
	:cd=\E[J:ce=\E[K:cl=\E[H\E[J:cm=\E[%i%d;%dH:ct=\E[3g:\
	:do=^J:nd=\E[C:pt:rc=\E8:rs=\Ec:sc=\E7:st=\EH:up=\EM:\
	:le=^H:bl=^G:cr=^M:it#8:ho=\E[H:nw=\EE:ta=^I:is=\E)0:\
	:li#53:co#180:am:xn:xv:LP:sr=\EM:al=\E[L:AL=\E[%dL:\
	:cs=\E[%i%d;%dr:dl=\E[M:DL=\E[%dM:dc=\E[P:DC=\E[%dP:\
	:im=\E[4h:ei=\E[4l:mi:IC=\E[%d@:ks=\E[?1h\E=:\
	:ke=\E[?1l\E>:vi=\E[?25l:ve=\E[34h\E[?25h:vs=\E[34l:\
	:ti=\E[?1049h:te=\E[?1049l:us=\E[4m:ue=\E[24m:so=\E[3m:\
	:se=\E[23m:mb=\E[5m:md=\E[1m:mr=\E[7m:me=\E[m:ms:\
	:Co#8:pa#64:AF=\E[3%dm:AB=\E[4%dm:op=\E[39;49m:AX:\
	:vb=\Eg:G0:as=\E(0:ae=\E(B:\
	:ac=\140\140aaffggjjkkllmmnnooppqqrrssttuuvvwwxxyyzz{{||}}~~..--++,,hhII00:\
	:po=\E[5i:pf=\E[4i:Km=\E[M:k0=\E[10~:k1=\EOP:k2=\EOQ:\
	:k3=\EOR:k4=\EOS:k5=\E[15~:k6=\E[17~:k7=\E[18~:\
	:k8=\E[19~:k9=\E[20~:k;=\E[21~:F1=\E[23~:F2=\E[24~:\
	:F3=\E[1;2P:F4=\E[1;2Q:F5=\E[1;2R:F6=\E[1;2S:\
	:F7=\E[15;2~:F8=\E[17;2~:F9=\E[18;2~:FA=\E[19;2~:kb=:\
	:K2=\EOE:kB=\E[Z:kF=\E[1;2B:kR=\E[1;2A:*4=\E[3;2~:\
	:*7=\E[1;2F:#2=\E[1;2H:#3=\E[2;2~:#4=\E[1;2D:%c=\E[6;2~:\
	:%e=\E[5;2~:%i=\E[1;2C:kh=\E[1~:@1=\E[1~:kH=\E[4~:\
	:@7=\E[4~:kN=\E[6~:kP=\E[5~:kI=\E[2~:kD=\E[3~:ku=\EOA:\
	:kd=\EOB:kr=\EOC:kl=\EOD:km:
SSH_AUTH_SOCK=/run/user/1000/keyring-qB69ZW/ssh
V=s
REVISION=r49389
MAKELEVEL=4
DEFAULTS_PATH=/usr/share/gconf/xubuntu.default.path
SESSION_MANAGER=local/robc-xub14v:@/tmp/.ICE-unix/1944,unix/robc-xub14v:/tmp/.ICE-unix/1944
PAGER=less
MFLAGS=-wr
LSCOLORS=Gxfxcxdxbxegedabagacad
XDG_CONFIG_DIRS=/etc/xdg/xdg-xubuntu:/usr/share/upstart/xdg:/etc/xdg:/etc/xdg
IS_TTY=1
PATH=/home/robbie/GitHub/widora/staging_dir/host/bin:/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin:/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin:/home/robbie/GitHub/widora/staging_dir/host/bin:/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin:/home/robbie/GitHub/widora/staging_dir/host/bin:/home/robbie/GitHub/widora/staging_dir/host/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/git/bin:/home/robbie/.bin:/home/robbie/.bin:/home/robbie/.rvm/bin:/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/git/bin:/home/robbie/.bin:/home/robbie/.bin:/home/robbie/.rvm/bin:/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/git/bin:/home/robbie/.bin:/home/robbie/Tools/gcc-arm-none-eabi-5_4-2016q2/bin:/home/robbie/.bin:/home/robbie/.rvm/bin
DESKTOP_SESSION=xubuntu
TOPDIR=/home/robbie/GitHub/widora
STY=2842.pts-0.robc-xub14v
XML_CATALOG_FILES=/usr/local/etc/xml/catalog
_=/usr/bin/env
LC_IDENTIFICATION=zh_CN.UTF-8
HOSTCC_NOCACHE=gcc
PWD=/home/robbie/GitHub/widora/toolchain/kernel-headers
JOB=dbus
STAGING_PREFIX=/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2
EDITOR=vim
LANG=C
PKG_CONFIG_LIBDIR=/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/lib/pkgconfig
GNOME_KEYRING_PID=1563
OPENWRT_BUILD=1
MANDATORY_PATH=/usr/share/gconf/xubuntu.mandatory.path
GDM_LANG=en_US
LC_MEASUREMENT=zh_CN.UTF-8
IM_CONFIG_PHASE=1
NO_TRACE_MAKE=make V=ss
BUILD_VARIANT=
GDMSESSION=xubuntu
MAKE_JOBSERVER=
SESSIONTYPE=
SHLVL=5
HOME=/home/robbie
UPDATE_ZSH_DAYS=30
XDG_SEAT=seat0
LANGUAGE=en_US
GREP_OPTIONS=
BISON_PKGDATADIR=/home/robbie/GitHub/widora/staging_dir/host/share/bison
CFLAGS=
DYLD_LIBRARY_PATH=/home/robbie/GitHub/widora/staging_dir/host/lib
UPSTART_INSTANCE=
STAGING_DIR=/home/robbie/GitHub/widora/staging_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2
LESS=-R
UPSTART_EVENTS=started xsession
LOGNAME=robbie
GCC_HONOUR_COPTS=s
WINDOW=7
TARGET_CXX_NOCACHE=mipsel-openwrt-linux-uclibc-g++
DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-y9i7bEDn1r
XDG_DATA_DIRS=/usr/share/xubuntu:/usr/share/xfce4:/usr/local/share/:/usr/share/:/usr/share
LC_CTYPE=en_US.UTF-8
PKG_CONFIG_PATH=/home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/lib/pkgconfig
OPENWRTVERSION=Chaos Calmer (r49389)
PKG_CONFIG=/home/robbie/GitHub/widora/staging_dir/host/bin/pkg-config
HOST_EXTRACFLAGS=
INSTANCE=
TEXTDOMAIN=im-config
UPSTART_JOB=startxfce4
DISPLAY=:0.0
XDG_RUNTIME_DIR=/run/user/1000
GLADE_CATALOG_PATH=:
XDG_CURRENT_DESKTOP=XFCE
LC_TIME=zh_CN.UTF-8
TEXTDOMAINDIR=/usr/share/locale/
EXTRA_OPTIMIZATION=-fno-caller-saves
COLORTERM=xfce4-terminal
LC_NAME=zh_CN.UTF-8
XAUTHORITY=/home/robbie/.Xauthority
yes '' | make -C /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29 HOSTCFLAGS="-O2 -I/home/robbie/GitHub/widora/staging_dir/host/include -I/home/robbie/GitHub/widora/staging_dir/host/usr/include -Wall -Wmissing-prototypes -Wstrict-prototypes" ARCH=mips CC="mipsel-openwrt-linux-uclibc-gcc" CFLAGS="-Os -pipe -mno-branch-likely -mips32r2 -mtune=24kec -mdsp -fno-caller-saves -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -msoft-float" CROSS_COMPILE=mipsel-openwrt-linux-uclibc- KBUILD_HAVE_NLS=no CONFIG_SHELL=bash oldconfig
make[4]: Entering directory `/home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29'
	HOSTCC  scripts/basic/fixdep
	HOSTCC  scripts/kconfig/conf.o
	SHIPPED scripts/kconfig/zconf.tab.c
	SHIPPED scripts/kconfig/zconf.lex.c
	SHIPPED scripts/kconfig/zconf.hash.c
	HOSTCC  scripts/kconfig/zconf.tab.o
	HOSTLD  scripts/kconfig/conf
scripts/kconfig/conf --oldconfig Kconfig
net/sched/Kconfig:43: warning: menuconfig statement without prompt
/boot/config-3.19.0-64-generic:1385:warning: symbol value 'm' invalid for OPENVSWITCH_GRE
/boot/config-3.19.0-64-generic:1386:warning: symbol value 'm' invalid for OPENVSWITCH_VXLAN
/boot/config-3.19.0-64-generic:1387:warning: symbol value 'm' invalid for OPENVSWITCH_GENEVE
/boot/config-3.19.0-64-generic:1944:warning: symbol value 'm' invalid for BMP085
/boot/config-3.19.0-64-generic:2606:warning: symbol value 'm' invalid for STMMAC_PLATFORM
/boot/config-3.19.0-64-generic:6365:warning: symbol value 'm' invalid for COMEDI_PCI_DRIVERS
/boot/config-3.19.0-64-generic:6420:warning: symbol value 'm' invalid for COMEDI_PCMCIA_DRIVERS
/boot/config-3.19.0-64-generic:6428:warning: symbol value 'm' invalid for COMEDI_USB_DRIVERS
#
# using defaults found in /boot/config-3.19.0-64-generic
#
	*
	* Restart config...
	*
	*
	* Machine selection
	*
System type
	1. Alchemy processor based machines (MIPS_ALCHEMY) (NEW)
	2. Texas Instruments AR7 (AR7) (NEW)
	3. Atheros AR71XX/AR724X/AR913X based boards (ATH79) (NEW)
	4. Broadcom BCM47XX based boards (BCM47XX) (NEW)
	5. Broadcom BCM63XX based boards (BCM63XX) (NEW)
	6. Cobalt Server (MIPS_COBALT) (NEW)
	7. DECstations (MACH_DECSTATION) (NEW)
	8. Jazz family of machines (MACH_JAZZ) (NEW)
	9. Ingenic JZ4740 based machines (MACH_JZ4740) (NEW)
	10. Lantiq based platforms (LANTIQ) (NEW)
	11. LASAT Networks platforms (LASAT) (NEW)
	12. Loongson family of machines (MACH_LOONGSON) (NEW)
	13. Loongson 1 family of machines (MACH_LOONGSON1) (NEW)
	14. MIPS Malta board (MIPS_MALTA) (NEW)
	15. MIPS SEAD3 board (MIPS_SEAD3) (NEW)
	16. NEC EMMA2RH Mark-eins board (NEC_MARKEINS) (NEW)
	17. NEC VR4100 series based machines (MACH_VR41XX) (NEW)
	18. NXP STB220 board (NXP_STB220) (NEW)
	19. NXP 225 board (NXP_STB225) (NEW)
	20. PMC-Sierra MSP chipsets (PMC_MSP) (NEW)
	21. Ralink based machines (RALINK) (NEW)
  > 22. SGI IP22 (Indy/Indigo2) (SGI_IP22) (NEW)
	23. SGI IP27 (Origin200/2000) (SGI_IP27) (NEW)
	24. SGI IP28 (Indigo2 R10k) (SGI_IP28) (NEW)
	25. SGI IP32 (O2) (SGI_IP32) (NEW)
	26. Sibyte BCM91120C-CRhine (SIBYTE_CRHINE) (NEW)
	27. Sibyte BCM91120x-Carmel (SIBYTE_CARMEL) (NEW)
	28. Sibyte BCM91125C-CRhone (SIBYTE_CRHONE) (NEW)
	29. Sibyte BCM91125E-Rhone (SIBYTE_RHONE) (NEW)
	30. Sibyte BCM91250A-SWARM (SIBYTE_SWARM) (NEW)
	31. Sibyte BCM91250C2-LittleSur (SIBYTE_LITTLESUR) (NEW)
	32. Sibyte BCM91250E-Sentosa (SIBYTE_SENTOSA) (NEW)
	33. Sibyte BCM91480B-BigSur (SIBYTE_BIGSUR) (NEW)
	34. SNI RM200/300/400 (SNI_RM) (NEW)
	35. Toshiba TX39 series based machines (MACH_TX39XX) (NEW)
	36. Toshiba TX49 series based machines (MACH_TX49XX) (NEW)
	37. Mikrotik RB532 boards (MIKROTIK_RB532) (NEW)
	38. Cavium Networks Octeon SoC based boards (CAVIUM_OCTEON_SOC) (NEW)
	39. Netlogic XLR/XLS based systems (NLM_XLR_BOARD) (NEW)
	40. Netlogic XLP based systems (NLM_XLP_BOARD) (NEW)
	41. Para-Virtualized guest system (MIPS_PARAVIRT) (NEW)
choice[1-41]: *
* Linux/mips 3.18.29 Kernel Configuration
*
*
* Machine selection
*
System type
	1. Alchemy processor based machines (MIPS_ALCHEMY)
	2. Texas Instruments AR7 (AR7)
	3. Atheros AR71XX/AR724X/AR913X based boards (ATH79)
	4. Broadcom BCM47XX based boards (BCM47XX)
	5. Broadcom BCM63XX based boards (BCM63XX)
	6. Cobalt Server (MIPS_COBALT)
	7. DECstations (MACH_DECSTATION)
	8. Jazz family of machines (MACH_JAZZ)
	9. Ingenic JZ4740 based machines (MACH_JZ4740)
	10. Lantiq based platforms (LANTIQ)
	11. LASAT Networks platforms (LASAT)
	12. Loongson family of machines (MACH_LOONGSON)
	13. Loongson 1 family of machines (MACH_LOONGSON1)
	14. MIPS Malta board (MIPS_MALTA)
	15. MIPS SEAD3 board (MIPS_SEAD3)
	16. NEC EMMA2RH Mark-eins board (NEC_MARKEINS)
	17. NEC VR4100 series based machines (MACH_VR41XX)
	18. NXP STB220 board (NXP_STB220)
	19. NXP 225 board (NXP_STB225)
	20. PMC-Sierra MSP chipsets (PMC_MSP)
	21. Ralink based machines (RALINK)
  > 22. SGI IP22 (Indy/Indigo2) (SGI_IP22)
	23. SGI IP27 (Origin200/2000) (SGI_IP27)
	24. SGI IP28 (Indigo2 R10k) (SGI_IP28)
	25. SGI IP32 (O2) (SGI_IP32)
	26. Sibyte BCM91120C-CRhine (SIBYTE_CRHINE)
	27. Sibyte BCM91120x-Carmel (SIBYTE_CARMEL)
	28. Sibyte BCM91125C-CRhone (SIBYTE_CRHONE)
	29. Sibyte BCM91125E-Rhone (SIBYTE_RHONE)
	30. Sibyte BCM91250A-SWARM (SIBYTE_SWARM)
	31. Sibyte BCM91250C2-LittleSur (SIBYTE_LITTLESUR)
	32. Sibyte BCM91250E-Sentosa (SIBYTE_SENTOSA)
	33. Sibyte BCM91480B-BigSur (SIBYTE_BIGSUR)
	34. SNI RM200/300/400 (SNI_RM)
	35. Toshiba TX39 series based machines (MACH_TX39XX)
	36. Toshiba TX49 series based machines (MACH_TX49XX)
	37. Mikrotik RB532 boards (MIKROTIK_RB532)
	38. Cavium Networks Octeon SoC based boards (CAVIUM_OCTEON_SOC)
	39. Netlogic XLR/XLS based systems (NLM_XLR_BOARD)
	40. Netlogic XLP based systems (NLM_XLP_BOARD)
	41. Para-Virtualized guest system (MIPS_PARAVIRT)
choice[1-41]: 22
OpenWrt specific image command line hack (IMAGE_CMDLINE_HACK) [N/y] (NEW) 
Endianness selection
> 1. Big endian (CPU_BIG_ENDIAN) (NEW)
choice[1]: 1
ARC console support (ARC_CONSOLE) [N/y] (NEW) 
*
* CPU selection
*
CPU type
> 1. R4x00 (CPU_R4X00) (NEW)
  2. R5000 (CPU_R5000) (NEW)
choice[1-2]: *
* Kernel type
*
Kernel code model
  1. 32-bit kernel (32BIT) (NEW)
> 2. 64-bit kernel (64BIT)
choice[1-2?]: KVM Guest Kernel (KVM_GUEST) [Y/n/?] y
Count/Compare Timer Frequency (MHz) (KVM_GUEST_TIMER_FREQ) [100] (NEW) 
Kernel page size
> 1. 4kB (PAGE_SIZE_4KB) (NEW)
  2. 16kB (PAGE_SIZE_16KB) (NEW)
  3. 64kB (PAGE_SIZE_64KB) (NEW)
choice[1-3]: Maximum zone order (FORCE_MAX_ZONEORDER) [11] (NEW) 
SmartMIPS or microMIPS ASE support
> 1. None (CPU_NEEDS_NO_SMARTMIPS_OR_MICROMIPS) (NEW)
choice[1]: 1
Allow for balloon memory compaction/migration (BALLOON_COMPACTION) [Y/n/?] y
Allow for memory compaction (COMPACTION) [Y/?] y
Page migration (MIGRATION) [Y/?] y
Enable KSM for page merging (KSM) [Y/n/?] y
Low address space to protect from user allocation (DEFAULT_MMAP_MIN_ADDR) [65536] 65536
Transparent Hugepage Support (TRANSPARENT_HUGEPAGE) [Y/n/?] y

...
...


*
* Virtualization
*
Virtualization (VIRTUALIZATION) [Y/?] y
Host kernel accelerator for virtio net (VHOST_NET) [M/n/?] m
VHOST_SCSI TCM fabric driver (VHOST_SCSI) [M/n/?] m
#
# configuration written to .config
#
make[4]: Leaving directory `/home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29'
mkdir -p /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-dev
make -C /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29 HOSTCFLAGS="-O2 -I/home/robbie/GitHub/widora/staging_dir/host/include -I/home/robbie/GitHub/widora/staging_dir/host/usr/include -Wall -Wmissing-prototypes -Wstrict-prototypes" ARCH=mips CC="mipsel-openwrt-linux-uclibc-gcc" CFLAGS="-Os -pipe -mno-branch-likely -mips32r2 -mtune=24kec -mdsp -fno-caller-saves -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -msoft-float" CROSS_COMPILE=mipsel-openwrt-linux-uclibc- KBUILD_HAVE_NLS=no CONFIG_SHELL=bash INSTALL_HDR_PATH="/home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-dev/" headers_install
make[4]: Entering directory `/home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29'
	CHK     include/generated/uapi/linux/version.h
	UPD     include/generated/uapi/linux/version.h
	WRAP    arch/mips/include/generated/asm/cputime.h
	WRAP    arch/mips/include/generated/asm/current.h
	WRAP    arch/mips/include/generated/asm/dma-contiguous.h
	WRAP    arch/mips/include/generated/asm/emergency-restart.h
	WRAP    arch/mips/include/generated/asm/hash.h
	WRAP    arch/mips/include/generated/asm/irq_work.h
	WRAP    arch/mips/include/generated/asm/local64.h
	WRAP    arch/mips/include/generated/asm/mcs_spinlock.h
	WRAP    arch/mips/include/generated/asm/mutex.h
	WRAP    arch/mips/include/generated/asm/parport.h
	WRAP    arch/mips/include/generated/asm/percpu.h
	WRAP    arch/mips/include/generated/asm/preempt.h
	WRAP    arch/mips/include/generated/asm/scatterlist.h
	WRAP    arch/mips/include/generated/asm/sections.h
	WRAP    arch/mips/include/generated/asm/segment.h
	WRAP    arch/mips/include/generated/asm/serial.h
	WRAP    arch/mips/include/generated/asm/trace_clock.h
	WRAP    arch/mips/include/generated/asm/ucontext.h
	WRAP    arch/mips/include/generated/asm/user.h
	WRAP    arch/mips/include/generated/asm/xor.h
	WRAP    arch/mips/include/generated/uapi/asm/auxvec.h
	WRAP    arch/mips/include/generated/uapi/asm/ipcbuf.h
	HOSTCC  scripts/unifdef
	INSTALL include/asm-generic (35 files)
	INSTALL include/drm (18 files)
	INSTALL include/linux/byteorder (2 files)
	INSTALL include/linux/caif (2 files)
	INSTALL include/linux/can (5 files)
	INSTALL include/linux/dvb (8 files)
	INSTALL include/linux/hdlc (1 file)
	INSTALL include/linux/hsi (1 file)
	INSTALL include/linux/isdn (1 file)
	INSTALL include/linux/mmc (1 file)
	INSTALL include/linux/netfilter/ipset (4 files)
	INSTALL include/linux/netfilter (86 files)
	INSTALL include/linux/netfilter_arp (2 files)
	INSTALL include/linux/netfilter_bridge (17 files)
	INSTALL include/linux/netfilter_ipv4 (9 files)
	INSTALL include/linux/netfilter_ipv6 (12 files)
	INSTALL include/linux/nfsd (5 files)
	INSTALL include/linux/raid (2 files)
	INSTALL include/linux/spi (1 file)
	INSTALL include/linux/sunrpc (1 file)
	INSTALL include/linux/tc_act (8 files)
	INSTALL include/linux/tc_ematch (4 files)
	INSTALL include/linux/usb (11 files)
	INSTALL include/linux/wimax (1 file)
	INSTALL include/linux (404 files)
	INSTALL include/misc (1 file)
	INSTALL include/mtd (5 files)
	INSTALL include/rdma (6 files)
	INSTALL include/scsi/fc (4 files)
	INSTALL include/scsi (3 files)
	INSTALL include/sound (11 files)
	INSTALL include/video (3 files)
	INSTALL include/xen (4 files)
	INSTALL include/uapi (0 file)
INSTALL include/asm (38 files)
make[4]: Leaving directory `/home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29'
cp -fpR /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/arch/mips/include/asm/asm.h /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/arch/mips/include/asm/regdef.h /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/arch/mips/include/asm/asm-eva.h /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-dev/include/asm/
touch /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/.configured
touch /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/.built
make[3]: Leaving directory `/home/robbie/GitHub/widora/toolchain/kernel-headers'
make[3]: Entering directory `/home/robbie/GitHub/widora/toolchain/kernel-headers'
cp -fpR /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-dev/* /home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/
mkdir -p /home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/stamp
touch /home/robbie/GitHub/widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/linux-3.18.29/.built
touch /home/robbie/GitHub/widora/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/stamp/.linux_installed
make[3]: Leaving directory `/home/robbie/GitHub/widora/toolchain/kernel-headers'



***********



make[4]: Entering directory `/home/robbie/GitHub/widora/target/linux/ramips'
rm -rf /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688
mkdir -p /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688
xzcat /home/robbie/GitHub/widora/dl/linux-3.18.29.tar.xz | tar -C /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688 -xf -
rm -rf /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/patches; mkdir -p /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/patches
cp -fpR "/home/robbie/GitHub/widora/target/linux/generic/files"/. "./files"/. /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/
find /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/ -name \*.rej -or -name \*.orig | xargs -r rm -f
touch /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.quilt_used
touch /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.prepared
if [ -s "/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/patches/series" ]; then (cd "/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29"; if quilt --quiltrc=- next >/dev/null 2>&1; then quilt --quiltrc=- push -a; else quilt --quiltrc=- top >/dev/null 2>&1; fi ); fi
Applying patch generic/000-keep_initrafs_the_default.patch
patching file init/do_mounts.c

...
...


Applying patch platform/0001-MIPS-ralink-add-verbose-pmu-info.patch
patching file arch/mips/ralink/mt7620.c

Applying patch platform/0002-MIPS-ralink-add-a-helper-for-reading-the-ECO-version.patch
patching file arch/mips/include/asm/mach-ralink/mt7620.h

...
...


Applying patch platform/999-pci-reset.patch
patching file arch/mips/ralink/reset.c

Now at patch platform/999-pci-reset.patch
touch "/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.quilt_checked"
/home/robbie/GitHub/widora/scripts/kconfig.pl  + /home/robbie/GitHub/widora/target/linux/generic/config-3.18 /home/robbie/GitHub/widora/target/linux/ramips/mt7688/config-3.18 > /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.target
awk '/^(#[[:space:]]+)?CONFIG_KERNEL/{sub("CONFIG_KERNEL_","CONFIG_");print}' /home/robbie/GitHub/widora/.config >> /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.target
echo "# CONFIG_KALLSYMS_EXTRA_PASS is not set" >> /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.target
echo "# CONFIG_KALLSYMS_ALL is not set" >> /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.target
echo "# CONFIG_KALLSYMS_UNCOMPRESSED is not set" >> /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.target
/home/robbie/GitHub/widora/scripts/metadata.pl kconfig /home/robbie/GitHub/widora/tmp/.packageinfo /home/robbie/GitHub/widora/.config 3.18 > /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.override
/home/robbie/GitHub/widora/scripts/kconfig.pl 'm+' '+' /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.target /dev/null /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.override > /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config
mv /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.old
grep -v INITRAMFS /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config.old > /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config
echo 'CONFIG_INITRAMFS_SOURCE=""' >> /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config
rm -rf /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/modules
export MAKEFLAGS= ; [ -d /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/user_headers ] || make -C /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29 HOSTCFLAGS="-O2 -I/home/robbie/GitHub/widora/staging_dir/host/include -I/home/robbie/GitHub/widora/staging_dir/host/usr/include -Wall -Wmissing-prototypes -Wstrict-prototypes" CROSS_COMPILE="mipsel-openwrt-linux-uclibc-" ARCH="mips" KBUILD_HAVE_NLS=no CONFIG_SHELL="bash" V='' CC="mipsel-openwrt-linux-uclibc-gcc" INSTALL_HDR_PATH=/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/user_headers headers_install
make[5]: Entering directory `/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29'
	CHK     include/generated/uapi/linux/version.h
	UPD     include/generated/uapi/linux/version.h
	HOSTCC  scripts/basic/fixdep
	WRAP    arch/mips/include/generated/asm/cputime.h
	WRAP    arch/mips/include/generated/asm/current.h
	WRAP    arch/mips/include/generated/asm/dma-contiguous.h
	WRAP    arch/mips/include/generated/asm/emergency-restart.h
	WRAP    arch/mips/include/generated/asm/hash.h
	WRAP    arch/mips/include/generated/asm/irq_work.h
	WRAP    arch/mips/include/generated/asm/local64.h
	WRAP    arch/mips/include/generated/asm/mcs_spinlock.h
	WRAP    arch/mips/include/generated/asm/mutex.h
	WRAP    arch/mips/include/generated/asm/parport.h
	WRAP    arch/mips/include/generated/asm/percpu.h
	WRAP    arch/mips/include/generated/asm/preempt.h
	WRAP    arch/mips/include/generated/asm/scatterlist.h
	WRAP    arch/mips/include/generated/asm/sections.h
	WRAP    arch/mips/include/generated/asm/segment.h
	WRAP    arch/mips/include/generated/asm/serial.h
	WRAP    arch/mips/include/generated/asm/trace_clock.h
	WRAP    arch/mips/include/generated/asm/ucontext.h
	WRAP    arch/mips/include/generated/asm/user.h
	WRAP    arch/mips/include/generated/asm/xor.h
	WRAP    arch/mips/include/generated/uapi/asm/auxvec.h
	WRAP    arch/mips/include/generated/uapi/asm/ipcbuf.h
	HOSTCC  scripts/unifdef
	INSTALL include/asm-generic (35 files)
	INSTALL include/drm (18 files)
	INSTALL include/linux/byteorder (2 files)
	INSTALL include/linux/caif (2 files)
	INSTALL include/linux/can (5 files)
	INSTALL include/linux/dvb (8 files)
	INSTALL include/linux/hdlc (1 file)
	INSTALL include/linux/hsi (1 file)
	INSTALL include/linux/isdn (1 file)
	INSTALL include/linux/mmc (1 file)
	INSTALL include/linux/netfilter/ipset (4 files)
	INSTALL include/linux/netfilter (86 files)
	INSTALL include/linux/netfilter_arp (2 files)
	INSTALL include/linux/netfilter_bridge (17 files)
	INSTALL include/linux/netfilter_ipv4 (9 files)
	INSTALL include/linux/netfilter_ipv6 (12 files)
	INSTALL include/linux/nfsd (5 files)
	INSTALL include/linux/raid (2 files)
	INSTALL include/linux/spi (1 file)
	INSTALL include/linux/sunrpc (1 file)
	INSTALL include/linux/tc_act (8 files)
	INSTALL include/linux/tc_ematch (4 files)
	INSTALL include/linux/usb (11 files)
	INSTALL include/linux/wimax (1 file)
	INSTALL include/linux (404 files)
	INSTALL include/misc (1 file)
	INSTALL include/mtd (5 files)
	INSTALL include/rdma (6 files)
	INSTALL include/scsi/fc (4 files)
	INSTALL include/scsi (3 files)
	INSTALL include/sound (11 files)
	INSTALL include/video (3 files)
	INSTALL include/xen (4 files)
	INSTALL include/uapi (0 file)
	INSTALL include/asm (38 files)
make[5]: Leaving directory `/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29'
. /home/robbie/GitHub/widora/include/shell.sh; grep '=[ym]' /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.config | LC_ALL=C sort | md5s > /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.vermagic
touch /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.configured
rm -f /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/vmlinux /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/System.map
make -C /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29 HOSTCFLAGS="-O2 -I/home/robbie/GitHub/widora/staging_dir/host/include -I/home/robbie/GitHub/widora/staging_dir/host/usr/include -Wall -Wmissing-prototypes -Wstrict-prototypes" CROSS_COMPILE="mipsel-openwrt-linux-uclibc-" ARCH="mips" KBUILD_HAVE_NLS=no CONFIG_SHELL="bash" V='' CC="mipsel-openwrt-linux-uclibc-gcc" modules
make[5]: Entering directory `/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29'
	HOSTCC  scripts/kconfig/conf.o
	SHIPPED scripts/kconfig/zconf.tab.c
	SHIPPED scripts/kconfig/zconf.lex.c
	SHIPPED scripts/kconfig/zconf.hash.c
	HOSTCC  scripts/kconfig/zconf.tab.o
	HOSTLD  scripts/kconfig/conf
scripts/kconfig/conf --silentoldconfig Kconfig
net/sched/Kconfig:43: warning: menuconfig statement without prompt
#
# configuration written to .config
#
make[5]: Leaving directory `/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29'
make[5]: Entering directory `/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29'
	CHK     include/config/kernel.release
	UPD     include/config/kernel.release
	CHK     include/generated/uapi/linux/version.h
	CHK     include/generated/utsrelease.h
	UPD     include/generated/utsrelease.h
	CC      kernel/bounds.s
	GEN     include/generated/bounds.h
	CC      arch/mips/kernel/asm-offsets.s
	GEN     include/generated/asm-offsets.h
	CALL    scripts/checksyscalls.sh
	HOSTCC  scripts/dtc/dtc.o
	HOSTCC  scripts/dtc/flattree.o
	HOSTCC  scripts/dtc/fstree.o
	HOSTCC  scripts/dtc/data.o
	HOSTCC  scripts/dtc/livetree.o
	HOSTCC  scripts/dtc/treesource.o
	HOSTCC  scripts/dtc/srcpos.o
	HOSTCC  scripts/dtc/checks.o
	HOSTCC  scripts/dtc/util.o
	SHIPPED scripts/dtc/dtc-lexer.lex.c
	SHIPPED scripts/dtc/dtc-parser.tab.h
	HOSTCC  scripts/dtc/dtc-lexer.lex.o
	SHIPPED scripts/dtc/dtc-parser.tab.c
	HOSTCC  scripts/dtc/dtc-parser.tab.o
	HOSTLD  scripts/dtc/dtc
	CC      scripts/mod/empty.o
	HOSTCC  scripts/mod/mk_elfconfig
	MKELF   scripts/mod/elfconfig.h
	HOSTCC  scripts/mod/modpost.o
	CC      scripts/mod/devicetable-offsets.s
	GEN     scripts/mod/devicetable-offsets.h
	HOSTCC  scripts/mod/file2alias.o
	HOSTCC  scripts/mod/sumversion.o
	HOSTLD  scripts/mod/modpost
	HOSTCC  scripts/kallsyms
	HOSTCC  scripts/sortextable
	CC [M]  fs/autofs4/init.o
	CC [M]  fs/autofs4/inode.o
	CC [M]  fs/autofs4/root.o
	CC [M]  fs/autofs4/symlink.o
make[5]: Leaving directory `/home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29'
touch /home/robbie/GitHub/widora/build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7688/linux-3.18.29/.modules
make -C image compile TARGET_BUILD=
```
