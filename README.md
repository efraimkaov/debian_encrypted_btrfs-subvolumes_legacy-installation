# debian_encrypted_btrfs-subvolumes_legacy-installation

## Installation guide for Debian GNU/Linux in the `Legacy mode` with btrfs subvolumes, disk encryption, swap file, [timeshift-autosnap-apt](https://github.com/wmutschl/timeshift-autosnap-apt) and [grub-btrfs](https://github.com/Antynea/grub-btrfs) (tested in version 11.0.0)

1. Boot into your Debian GNU/Linux in the Legacy mode

   * When you boot up your CD/DVD make sure you will see `Debian GNU/Linux installer menu (BIOS mode)`

2. Preparing your installation

   * Choose your language, configure your keyboard, etc.

   * Prepare your partitions manually

      * Example of partitioning scheme

      | DEVICE | SIZE | PTYPE | FSTYPE | MOUNTPOINT |
      | --- | --- | --- | --- | --- |
      | /dev/sda1 | 256 MB - 1 GB | 'Linux' | ext2 | /boot |
      | /dev/mapper/sda2_crypt | 30 GB - 100% | 'Linux' | btrfs | / |

   * Finish partitioning and write changes to disk

      * Write the changes to disks

3. Create and configure the subvolumes

   * **VERY IMPORTANT**, make sure you read carefully

      * `nossd` - By default, BTRFS will enable or disable SSD optimizations, but with regular SSD your system will complety freeze occasionally with `ssd` option enabled. I recommend you to use `nossd` option for both HDD or SSD.

      * `noatime` - Under read intensive work-loads, specifying `noatime` significantly improves performance because no new access time information needs to be written. Note that `noatime` may break applications that rely on `atime` uptimes like the venerable Mutt (unless you use maildir mailboxes).

      * `space_cache` - The free `space_cache` greatly improves performance when reading block group free space into memory. However, managing the `space_cache` consumes some resources, including a small amount of disk space.

      * `compress=zstd` - zstd expose the compression level as a tunable knob with higher levels trading speed and memory for higher compression ratios.

      * `discard=async` - **WARNING:** BTRFS documentation recommend you to use `fstrim.timer` service over `discard=async` option. By default many distributions have enabled `fstrim.timer` service. I highly don't recommend you to have enabled both `fstrim.timer` service and `discard=async` option. With `discard=async` option enabled your system will complety freeze occasionally.

      * `autodefrag` - **WARNING:** Defragmenting will break up the reflinks of COW data (for example files copied with cp --reflink, snapshots or de-duplicated data). This may cause considerable increase of space usage depending on the broken up reflinks.

   * Move to `tty2` with `Ctrl+Alt+F2`

   * Press `Enter` to activate the console

   ```
   # umount -l /target
   # mount -o defaults,subvolid=5,nossd,noatime,space_cache,compress=zstd /dev/mapper/sda2_crypt /mnt
   # mv /mnt/@rootfs /mnt/@
   # btrfs subvolume create /mnt/@home
   # btrfs subvolume create /mnt/@swap
   # mkdir -p /mnt/@/home
   # mkdir -p /mnt/@/swap
   # sed -i 's/btrfs   defaults,subvol=@rootfs/btrfs defaults,subvol=@,nossd,noatime,space_cache,compress=zstd/' /mnt/@/etc/fstab
   # echo "/dev/mapper/sda2_crypt /home btrfs defaults,subvol=@home,nossd,noatime,space_cache,compress=zstd 0 0" >> /mnt/@/etc/fstab
   # echo "/dev/mapper/sda2_crypt /swap btrfs defaults,subvol=@swap,nossd,noatime,space_cache,compress=zstd 0 0" >> /mnt/@/etc/fstab
   # btrfs subvolume list /mnt
   # umount -l /mnt
   # mount -o defaults,subvolid=256,subvol=@,nossd,noatime,space_cache,compress=zstd /dev/mapper/sda2_crypt /target
   # mount -o defaults,subvolid=258,subvol=@home,nossd,noatime,space_cache,compress=zstd /dev/mapper/sda2_crypt /target/home
   # mount /dev/sda1 /target/boot
   ```
   
   * Deactivate the console with `Ctrl+d`

   * Move back to `tty1` with `Ctrl+Alt+F1` if you are using the `Text installer` or to `tty5` with `Ctrl+Alt+F5` if you are using the `Graphical installer`

4. Installing the system

   * Install the base system, Select and install software, etc.

   * Finish the installation

5. Post-Installation steps

   * Soon
