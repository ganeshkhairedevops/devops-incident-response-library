# Jenkins Job Setup: Automated Database Backup

## Objective
Create a Jenkins job named **database-backup** to:
- Take a MySQL dump of database `kodekloud_db01` from `stapp01`
- Save it as `db_$(date +%F).sql`
- Copy it to `/home/natasha/db_backups` on `ststor01`
- Run the job every 10 minutes

---

## Prerequisites
- Jenkins access:
  - Username: `admin`
  - Password: `Adm!n321`
- SSH access configured between:
  - Jenkins → stapp01
  - Jenkins → ststor01

---

## Step 1: Login to Jenkins
1. Open Jenkins UI
2. Login using provided credentials

---

## Step 2: Create Jenkins Job
1. Click **New Item**
2. Enter job name: `database-backup`
3. Select **Freestyle Project**
4. Click **OK**

---

## Step 3: Configure Build Step

Go to:
**Build → Add Build Step → Execute Shell**

Add the following script:

```bash
#!/bin/bash

DB_NAME=kodekloud_db01
DB_USER=kodekloud_roy
DB_PASS=asdfgdsd
DATE=$(date +%F)
DUMP_FILE=db_${DATE}.sql

# Take DB dump from application server
ssh tony@stapp01 "mysqldump -u ${DB_USER} -p${DB_PASS} ${DB_NAME}" > ${DUMP_FILE}

# Copy dump to storage server
scp ${DUMP_FILE} natasha@ststor01:/home/natasha/db_backups/
```
## Step 4: Configure Build Trigger
1. Go to **Build Triggers**
2. Enable **Build periodically**
3. Add the cron schedule:
```bash
*/10 * * * *
```
## Step 5: Setup Passwordless SSH (Important)

### Switch to Jenkins user:
```bash
sudo su - jenkins
```
### Generate SSH key:
```bash
ssh-keygen -t rsa
```
### Copy key to App Server:
```bash
ssh-copy-id tony@stapp01
```
Password:
```bash
Ir0nM@n
```
### Copy key to Storage Server:
```bash
ssh-copy-id natasha@ststor01
```
Password: 
```bash
Bl@kW
```
## Step 6: Fix Host Key Verification
Run:
```bash
ssh tony@stapp01
ssh natasha@ststor01
```
Type yes when prompted.

## Step 7: Test Connectivity
```bash
ssh tony@stapp01 hostname
ssh natasha@ststor01 hostname
```
Expected:

- No password prompt
- Commands execute successfully

## Step 8: Save and Run Job
1. Click **Save**
2. Click **Build Now**

## Step 9: Verify Backup

Login to storage server:
```bash
ssh natasha@ststor01
ls -l /home/natasha/db_backups
```
Expected output:

db_YYYY-MM-DD.sql

---

## Final Outcome
- Automated database backups created
- Files transferred securely
- Job runs every 10 minutes without manual intervention