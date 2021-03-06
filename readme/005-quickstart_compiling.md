---
title: "Compiling quickstart"
permalink: /docs/readme/quickstart/
excerpt: "Kismet has many many configuration knobs and options, but check here for the quickest way to get Kismet working with the latest release (or git version) and what you need to compile and do the initial configuration."
docgroup: "readme"
redirect_from:
  - /docs/readme/
  - /README-latest.html
---

## Compiling or packages?

Often, distributions lag behind software releases and may be offering older - sometimes significantly older - packages.  To get the latest, you can either install Kismet from [a package in the Kismet official repositories](/docs/readme/packages) or you can compile it from source.

If you're looking to make changes to the Kismet code, or if you're installing to a distribution not currently supported by the official Kismet packages, you'll definitely need to compile from source.

If you're installing onto a very resource constrained system, like a Raspberry Pi, you may want to consider either the packages, or making a cross compiling environment - modern C++ can be very resource intensive to compile, and a Raspberry Pi 3 or Raspberry Pi 0 is unlikely to be able to successfully compile natively.  The official packages for these environments are built on an Intel server with an emulated Pi docker environment to overcome these hurdles.

## Compiling: Quick Setup

Kismet has many configuration knobs and options; but for the quickest way to get the basics working:

1. Uninstall any existing Kismet installs.  If you installed Kismet using a package from your distribution, uninstall it the same way; if you compiled it yourself, be sure to remove it

2. Install dependencies.  Kismet needs a number of libraries and  development headers to compile; these should be available in nearly all distributions.

   * *Linux Ubuntu/Debian/Kali/Mint*

    ```bash
    $ sudo apt install build-essential git libmicrohttpd-dev pkg-config zlib1g-dev libnl-3-dev libnl-genl-3-dev libcap-dev libpcap-dev libnm-dev libdw-dev libsqlite3-dev libprotobuf-dev libprotobuf-c-dev protobuf-compiler protobuf-c-compiler libsensors4-dev libusb-1.0-0-dev python3 python3-setuptools python3-protobuf python3-requests python3-numpy python3-serial python3-usb python3-dev librtlsdr0 libubertooth-dev libbtbb-dev
    ```

    On some older distributions, `libprotobuf-c-dev` may be called `libprotobuf-c0-dev`.
   
    For rtlsdr rtl_433 support, you will also need the rtl_433 tool from https://github.com/merbanan/rtl_433 if it is not otherwise provided by your distribution.

   * *Linux Fedora (and related)*

    ```bash
    $ sudo dnf install make automake gcc gcc-c++ kernel-devel git libmicrohttpd-devel pkg-config zlib-devel libnl3-devel libcap-devel libpcap-devel NetworkManager-libnm-devel libdwarf libdwarf-devel elfutils-devel libsqlite3x-devel protobuf-devel protobuf-c-devel protobuf-compiler protobuf-c-compiler lm_sensors-devel libusb-devel fftw-devel
    ```

    You will also need the related python3, rtlsdr, and ubertooth packages.

   * *Other Linux distributions*

    Most distributions will have equivalent packages.  If your distribution splits binary and development packages, make sure to install both if you're compiling.

   * *OSX*

    OSX requires the XCode toolchain from the Apple store.  Once installed, you will need to launch the XCode IDE at least once to accept the license; do so before using the command line tools.

    You will need to install the `brew` tool from [brew.sh](https://brew.sh).  There are other package managers for OSX; feel free to use any of them which have the required packages, but Brew is known to work.

    Install the required packages via Brew:
    ```bash
    % brew install pkg-config libmicrohttpd python3 libpcap protobuf protobuf-c pcre librtlsdr libbtbb ubertooth
    ```

3. Clone Kismet from git.  If you haven't cloned Kismet before:
   ```bash
   $ git clone https://www.kismetwireless.net/git/kismet.git
   ```

    If you have a Kismet repo already:

    ```bash
   $ cd kismet
   $ git pull
    ```

4. Run configure.  This will find all the specifics about your system and prepare Kismet for compiling.  If you have any missing dependencies or incompatible library versions, they will show up here.
   ```bash
   $ cd kismet
   $ ./configure
   ```

   Pay attention to the summary at the end and look out for any warnings! The summary will show key features and raise warnings for missing dependencies which will drastically affect the compiled Kismet.

   If you're compiling for a remote capture platform *only*, check the [remote capture docs](/docs/readme/datasources_remote_capture/) for more information.

5. Compile Kismet.
   ```bash
   $ make
   ```

   You can accelerate the process by adding `-j #`, depending on how many CPUs you have.  To automatically compile on all the available cores:
   ```bash
   $ make -j$(nproc)
   ```

   C++ uses quite a bit of RAM to compile, so depending on the RAM available on your system you may need to limit the number of processes you run simultaneously.

6.  Install Kismet.  Generally, you should install Kismet as suid-root; Kismet will automatically add a group and install the capture binaries accordingly.

   When installed suid-root, Kismet will launch the binaries which control the channels and interfaces with the needed privileges, but will keep the packet decoding and web interface running without root privileges.
   ```bash
   $ sudo make suidinstall
   ```

7.  Add your user to the `kismet` group (Linux)
   ```bash
   $ sudo usermod -aG kismet $USER
   ```
   This will add your current logged in user to the `kismet` group.

   On OSX, Kismet is installed under the `staff` group, which the default user is part of.

8.  Log out and back in.  Linux does not update groups until you log in; if you have just added yourself to the Kismet group you will have to re-log in.

9.  Check that you are in the Kismet group with:
   ```bash
   $ groups
   ```
   If you are not in the `kismet` group, you should log out entirely, or reboot.

10.  You're now ready to run Kismet!  Point it at your network interface... Different distributions (and kernel versions, and distribution versions) name interfaces differently; your interface may be `wlan0` or `wlan1`, or it may be named something like `wlp0s1`, or it may be named using the MAC address of the card and look like `wlx00c0ca8d7f2e`.

   You can now start Kismet on Linux with something like:
   ```bash
   $ kismet -c wlan0
   ```

   Or on OSX with:
   ```bash
   $ kismet -c en1
   ```

   *or*, you can just launch Kismet and then use the new web UI to select the card you want to use, by launching it with just:
   ```bash
   $ kismet
   ```

   The name of your interface will be different depending on your kernel, distribution, etc;  when in doubt, run Kismet with no defined sources and select one from the 'Data Sources' option in the top-left menu in the web UI.

   Remember, until you add a data source, Kismet will not be capturing any packets!

   *THE FIRST TIME YOU RUN KISMET*, you *MUST* go to the Kismet WebUI and set your login and password.

   This login will be saved in the config file: `~/.kismet/kismet_httpd.conf` which is in the *home directory of the user* starting Kismet when installed in suidroot mode.  This is the preferred way to run Kismet.

   If you start Kismet as or via sudo (or via a system startup script where it runs as root), this will be in *roots* home directory: `/root/.kismet/kismet_httpd.conf`

11.  Point your browser at http://localhost:2501 (or the address of the server Kismet is running on)

   If you are running Kismet on your laptop (or other system with a browser), you can see the Kismet UI at `http://localhost:2501`.

   If you are running Kismet on a Raspberry Pi, Wi-Fi Pineapple, or other device, you will need to point your computer at the address *of the device running Kismet*.  You will need to have the system running Kismet plugged into wired Ethernet, or it will need a second Wi-Fi card configured to connect to your network:  You cannot run Kismet and connect to a network on the same Wi-Fi card at the same time.

   You will be prompted to do basic configuration - Kismet has many options in the web UI which can be tweaked.  Explore and have fun!

