# Network Traffic Analysis with Wireshark

## Project Overview

This beginner SOC analyst project explores network traffic analysis using packet captures (PCAP/PCAPNG) and Wireshark. Its planned labs cover normal traffic, HTTP traffic, and suspicious activity, with an emphasis on protocol analysis, evidence collection, timelines, and investigation reporting.

## Learning Objectives

- Analyze TCP/IP protocols and investigate DNS traffic.
- Examine TCP handshakes and HTTP/HTTPS traffic.
- Inspect packets in Wireshark using display filters, Follow TCP Stream, and statistics.
- Investigate network traffic and identify potentially unusual behavior.
- Collect evidence and prepare incident reports.

## Technologies & Tools

- **Wireshark:** planned tool for capturing and inspecting packets.
- **Windows PowerShell:** environment for the example traffic-generation commands below.
- **Git:** version control for the repository.
- **Markdown:** format for documentation and planned reports.

Wireshark and PowerShell are part of the proposed lab workflow; no capture or analysis demonstrating their use is currently included in the repository.

## Prerequisites

- Basic networking and TCP/IP fundamentals.
- Familiarity with DNS, HTTP/HTTPS, and ICMP.
- Basic Wireshark concepts, including packet capture and display filters.
- Basic Windows PowerShell and Git usage.
- Wireshark and Npcap installed on Windows for live capture, or an authorized PCAP/PCAPNG file to inspect.

## Repository Structure

```text
soc-network-traffic-analysis/
├── README.md
├── .gitignore
├── docs/
│   └── .gitkeep
├── labs/
│   ├── 01-normal-traffic/
│   │   ├── captures/.gitkeep
│   │   ├── screenshots/.gitkeep
│   │   └── reports/.gitkeep
│   ├── 02-http-investigation/
│   │   ├── captures/.gitkeep
│   │   ├── screenshots/.gitkeep
│   │   └── reports/.gitkeep
│   └── 03-suspicious-traffic/
│       ├── captures/.gitkeep
│       ├── screenshots/.gitkeep
│       └── reports/.gitkeep
├── filters/
│   └── .gitkeep
└── scripts/
    └── .gitkeep
```

- [`docs/`](docs/): reserved for general project documentation.
- [`labs/`](labs/): case study workspaces; each has directories for captures, screenshots, and reports.
- [`filters/`](filters/): reserved for reusable Wireshark filters.
- [`scripts/`](scripts/): reserved for future supporting scripts; none are currently included.
- `.gitkeep` files preserve empty directories in Git. [`.gitignore`](.gitignore) excludes PCAP/PCAPNG files and common sensitive and temporary files.

## Getting Started

1. Clone the repository and enter it:

   ```powershell
   git clone https://github.com/BonM2/soc-network-traffic-analysis.git
   cd soc-network-traffic-analysis
   ```

2. Download and install [Wireshark for Windows](https://www.wireshark.org/download.html). During installation, install Npcap when offered for live packet capture, following the installer instructions.
3. Open Wireshark and select the network interface carrying your authorized traffic. Start a capture, or use **File → Open** to inspect an authorized PCAP/PCAPNG file instead.
4. For a live capture, generate a small amount of traffic in PowerShell, then stop the capture:

   ```powershell
   nslookup example.com
   curl.exe --http1.1 -I https://example.com
   ping -n 4 1.1.1.1
   ```

   `nslookup` requests DNS resolution; `curl.exe` requests HTTPS response headers over HTTP/1.1; and `ping` sends four ICMP echo requests. Results depend on connectivity, DNS caching or encrypted DNS, local policies, and the selected interface. ICMP may be blocked, and HTTPS request and response content is encrypted unless decryption is configured.
5. Enter a display filter such as `dns`, `tcp`, `tls`, `icmp`, or `http` in Wireshark's filter bar. Display filters change what is shown, not the captured packets. The `http` filter shows decoded HTTP, not ordinary encrypted HTTPS contents.
6. Begin the [normal traffic lab](labs/01-normal-traffic/): inspect visible DNS queries and responses, a TCP handshake, TLS traffic, and ICMP where available. Use **Statistics → Protocol Hierarchy** and **Follow → TCP Stream** where useful. Record packet numbers, timestamps, endpoints, observations, and limitations for a future report. A dedicated lab guide is not yet available.

## Case Studies

| Case Study | Objective | Protocols | Status |
| --- | --- | --- | --- |
| Normal Network Traffic Analysis | Examine normal network activity and protocol exchanges. | DNS, TCP, TLS, ICMP | Planned |
| HTTP Traffic Investigation | Examine requests, responses, headers, status codes, and TCP streams in a suitable capture. | HTTP, TCP | Planned |
| Suspicious Network Traffic Investigation | Investigate potentially suspicious traffic, collect evidence, and prepare a report. | To be determined by the capture | Planned |

## Analysis Methodology

**PCAP Collection → Packet Filtering → Protocol Analysis → Evidence Collection → Investigation → Reporting**

1. Collect an authorized capture or obtain a permitted dataset; record its source and capture context.
2. Apply display filters to focus on relevant traffic while retaining the original capture.
3. Inspect protocol exchanges, streams, and statistics; establish a normal baseline when possible.
4. Record packet numbers, timestamps, endpoints, screenshots, and relevant context as evidence.
5. Build a timeline and investigate anomalies, distinguishing observations from hypotheses and noting limitations.
6. Report the method, evidence, assessment, and any recommended follow-up.

## Expected Deliverables

The following are planned outputs, not completed results:

- PCAP analysis reports.
- Reusable Wireshark display filters.
- Investigation screenshots.
- Network traffic timelines.
- An incident report for the suspicious traffic case.