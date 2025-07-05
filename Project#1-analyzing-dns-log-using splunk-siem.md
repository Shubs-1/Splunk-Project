# Analyzing DNS Log Files Using Splunk SIEM

## Introduction
DNS (Domain Name System) logs are crucial for understanding network activity and identifying potential security threats. Splunk SIEM (Security Information and Event Management) provides powerful capabilities for analyzing DNS logs and detecting anomalies or malicious activities.

## Prerequisites
Before analyzing DNS logs in Splunk, ensure the following:
- Splunk instance is installed and configured.
- DNS log data sources are configured to forward logs to Splunk.

## Steps to Upload Sample DNS Log Files to Splunk SIEM

### 1. Prepare Sample DNS Log Files
- Obtain sample [DNS log file](https://www.secrepo.com/maccdc2012/dns.log.gz) in a suitable format (e.g., text files).
- Ensure the log files contain relevant DNS events, including source IP, destination IP, domain name, query type, response code, etc.
- Save the sample log files in a directory accessible by the Splunk instance.

### 2. Upload Log Files to Splunk
- Log in to the Splunk web interface.
- Navigate to **Settings** > **Add Data**.
- Select **Upload** as the data input method.

### 3. Choose File
- Click on **Select File** and choose the sample DNS log file you prepared earlier.

### 4. Set Source Type
- In the **Set Source Type** section, specify the source type for the uploaded log file.
- Choose the appropriate source type for DNS logs (e.g., `dns` or a custom source type if applicable).

### 5. Review Settings
- Review other settings such as index, host, and sourcetype.
- Ensure the settings are configured correctly to match the sample DNS log file.

### 6. Click Upload
- Once all settings are configured, click on the **Review** button.
- Review the settings one final time to ensure accuracy.
- Click **Submit** to upload the sample DNS log file to Splunk.

### 7. Verify Upload
- After uploading, navigate to the search bar in the Splunk interface.
- Run a search query to verify that the uploaded DNS events are visible.

 ```spl
  index=<your_dns_index> sourcetype=<your_dns_sourcetype>
 ```
  
## Steps to Analyze DNS Log Files in Splunk SIEM

### 1. Search for DNS Events   
- Open Splunk interface and navigate to the search bar.   
- Enter the following search query to retrieve DNS events   
```
index=* sourcetype=dns_sample
```
![DNS I4](https://github.com/user-attachments/assets/a313218e-5162-440d-a78e-d7977017b24d)

### 2. Extract Relevant Fields
- Identify key fields in DNS logs such as source IP, destination IP, domain name, query type, response code, etc.   
- As mentioned below,  | regex _raw="(?i)\b(dns|domain|query|response|port 53)\b": This regex searches for common DNS-related keywords in the raw event data.
- Example extraction command:
```
index=* sourcetype=dns_sample | regex _raw="(?i)\b(dns|domain|query|response|port 53)\b"
```
![DNS I5](https://github.com/user-attachments/assets/5f5a45be-2687-4897-9b2b-a453c18d2feb)

Regex:
• (?i) makes the pattern case-insensitive.
• \b denotes word boundaries, so you match whole words only (e.g. it catches "DNS" but not "dnslookup").
(dns|domain|query|response|port 53) uses a group with alternation to match any of those specific DNS-related terms.

### 3. Match Fully Qualified Domain Names (FQDNs)

- This regex is designed to match domain names or hostnames in a string.
- You’ll typically use this in Splunk to extract or search for domain names in raw logs, which can be incredibly useful for DNS analysis, spotting uncommon TLDs, or filtering traffic for tunneling use cases.
```
index="*" sourcetype=dns | regex _raw="\b(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}\b"
```
![DNS I6](https://github.com/user-attachments/assets/4a864a9a-cd18-41c5-a9b6-079c8760ee12)
  Regex :
- \b ensures the pattern starts and ends at a word boundary.
- (?:[a-zA-Z0-9-]+\.)+ matches one or more subdomains or domain labels ending with a period. The ?: makes it a non-capturing group.
- [a-zA-Z]{2,} captures the top-level domain (like .com, .in, .org), requiring at least two alphabetic characters.
  The final \b keeps it cleanly bounded.

