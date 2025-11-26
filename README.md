## Splunk SSH Login Analysis — SOC Investigation Project

This project focuses on analyzing SSH authentication logs using Splunk.
The goal is to identify attacker behavior, detect brute-force attempts, and understand authentication patterns by using six core SOC-level detections.
By identifying unusual login patterns and failed login spikes, it helps security analysts respond faster and strengthen system security. Ideal for SOC teams or anyone monitoring server access for potential threats.

Why analyze SSH logins? SSH is a common method for remote server access, and attackers often target it using brute-force attacks, stolen credentials, or unauthorized login attempts.

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

---

## 🚀 Getting Started / How to Run This Project

This project was built completely inside Splunk using SSH authentication logs.  
If you want to replicate or run this project on your own system, follow the steps below:

### 1️⃣ Prepare Your SSH Log Dataset
I used a sample SSH authentication dataset that includes fields like:
- `id.orig_h` (source/attacker IP)
- `id.resp_h` (destination/server IP)
- `auth_success`
- `auth_attempts`

You can use any SSH log source (Splunk sample data, Zeek logs, or your own lab logs).

In Splunk:
- Go to **Settings → Add Data**
- Upload your log file (or point to a directory)
- Select **Search & Reporting** as the app
- Index it under `main` (or any index of your choice)

### 2️⃣ Run the SPL Queries
Inside **Search & Reporting**, copy each SPL file from the `/spl/` folder and paste it into the Splunk search bar.

Example:
- `01_top_attacker.spl` → shows the highest failed login source
- `02_top10_attackers.spl` → bar chart for top brute-forcers  
…and so on.

Running each query once verifies that your dataset is loading correctly.

### 3️⃣ Import the Dashboard (XML)
To load the dashboard I created:

1. Go to **Dashboards**
2. Click **Create New Dashboard**
3. Select **Classic (Simple XML)**
4. Click **Source**
5. Paste the XML from `dashboard/ssh_login_dashboard.xml`
6. Save

This recreates the exact same dashboard shown in the screenshots folder.

### 4️⃣ Adjust Time Range
Set time range to **All Time** (important because the dataset is static).

After this, all 6 visual panels will load properly.

---

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

---

## 📸 Dashboard Preview

### 🔹 Top Attacker
<img src="images/Screenshot%202025-11-26%20141512.png" width="600">

### 🔹 Top 10 Attackers
<img src="images/Screenshot%202025-11-26%20142816.png" width="600">

### 🔹 Success vs Failed Logins
<img src="images/Screenshot%202025-11-25%20164045.png" width="600">

### 🔹 Compromised IPs
<img src="images/Screenshot%202025-11-25%20164103.png" width="600">

### 🔹 Top Targeted Servers
<img src="images/Screenshot%202025-11-25%20164131.png" width="600">

### 🔹 Attempt Intensity Per IP
<img src="images/Screenshot%202025-11-25%20164200.png" width="600">

---

## 📊 What This Project Demonstrates

-This project shows practical SOC investigation skills using Splunk:
-Identifying brute-force attackers
-Classifying authentication attempts
-Detecting compromised IPs
-Finding the most targeted servers
-Analyzing login patterns
-Understanding attacker behavior through raw log analysis

These 6 detections form a complete SSH authentication analysis workflow.

---

## 📌 Observations, Challenges & Learnings

### 🔍 Key Observations From the Dashboard
Working through this dashboard helped me understand SSH login activity from both a high-level and attack-pattern perspective.  
Some things I noticed:

- A small number of attacker IPs were responsible for most of the failed login attempts.
- Success vs failed attempts clearly showed typical brute-force behavior (large failure spike with very few successes).
- Only a few source IPs showed both success and failure, which helped me identify potentially compromised IPs.
- The targeted servers panel showed that attackers usually hit the same destination repeatedly.

Overall, the dashboard made it easy to spot unusual SSH login patterns at a glance.

---

### ⚠️ Challenges I Faced
This project wasn’t just “run a query and make a dashboard”.  
I faced some real issues that I had to troubleshoot:

- **Field mismatch issues**: My logs used `id.orig_h`, `auth_success`, `id.resp_h`, etc. I had to adjust SPL queries to make sure they worked with my exact dataset.
- **Multiple events showing unexpected results**: Some queries returned too many events or zero events until I added the correct filters.
- **Dashboard configuration confusion**: Initially, I didn’t find the inline search or the right dashboard options, so I switched to Simple XML and edited the entire dashboard structure manually.
- **Mapping screenshots correctly**: Since I took all screenshots separately, I had to carefully match each one with the correct dashboard panel to keep the README clean and professional.

These challenges helped me understand Splunk more deeply, especially how it handles fields, queries, visualizations, and dashboard layouts.

---

### 🎯 What I Learned
This project helped me learn more than just SPL syntax:

- **How to analyze SSH logs like a SOC analyst** — spotting attackers, brute-force attempts, compromised IPs, and target servers.
- **How to structure a Splunk dashboard** that tells a clear security story instead of random charts.
- **How to cleanly organize a security project for GitHub** with screenshots, queries, dashboards, and documentation.
- **Troubleshooting Splunk searches** when queries return too many/zero events.
- **Importance of visual clarity** — the dashboard quickly highlights anomalies that would be hard to spot just by reading logs.

Overall, this project gave me hands-on experience in log analysis, brute-force detection, query building, and real SOC-style investigation.

---

## 🧑‍💻 How to Use This Project

-Upload your SSH log JSON file into Splunk.
-Set sourcetype to _json (recommended).
-Copy each .spl file’s content into Splunk search.

**Use the outputs to build visual panels:
-single value
-bar charts
-pie chart
-tables

---

## 📝 Conclusion

This project provides a clear and structured analysis of SSH login events in Splunk using six focused detections.
It demonstrates hands-on SOC skills including threat hunting, brute-force detection, and authentication pattern analysis.
