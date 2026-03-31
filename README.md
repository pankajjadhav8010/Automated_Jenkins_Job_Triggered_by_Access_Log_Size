# Automated Jenkins Job Triggered by Access Log Size

## Project Overview

This project implements an automated log monitoring and archival system designed to prevent server issues caused by oversized access logs. When the log file exceeds a defined threshold (1 GB), a monitoring script automatically triggers a Jenkins job. The job uploads the log file to Amazon S3 and clears it locally to free disk space.

## Objective

* Monitor access log size continuously
* Trigger a Jenkins job automatically when the log exceeds a threshold
* Upload logs to Amazon S3 for storage
* Clear the log file after successful upload
* Ensure reliability, logging, and automation

## Architecture Overview

### Components Used

* Shell Script (Monitoring)
* Jenkins (Automation Server)
* AWS S3 (Storage)
* Cron (Scheduling)
* AWS CLI

## Workflow

1. Cron runs the monitoring script at regular intervals
2. The script checks the size of the access log file
3. If the file size exceeds 1 GB, the script triggers a Jenkins job via API
4. Jenkins pipeline performs the following steps:

   * Creates a backup of the log file
   * Uploads the backup to S3
   * Verifies the upload
   * Clears the original log file
5. Logs are recorded for tracking and debugging

## Project Structure

```
.
├── monitor_log.sh
├── Jenkinsfile
├── backup.log
├── screenshots/
└── README.md
```

## Monitoring Script

### Description

The script checks the size of the Apache access log and triggers a Jenkins job if the size exceeds the defined threshold.

### Script

```bash
#!/bin/bash

LOG_FILE="/var/log/httpd/access.log"
THRESHOLD=$((1024 * 1024 * 1024))
JENKINS_URL="http://<jenkins-url>/job/log-archive/build"
USER="your_username"
API_TOKEN="your_api_token"
LOG_OUTPUT="/home/ec2-user/backup.log"

if [ ! -f "$LOG_FILE" ]; then
    echo "$(date) - Log file not found" >> $LOG_OUTPUT
    exit 1
fi

FILE_SIZE=$(stat -c%s "$LOG_FILE")

echo "$(date) - Current size: $FILE_SIZE" >> $LOG_OUTPUT

if [ "$FILE_SIZE" -ge "$THRESHOLD" ]; then
    echo "$(date) - Threshold exceeded. Triggering Jenkins job" >> $LOG_OUTPUT

    CRUMB=$(curl -s --user "$USER:$API_TOKEN" \
    "$JENKINS_URL/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)")

    curl -X POST "$JENKINS_URL" \
    --user "$USER:$API_TOKEN" \
    -H "$CRUMB"
else
    echo "$(date) - Log size within limit" >> $LOG_OUTPUT
fi
```

## Jenkins Pipeline Configuration

### Description

The Jenkins pipeline handles backup, upload, verification, and cleanup of the log file.

### Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        LOG_FILE = "/var/log/httpd/access.log"
        S3_BUCKET = "your-s3-bucket-name"
    }

    stages {
        stage('Backup Log') {
            steps {
                sh '''
                cp $LOG_FILE /tmp/access.log
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                sh '''
                aws s3 cp /tmp/access.log s3://$S3_BUCKET/access-$(date +%F-%H-%M-%S).log
                '''
            }
        }

        stage('Verify Upload') {
            steps {
                sh '''
                aws s3 ls s3://$S3_BUCKET/
                '''
            }
        }

        stage('Clear Log File') {
            steps {
                sh '''
                > $LOG_FILE
                '''
            }
        }
    }

    post {
        success {
            echo 'Log archived and cleared successfully'
        }
        failure {
            echo 'Job failed. Check logs'
        }
    }
}
```

## Automation with Cron

Edit crontab:

```
crontab -e
```

Add the following line to run the script every 5 minutes:

```
*/5 * * * * /bin/bash /path/to/monitor_log.sh >> /path/to/backup.log 2>&1
```

## Verification Steps

### Verify S3 Upload

```
aws s3 ls s3://your-s3-bucket-name/
```

### Verify Log File Cleared

```
cat /var/log/httpd/access.log
```

The output should be empty after successful execution.

## Security Considerations

* Avoid hardcoding credentials in scripts
* Use Jenkins credentials store for API tokens
* Use IAM roles instead of access keys when running on EC2
* Restrict script permissions using:

  ```
  chmod 700 monitor_log.sh
  ```

## Optional Enhancements

* Compress logs before uploading to S3
* Add email or Slack notifications from Jenkins
* Parameterize the Jenkins job
* Integrate monitoring with CloudWatch
* Use logrotate for standard log management

## Key Learnings

* Automating system monitoring using shell scripting
* Integrating Jenkins with external systems via API
* Using AWS CLI for storage operations
* Designing fault-tolerant workflows
* Handling real-world infrastructure challenges

## Conclusion

This project demonstrates a practical approach to managing large log files using automation and cloud services. It helps prevent disk space issues and ensures that logs are safely archived for future use.
