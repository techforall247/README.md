# AWS Cloud Engineer Roadmap

Essential shell scripting tasks for AWS Cloud Engineers to automate routine tasks, improving efficiency and reliability in managing backups, monitoring system health, and ensuring service uptime.

### 1. File Backup with Datestamp and Send Mail Notification

Creates a backup, timestamps it, and sends an email notification.

```bash
#!/bin/bash
TIME=$(date +%b-%d-%y)
FILENAME=backup-$TIME.tar.gz
SRCDIR=/imp-data
DESDIR=/mybackupfolder

tar -cpzf $DESDIR/$FILENAME $SRCDIR

if [ $? -eq 0 ]; then
    echo "Backup Successful: $FILENAME" | mailx -s "Backup Status" itsupport@gmail.com
else
    echo "Backup Failed" | mailx -s "Backup Status" itsupport@gmail.com
    exit 1
fi

find $SRCDIR -mindepth 1 -mtime +1 -delete
```

### 2. MySQL DB Backup, Copy to AWS S3, and Delete Old Backups

Performs a MySQL backup, uploads it to S3, and deletes the local backup.

```bash
#!/bin/bash

FILE=DB_dump_$(date +%Y%m%d).sql

mysqldump -u root -p password target_db > /backup/$FILE
gzip /backup/$FILE
aws s3 cp /backup/$FILE.gz s3://targetbucket/SQL/$(date +%Y%m%d)/
rm -rf /backup/$FILE.gz /backup/$FILE

exit 0
```

### 3. MySQL DB Backup Locally and Send Mail Notification

Creates a local MySQL backup, compresses it, and sends an email notification.

```bash
#!/bin/bash

filename=live-DB-$(date +%Y-%m-%d).sql

mysqldump -u root -p password target_db > /backup/DB/$filename
zip /backup/DB/$filename.zip /backup/DB/$filename
rm -rf /backup/DB/$filename
echo "MySQL backup completed" | mail -s "MySQL Backup" itsupport@gmail.com
find /backup/DB/ -mindepth 1 -mtime +3 -delete
```

### 4. Disk Usage Check and Send Mail Notification

Checks if disk usage exceeds 80% and sends an email alert if it does.

```bash
#!/bin/bash
CURRENT=$(df / | grep / | awk '{print $5}' | sed 's/%//g')
THRESHOLD=80

if [ "$CURRENT" -gt "$THRESHOLD" ]; then
    mail -s 'Disk Space Alert' itsupport@gmail.com <<EOF
Root partition free space is critically low. Used: $CURRENT%
EOF
fi
```

### 5. Service Up or Down Check and Mail Notification

Checks if a service (e.g., Apache) is running, sends an alert if it is down, and attempts to start it.

```bash
#!/bin/bash
UP=$(systemctl is-active apache2)

if [ "$UP" != "active" ]; then
    echo "Webserver is down." | mail -s "Webserver Down" itsupport@gmail.com
    sudo systemctl start apache2
fi
```

### 6. Check Server Memory and Send Mail Notification

Checks available memory and sends an alert if it falls below a threshold.

```bash
#!/bin/bash
subject="Server Memory Alert"
To="server.monitor@example.com"
free=$(free -mt | grep Total | awk '{print $4}')

if [ "$free" -le 100 ]; then
    echo -e "Warning: Low memory!\n\nFree memory: $free MB" | mail -s "$subject" $To
fi

exit 0
```

These shell scripts enhancing the reliability and efficiency of your cloud infrastructure. Adapt these scripts to fit your specific needs and boost your productivity.