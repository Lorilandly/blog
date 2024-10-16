---
title: "Running Ubuntu(ARM) on M1 Mac with QEMU"
date: 2024-09-18
---

For my Embedded Systems course, I will be creating a kernel on my Adafruit feather board. As the the class will only provide support for Ubuntu, I will be programming on a VM for the rest of the project. 

## Preparations
You could totally use a VM application with a fancy GUI interface. That will probably be easier to setup and use, and require much less tinkering. But what's the fun in that! Since I will not need to use the Ubuntu desktop at all (I'm just doing some coding work with SSH), I could totally be running the VM without display and have it clean and tiny~

You will need the following prepared on your laptop:
- `brew`
- `qemu`
- Download Ubuntu Server image for ARM from their website

The M-series Mac is running on the ARM architecture CPU. If I want to enable CPU passthrough to save some of my laptop's precious battery, I would need to run the ARM version of Ubuntu Server. Unlike the commonly used X86_64 architecture which have a bios system that enable booting directly from one partition, ARM only supports the more advanced and more complicated efi booting, which requires an efi partition to work. If I'm not mistaken, the efi partition basically contains a bootloader that follows the efi protocol.

```bash
curl -L https://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/QEMU_EFI.fd -o QEMU_EFI.fd
```

This command downloads the efi partition from an external source. On a linux system, there are more standard way to do this and I'll talk about it later.

Next, we will create a partition for the VM to run on. Since I'll not be putting a lot of data into the VM, I'm going to allocate 10G initially.

```bash
qemu-img create -f raw ubuntu.img 10G
```

Note that the image format here is `raw`, not `qcow2` used commonly for X86_64. `qcow2` will not work, I tried that so you don't have to, though I didn't find anything about why it's not working.

## Install Ubuntu in your VM
With these two images ready, You can start to install Ubuntu onto your VM.

```bash
qemu-system-aarch64 \
        -M virt,highmem=off \
        -accel hvf \
        -cpu host \
        -smp 2 \
        -m 2G \
        -nographic \
        -drive if=virtio,format=raw,file=./ubuntu.img \
        -bios QEMU_EFI.fd
        -cdrom path/to/ubuntu-24.04.iso
```

In the launch command, `-M` sets the machine type to `virt`, `-accel hvf` enables the MacOS accelerator framework, `-cpu host` enables passthrough, `-cmp 2` allocates two CPU threads, and lastly `-m 2G` gives the VM 2 gigs of memory. 

After you get into the VM, follow the Ubuntu installing wizard to install Ubuntu. The next time you boot up, you will not need the `-cdrom` argument.

```bash
qemu-system-aarch64 \
        -M virt,highmem=off \
        -accel hvf \
        -cpu host \
        -smp 2 \
        -m 2G \
        -nographic \
        -drive if=virtio,format=raw,file=./ubuntu.img \
        -bios QEMU_EFI.fd
```

## Connecting to the VM with SSH
When you start your VM, you will find that the terminal of the VM is connected to your machine, but the size of the terminal is fixed and it only occupies a small portion of your screen. It bugs you so badly and start scratching your head finding ways you could extend the terminal to the entirety of your screem. 

But then you realize that you don't really need the terminal to be working anyways. Just connect to the VM through SSH! To do so, you will just need to attach the SSH port of the VM to your local machine like so.

```bash
qemu-system-aarch64 \
        -M virt,highmem=off \
        -accel hvf \
        -cpu host \
        -smp 2 \
        -m 2G \
        -nographic \
        -drive if=virtio,format=raw,file=./ubuntu.img \
        -bios QEMU_EFI.fd \
        -net user,hostfwd=tcp::10022-:22 -net nic
```

What the line added above basically did is that it connected the port 22 on the VM to the port 10022 on the host machine with a TCP tunnel. Now you can connect to the VM on the port `10022` on your localhost.

## Allowing the VM to access USB devices
The reason I'm creating this VM is to write code for my little embedded board. For that I'll of course need the VM to connect to the little thing. For that the USB device need to be forwarded to the VM. For that, you first need to confirm the ID of your USB device. On MacOS, you can find the information by looking under the USB section in your "System Information" APP. A USB device will have a vender ID and a product ID. These tells your computer what device have been connected. You don't need to decypher the ID's, just record them.

```bash
-device qemu-xhci \
-device usb-host,vendorid=0x1d50,productid=0x6018
```

The first line here attaches a USB bus to the VM, and the second line pass through the specific USB device to the VM. My device has a vendor ID of `0x1d50` and product ID of `0x6018`, you can look up what kind of device I'm using if you want :)


## Another way of doing bootloader
Currently the bios conbines the EFI partition and the varstore partition. Apparently it is a better practice to separate these partitions so that we have a read only partition and R/W partition. For that we will need to download the EFI from another external source.

```bash
  curl -O https://yum.oracle.com/repo/OracleLinux/OL7/latest/aarch64/getPackage/AAVMF-1.5.1-1.el7.noarch.rpm
rpm2cpio AAVMF-1.5.1-1.el7.noarch.rpm | cpio -ivd usr/share/AAVMF/AAVMF_{VARS,CODE}.fd
```

Prep the images like so

```bash
dd if=/dev/zero of=pflash0.img bs=1m count=64
dd if=/dev/zero of=pflash1.img bs=1m count=64
dd if=usr/share/AAVMF/AAVMF_CODE.fd of=pflash0.img conv=notrunc
dd if=usr/share/AAVMF/AAVMF_VARS.fd of=pflash1.img conv=notrunc
```

And replace the `-bios` with the following

```bash
    -drive file=pflash0.img,format=raw,if=pflash,readonly=on \
    -drive file=pflash1.img,format=raw,if=pflash \
```

## TBC...
```bash
truncate -s 64m efi.img
dd if=/usr/share/qemu-efi-aarch64/QEMU_EFI.fd of=efi.img conv=notrunc
```

### References
> https://blogs.oracle.com/linux/post/oracle-linux-9-with-qemu-on-an-m1-mac
> https://adonis0147.github.io/post/qemu-macos-apple-silicon/
> https://ubuntu.com/server/docs/boot-arm64-virtual-machines-on-qemu
