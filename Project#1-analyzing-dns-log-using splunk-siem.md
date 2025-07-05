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
- (?i) makes the pattern case-insensitive.
- \b denotes word boundaries, so you match whole words only (e.g. it catches "DNS" but not "dnslookup").
- (dns|domain|query|response|port 53) uses a group with alternation to match any of those specific DNS-related terms.

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

### 4. Detect Suspiciously Long Subdomains (Tunneling Clue)

This regex filters for fully qualified domain names (FQDNs) with four or more labels, which is super relevant in detecting suspicious or unusually nested domains—common in DNS tunneling or command-and-control traffic.
```
index="*" sourcetype=dns | regex _raw="\b(?:[a-zA-Z0-9-]+\.){4,}[a-zA-Z]{2,}\b"
```
![DNS I7](https://github.com/user-attachments/assets/e18a6134-fcdc-4cd3-a260-48f6c2b473e9)

  Regex : 
- \b → Word boundary to anchor the match neatly.
- (?:[a-zA-Z0-9-]+\.){4,} → A non-capturing group that matches at least four domain parts (e.g. sub.sub.sub.sub.).
- [a-zA-Z]{2,} → The top-level domain (e.g. com, info, xyz), requiring at least 2 characters.
- Final \b → Ensures we don’t match trailing punctuation or partial words.

This makes it a solid detector for potential data exfiltration attempts or suspicious domain generation. You could even pair it with entropy checks or frequency analysis on the subdomains for deeper threat detection.

### 5. Reverse DNS Lookup Detection
This regex is tailored to match reverse DNS lookup addresses in the .in-addr.arpa domain, which is how IP-to-hostname mappings are queried for IPv4 addresses. 
This is great for spotting PTR record queries or DNS logs involving reverse lookups.
```
 index="*" sourcetype=dns | regex _raw="\b(\d{1,3}\.){3}\d{1,3}\.in-addr\.arpa\b"
```
![DNS I8](https://github.com/user-attachments/assets/f30029cc-bea3-416a-a566-1e5dd1d7bd61)

Regex : 
- \b → Word boundary to anchor the pattern.
- (\d{1,3}\.){3} → Matches three groups of 1–3 digits followed by a dot, essentially the first three octets of an IPv4 address.
- \d{1,3} → The last octet of the IP.
- \.in-addr\.arpa → Matches the literal domain used in reverse DNS zones.
- Final \b → Ensures a clean word boundary at the end.

### 6. Identify Anomalies
anomalies refer to unusual or unexpected patterns in DNS traffic that deviate from normal behavior. These can be early indicators of misconfigurations, performance issues, or even cyber threats like data exfiltration or malware communication.
		• Look for unusual patterns or anomalies in DNS activity.
		• Example query to identify spikes
	
```
 index="*" sourcetype=dns | stats count by fqdn |sort - count
 ```
![DNS I9](https://github.com/user-attachments/assets/80e85bfc-9d9e-4c51-9ff6-2ed4a28cbc33)

1. index="*"
		• This tells Splunk to search across all indexes. It’s broad and useful when you're not sure where the data is stored or if your data spans multiple indexes.
2. sourcetype=dns
		• This filters events to only those tagged with the dns sourcetype, so you’re focusing purely on DNS-related logs.
3. stats count by fqdn
		• This command groups events by the fqdn (fully qualified domain name) field and counts how many times each domain appears in the dataset.
		• So, you're basically saying: “How many DNS queries did I see for each unique domain?”
4. sort - count
The - means descending order, so this step orders the output from the most-queried domain to the least-queried.

### 7. Identify the most frequently queried domain names
```
index=dns_lab sourcetype="json"
| stats count by query
| sort -count
```
![DNS I1](https://github.com/user-attachments/assets/d00aea26-d4ad-4d12-b2b3-67657ae71c54)

### 8. Find the most active user IPs generating DNS traffic![image](https://github.com/user-attachments/assets/357d4a7b-32dc-49dd-a021-cc999ff00fdf)

```
index=dns_lab sourcetype="json"
| stats count by "id.orig_h"
| sort -count
```
![DNS I2](https://github.com/user-attachments/assets/c65f8304-a4db-411a-be44-1a447ee843c6)

### 9. Breakdown of DNS query types (A, AAAA, CNAME, PTR)

```
index=dns_lab sourcetype="json"
| stats count by qtype
```
![DNS I3](https://github.com/user-attachments/assets/ad7bb362-a50c-4374-8e11-1580e60bf336)
