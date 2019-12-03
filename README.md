CRAN Solaris 10/11 Setup for Package Testing
================
Balasubramanian Narasimhan
November 1, 2019

## Environment

  - Macbook pro running MacOS 10.15.1
  - [Virtual Box Version 6.0.14 r133895
    (Qt5.6.3)](https://www.virtualbox.org/)
  - Enable function keys to work as standard function keys via system
    preferences for setup of Solaris and set it back as needed
  - I do everything as `root` (a bit lazy, yes) and so I create a
    directory `/root` where I keep files, sources and build things,
    including the `config.site` and `mapfile` referenced below.

### Config and Mapfile

The `config.site` provides all the flags for building with Oracle
Developer Studio and override default settings. Note the reference to a
`mapfile` which I place in `/root/mapfile` (more below).

``` bash
#! /bin/sh

## From Solaris specs on CRAN
CC='cc -xc99'
CPPFLAGS="-I/opt/csw/include -I/usr/local/include"
LDFLAGS="-L/opt/csw/lib -L/usr/local/lib -M/root/mapfile"
CFLAGS='-O -xlibmieee -xlibmil -xtarget=native -xcache=generic -nofstore'
FC=f95
FFLAGS='-O -libmil -xtarget=native -xcache=generic -nofstore'
CXX=CC
CXXSTD="-std=c++11 -library=stdcpp,CrunG3"
CXXFLAGS="-O -xlibmil -xtarget=native -xcache=generic -nofstore"
CXX98STD="-compat=g -library=stdcpp,CrunG3"
CXX11STD=$CXXSTD
SAFE_FFLAGS="-O -fstore"
FCLIBS_XTRA="-lfsu /opt/developerstudio12.6/lib/libfui.so.2"
R_LD_LIBRARY_PATH="/opt/developerstudio12.6/lib:/usr/local/lib:/opt/csw/lib"
```

### Mapfile

The `mapfile` below is something that overrides assumptions the
compilers make about hardware; see
[reference](https://docs.oracle.com/cd/E19120-01/open.solaris/819-0690/chapter2-20/index.html).
Without this fix, expect a strange error about `sysdata` not being
loaded\! My goal is not speed, but mere checking, so I go with the bare
minimum.

``` c
$mapfile_version 2
CAPABILITY {
    HW = ;
};
```

## 1\. Solaris 10 Instructions

### 1.1. Set up Solaris 10 OS

*A one-time setup\!*

  - Dowload template from
    [Oracle](https://download.oracle.com/otn/solaris/vm/Solaris10_1-13_VM.ova).
  - Import appliance (`.ova` file) into vbox.
  - After import, tweak settings
      - System/Motherboard: Base Memory 8G, 1 processor
      - Display/Video Memory: 32Mb
      - General/Advanced: Enable Shared Clipboard
      - Shared Folders: Local Downloads folder (`Folder Name:
        Downloads`), automounted to Solaris (`Mount point: Downloads`),
        although it is seen at `/mnt/sf_Downloads`\!
      - Save settings
  - Boot into Solaris for the first time.
      - Choose timezone, and en\_US.UTF-8 as Locale
      - Setup root password which we will use for everything
      - To avoid hassles, decline `NIS+` or `NIS` and just go with `DNS`
        (which is what I do), or even none if you don’t know what those
        terms mean. (The latter, however, could be limiting for dns name
        resolution and downloads.)
      - Go with default for all else

### 1.2. Set up Oracle Developer Studio

*A one-time setup unless the version of Developer Studio changes\!*

  - Log in as `root`. (As noted earlier I do everything as `root`.)
  - Create a directory called `/root` and save `config.site` and
    `mapfile` above there.
  - Install Developer Studio (v12.6 as of this writing)
      - Download [Oracle Developer Studio tar gz
        file](https://www.oracle.com/technetwork/server-storage/developerstudio/overview/index.html)
      - Unzip, untar, install and patch.

<!-- end list -->

``` bash
bzcat -d OracleDeveloperStudio12.6-solaris-x86-pkg.tar.bz2 | tar xf -
cd OracleDeveloperStudio12.6-solaris-x86-pkg
./developerstudio.sh --non-interactive
./install_patches.sh
```

### 1.3. Set up Open CSW

*A one-time setup.*

  - Install needed packages from Open CSW Project tree

<!-- end list -->

``` bash
pkgadd -d http://get.opencsw.org/now
/opt/csw/bin/pkgutil -U
/opt/csw/bin/pkgutil -y -i libiconv_dev libz_dev libreadline_dev liblzma_dev libpcre_dev libcurl_dev libssh2_dev libssl_dev libcares_dev librtmp_dev libkrb5_dev libk5crypto3 liblber2_4_2 libbrotli_dev libicu_dev pkg-config gmake texlive gtar curl wget emacs 
```

  - Add symbolic link manually, otherwise, expect (incorrect) error
    message about curl version during configure\!

<!-- end list -->

``` bash
ln -s /opt/csw/lib/liblber-2.4.so.2 /opt/csw/lib/liblber.so
```

### 1.4. Build `R-patched`

*Repeat with each release of R.*

I build in `/root` and install to `/root/RHOME`. Always build
`R-patched` first since `R-devel` can be unstable leaving you wondering
if the toolchain is wrong or the R-devel source is unstable\!

  - Set up environment and directory

<!-- end list -->

``` bash
mkdir /root
export PATH=/usr/ccs/bin:/opt/developerstudio12.6/bin:/usr/sfw/bin:/usr/xpg4/bin:/usr/xpg6/bin::/opt/csw/bin:$PATH
cd /root
mkdir RHOME ## where R will land
```

  - Download `R-devel.tar.gz` or `R-patched.tar.gz` and untar.
  - Copy above `config.site` over the supplied `config.site` in
    `R-devel` or `R-patched` directory
  - Configure and make in the main source directory

<!-- end list -->

``` bash
./configure --prefix=/root/RHOME --enable --with-internal-tzcode
make
make install
```

  - Add `/root/RHOME` to path and invoke R

<!-- end list -->

``` bash
export PATH=$PATH/RHOME/bin:$PATH
R
```

### 1.5. Build `R-devel`

*Repeat with each release of R.*

Repeat 1.4 with R-devel sources.

## 2\. Solaris 11 Notes

Avoid this if you can. Solaris X window server is appalling, is
blindingly slow, hangs and crashes. But the following should work.

### 2.1. Set up Solaris 11

*One time setup.*

  - Download template from
    [Oracle](http://www.oracle.com/technetwork/server-storage/solaris11/downloads/vm-templates-2245495.html).
  - Import appliance (`.ova` file) into vbox.
  - After import, tweak settings
      - System: Base Memory 8G, 2 processors
      - Display: 32Mb
      - Shared Folders: Local Downloads folder (`Name: Downloads`),
        automounted to Solaris (`/Downloads`), although it is see at
        `/mnt/sf_Downloads`\!
      - Disable audio on Solaris 11 or it will freeze over and over\!
      - Save settings
  - Boot into Solaris for the first time.
      - Choose timezone, and en\_US.UTF-8 as Locale
      - Setup root password and a standard account
      - Go with default for all else.
      - Log in as user, decline location services and online accounts
        synchronization. Exit the welcome app and reboot to be safe

### 2.2. Set up Oracle Developer Studio

*A one-time setup unless the version of Developer Studio changes\!*

  - Install Developer Studio (v12.6 as of this writing)
      - Download [certificate and
        keys](https://pkg-register.oracle.com/register/certificate/)
      - [Request
        access](https://pkg-register.oracle.com/register/repos/) and
        follow instructions on solaris machine.
  - Update everything: `sudo pkg update`

### 2.3. Follow Instructions for Solaris 10

From 1.3 onwards.

## Miscellaneous Notes

1.  To check/install a package which has `GNU Make` as a systems
    requirement, use

<!-- end list -->

``` bash
MAKE=gmake R CMD ...
```

2.  Some packages cannot be installed using the Oracle development
    tools. The only recourse is the GNU tools. For this purpose, one
    needs to jump through several hoops.

<!-- end list -->

  - Setup a directory `~/.R/Makevars` file with the following contents

<!-- end list -->

``` bash
CC=gcc
CFLAGS=-m32 -I/opt/csw/include -I/usr/local/include
CPPFLAGS=-m32 -I/opt/csw/include -I/usr/local/include
CPICFLAGS=
CXXPICFLAGS=
CXXSTD=
F77=gfortran
F77FLAGS=-m32
CXX=g++
CXXFLAGS=-m32 -I/opt/csw/include -I/usr/local/include
FC=$F77
LDFLAGS=-L/opt/csw/lib -L/usr/local/lib
```

I also set my path as follows

``` bash
export PATH=/opt/csw/gnu:/root/RHOME/bin:$PATH
```

so that the OpenCSW tools are found first. After this you can install
packages like `stringi` which is needed by `stringr` which is needed for
`knitr`. When done, move the directory back to `~/.R-gnu` to keep for
future use.
