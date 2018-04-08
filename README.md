# ntfs-rootfs
A guide on how to use NTFS as the root filesystem on Linux or possibly any other UNIX OS.

Go visit the wiki (on this repository) on how to install and setup.

## Currently supported:
* ArchLinux (and derivatives)

## Stuff I have to say....
QUICK WARNING: This will probably be impractical and/or hard to set up. So it is NOT recommended for new Linux users.

And no, it's not a joke. You can actually do this on a real machine...

## Why?

Because:
 * what if you wanted a single filesystem
 * what if you ran out of filesystems to create (can happen on MBR pretty easy)
 * what if you don't like partitioning and are scared of messing up your data again
 * what if you like to preserve space
 * makes it easier to access Linux files from Windows (because ext2fsd broke as fuck)
 * some other reason that I can't think of right now...
