# **CRON-Jobs**

# **Commonly Used Cron Jobs by System Administrators**  

System administrators frequently use cron jobs for automation, scheduled maintenance, monitoring, and backup tasks. Below are some of the most commonly used cron jobs and their purposes.

---

## **1. Common Cron Jobs and Their Purpose**  

### **1.1 System Maintenance and Monitoring**
- **Clearing logs to free up disk space:**  
  ```bash
  0 0 * * * echo "" > /var/log/syslog
  ```
  *Runs daily at midnight to empty the system log file.*

- **Monitoring disk usage and sending alerts:**  
  ```bash
  0 6 * * * df -h | mail -s "Disk Usage Report" admin@example.com
  ```
  *Runs every day at 6 AM and emails the disk usage report to the admin.*

- **Restarting a crashed service automatically:**  
  ```bash
  */5 * * * * systemctl is-active --quiet nginx || systemctl restart nginx
  ```
  *Runs every 5 minutes and restarts Nginx if it is not active.*

### **1.2 Backup and Cleanup Jobs**
- **Automatic database backup:**  
  ```bash
  0 2 * * * mysqldump -u root -p'yourpassword' mydatabase > /backups/db_$(date +\%F).sql
  ```
  *Runs every day at 2 AM, backs up a MySQL database, and stores it with the date in the filename.*

- **Clearing temporary files older than 7 days:**  
  ```bash
  0 3 * * * find /tmp -type f -mtime +7 -exec rm -f {} \;
  ```
  *Runs every day at 3 AM to delete temporary files older than 7 days.*

- **Rotating logs and keeping only the last 10:**  
  ```bash
  0 4 * * * find /var/log/myapp/ -type f -name "*.log" -mtime +10 -delete
  ```
  *Deletes logs older than 10 days at 4 AM daily.*

### **1.3 System Security and Updates**
- **Updating package lists automatically:**  
  ```bash
  0 5 * * * apt update && apt upgrade -y
  ```
  *Runs every day at 5 AM and updates the system packages.*

- **Checking for failed SSH logins and emailing alerts:**  
  ```bash
  0 1 * * * grep "Failed password" /var/log/auth.log | mail -s "SSH Failed Login Attempts" admin@example.com
  ```
  *Runs at 1 AM and emails a report of failed SSH login attempts.*

- **Blocking IPs with too many failed SSH attempts:**  
  ```bash
  */10 * * * * awk '/Failed password/ {print $(NF-3)}' /var/log/auth.log | sort | uniq -c | awk '$1 > 5 {print $2}' | while read ip; do iptables -A INPUT -s $ip -j DROP; done
  ```
  *Runs every 10 minutes and blocks IPs with more than 5 failed login attempts.*

### **1.4 Web Server and Application Management**
- **Clearing cache files in a web application:**  
  ```bash
  0 * * * * rm -rf /var/www/html/cache/*
  ```
  *Runs every hour to clear the cache directory.*

- **Restarting a web server at a specific time:**  
  ```bash
  0 3 * * 0 systemctl restart apache2
  ```
  *Runs every Sunday at 3 AM to restart Apache.*

- **Monitoring a process and restarting it if not running:**  
  ```bash
  */5 * * * * pgrep myapp || systemctl restart myapp
  ```
  *Runs every 5 minutes and restarts "myapp" if it is not running.*

---

## **2. Example Cron Job: Automatic Database Backup**

### **Step 1: Open the Crontab**
```bash
crontab -e
```

### **Step 2: Add the Cron Job**
```bash
0 2 * * * mysqldump -u root -p'password' mydatabase > /backups/db_$(date +\%F).sql
```
- `0 2 * * *` → Runs at 2:00 AM every day.
- `mysqldump` → Command to dump the database.
- `-u root -p'password'` → MySQL username and password.
- `mydatabase` → The name of the database to back up.
- `>/backups/db_$(date +\%F).sql` → Saves the backup with the current date.

### **Step 3: Save and Exit**
- If using `nano`, press `CTRL + X`, then `Y`, and `Enter`.

### **Step 4: Verify the Cron Job**
List all cron jobs:
```bash
crontab -l
```

### **Step 5: Test the Cron Job**
Run the command manually to check for errors:
```bash
mysqldump -u root -p'password' mydatabase > /backups/db_$(date +\%F).sql
```

---

## **3. Advanced Cron Job: Custom Shell Script Execution**

### **Example: Automated System Health Check Script**

1. **Create a shell script:**
    ```bash
    nano /usr/local/bin/system_health_check.sh
    ```

2. **Add the following content:**
    ```bash
    #!/bin/bash
    TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")
    LOG_FILE="/var/log/system_health_$TIMESTAMP.log"
    
    echo "System Health Check - $TIMESTAMP" > $LOG_FILE
    echo "Uptime:" >> $LOG_FILE
    uptime >> $LOG_FILE
    echo "Disk Usage:" >> $LOG_FILE
    df -h >> $LOG_FILE
    echo "Memory Usage:" >> $LOG_FILE
    free -m >> $LOG_FILE
    
    mail -s "System Health Report" admin@example.com < $LOG_FILE
    ```

3. **Give execute permissions:**
    ```bash
    chmod +x /usr/local/bin/system_health_check.sh
    ```

4. **Schedule the script in crontab:**
    ```bash
    0 4 * * * /usr/local/bin/system_health_check.sh
    ```
    *Runs daily at 4 AM and sends a system health report via email.*

---

## **4. Summary**
- **Cron jobs are used for automation, maintenance, monitoring, and security.**
- **System admins commonly schedule tasks like backups, log cleanup, security scans, and service restarts.**
- **Using `crontab -e`, admins define jobs with specific schedules.**
- **Debugging cron issues involves checking logs and testing commands manually.**
- **Advanced cron jobs can execute shell scripts for more complex tasks.**

---
