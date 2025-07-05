# Analyzing FTP Log Files Using Splunk SIEM

## Introduction
FTP (File Transfer Protocol) log files contain valuable information about file transfers within a network. Analyzing FTP logs using Splunk SIEM enables security professionals to monitor file transfer activities, detect anomalies, and identify potential security threats.

## Project Overview
In this project, we will upload sample FTP log files to Splunk SIEM and perform various analyses to gain insights into FTP activity within the network.

## Prerequisites
Before starting the project, ensure the following:
- Splunk instance is installed and configured.
- FTP log data sources are configured to forward logs to Splunk.

## Steps to Upload Sample FTP Log Files to Splunk SIEM

### 1. Prepare Sample FTP Log Files
- Obtain sample [FTP log files](https://www.secrepo.com/maccdc2012/ftp.log.gz) in a suitable format (e.g., text files).
- Ensure the log files contain relevant FTP events, including timestamps, source IP, username, commands, filenames, etc.
- Save the sample log files in a directory accessible by the Splunk instance.

### 2. Upload Log Files to Splunk
- Log in to the Splunk web interface.
- Navigate to **Settings** > **Add Data**.
- Select **Upload** as the data input method.

### 3. Choose File
- Click on **Select File** and choose the sample FTP log file you prepared earlier.

### 4. Set Source Type
- In the **Set Source Type** section, specify the source type for the uploaded log file.
- Choose the appropriate source type for FTP logs (e.g., `ftp` or a custom source type if applicable).

### 5. Review Settings
- Review other settings such as index, host, and sourcetype.
- Ensure the settings are configured correctly to match the sample FTP log file.

### 6. Click Upload
- Once all settings are configured, click on the **Review** button.
- Review the settings one final time to ensure accuracy.
- Click **Submit** to upload the sample FTP log file to Splunk.

### 7. Verify Upload
- After uploading, navigate to the search bar in the Splunk interface.
- Run a search query to verify that the uploaded FTP events are visible.


## Steps to Analyze FTP Log Files in Splunk SIEM
