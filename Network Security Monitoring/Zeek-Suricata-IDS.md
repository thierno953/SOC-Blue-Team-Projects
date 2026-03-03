# IDS deployment & rules | Network Security, Detection

This project focuses on detecting and analyzing malicious network traffic using Suricata and Zeek.
The goal is to validate suspicious activity observed in a PCAP file and confirm malware behavior through IDS alerts, network logs, file extraction, and OSINT.

## Suricata Detection

Suricata was used to analyze the PCAP file in offline mode.

```sh
sudo suricata -r Zeek_Suricata.pcap
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_01.png)

Suricata generated multiple alerts. The `fast.log` file was parsed to identify the most frequent alert types.

```sh
cat fast.log | cut -d '}' -f 2 | cut -d ':' -f 1 | cut -d ' ' -f 2 | sort | uniq -c | sort -nr
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_02.png)

This confirms that the traffic was detected as suspicious by IDS signatures.

## Zeek Traffic Analysis

Zeek was used to extract network metadata and build a clear activity timeline.

```sh
sudo /opt/zeek/bin/zeek -r Zeek_Suricata.pcap
ls
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_03.png)

### Connection Analysis

```sh
cat conn.log | zeek-cut ts id.orig_h id.resp_h id.resp_p service
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_04.png)

This shows repeated outbound connections to external hosts over HTTP, indicating possible command-and-control communication.

### DNS Analysis

DNS traffic was reviewed to identify suspicious domain activity.

```sh
cat dns.log | less
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_05.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_06.png)

Filtered DNS queries:

```sh
cat dns.log | zeek-cut ts id.orig_h query
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_07.png)

Several suspicious and newly registered domains were observed, consistent with malware C2 behavior.

## OSINT Validation

The identified domains and IPs were checked using **VirusTotal** and **Whois. Domaintools**.

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_08.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_09.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_10.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_11.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_12.png)

Results confirmed links to malicious infrastructure associated with Pikabot.

## HTTP Activity Analysis

```sh
cat http.log | zeek-cut ts id.orig_h id.resp_h method uri user_agent
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_13.png)

HTTP GET requests were observed, showing file downloads from suspicious external servers.

## File Extraction and Analysis

```sh
cat files.log | zeek-cut ts id.orig_h id.resp_h mime_type filename
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_14.png)

The downloaded files were identified as binary payloads.

VirusTotal analysis confirmed malicious behavior.

## Virustotal

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_15.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_16.png)

File inspection:

```sh
files *
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_17.png)

Extracted files were organized and unpacked.

```sh
mv extract-* zip/
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_18.png)

```sh
unzip extract-*
sha256sum independert.msi
```

![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_19.png)
![Zeek & Suricata](./assets/assets_Zeek-Suricata-IDS/Zeek&Suricata-Network-Detection_20.png)

The SHA256 hash was flagged as malicious by multiple security vendors.

## Conclusion

This analysis confirms malicious network activity linked to Pikabot malware.
Suricata detected the threat, Zeek provided detailed network visibility, and OSINT validated the malware infrastructure and payload.

- **Key takeaways:**

  - IDS alerts confirmed suspicious traffic

  - DNS and HTTP behavior matched C2 patterns

  - Malware payload was successfully extracted and verified

---

### Useful Resources

- [AbuseIPDB](https://www.abuseipdb.com/)
- [VirusTotal](https://www.virustotal.com/gui/home/upload)
- [DomainTools](https://whois.domaintools.com/)
- [AlienVault OTX](https://otx.alienvault.com/browse/global/pulses?sort=-modified&page=1&include_inactive=0&limit=10)
- [Wireshark](https://2.na.dl.wireshark.org/win64/Wireshark-4.2.3-x64.exe)
