⚙️ Automated Jenkins Job Triggered by Access Log Size
📌 Project Overview

This project implements an automated log monitoring and archival system to prevent server crashes caused by oversized access logs.

When the log file exceeds a defined threshold (1GB), a script automatically triggers a Jenkins job that uploads the log to Amazon S3 and clears the file to free disk space.

🎯 Objective
Monitor access log size in real-time
Automatically trigger a Jenkins job when size exceeds threshold
Upload logs to S3 for storage
Clear log file after successful upload
Ensure automation, logging, and reliability
🏗️ Architecture Overview

Components Used:

Shell Script (Monitoring)
Jenkins (Automation Server)
Amazon Web Services (S3 Storage)
Cron (Scheduling)
AWS CLI

⚙️ Workflow

Cron runs monitoring script every few minutes
Script checks log file size
If size > 1GB → triggers Jenkins job
Jenkins job:
Uploads log file to S3
Verifies upload
Clears log file
Logs and notifications generated

📂 Project Structure

.
├── monitor_log.sh          # Monitoring script
├── Jenkinsfile            # Pipeline definition
├── backup.log             # Execution logs
├── screenshots/           # Proof of execution
└── README.md

🧪 Monitoring Script

📌 Function:
Checks /var/log/httpd/access.log
Triggers Jenkins job via API if size exceeds 1GB
🧾 Sample Script:
#!/bin/bash

LOG_FILE="/var/log/httpd/access.log"
THRESHOLD=$((1024 * 1024 * 1024))  # 1GB
JENKINS_URL="http://<jenkins-url>/job/log-archive/build"
API_TOKEN="your_api_token"
USER="your_username"

FILE_SIZE=$(stat -c%s "$LOG_FILE")

if [ "$FILE_SIZE" -ge "$THRESHOLD" ]; then
    echo "Threshold exceeded. Triggering Jenkins job..."
    
    curl -X POST "$JENKINS_URL" \
    --user "$USER:$API_TOKEN"
else
    echo "Log size within limit."
fi
🔧 Jenkins Job Configuration
Option 1: Pipeline Job (Recommended)
pipeline {
    agent any

    environment {
        LOG_FILE = "/var/log/httpd/access.log"
        S3_BUCKET = "your-s3-bucket-name"
    }

    stages {
        stage('Upload to S3') {
            steps {
                sh '''
                aws s3 cp $LOG_FILE s3://$S3_BUCKET/access.log
                '''
            }
        }

        stage('Verify Upload') {
            steps {
                sh '''
                aws s3 ls s3://$S3_BUCKET/access.log
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
            echo 'Log archived and cleared successfully.'
        }
        failure {
            echo 'Job failed. Check logs.'
        }
    }
}
⏱️ Automation with Cron


Edit crontab:

crontab -e
Run every 5 minutes:
*/5 * * * * /bin/bash /path/to/monitor_log.sh >> /path/to/backup.log 2>&1
📊 Verification Steps
✅ S3 Upload
Check file in S3 bucket:
aws s3 ls s3://your-s3-bucket-name/
✅ Log Cleared
cat /var/log/httpd/access.log
Output should be empty

📸 Deliverables

✅ Monitoring script (monitor_log.sh)
✅ Jenkins Pipeline (Jenkinsfile)
✅ Screenshot of Jenkins job success
✅ Screenshot of file in S3
✅ Proof of cleared log file
📢 Optional Enhancements
📧 Email Notification
Configure Jenkins → “Post-build Actions”
Add Email Notification plugin
🔄 Parameterized Job
parameters {
    string(name: 'LOG_FILE', defaultValue: '/var/log/httpd/access.log')
}

🔐 Security Considerations

Store Jenkins API token securely
Use IAM role with limited S3 access
Restrict script permissions:
chmod 700 monitor_log.sh
Avoid hardcoding credentials

🚀 Key Learnings

Automating log monitoring
Integrating Jenkins with AWS
Using AWS CLI for storage operations
Preventing disk overflow issues
Real-world DevOps automation
🔮 Future Enhancements
Compress logs before upload (.gz)
Add CloudWatch monitoring
Use Docker for containerized execution
Integrate Slack alerts
Implement log rotation using logrotate

🤝 Conclusion

This project provides a robust, automated solution to manage large log files and prevent server crashes. It demonstrates real-world DevOps practices including monitoring, automation, and cloud integration.
