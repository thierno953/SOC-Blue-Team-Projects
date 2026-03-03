# Network Traffic Analysis

Investigation of suspicious network activity. Multiple HTTP requests, malware download, and malicious DNS activity were identified via Wireshark and confirmed through OSINT. Activity is linked to Pikabot malware infrastructure.

### Top Talkers

- Three IPs generated the most traffic (~1 MB):

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_01.png)

### Initial HTTP Activity

- First HTTP request observed at 16:01 UTC (packet 6).

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_02.png)

- First HTTP request at 16:01 UTC (packet 6)
- HTTP stream shows communication with:
  - Domain: jinjadiocese[.]com
  - IP: 68.66.226.89

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_03.png)

### OSINT - Domain & IP

- 68.66.226.89
  - AbuseIPDB flagged
  - VirusTotal detections

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_04.png)
![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_05.png)

- jinjadiocese[.]com
  - Whois: created 2025-01-19, updated 2025-01-19
  - VirusTotal: 2 detections
  - Linked to suspicious activity in OTX

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_06.png)
![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_07.png)

### Suspicious File Download

- Packet 17 shows second GET request.
- File delivered: `GURVU.zip`
- MIME type: application/octet-stream (malware payload)

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_09.png)
![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_10.png)

- This is typical for malware payload delivery.

### DNS Activity

- Packet 117: multiple `.dat` domains queried
- Responses: no records -> possible C2 instructions

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_11.png)
![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_12.png)

### Additional domains resolved (Pikabot)

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_13.png)

### Malware File Analysis

- Exported `GURVU.zip` via:
  - `File -> Export Objects -> HTTP -> packet 111`

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_14.png)
![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_15.png)

### SHA256 Hash

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_16.png)

### VirusTotal Result

- 28 vendors flagged the file as malicious

![Network](./assets/assets_Network-Forensics/Network_Traffic_Analysis_17.png)

---

### Conclusion

- Activity confirmed as malicious Pikabot infrastructure
- HTTP download, DNS C2 queries, and malware hash confirm compromise potential
- Recommended SOC actions: block IPs/domains, isolate file, update IDS signatures, monitor DNS

---

### Useful Resources

- [AbuseIPDB](https://www.abuseipdb.com/)
- [VirusTotal](https://www.virustotal.com/gui/home/upload)
- [DomainTools](https://whois.domaintools.com/)
- [AlienVault OTX](https://otx.alienvault.com/browse/global/pulses?sort=-modified&page=1&include_inactive=0&limit=10)
- [Wireshark](https://2.na.dl.wireshark.org/win64/Wireshark-4.2.3-x64.exe)
