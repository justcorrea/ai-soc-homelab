# AI-Driven SOC Home Lab

Kali-based SOC home lab that captures ICMP traffic with TShark, analyzes packet volume in Python, and sends JSON alerts to an AI SOC agent (Airia) via REST API for automated triage.

## Lab Overview

This project simulates a junior SOC analyst workflow:

1. An "attacker" host sends ICMP traffic (ping) to an internal server.
2. The internal server captures and filters network traffic using TShark.
3. A Python script parses the captured traffic, counts packets per source IP, and flags high-volume sources as suspicious.
4. The script generates a JSON alert with context (packet counts, time window, destination metadata).
5. The alert is sent to an Airia AI agent, which returns risk scoring, escalation decisions, and recommended actions.

## Environment

- **Internal server:** Kali Linux VM (VirtualBox, bridged networking)
- **Attacker:** Windows 10/11 host sending ICMP (ping) to the Kali VM
- **Tools & tech:**
  - Python 3
  - TShark (Wireshark CLI)
  - JSON & REST APIs
  - Airia AI SOC agent (API integration)

## Files

- `soc_lab.py` – main automation script:
  - Captures ICMP traffic with TShark
  - Converts PCAP to CSV
  - Counts packets per source IP and applies a threshold
  - Generates a JSON alert
  - Sends the alert to an Airia AI agent via REST API
- `README.md` – project overview and usage notes

> **Note:** `AIRIA_API_URL` and `AIRIA_API_KEY` are placeholders. Replace them with your own values if you want to run the script end-to-end.

## Workflow Details

1. **Traffic capture (TShark)**
   - Capture interface: `eth0` on the Kali VM
   - Capture filter: ICMP traffic destined to the internal server IP  
     Example filter: `icmp and dst host <your_kali_ip>`

2. **PCAP → CSV conversion**
   - TShark exports key fields (timestamp, source IP, destination IP, protocol, frame length) to a CSV file for easier parsing.

3. **Analysis & detection (Python)**
   - Reads the CSV file and counts the number of packets per `ip.src`.
   - Compares counts against a configurable threshold (e.g., `THRESHOLD = 40`).
   - Flags the first source IP that exceeds the threshold as "suspicious."

4. **Alert generation (JSON)**
   - Builds a JSON alert containing:
     - Alert ID
     - Indicator type/value (suspicious IP)
     - Destination host/IP
     - Evidence (packet count, capture duration, data source)
     - An analyst question for the AI agent

5. **AI SOC triage (Airia API)**
   - Sends the alert JSON to an Airia AI agent endpoint.
   - Receives a structured response with:
     - Risk score / severity
     - Recommended SOC actions
     - Escalation decision
     - Executive summary

## How to Use (High-Level)

1. Configure your Kali VM IP and interface in `soc_lab.py`:
   - `INTERFACE`
   - `DESTINATION_IP`
   - TShark capture filter (`icmp and dst host <your_kali_ip>`)

2. Replace the Airia placeholders with your own values if you have an account:
   - `AIRIA_API_URL`
   - `AIRIA_API_KEY`

3. From your attacker host (Windows), run a longer ping to the Kali IP:
   - `ping <kali_ip> -n 80`

4. On Kali, run the script:
   - `python3 soc_lab.py`

5. Review the console output and Airia’s response to see how the AI agent triages the alert.

## Skills Demonstrated

- Linux (Kali) administration and VirtualBox networking
- Packet capture and filtering with TShark
- Python scripting for basic detection logic and automation
- Working with JSON and REST APIs
- Understanding of SOC alert triage concepts and AI-assisted analysis
