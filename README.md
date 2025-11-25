Splunk SSH Login Analysis — SOC Investigation Project

This project focuses on analyzing SSH authentication logs using Splunk.
The goal is to identify attacker behavior, detect brute-force attempts, and understand authentication patterns by using six core SOC-level detections.

All queries in this project are written specifically for Zeek-style SSH logs, with fields such as id.orig_h, id.resp_h, auth_success, auth_attempts, and event_type.

## 📁 Project Structure
/Splunk-SSH-Login-Analysis
│
├── 01_top_attacker.spl
├── 02_top10_attackers.spl
├── 03_success_vs_failed.spl
├── 04_compromised_ips.spl
├── 05_top_dest_servers.spl
├── 06_attempts_per_ip.spl
│
└── README.md   ← (this file)

## 📌 About the Dataset

The dataset contains SSH login events in JSON format.
Each event includes:

id.orig_h — Source IP (attacker / user)

id.resp_h — Destination server IP

auth_success — true/false login result

auth_attempts — number of attempts

event_type — Successful or Failed SSH Login

Additional Zeek network metadata

Each event represents one SSH authentication attempt.

##  Queries Implemented in This Project

Below are the six detections implemented in Splunk, along with the .spl files that contain the exact queries.

### 1. Top Attacker (Single Value Panel)

File: 01_top_attacker.spl

index=* auth_success=false
| stats count by id.orig_h
| sort -count
| head 1


✔ Identifies the IP with the highest number of failed SSH logins — usually the main attacker.
✔ Useful for brute-force investigation.

### 2. Top 10 Attackers (Bar Chart)

File: 02_top10_attackers.spl

index=* auth_success=false
| stats count by id.orig_h
| sort -count
| head 10


✔ Shows the top 10 source IPs generating failed SSH logins.
✔ Helps identify botnets or multiple attack sources.

### 3. Success vs Failed Logins (Pie Chart)

File: 03_success_vs_failed.spl

index=* 
| stats count by auth_success


✔ Displays the ratio of successful vs failed logins.
✔ Useful for understanding authentication behavior and brute-force activity.

### 4. Compromised IPs (Table)

File: 04_compromised_ips.spl

index=*
| stats values(auth_success) as status by id.orig_h
| search status="true" AND status="false"


✔ Identifies IPs that had both failed and successful logins.
✔ These IPs may have successfully brute-forced the SSH service.

### 5. Top Targeted Servers (Bar Chart)

File: 05_top_dest_servers.spl

index=* auth_success=false
| stats count by id.resp_h
| sort -count
| head 5


✔ Shows which destination SSH servers were targeted the most.
✔ Helps identify servers under heavy attack load.

### 6. Attempt Intensity Per Attacker (Table)

File: 06_attempts_per_ip.spl

index=* 
| stats sum(auth_attempts) as attempts by id.orig_h
| sort -attempts
| head 5


✔ Measures how many total attempts each IP generated.
✔ Helps identify aggressive attackers.

## 📊 What This Project Demonstrates

-This project shows practical SOC investigation skills using Splunk:
-Identifying brute-force attackers
-Classifying authentication attempts
-Detecting compromised IPs
-Finding the most targeted servers
-Analyzing login patterns
-Understanding attacker behavior through raw log analysis

These 6 detections form a complete SSH authentication analysis workflow.

## 🧑‍💻 How to Use This Project

-Upload your SSH log JSON file into Splunk.
-Set sourcetype to _json (recommended).
-Copy each .spl file’s content into Splunk search.

**Use the outputs to build visual panels:
-single value
-bar charts
-pie chart
-tables

## 📝 Conclusion

This project provides a clear and structured analysis of SSH login events in Splunk using six focused detections.
It demonstrates hands-on SOC skills including threat hunting, brute-force detection, and authentication pattern analysis.
