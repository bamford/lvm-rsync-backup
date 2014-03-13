lvm-rsync-backup
================

Bash scripts for efficiently and safely backing up a linux system
using LVM snapshots and rsync.

This produces a set of folders, each of which (apparently) contain a
full copy of the backed-up filesystem at particualar times in the
past.  In reality, the files are hardlinked, so they only take up the
space of the oldest backup plus any changes made since.

The scripts aren't perfect, and may need some tweaking to work with
your system.

Usage
-----

You need LVM set up, with enough unallocated space in volume groups to
create logical volumes for the backup and the temporary snapshop
buffer.  The backup size must be large enough to contain all the data
in the logical volume to be backed up and any changes that may occur
over the period that the backups are kept for.  The buffer size just
needs to be large enough to contain any changes to the logical volume
that is being backed up.

The main script is `backup-daily`:
    backup-daily BACKUP_VOLUME BACKUP_MOUNT BACKUP_SIZE BUFFER_SIZE

For example, say you have a filesystem on the logical volume device
`/dev/mapper/vg_system-lv_home` and you wish to create a backup
mounted at `/mnt/backup` with 64Gb of backup space and 592Mb of buffer
space, the command is:
    backup-daily /dev/mapper/vg_system-lv_home /mnt/backup 64G 592M

The other scripts, `backup-weekly` and `backup-monthly`, simply delete
the oldest backups and rename intermediate backups.

For automated use, put the scripts somewhere convenient,
e.g. `/etc/backups/`, and set up a cron job to run them with
appropriate frequency (see the *.cron examples).
