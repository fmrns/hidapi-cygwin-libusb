# libusb patch for HIDAPI on Cygwin

## background

When I use some application with
[HIDAPI](https://github.com/libusb/hidapi) on Windows, privilege
escalation is required to work properly. HIDAPI uses hid.dll on
Windows, and libusb on some OSes. So I have made a PoC: By forcing
HIDAPI to use libusb on Windows, application does not need to escalate
its privilege. On the situation, the libusbK driver is the only one
that needs administrator privilege.

cf. [libusb with libusbK](https://github.com/libusb/libusb/wiki/FAQ#libusbwin32_libusbK_and_libusb_project)

## CAUTION

Trying this PoC can [damage your device](https://github.com/libusb/hidapi/issues/105).

Trying this PoC can corrupt your Windows environment.

Trying this PoC can corrupt your Cygwin environment.

Please try this PoC in the experimental environment, and only if you
know what you are doing.

### setup

1. Install cygport, libusb1.0, libusb1.0-devel, libiconv,
 libiconv-devel, and *src* of libhidapi0 with setup-x86_64.exe

You may need more below, please install them as appropriate in that
case.

2. Apply patches.

```
$ cd /usr/src/hidapi-0.8.0-rc1-1.src/
$ patch < .../hidapi.cygport.patch
$ cp -p .../hidapi-0.8.0-rc1-1_1.src.patch .
$ cygport hidapi.cygport prep 
```

3. Build binaries.

```
$ cygport hidapi.cygport compile
$ cygport hidapi.cygport install
$ cygport hidapi.cygport package
```

4. Install them locally, only if you know what you are doing.

```
$ tar      -tJvf hidapi-0.8.0-rc1-1_1.x86_64/dist/hidapi/hidapi0/hidapi0-0.8.0-rc1-1_1.tar.xz
$ tar -C / -xJvf hidapi-0.8.0-rc1-1_1.x86_64/dist/hidapi/hidapi0/hidapi0-0.8.0-rc1-1_1.tar.xz
$ tar      -tJvf hidapi-0.8.0-rc1-1_1.x86_64/dist/hidapi/libhidapi-devel/libhidapi-devel-0.8.0-rc1-1_1.tar.xz
$ tar -C / -xJvf hidapi-0.8.0-rc1-1_1.x86_64/dist/hidapi/libhidapi-devel/libhidapi-devel-0.8.0-rc1-1_1.tar.xz
```

5. Install [libusbK](https://sourceforge.net/projects/libusbk/files/libusbK-release/)

I have tried with V3.1.0.0. libusbK.dll is installed on %WINDIR%/system32.

6. Install a device driver.

Run c:/libusbK-dev-kit/libusbK-inf-wizard.exe and select WinUSB, the
target device, 'Client installer,' and 'Finish & Install Driver now.'
The device is shown in the section 'Universal Serial Bus devices' in
Device Manager. You can select libusbK instead of WinUSB, and the
section is 'libusbK Usb Devices' for it.

7. That's all. Test your application.

When you want to remove the libusbK device drivers, please remeber to
click the check box saying "Uninstall Driver and Software."


## libfido2

My motivation of this PoC has come from libfido2. libfido2 uses
hid.dll on Windows(incl. MSYS2, and Cygwin). Both ssh-keygen -t
ecdsa-sk and -t ed25519-sk without priviledge result in the message
below.

```
You may need to touch your authenticator to authorize key generation.
Key enrollment failed: invalid format
```

libfido2 uses HIDAPI for FreeBSD, so forcing libfido2 to use HIDAPI on
Windows may be what I want. Making libfido2 to use libusb 1.0 directly
is the better solution for simplicity of composition. But
[OpenSSH SK WinHello](https://github.com/tavrez/openssh-sk-winhello),
which is a plug-in for ssh-sk-helper using WinHello API, is the best,
since it works with default device driver.


### setup

0. If you really want to use -sk key types without priviledge
escalation on Windows, please leave here, and go to
[OpenSSH SK WinHello](https://github.com/tavrez/openssh-sk-winhello).

1. Install gcc-fortran, and *src* of libfido2 with setup-x86_64.exe

The above *customized* hidapi0, which uses libusb 1.0 is
mandatory. You may need more below, please install them as appropriate
in that case.

2. Apply patches.

```
$ cd  /usr/src/libfido2-1.5.0-2.src/
$ patch < .../libfido2.cygport.patch
$ cp -p .../libfido2-1.5.0-2_1.src.patch .
$ cygport libfido2.cygport prep 
```

3. Build binaries.

```
$ cygport libfido2.cygport compile
$ cygport libfido2.cygport install
$ cygport libfido2.cygport package
```

4. Install them locally, only if you know what you are doing.

```
$ tar      -tJvf libfido2-1.5.0-2_1.x86_64/dist/libfido2/libfido2-1.5.0-2_1.tar.xz
$ tar -C / -xJvf libfido2-1.5.0-2_1.x86_64/dist/libfido2/libfido2-1.5.0-2_1.tar.xz
```

5. Check if libfido2 loads libusb 1.0 (via HIDAPI).

```
$ ldd /usr/bin/fido2-token
        ntdll.dll => /cygdrive/c/WINDOWS/SYSTEM32/ntdll.dll (0x7ffa9f1d0000)
        KERNEL32.DLL => /cygdrive/c/WINDOWS/System32/KERNEL32.DLL (0x7ffa9e500000)
        KERNELBASE.dll => /cygdrive/c/WINDOWS/System32/KERNELBASE.dll (0x7ffa9cbf0000)
        cygfido2-1.dll => /usr/bin/cygfido2-1.dll (0x5c1b00000)
        cygcrypto-1.1.dll => /usr/bin/cygcrypto-1.1.dll (0x3fb610000)
        cygwin1.dll => /usr/bin/cygwin1.dll (0x180040000)
        cygcbor-0.dll => /usr/bin/cygcbor-0.dll (0x3ff550000)
        cyghidapi-libusb-0.dll => /usr/bin/cyghidapi-libusb-0.dll (0x5282e0000)
        cyggcc_s-seh-1.dll => /usr/bin/cyggcc_s-seh-1.dll (0x3ffba0000)
        cygz.dll => /usr/bin/cygz.dll (0x3ff0d0000)
        cygiconv-2.dll => /usr/bin/cygiconv-2.dll (0x3ff7b0000)
        cygusb-1.0.dll => /usr/bin/cygusb-1.0.dll (0x3fada0000)
```

6. Install libusbK driver as above.

Both WinUSB and libusbK drivers are expected to work properly. If your
device has multiple interfaces with same Vendor ID and Product ID,
install drivers of all interfaces binding to 'HidUsb.' Do not select
devices with, such as WUDFRd, and usbccgp.

Now you can see the list of devices without priviledge.

```
$ fido2-token -d -L
0001:0011:00: vendor=0x1050, product=0x0120 (Yubico Security Key by Yubico)
0001:0012:01: vendor=0x1050, product=0x0407 (Yubico YubiKey OTP+FIDO+CCID)
0001:0013:00: vendor=0x1ea8, product=0xf025 (ExcelSecu EsecuFIDO HID)
```

A Thetis U2F key (not FIDO2), which is not shown even by priviledged
original fido2-token with hid.dll in my Windows environment, is also
shown.

After removing drivers, these devices are shown as HID-compliant fido
in Human Interface Device section in Device Manager.

## TODO

Using libusb 1.0 directly may be better as described above, but maybe
no one wants. If all functionality of libfido2 can be implemented with
WinHello API (I don't think so), it may help some people.
