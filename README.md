# FreedomNAS images

Ready-to-flash disk images and installer ISOs of
[FreedomNAS](https://codeberg.org/FreedomNAS), the free, OS-agnostic NAS
built on OpenZFS. Write an image to a disk, boot it, and finish the setup in
your browser.

This repository holds no code. The files are in its
[**Releases**](https://github.com/FreedomNAS/freedomnas-image/releases):
each one is uploaded there by the `build-images` workflow of
[freedomnas-image-builder](https://codeberg.org/FreedomNAS/freedomnas-image-builder)
on Codeberg, where FreedomNAS is developed.

## Which release

Every run of the workflow publishes one release, `images-<n>`:

| Release name | Built from | For |
|--------------|------------|-----|
| `FreedomNAS images, build <n> (vX.Y.Z)` | the release tag `vX.Y.Z` | everyday use |
| `FreedomNAS images, build <n> (dev)` | the `dev` branch | testing what comes next |

Pick the newest release whose name carries a version number unless you want to
try the development builds. All of them are marked as pre-releases until
FreedomNAS reaches 1.0.

## Which file

FreedomNAS runs on several operating systems, called editions. Every image
is a complete system: the operating system, OpenZFS, and FreedomNAS with its
web interface.

| File | Edition | Init system |
|------|---------|-------------|
| `debian.img.zst` | Debian 13 (trixie) | systemd |
| `freebsd.img.zst` | FreeBSD | rc |
| `void.img.zst` | Void Linux | runit |
| `artix-openrc.img.zst` | Artix Linux | OpenRC |
| `artix-dinit.img.zst` | Artix Linux | dinit |
| `artix-runit.img.zst` | Artix Linux | runit |
| `gentoo.img.zst` | Gentoo | OpenRC (dev builds) |
| `freedomnas-live.iso` | installer for the Linux editions | — |
| `freedomnas-live-freebsd.iso` | installer for FreeBSD (dev builds) | — |

If you have no preference, take **Debian**.

The two ISOs are for machines that cannot take a flashed image. They boot
on UEFI and BIOS, from a USB stick or optical media, and start the
FreedomNAS installer, which installs the edition you choose onto a disk.

## Requirements

| | Minimum | Recommended |
|-|---------|-------------|
| CPU | x86_64 (amd64) | |
| RAM | 2 GB | 8 GB or more, ZFS likes memory |
| System disk | 16 GB | 32 GB SSD |
| Network | 1 × Ethernet | 1 GbE or faster |

The system disk holds only the operating system. Keep your data on other
disks: FreedomNAS creates its data pools on them from the web interface.

## Flashing an image

1. Download the image and its `.sha256` file, and check it:

   ```sh
   sha256sum -c debian.img.zst.sha256
   ```

   A file larger than 2 GB is split into `<file>.part-aa`, `<file>.part-ab`,
   and so on. Download all the parts and join them first:

   ```sh
   cat debian.img.zst.part-* > debian.img.zst
   ```

2. Write it to the system disk or USB stick. **Everything on that disk is
   erased.**

   - with [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
     ("Use custom") or [Rufus](https://rufus.ie/) 4.7 or newer, which read `.zst` files
     directly;
   - from a terminal, replacing `/dev/sdX` with the right disk:

     ```sh
     zstd -dc debian.img.zst | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
     ```

   - or decompress it first (`zstd -d debian.img.zst`, or
     [PeaZip](https://peazip.github.io/) on Windows) and write the `.img`
     with any tool. balenaEtcher does not read `.zst` files, only the
     decompressed `.img`.

   The ISOs are written the same way, without `zstd`:

   ```sh
   sudo dd if=freedomnas-live.iso of=/dev/sdX bs=4M status=progress conv=fsync
   ```

3. Boot the machine from that disk. On the first boot the system grows to
   fill the whole disk by itself.

## First boot

Find the address the machine got from your router, and open in a browser:

```
https://<address>:8080
```

The browser warns about the certificate, because it is self-signed: accept
it. Sign in with the account every image comes with:

| Username | Password |
|----------|----------|
| `admin` | `freedomnas` |

The setup wizard starts right away and asks for your own administrator and
password, the host name, the time zone and the pool that keeps the
configuration. SSH is off until you turn it on there.

If you are reinstalling, leave your data disks connected: FreedomNAS finds
the configuration saved on its pool and offers to restore users, shares and
settings. It works across editions too, for example from a Void install to
a Debian one.

## Documentation and source

- Documentation: <https://freedomnas.codeberg.page/freedomnas-docs/>,
  also available offline in every installed system's web interface at `/docs/`
- Source code, issues and development: <https://codeberg.org/FreedomNAS>
- How these images are built: [freedomnas-image-builder](https://codeberg.org/FreedomNAS/freedomnas-image-builder)

## License

The files in this repository are under the Mozilla Public License 2.0
(`LICENSE`). The images contain software under many licenses: FreedomNAS is
MPL-2.0, and each operating system and package inside keeps its own.
