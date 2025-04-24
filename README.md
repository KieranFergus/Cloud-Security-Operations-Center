# Azure Honeypot Lab – Cybersecurity Logging & Detection

This project followed a guided lab to deploy a Windows-based honeypot in Microsoft Azure, analyze security logs using Azure Sentinel and KQL, and create an attack map. My goal was to explore detection visualization techniques used in modern cloud SOC environments.

---

## Tools & Services Used

- Microsoft Azure (Virtual Machines, Network Security Groups)
- Windows 10 (target/honeypot)
- Azure Sentinel (SIEM)
- Log Analytics Workspace (LAW)
- Kusto Query Language (KQL)
- Watchlists (GeoIP enrichment)
- Event Viewer (Windows logs)

---

## Lab Summary

| Part 1 | Create honeypot VM and expose services |
| Part 2 | Observe failed login attempts and analyze raw Windows logs |
| Part 3 | Forward logs to Azure Sentinel and query using KQL |
| Part 4 | Enrich logs using a GeoIP watchlist |
| Part 5 | Visualize attacker location with an attack map |

Here is a visual representation of the lab:
![Architecture](./images/Architecture.png)

---

## Part 1: Deploying the Honeypot VM

- Created a Windows 10 VM in Azure.
- Configured Network Security Group (NSG) to allow all inbound traffic.
- Disabled Windows Firewall (`wf.msc` → Properties → Turn off all profiles).

Disabled the Windows Defender firewall within the Virtual Machine instance:
![Firewall Rules](./images/VMFirewall.png)

---

## Part 2: Simulate & Observe Brute Force Logins

- Attempted 3+ failed logins using fake user (e.g., `employee`).
- Logged in successfully afterward.
- Opened **Event Viewer → Security logs**, filtered for:
  - Event ID `4625` (failed login attempts)

---

## Part 3: Log Forwarding & Sentinel Integration

- Created a **Log Analytics Workspace (LAW)**.
- Deployed **Azure Sentinel** and connected it to the LAW.
- Enabled the **“Windows Security Events via AMA”** data connector.
- Queried logs in Sentinel using KQL:

```kql
SecurityEvent
| where EventID == 4625
```

---

## Part 4: GeoIP Enrichment Using Watchlists

- Imported a CSV (`geoip-summarized.csv`) as a **Sentinel Watchlist**:
  - Name: `geoip`
  - Search key: `network`
- Applied `ipv4_lookup()` on attacker IPs to derive location data:

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == "<attacker-IP>"
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
```

---

## Part 5: Attack Map Visualization

- Created a custom **Sentinel Workbook**
- Deleted default visualizations and added a **Query + Map** element.
- Imported JSON from `map.json` to visualize attacker IP geolocation.

---

## Key Takeaways

- Built a honeypot in Azure and simulated credential attacks.
- Used **Event Viewer** and **Azure Sentinel** to detect and visualize events.
- Learned basic **KQL queries** for event filtering.
- Enriched raw logs with external data (GeoIP) using **Watchlists**.
- Visualized attack origin using **Sentinel Workbooks**.


