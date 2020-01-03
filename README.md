# DevOps Foundation LAB 1 (quick tutorial)
## Requirements

Make a local backup scheme using rsync.

Requirements list:

 - Make a local backup scheme using *rsync*.
 - *Compress* backed up data and have a single archive file as a result.
 - Use *crontab* for scheduling daily backup at 11:00pm.
 - Write logs to /var/logs/

## 1. myBackuper (bash-script)

    #!/bin/bash
    
    # use "man rsync" to learn more about certain parameters used below in this partucular case
    # note: do not use --compress parameter in case of local backup scheme, as it doesn't add any effectivness and just uses your server's CPU resources to compress and decompress data on a fly. This parameter has nothing to do with archiving direcotry to a single compressed file. This parameter can be handy only for remote backup schemes when a network connection is a bottle-neck.
    
    # creating a directory for rsync to backup to (in case it doesn't exist).
    mkdir -p /media/myBackuper_data
    
    # creating a directory for rsync to store log files in (in case it doesn't exist).
    mkdir -p /var/log/myBackuper
    
    sudo rsync -av --progress --delete-excluded --log-file=/var/log/myBackuper/$(date +"%d-%m-%Y_%H-%M")_myBackuper.log --exclude "/home/*/not_important_files" /home /media/myBackuper_data
    
    # creating a directory for storing the final archive (.tar.gz) file in case it doesn't exist.
    mkdir -p /media/myBackuper_archives
    
    # compressing the already backed up data to a single archive file (using gzip as a compression method).
    tar -zcvf /media/myBackuper_archives/$(date +"%d-%m-%Y_%H-%M")_myBackuper.tar.gz /media/myBackuper_data


We need to give ability (to all users, in our demonstrational case) to execute the above script by setting the following permission:

> chmod +x myBackuper.sh

## 2. Cron job

> Without root privileges *rsync* will not be able to copy all the content of /home directory.
> So, here is a lifehack to that:

 - sudo cp /home/you-user-account/my_scripts/myBackuper.sh /root
 - sudo crontab -e

 Add following line at the end of the file:

 - 0 23 * * * /root/myBackuper.sh

This will run our script every day at 11:00pm.

**P.S.** Make sure, your system's local time (and timezone!) and date are set correctly. Use *date* command to see the current time and date. In case you need to adjust it on Ubuntu, follow [these instructions](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29).
