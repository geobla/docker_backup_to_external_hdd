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
DOCKER_DATA="/var/lib/docker"
COMPOSE_CONFIGS="/home/your_username/Docker"  # Adjusted path

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

## Restore your Docker-related data from the backup

To reinstall Docker and restore your Docker-related data from the backup after formatting your local server or in the event of a server crash, you can follow these steps:

### Install Docker

If Docker is not already installed on your freshly formatted server, you'll need to install it.  

You can follow the official Docker installation instructions for Ubuntu: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

### Mount Your External HDD

Ensure that your external HDD is connected to your server and properly mounted. You can use the `mount` command

To specify which device is your external HDD when using the `mount` command, you can check the output of the `lsblk` or `fdisk -l` command to identify the device name associated with your external HDD. 

Once you've identified the correct device, you can use it in the `mount` command.

Here are the steps:

1. Open a terminal on your Linux server.

2. Run either `lsblk` or `fdisk -l` to list all the block devices, including the external HDD. These commands will display information about the devices connected to your system.

   ```bash
   lsblk
   ```

   Look for the entry that corresponds to your external HDD. It will typically have a size and partition information that matches your HDD.

3. Once you've identified the device name (e.g., `/dev/sdb`, `/dev/sdc`, etc.) associated with your external HDD, you can use it in the mount command to mount the HDD to a specific directory. 

   For example, to mount it to `/media/myhdd`, you can use:

   ```bash
   sudo mount /dev/sdX /media/myhdd
   ```

   Replace `/dev/sdX` with the actual device name of your external HDD.

   Be <ins>extremely careful</ins> to specify the correct device name, as mounting the wrong device can result in data loss.

5. After mounting, you can use the mount command without any arguments to verify that the external HDD is mounted:

   ```bash
   mount
   ```

   You should see an entry indicating that your external HDD is mounted at the specified mount point.


### Locate the Backup

Locate the backup directory on your external HDD that contains the Docker-related data. For example, if you used the provided backup script, the backups would be stored in `/media/myhdd/backup`.

### Use the rsync command to restore Docker-related data from your backup while ensuring correct permissions and ownership

### Containers Data

Assuming your containers data is in the backup directory `backup_containers`, use `rsync` to copy the contents back to `/var/lib/docker/containers`. 

Ensure you replace backup_containers with the actual directory name in your backup:

```bash
sudo rsync -avP /path/from/external/drive/backup_containers/ /var/lib/docker/containers/
```

### Docker Compose Configurations

If you had Docker Compose stack configurations in your backup directory `backup_compose`, copy them back to their original location. Replace `backup_compose` with your actual backup directory:

```bash
sudo rsync -avP /path/from/external/drive/backup_compose/ /home/your_username/Docker/
```

Replace `your_username` with your actual username.

### Docker Volumes

For Docker volumes, copy the contents of the volumes directory from the backup to `/var/lib/docker/volumes`. Replace `backup_volumes` with your actual backup directory:

```bash
sudo rsync -avP /pathfrom/external/drive/backup_volumes/ /var/lib/docker/volumes/
```

### Permissions and Ownership

After copying the data, you may need to set the correct permissions and ownership to ensure Docker can access the restored data. Run the following commands for each of the directories you copied data to:

For example, if you restored containers data:

```bash
sudo chown -R root:root /var/lib/docker/containers
```

For Docker Compose configurations:

```bash
sudo chown -R your_username:your_username /home/your_username/Docker
```

Replace your_username with your actual username.

And for Docker volumes:

```bash
sudo chown -R root:root /var/lib/docker/volumes
```

### Start Docker

Once you have restored the Docker-related data, you can start Docker:

```bash
sudo systemctl start docker
```

### Check Docker Containers

To ensure that your Docker containers are up and running, you can use the docker ps command:

```bash
docker ps
```

### Test Applications

Verify that your Dockerized applications and services are functioning as expected.

By following these steps, you should be able to restore your Docker-related data from the backup and ensure that permissions and ownership are correctly set for Docker to work with the restored data.

## Considerations

Docker images are not included in this backup by default. Decide whether to back up Docker images based on your requirements.

## Conclusion

You now have a Docker backup solution in place that performs daily incremental backups of Docker-related data to your external HDD.

Happy backing up!





























