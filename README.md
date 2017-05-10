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

Run the set of scripts to perform the first backup and check they are working.  This may take a while if you have a lot of data to backup.

    ~/backup/bin/backup-daily.cron

    ~/backup/bin/backup-weekly.cron

    ~/backup/bin/backup-monthly.cron

You should now have folders `daily.0`, `weekly.0`, `monthly.0`.  Running `ls -l` will show you the backup date and time of each.  If you navigate into one, you should see the entire folder as it was at the time of the backup.

Run them again:

    ~/backup/bin/backup-daily.cron

    ~/backup/bin/backup-weekly.cron

    ~/backup/bin/backup-monthly.cron

Now you should also have folders `daily.1`, `weekly.1`, `monthly.1`.  The scripts will maintain daily snapshots for the past four days, weekly snapshots for the past four weeks, and monthly snapshots for the past four months. 

If everything looks ok, add these scripts to your cron table.  Here we pick some random times in the evening to avoid all the backups occuring at the same time:

    min=`shuf -i 0-59 -n1`
    daily_hour=`shuf -i 21-23 -n1`
    weekly_hour=$(( $(( $daily_hour + 1 )) % 24 ))
    monthly_hour=$(( $(( $weekly_hour + 1 )) % 24 ))
    crontab -l > crontab.backup
    echo "$min $daily_hour * * * $HOME/backup/bin/backup-daily.cron &> /dev/null" >> crontab.backup
    echo "$min $weekly_hour * * * $HOME/backup/bin/backup-weekly.cron &> /dev/null" >> crontab.backup
    echo "$min $monthly_hour * * * $HOME/backup/bin/backup-weekly.cron &> /dev/null" >> crontab.backup
    crontab crontab.backup

And you're done!