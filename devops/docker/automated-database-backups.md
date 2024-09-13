# 🗄️ Automated Database Backups with Docker, Cron, and Google Cloud Storage

Welcome to the guide on setting up automated database backups using Docker, cron, and Google Cloud Storage (GCS). This solution is particularly useful for maintaining regular backups of your PostgreSQL database running in a Docker container.

## 🚀 Prerequisites

Before we begin, make sure you have:

- Docker installed on your system
- A Google Cloud Platform (GCP) account
- The `gcloud` CLI tool installed
- `gcsfuse` installed using [GCP docs]( https://cloud.google.com/storage/docs/gcsfuse-quickstart-mount-bucket#install)

## 📋 Step-by-Step Guide

### 🌟 Step 1: Set Up Google Cloud Storage

1. Create a new bucket in Google Cloud Storage.
2. Create a service account with necessary permissions to access the bucket.
3. Download the JSON key file for the service account.

### 🔑 Step 2: Configure Google Cloud SDK

1. Authenticate with your service account:

   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/keyfile.json"
   gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
   ```

2. Set your project ID:

   ```bash
   gcloud config set project YOUR_PROJECT_ID
   ```

### 🏔️ Step 3: Mount GCS Bucket

1. Create a directory to mount your GCS bucket:

   ```bash
   mkdir -p ~/Documents/auto-dump-db/mount_bucket_dir
   ```

2. Mount the bucket:

   ```bash
   gcsfuse YOUR_BUCKET_NAME ~/Documents/auto-dump-db/mount_bucket_dir
   ```

### 🔒 Step 4: Configure Passwordless Sudo for Docker Exec

1. Open the sudoers file:

   ```bash
   sudo visudo
   ```

2. Add the following line, replacing `your_username` with your actual username and `your_docker_container` with your actual container name:

   ```
   your_username ALL=(ALL) NOPASSWD: /usr/bin/docker exec -t your_docker_container pg_dumpall -c -U postgres
   ```

3. Save and exit the editor.

### 📜 Step 5: Create the Backup Script

1. Create a new file named `backup_db.sh`:

   ```bash
   nano ~/Documents/auto-dump-db/backup_db.sh
   ```

2. Add the following content:

   ```bash
   #!/bin/bash
   
   # Ensure the GCS bucket is mounted
   if ! mountpoint -q ~/Documents/auto-dump-db/mount_bucket_dir; then
     gcsfuse YOUR_BUCKET_NAME ~/Documents/auto-dump-db/mount_bucket_dir
   fi
   
   # Perform the database dump
   sudo /usr/bin/docker exec -t your_docker_container pg_dumpall -c -U postgres > ~/Documents/auto-dump-db/mount_bucket_dir/produzione-backup-database/dump_$(date +%d-%m-%Y_%H_%M_%S).sql
   
   # Check if the dump was successful
   if [ $? -eq 0 ]; then
     echo "Database backup completed successfully."
   else
     echo "Database backup failed." >&2
   fi
   ```

3. Make the script executable:

   ```bash
   chmod +x ~/Documents/auto-dump-db/backup_db.sh
   ```

### ⏰ Step 6: Set Up Cron Job

1. Open your crontab file:

   ```bash
   crontab -e
   ```

2. Add the following line to run the backup daily at 2:00 AM:

   ```
   0 2 * * * ~/Documents/auto-dump-db/backup_db.sh >> ~/Documents/auto-dump-db/backup.log 2>&1
   ```

### 🧪 Step 7: Testing

1. Test the passwordless sudo configuration:

   ```bash
   sudo /usr/bin/docker exec -t gimmidb pg_dumpall -c -U postgres > ~/Documents/auto-dump-db/mount_bucket_dir/produzione-backup-database/test_dump.sql
   ```

2. Test the backup script:

   ```bash
   ~/Documents/auto-dump-db/backup_db.sh
   ```

3. Check the GCS bucket for the new backup file.

## 🔍 Troubleshooting

- If the cron job fails, check the `~/Documents/auto-dump-db/backup.log` file for error messages.
- Ensure that the GCS bucket is properly mounted before running the backup.
- Verify that the Docker container is running and accessible.

## 🛡️ Security Considerations

- Keep your service account key file secure and restrict its permissions.
- Regularly rotate your service account keys.
- Monitor access to your GCS bucket and enable audit logging.

By following this guide, you'll have set up an automated system for backing up your PostgreSQL database to Google Cloud Storage. Remember to regularly test your backup and recovery process to ensure data integrity and availability.

Happy backing up! 🎉🗄️🚀
