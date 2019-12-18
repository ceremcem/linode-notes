# Converting rootfs to BTRFS

> Note: Converting rootfs to BTRFS via `btrfs-convert` does
> not work at this time. System succesfully boots, however an 
> exception is thrown by the kernel and rootfs is re-mounted readonly.

> Note2: You don't have to rent a temporary disk volume, you may 
> use a storage over network if you like. Adapt the below procedure
> accordingly if you prefer to do so.

1. Rent a second disk volume.

2. Boot in Rescue Mode. Finnix OS will be launched in a few seconds.

    1. Be sure you attached the newly rent disk before booting in rescue mode

3. Backup your rootfs:
       
        mkfs.btrfs /dev/sdc 
        mkdir /mnt/a /mnt/c
        mount /dev/sda /mnt/a
        mount /dev/sdc /mnt/c
        rsync -avP /mnt/a/ /mnt/c/

4. Format the rootfs disk:

        umount /dev/sda
        mkfs.btrfs /dev/sda
        mount /dev/sda /mnt/a

5. Copy your rootfs files back to /dev/sda 

        rsync -avP /mnt/c/ /mnt/a/

6. Edit `/mnt/a/etc/fstab` accordingly to change rootfs type to BTRFS: 

        /dev/sda       /               btrfs    defaults          0       1

7. Use the UUID value of the following command in the next step: 

        blkid | grep /dev/sda

8. Edit `/mnt/a/boot/grub/grub.cfg` to replace UUID of `/dev/sda` in the menuentries accordingly

9. Reboot the system

10. If you successfully booted, update grub.cfg properly:

        update-grub

11. Do not forget to install btrfs-tools:

        apt-get install btrfs-tools


# Move `/` to a subvolume (`rootfs`)

1. `btrfs sub snap / /rootfs`
2. Create the new top subvolume mountpoint:

        mkdir /rootfs/mnt/myhost

3. Edit the `/etc/fstab` now (to save one reboot):

        /dev/sda  /             btrfs   subvol=/rootfs,rw,noatime 0 1
        /dev/sda  /mnt/myhost   btrfs   subvolid=5,noatime  0 1

4. Edit the `linux ...` lines in `/boot/grub/grub.cfg` accordingly to mount /rootfs subvolume at boot time: 

        ... rootflags=subvol=rootfs ... 

5. Reboot 

6. Check if everything went fine so far:

      1. Check if there is any BTRFS error: 

              # cat /var/log/messages | grep -i btrfs | grep -i error
  
      2. Check that our new / is the rootfs subvolume:

              # btrfs subvolume show /
                      Name:                   rootfs
                      ...

7. If everything went fine, update grub to reflect the appropriate changes:

          ### do not forget to backup your /boot/grub/grub.cfg first 
          # update-grub 

8. Reboot  

9. If everything still okay, cleanup the old files: 
    
          # find /mnt/myhost -maxdepth 1 -mindepth 1 -not -name rootfs -exec rm -r {} \;



10. Fix your Grub `configfile` location: 

      > You can load your configfile inside Grub Rescue Shell by the following command: 
      > 
      >       configfile (hd0)/rootfs/boot/grub/grub.cfg


      1. `btrfs sub create /mnt/myhost/boot`
      2. `mv /boot/* /mnt/myhost/boot/`

      3. Add the following line to `/etc/fstab` to mount `/boot` from `/mnt/myhost/boot`: 

             /dev/sda       /boot           btrfs    subvol=/boot,rw,noatime       0     1

      4. `update-grub`



