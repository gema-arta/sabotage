# Sabotage Linux

This is sabotage, an experimental distribution based on musl libc and busybox.

Currently sabotage supports i386 and x86_64, MIPS, PowerPC32 and ARM(v4t+).

The prefered way to build sabotage is using a native linux environment
for the desired architecture; however it is now also possible to cross-compile
large parts of it. as cross-compiling is very hairy and support for it
is quite new in sabotage, breakage is to be expected.
Native builds though are well tested and can be considered very stable.

## Requirements:

* ~4G free disk space
* root access, or a linux 3.8+ host kernel with USER_NS support 
  (only for native build)
* usual GCC 4 toolchain
* git
* lots of time and a fair bit of Linux knowledge

## X-Compile Requirements

* latest musl-cross for your target arch
* latest butch >= 0.4.0 installed and compiled for the build host in $PATH
* pkgconf symlinked as pkg-config in PATH (before other pkg-config versions)
* anything that is listed in deps.host sections in the packages you build,
  installed on your host.
  optimal (and only tested configuration) is to build from a sabotage host
  (rootfs) that has all the same packages installed that you want to compile.

This system has been built *natively* on Debian 6.0 and 7.0, Ubuntu 13.04, 
Suse 11.4, aboriginal 1.1 and 1.2, and Fedora 19 systems.

You can bootstrap your own build from the scripts at 

https://github.com/sabotage-linux/sabotage

or use a ready-to-boot disk image either for qemu, vbox 
or to extract the rootfs, to be found at:

* DE : http://ftp.barfooze.de/pub/sabotage/
* GR : http://foss.aueb.gr/mirrors/linux/sabotage/
* UK : http://dl.2f30.org/mirrors/sabotage/

the DE mirror is the master from which the other mirrors are synced
after some hours.

sha512 checksums for the releases are announced on the sabotage
and musl mailinglists, and are archived here:

http://openwall.com/lists/sabotage/


**READ THE COOKBOOK FIRST**.

## Native Build instructions:

**DO NOT RUN SCRIPTS YOU HAVE NOT READ**.

    cp KEEP/config.stage0 config
    vi config

set SABOTAGE_BUILDDIR, A, and MAKE_THREADS variables. 
if you have an appropriate kernel and no root privs, set SUPER.
the other values can be left as-is usually.

NOTE: it is possible to build i386 sabotage from within an existing
32bit chroot on a 64bit sys, however you need to "impersonate" a 32bit sys
(this means uname has to lie and report a 32bit sys) using the linux32 command.
the enter-chroot script will try to detect this scenario automatically
and builds an embedded version (KEEP/linux32.c) if linux32 is not alread
installed.

    ./build-stage 0  # build toolchain 
                     # ~2min on an AMD FX 8core, 75min on ARM Cortex A8 800Mhz
    ./enter-chroot   # enter $R chrooted, needs root password or SUPER, see above
    cd /src
    vi config        # set your MAKE_THREADS, etc
    butch install stage1

and after that, feel free to install optional packages:
you can look at what's available using 'ls /src/pkg'

if you want the default kernel (otherwise you have to build it yourself)

    butch install kernel

    butch install core # installs a sane subset needed for a developer base system
    butch install xorg # install everything needed to get xfbdev
    butch install pkg # installs additional things, such as file, git, gdb ...
    butch install world #installs almost everything

Run "butch" and look at the usage information it outputs for further options.

butch uses build templates that allow a high level of customization.
in case you're interested, take a look at 
KEEP/butch_template_configure_cached.txt to see how it works. 
this is the base template used by sabotage, which
is responsible for things like providing a tuned config.cache for faster
configure runs, installing packages into a custom directory in /opt, 
creation of filelists, etc.

## X-Compile instructions:

    cp KEEP/config.cross .
    vi config.cross #set your vars
    A=microblaze CONFIG=./config.cross utils/setup-rootfs #initialize rootfs
    A=microblaze CONFIG=./config.cross butch install nano #start building stuff

when you're done compiling, exit the chroot and
- either use the rootfs directly (by copying it to some disk)
- use utils/run-emulator.sh to boot the system directly in qemu.
  running the rootfs directly in qemu has pretty poor hdd performance,
  because the FS is mounted via 9P network, so it's not recommended
  to build packages, but it's practical for testing.
- use utils/write-hd-image.sh to create an image file.
  the image file can be directly booted in qemu.
  to convert it into virtual box format use "VBoxManage convertfromraw".


## RUNNING SABOTAGE FOR THE FIRST TIME

The default root password is "sabotage".

the sshd service can be started using "sv u sshd"
(will create keys automatically on first use).
to make the service autostart on boot, remove /etc/service/sshd/down.

check and edit /etc/rc.local for other things to autostart,
such as network config, dhcp, linux console keymap...

if you have X installed, you want to edit /bin/X for the correct evdev
settings (see examples provided in there), then run "startx".
also check /etc/xinitrc for X11 keyboard config.

## NOTE TO CONTRIBUTORS

if you want to add packages, start from KEEP/pkg_skel/autoconf template.

    cp KEEP/pkg_skel/autoconf pkg/my_new_pkg
    utils/dlinfo.sh http://1.2.3.4/my_new_pkg.tar.xz

that'll spit out the filesize, sha512sum boilerplate for easy copy&paste.

please do not use HTTPS or FTP mirrors.
HTTPS is unsupported by busybox wget, and FTP is a broken, ancient protocol
which needs a second data connection (i.e. open port on the client).

this can cause problems when behind a NAT router or socks proxy.
Downloads from git or other source repositories are not desired, because
that would introduce a build-time dependency on an internet connection.

sabotage is designed so you can download all packages in advance when you
have internet connection, and then build everything offline.

please use unified diff format  (diff -u) for patches.

since sabotage ships all tarballs when an ISO or HD image is distributed (to
fulfill the GPL), space considerations are a top issue.
so if available, ALWAYS USE a TAR.XZ (preferred) or TAR.BZ2 download URL.

it is necessary that you create git branches for your work:
this allows me to checkout your changes and rebase as i see fit,
and you can easily pull back into your master without getting merge conflicts.

do not commit more than one change/package in a single commit, as it makes
it much more work than needed to pick the good commits and leave away bad ones.
use a meaningful commit message that mentions the package name.


## CONTACT
There is a mailinglist: sabotage@lists.openwall.com,
mail sabotage-subscribe@lists.openwall.com and follow instructions to get on it.

Archives are at http://openwall.com/lists/sabotage/ .

You can also /join #sabotage or #musl on irc.freenode.net for realtime help.

## DONATIONS

donations in bitcoins are welcome and can be sent to

1HXhSKSyBUGAAga29WbpTkKGpruQq9J8Bb .


