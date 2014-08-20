lvm-rsync-backup
================

Bash scripts for efficiently and (relatively) safely backing up a
linux system using rsync.

This branch does not use LVM.  LVM provides a bit more safety,
especially for highly active systems, by ensuring files can't change
as the backup progresses.  However, for many purposes this is not
so much of an issue.

These scripts produce a set of folders, each of which (apparently) contain
a full copy of the backed-up filesystem at particualar times in the
past.  In reality, the files are hardlinked, so they only take up the
space of the oldest backup plus any changes made since.

The scripts aren't perfect, and may need some tweaking to work with
your system.

Usage
-----

The main script is `backup-daily`:

    backup-daily FROM_DIR TO_DIR

For example, to backup everything below `/home/username` to another
physical volume mounted at `/mnt/backup` the command is:

    backup-daily /home/username /mnt/backup

The other scripts, `backup-weekly` and `backup-monthly`, simply delete
the oldest backups and rename intermediate backups.

For automated use, put the scripts somewhere convenient,
e.g. `/etc/backups/`, and set up a cron job to run them with
appropriate frequency (see the *.cron examples).
