---
title: Taking Jenkins Backup Using Console
weight: 3
---
# Jenkins Backup – Backup/Upload/Restore

## Manual Backup

- Go to "Manage Jenkins" and select "ThinBackup".
- Click on "Backup Now" to create a backup under say "/opt/jenkins-backup". You can define any backup under any path.

## Automated Backup

- Go to "Manage Jenkins" and select "ThinBackup".
- Click on "Settings".

- Under "Backup schedule for full backups", specify the cron notation for the schedule. Currently set to "H 2 * * *".

## Upload Backup

- Job [Jenkins-Backup](http://hostname:8080/job/Jenkins-Backup/) uploads the daily backup to GCP bucket "strg-bakp-jnk" under "jenkins-backup".
- Currently set to "H 3 * * *".

## Restore Backup

- Go to "Manage Jenkins" and select "ThinBackup".
- Click on "Restore".

- Select the backup you want to restore from the dropdown.
- Select the option "Restore plugins".

- Click on "Restore".