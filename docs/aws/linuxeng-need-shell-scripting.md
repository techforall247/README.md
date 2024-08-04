## Linux - AWS Cloud Engineer Roadmap: Essential Shell Scripting Tasks

Essential shell scripting tasks for AWS Cloud Engineers. 

These scripts help automate routine tasks, improving efficiency and reliability in managing backups, monitoring system health, and ensuring service uptime. 

Roadmap to get you started with shell scripting for common administrative tasks.

### 1. File Backup with Datestamp and Send Mail Notification

This script creates a backup of a specified directory, timestamps the backup file, and sends an email notification about the status of the backup process.

```bash
#!/bin/bash
TIME=`date +%b-%d-%y`
FILENAME=backup-$TIME.tar.gz
SRCDIR=/imp-data
DESDIR=/mybackupfolder

tar -cpzf $DESDIR/$FILENAME $SRCDIR

if [ "$?" = "0" ]; then
    echo "Backup Process was Successful. A new backup file $FILENAME has been created" | mailx -s "Backup Status Successful" itsupport@gmail.com
else
    echo "Backup Process Failed. Please contact System Administrator" | mailx -s "Backup Status Failed" itsupport@gmail.com
    exit 1
fi

find /imp-data -mindepth 1 -mtime +1 -delete
```

### 2. MySQL DB Backup, Copy to AWS S3, and Delete Old Backups

This script performs a backup of a MySQL database, compresses it, uploads it to an AWS S3 bucket, and then deletes the local backup file.

```bash
#!/bin/bash

FILE=DB_dump_`date +%Y%m%d`.sql

/usr/bin/mysqldump -u root -p password target_db > /backup/$FILE

/bin/gzip /backup/$FILE

/usr/local/bin/aws s3 cp /backup/$FILE.gz s3://targetbucket/SQL/`date +%Y%m%d`/

/bin/rm -rf /backup/$FILE.gz /backup/$FILE

exit 0
```

### 3. MySQL DB Backup Locally and Send Mail Notification

This script creates a local backup of a MySQL database, compresses it, and sends an email notification upon completion.

```bash
#!/bin/bash

filename=live-DB-`date +%Y-%m-%d`.sql

/usr/bin/mysqldump -u root -p password target_db > /backup/DB/$filename

/usr/bin/zip /backup/DB/$filename.zip /backup/DB/$filename

rm -rf /backup/DB/$filename

echo "example.com Mysql-backup completed" | mail -s Mysql-Backup itsupport@gmail.com

find /backup/DB/ -mindepth 1 -mtime +3 -delete
```

### 4. Disk Usage Check and Send Mail Notification

This script checks if disk usage exceeds 80% and sends an email alert if it does.

```bash
#!/bin/bash
CURRENT=$(df / | grep / | awk '{ print $5}' | sed 's/%//g')
THRESHOLD=80

if [ "$CURRENT" -gt "$THRESHOLD" ]; then
    mail -s 'Disk Space Alert' itsupport@gmail.com <<EOF
Your root partition remaining free space is critically low on AOL appserver. Used: $CURRENT%
EOF
fi
```

### 5. Service Up or Down Check and Mail Notification

This script checks if a service (e.g., Apache) is running and sends an email alert if it is down, then attempts to start the service.

```bash
#!/bin/bash
UP=$(/etc/init.d/apache2 status | grep running | grep -v not | wc -l)
if [ "$UP" -ne 1 ]; then
    echo "webserver is down on AOL." | mail -s "webserver is down on AOL." itsupport@gmail.com
    sudo service apache2 start
else
    echo "All is well."
fi
```

### 6. Check Server Memory and Send Mail Notification

This script checks if the available memory falls below a certain threshold and sends an alert if it does.

```bash
#!/bin/bash
subject="Server Memory Status Alert"
To="server.monitor@example.com"

free=$(free -mt | grep Total | awk '{print $4}')

if [[ "$free" -le 100 ]]; then
    echo -e "Warning, server memory is running low!\n\nFree memory: $free MB" | mail -s "$subject" $To
fi
exit 0
```

These shell scripts are essential tools for any AWS Cloud Engineer. 

They automate critical tasks such as backups, system monitoring, and service management, helping to ensure smooth and efficient operation of your cloud infrastructure. 

Implement these scripts and adapt them to fit your specific needs to enhance your system’s reliability and your productivity.

Stay tuned for more tips and tutorials on shell scripting and cloud management!
