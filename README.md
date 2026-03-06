# ec2 backup automation with s3

## overview
this project demonstrates how to automate backups of a project hosted on an ec2 instance and store them in an s3 bucket using aws cli, bash scripts, and cron.

the system performs:
- automatic code update from github
- automatic backup creation
- upload backup to s3
- restore backup from s3 when required

---

## architecture

local machine
    │
    │ git push
    ▼
github repository
    │
    ▼
ec2 instance
    │
    ├─ update-site.sh (cron job)
    │
    ├─ create backup (.tar.gz)
    │
    ▼
s3 bucket (backup storage)

---

## step 1 – launch ec2 instance

launch a t2.micro instance from aws console and connect using ssh.


ssh -i key.pem ec2-user@public-ip


---

## step 2 – install aws cli


sudo yum update -y
sudo yum install aws-cli -y


verify:


aws --version


---

## step 3 – create project folder


mkdir flower-shop
cd flower-shop


add sample files for testing.

---

## step 4 – create s3 bucket

create an s3 bucket from aws console to store backups.

verify connection:


aws s3 ls


---

## step 5 – backup script

create `backup.sh`


#!/bin/bash

DATE=$(date +%F-%H-%M-%S)
BACKUP_FILE="flower-shop-$DATE.tar.gz"

tar -czvf $BACKUP_FILE /home/ec2-user/flower-shop

aws s3 cp $BACKUP_FILE s3://your-bucket-name/


make executable:


chmod +x backup.sh


---

## step 6 – restore script

create `restore.sh`


#!/bin/bash

FILE=$1

aws s3 cp s3://your-bucket-name/$FILE .

tar -xzvf $FILE


make executable:


chmod +x restore.sh


restore example:


./restore.sh backup-file.tar.gz


---

## step 7 – automation with cron

edit cron:


crontab -e


add:


*/5 * * * * /home/ec2-user/update-site.sh


cron runs every 5 minutes to update the project and create backups.

---

## validation

check backups locally:


ls ~/backups


check backups in s3:


aws s3 ls s3://your-bucket-name/


restore backup if needed:


./restore.sh backup-file-name.tar.gz


---

## technologies used

- aws ec2
- aws s3
- aws cli
- bash scripting
- cron
- git