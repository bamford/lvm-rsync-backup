lvm-rsync-backup
================

Bash scripts for efficiently and (relatively) safely backing up a
linux system using rsync.

This branch is a customised example for the Nottingham Astronomy 
`captain` system, although it would equally apply to most modern servers.

These scripts produce a set of folders, each of which (apparently)
contain a full copy of the backed-up filesystem at particualar times
in the past.  In reality, the files are hardlinked, so they only take
up the space of the oldest backup plus any changes made since.  It
therefore behaves similarly to OSX Time Machine.


Usage
-----

First ssh to captain (you may need to enter your password):

    ssh captain

If you haven't already got an ssh key on captain, create one:

    ssh-keygen

Copy your ssh key to the system you want to backup (called `mymachine` here):

    ssh-copy-id mymachine

Create a folder `backup` in your home folder (or elsewhere if you really want), containing another folder called `bin`:

    mkdir -p ~/backup/bin

Get the scripts:

    cd ~/backup/bin
    wget https://github.com/bamford/lvm-rsync-backup/archive/captain.zip
    unzip captain.zip
    mv lvm-rsync-backup-captain/* .
    rm -rf lvm-rsync-backup-captain captain.zip

Edit the `backup-daily.cron` script so it contains the correct machine, folder and filenames.  See the file for an example.

Edit the `backup-weekly.cron` and `backup-monthly.cron` scripts to match.

Run the daily script to perform the first backup and check it is working.  This may take a while if you have a lot of data to backup (subsequent backups will typically be much faster).

    ~/backup/bin/backup-daily.cron

This should produce a folder `daily.0`.

If everything looks ok, add these scripts to your cron table.  Here we pick some random times in the evening to avoid all the backups occuring at the same time:

    min=`shuf -i 0-59 -n1`
    daily_hour=`shuf -i 21-23 -n1`
    weekly_hour=$(( $(( $daily_hour + 1 )) % 24 ))
    monthly_hour=$(( $(( $weekly_hour + 1 )) % 24 ))
    crontab -l > crontab.backup
    echo "$min $daily_hour * * * $HOME/backup/bin/backup-daily.cron &> /dev/null" >> crontab.backup
    echo "$min $weekly_hour * * 6 $HOME/backup/bin/backup-weekly.cron &> /dev/null" >> crontab.backup
    echo "$min $monthly_hour 1 * * $HOME/backup/bin/backup-weekly.cron &> /dev/null" >> crontab.backup
    crontab crontab.backup

The scripts will now run automatically to maintain daily snapshots for
the past four days (`daily.{0,1,2,3}`), weekly snapshots for the past
four weeks (`weekly.{0,1,2,3}`), and monthly snapshots for the past
four months (`monthly.{0,1,2,3}`). Adapt the scripts if you want a
different arrangement. Each of these folders contains a snapshot of
the target folder as it was at the time of the backup. Running `ls -l`
will show you the backup date and time of each folder and the files
within.

That's it!  Check back occasionally to make sure the full set of
folders appears.  If you ever need to recover a file or folder from a
day, week or month ago, just find it in the appropriate folder and
copy (or rsync) it where you want it.