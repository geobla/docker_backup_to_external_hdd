# Docker Backup on Ubuntu Server to External HDD

## Introduction

This guide outlines the steps to set up a backup solution for Docker-related data on your local Ubuntu server. The backup will be stored on an external HDD.

## Prerequisites

Before you begin, ensure the following:

- Ubuntu Server is installed and running on your server.
- Docker is installed and configured.
- You have connected an external HDD to your server and it's mounted, e.g., at `/media/myhdd`.

## Backup Strategy

We will create a backup script that performs incremental backups of Docker-related data, including container data, Docker Compose stack configurations, and Docker volumes.

## Setup

### Switch to the root user's environment:

```bash
sudo su
```

You may be prompted to enter your user password to confirm your identity.

After running sudo su, your terminal prompt should change to indicate that you are now the root user. It might look something like this:

```bash
root@your-server-name:~#
```

### Identify Docker Data Directories

Determine the Docker-related data directories that you want to back up. These typically include:

- Docker container data: `/var/lib/docker/containers`
- Docker Compose stack configurations: `/home/your_username/Docker` (Adjust the path as needed)
- Docker volumes: `/var/lib/docker/volumes`

### Create a Backup Directory

Create a directory on your external HDD where backups will be stored, e.g., `/media/myhdd/backup`.

```bash
mkdir -p /media/myhdd/backup
```

### Create a Backup Script Directory:
Create a directory specifically for your backup scripts. You can place it in your home directory or another location that suits your organization. Let's create a directory named `backup-scripts` in your home directory:

```bash
mkdir ~/backup-scripts
```

### Go to the directory:

```bash
cd ~/backup-scripts
```

### Create the Backup Script:

Create a shell script to perform the backup. You can use a text editor like `nano` or `vim` to create a file named `backup_docker.sh`.

```bash
nano backup_docker.sh
```

### Import the Backup Script code:

```bash
#!/bin/bash

# Paths to Docker-related data
CONTAINER_DATA="/var/lib/docker/containers"
COMPOSE_CONFIGS="/home/your_username/Docker"  # Adjusted path
DOCKER_VOLUMES="/var/lib/docker/volumes"

# Destination backup directory on USB HDD
BACKUP_DIR="/media/myhdd/backup"

# Backup timestamp
TIMESTAMP=$(date +%Y%m%d%H%M%S)

# Create a directory for this backup
BACKUP_DEST="$BACKUP_DIR/docker_backup_$TIMESTAMP"
mkdir -p "$BACKUP_DEST"

# Perform the backup using rsync
rsync -avP "$CONTAINER_DATA" "$COMPOSE_CONFIGS" "$DOCKER_VOLUMES" "$BACKUP_DEST"

# Optional: Add a timestamp file to indicate when the backup was created
echo "Backup created on $(date)" > "$BACKUP_DEST/backup_timestamp.txt"
```

### Set Script Permissions:
Ensure that the script is executable:

```bash
chmod +x ~/backup-scripts/backup_docker.sh
```

### Run the Backup Script:
Execute the script to perform the backup.

```bash
./backup_docker.sh
```

This script will create a timestamped directory under `/media/myhdd/backup`, copy the Docker-related data to that directory, and create a timestamp file for reference.

### Verify the Backup
Check the backup directory to ensure that the data was copied successfully:

```bash
ls /media/myhdd/backup
```

You should see a directory with a timestamped name containing the backed-up Docker data.

This script will create a point-in-time backup of your Docker data and configurations. Monitor your backups and test the restoration process periodically to ensure data integrity.

## Schedule it to run periodically using cron to ensure regular backups

### Edit the Crontab

Open your user's crontab file for editing. Use the following command to open it in your default text editor:

```bash
crontab -e
```

### Schedule the Daily Incremental Backup

Add a line to the crontab file to schedule your daily incremental backup. For example, to run the backup script every day at midnight, you can use the following line:

```bash
0 0 * * * /bin/bash ~/backup-scripts/backup_docker.sh
```

This line specifies that the script should run at 00:00 (midnight) every day. You can adjust the schedule according to your preferences. 

### The cron format is as follows

```sql
Minute (0 - 59) Hour (0 - 23) Day of Month (1 - 31) Month (1 - 12) Day of Week (0 - 7, where both 0 and 7 represent Sunday)
```

After making your changes, save and exit the text editor.

### Verify Crontab Entry

To verify that your cron job is scheduled correctly, you can list your user's crontab entries:

```bash
crontab -l
```

This command will display the scheduled cron jobs for your user, including the newly added backup job.

Your backup script is now scheduled to run daily at the specified time, performing incremental backups of your Docker-related data.

### Exit root user

When you're done with the root session, you can exit the root shell and return to your regular user session by typing:

```bash
exit
```

This will bring you back to your user's shell.

## Considerations

Docker images are not included in this backup by default. Decide whether to back up Docker images based on your requirements.

## Conclusion

You now have a Docker backup solution in place that performs daily incremental backups of Docker-related data to your external HDD.

Happy backing up!





























