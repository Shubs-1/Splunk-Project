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

### 1. Search for FTP Events
  
- Open Splunk interface and navigate to the search bar.   
- Enter the following search query to retrieve DNS events   
```
index="*" sourcetype=FTP_Logs
```
<img width="1325" height="607" alt="FTP I1" src="https://github.com/user-attachments/assets/5b4181ec-c63b-40a5-a076-0a57dbaeeff2" />

### 2. Extract Relevant Fields

- Identify key fields in FTP logs such as timestamps, source IP, username, commands, filenames, etc.
- Use Splunk's field extraction capabilities or regular expressions to extract these fields for better analysis.
- Example extraction command
  
```
| rex field=_raw "^(?<timestamp>\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2}:\d{2}).*?(?<source_ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}).*?(?<username>\w+).*?(?<command>[A-Z]+).*?(?<file_path>\/[\w\/.-]+)"
```
<img width="1380" height="641" alt="FTP I2" src="https://github.com/user-attachments/assets/04ddbff5-4133-4a22-9269-aaa2a22d466c" />

#### Explanation:
  
- ^: Start of the line.
- (?<timestamp>\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2}:\d{2}): Matches and captures the timestamp in the format "YYYY-MM-DD HH:MM:SS".
- .*?: Matches any character (except for line terminators) as few times as possible.
- (?<source_ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}): Matches and captures the source IP address.
- (?<username>\w+): Matches and captures the username (assuming it consists of alphanumeric characters).
- (?<command>[A-Z]+): Matches and captures the FTP command (assuming it consists of uppercase letters).
- (?<file_path>\/[\w\/.-]+): Matches and captures the file path (assuming it starts with "/" and can contain alphanumeric characters, "/", ".", and "-").

### 3. Analyze File Transfer Activity
#### a. Determine the frequency and volume of file transfers:
- To determine frequency and volume of file transfers from your FTP logs, we need to isolate events that represent file transfers—typically commands like STOR (upload) and RETR (download) are your key indicators.
- Example extraction command
  
```
index="*" sourcetype=FTP_Logs command="STOR" OR command="RETR" | stats count by command | rename count as transfer_files
```
<img width="1520" height="302" alt="FTP I3" src="https://github.com/user-attachments/assets/dbe71147-85df-47fe-a526-2a9b2f9834d5" />

#### b. Identify top users or IP addresses involved in file transfers : 
- This will help to identified involved IP address in transfer
- Example extraction command
  
```
• index="*" sourcetype=FTP_Logs command="STOR" OR command="RETR" | stats count by command, src_ip, dst_ip | rename count as transfer_files
```
<img width="1482" height="645" alt="FTP I4" src="https://github.com/user-attachments/assets/c7137667-e4cc-4f95-99d4-80fe31bd424e" />

#### c. Analyze the types of files being transferred (e.g., documents, executables, archives) :
- Example extraction command
  
```
index="*" sourcetype=FTP_Logs command="STOR" OR command="RETR" | rex "command=\w+\s+(?<filename>[\S]+)" | stats count by command files
```
<img width="1478" height="636" alt="FTP I5" src="https://github.com/user-attachments/assets/5e80fb29-21f6-4a22-8926-6606795ea120" />

### 4. Analyze File Transfer Activity
#### a. Look for unusual patterns in file transfer activity :

- Identify anomalies in file transfer volume, timing, or destinations
- Detect unauthorized access or suspicious user behavior
- Spot rare or sensitive file types being transferred (e.g., .exe, .zip, .docx)
- Example extraction command
  
```
index="*" sourcetype=FTP_Logs | stats count sum(bytes_transfer) by User_ID, command, file
```
<img width="1372" height="610" alt="FTP I6" src="https://github.com/user-attachments/assets/8678013f-6c47-4ae1-bdd2-7575b7ede2fa" />

#### b. Analyze sudden spikes or drops in file transfer volume :

```
index="*" sourcetype=FTP_Logs | timechart span=15m sum(bytes_transfer) as "total_transfer" by User_ID
```
<img width="1387" height="442" alt="FTP I7" src="https://github.com/user-attachments/assets/5b19c2ca-cbe7-4585-8788-358bb2c2ea8b" />

#### c. Investigate file transfers to or from suspicious IP addresses :

```
index="*" sourcetype=FTP_Logs command="STOR" OR command="RETR" | stats count by command, src_ip, dst_ip, file | rename count as transfer_files
```
<img width="1523" height="637" alt="FTP I8" src="https://github.com/user-attachments/assets/0cff8194-6f5c-4dcb-b664-696c1db1cd61" />


