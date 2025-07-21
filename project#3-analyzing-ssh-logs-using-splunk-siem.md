# Analyzing SSH Log Files Using Splunk SIEM

## Introduction
SSH (Secure Shell) log files contain valuable information about remote access to servers, including login attempts, commands executed, and session details. Analyzing SSH logs using Splunk SIEM enables security professionals to monitor access to critical systems, detect anomalies, and identify potential security threats.

## Project Overview
In this project, we will upload sample SSH log files to Splunk SIEM and perform various analyses to gain insights into SSH activity within the network.

## Prerequisites
Before starting the project, ensure the following:
- Splunk instance is installed and configured.
- SSH log data sources are configured to forward logs to Splunk.

## Steps to Upload Sample SSH Log Files to Splunk SIEM

### 1. Prepare Sample SSH Log Files
- Obtain sample [SSH log files](https://www.secrepo.com/maccdc2012/ssh.log.gz) in a suitable format (e.g., text files).
- Ensure the log files contain relevant SSH events, including timestamps, source IP addresses, usernames, actions (login, logout), etc.
- Save the sample log files in a directory accessible by the Splunk instance.

### 2. Upload Log Files to Splunk
- Log in to the Splunk web interface.
- Navigate to **Settings** > **Add Data**.
- Select **Upload** as the data input method.

### 3. Choose File
- Click on **Select File** and choose the sample SSH log file you prepared earlier.

### 4. Set Source Type
- In the **Set Source Type** section, specify the source type for the uploaded log file.
- Choose the appropriate source type for SSH logs (e.g., `syslog` or a custom source type if applicable).

### 5. Review Settings
- Review other settings such as index, host, and sourcetype.
- Ensure the settings are configured correctly to match the sample SSH log file.

### 6. Click Upload
- Once all settings are configured, click on the **Review** button.
- Review the settings one final time to ensure accuracy.
- Click **Submit** to upload the sample SSH log file to Splunk.

### 7. Verify Upload
- After uploading, navigate to the search bar in the Splunk interface.
- Run a search query to verify that the uploaded SSH events are visible.


## Steps to Analyze SSH Log Files in Splunk SIEM

### 1. Search for SSH Events
  
- Open Splunk interface and navigate to the search bar.
- Enter the following search query to retrieve SSH
```
source="ssh.log.gz" sourcetype="SSHlogs
```
<img width="1572" height="641" alt="SSH I1" src="https://github.com/user-attachments/assets/568a5bd8-6057-4653-ad00-63d317f8e368" />

### 2. Search for SSH Events
  
- Identify top users or source IP addresses accessing the SSH server:
- Enter the following search query to retrieve SSH
```
source="ssh.log.gz" sourcetype="SSHlogs" | top limit=10 src_ip
```
<img width="1433" height="423" alt="SSH I2" src="https://github.com/user-attachments/assets/ee94ca0d-36ec-4324-9b2d-e50b48856a42" />

### 3. Analyze successful vs. failed SSH login attempts:

  
- It will give you the count of both successfull and failed SSH login event
- Enter the following search query to retrieve SSH
```
source="ssh.log.gz" sourcetype="SSHlogs" | stats count by login_status

```
<img width="1482" height="282" alt="SSH I3" src="https://github.com/user-attachments/assets/b34938aa-7e34-4a8e-a21a-8989a02c595b" />

### 4. Detect Anomalies
#### a. Analyze failed login attempts:
- Enter the following search query to retrieve SSH
```
source="ssh.log.gz" sourcetype="SSHlogs" | search login_status="failure"
```
<img width="1412" height="640" alt="SSH I4" src="https://github.com/user-attachments/assets/3b240cb4-2e6b-4de8-ab47-e2c3cded26da" />

#### b. High frequency of failed logins

- Enter the following search query to retrieve SSH
```
source="ssh.log.gz" sourcetype="SSHlogs" | search login_status="failure" | stats count by src_ip
| where count > 10 | sort -count
```
<img width="1437" height="597" alt="SSH I5" src="https://github.com/user-attachments/assets/bd23ed04-911e-41af-a673-0665409a12fe" />

#### c. Spot outliers in traffic volume or behavior
- It will give output with rare IP addresses.
- Enter the following search query to retrieve SSH
```
source="ssh.log.gz" sourcetype="SSHlogs" | stats count by src_ip
| eventstats avg(count) as avg_count stdev(count) as std_dev
| where count > avg_count + (2 * std_dev)<img width="561" height="68" alt="image" src="https://github.com/user-attachments/assets/e55674b7-0c4d-40e9-a821-5844d8bddd34" />

```
<img width="1518" height="281" alt="SSH I6" src="https://github.com/user-attachments/assets/04105330-7cd4-4416-aa1b-282d40ec64a6" />


