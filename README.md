# Home SOC in Azure (Cloud Honeypot)

## Objective
I built an intentionally exposed honeypot in Azure to see how the public internet actually probes a host in real time. The purpose was to practice cloud deployment and end-to-end telemetry: stand up a VM, wire logs into a SIEM (Log Analytics + Microsoft Sentinel), and capture clean endpoint data with Sysmon. The goal was to turn unsolicited scans/brute-force traffic into actionable detections and investigations—write KQL/SIEM rules, triage incidents, pivot across entities (IP, account, process), and document guardrails (NSGs, budgets, auto-shutdown) so the lab stays safe and affordable. In short: deploy a realistic target, observe real attacks, and build the detection/response  a SOC analyst needs.

### Skills Learned
- Practical understanding of cloud-based SIEM operations using Log Analytics and Microsoft Sentinel (data connectors, tables, incidents).
- Proficiency writing and tuning KQL to investigate authentication failures, brute-force patterns, and process activity at scale.
- Ability to interpret unsolicited internet telemetry (scans, RDP/SSH brute force) and separate noise from actionable signals.
- Experience building detections and visualizations (analytics rules, workbooks, geo-mapping via watchlists) to surface attacker trends.
- Stronger grasp of Azure networking and security controls (NSGs, public IP exposure, VM/agent health) and their effect on telemetry.
- Endpoint instrumentation skills with Sysmon and Azure Monitor Agent, validating field quality, timestamps, and host identity.
- Improved incident triage, enrichment, and tuning discipline—reducing false positives while keeping high-fidelity alerts.


### Tools Used
- **Azure**: Resource Groups, Virtual Network/Subnet, **Network Security Groups (NSG)**, Public IPs, **Log Analytics Workspace**, **Microsoft Sentinel**, Defender for Cloud.
- **Windows Server / Windows 10** (honeypot VM) or **Ubuntu** (for SSH bait).
- **Sysmon** + **Azure Monitor Agent (AMA)** for endpoint telemetry into Log Analytics.
- **Microsoft Sentinel** (or **Splunk** if forwarding) for analytics rules, incidents, and dashboards.
- **PowerShell/Bash** for setup checks
-  **nmap** for external reachability tests.

## Video Walkthrough
  **Full end-to-end video walkthrough: https://youtu.be/sPw7q_KpGQw.**

## Additional Needed Files
   https://drive.google.com/file/d/13EfjM_4BohrmaxqXZLB5VUBIz2sv9Siz/view.
   https://drive.google.com/file/d/1ErlVEK5cQjpGyOcu4T02xYy7F31dWuir/view

## Attack map after 72 hours
<img width="1045" height="683" alt="Attack map 72 hours" src="https://github.com/user-attachments/assets/5ec64623-4de5-40ca-99d8-99c1b940bf9a" />

## Steps

1) **Create Resource Group, Virtual Network, and Subnet**  

<img width="1873" height="1094" alt="image" src="https://github.com/user-attachments/assets/ef9bbc79-e114-4751-9a24-4648d895743f" />
<img width="1873" height="1066" alt="image" src="https://github.com/user-attachments/assets/91336101-928d-4caa-855d-b509a77f9044" />

*Ref 1:* Create a dedicated Resource Group and a Virtual Network with a lab Subnet to keep everything organized and isolated.

2) **Begin VM Creation (Basics Tab)**  
<img width="1873" height="1904" alt="image" src="https://github.com/user-attachments/assets/fdc432eb-f7dd-4547-90cd-e61b6cec4a7f" />
  
*Ref 2:* Select the image (Windows/Ubuntu), size, admin username/password (or SSH key), and accept licensing. Use a small SKU to control costs.
*Ref 3:* Ensure the the VM is placed on the previously created VNet/Subnet. Leave defaults unless you need a specific NIC name or public IP.

3) **Advanced/Management Tabs (Leave Defaults)**  
<img width="1830" height="1645" alt="image" src="https://github.com/user-attachments/assets/f324bd64-bdda-43c0-ba66-5ce25bf4071c" />
 
*Ref 4:* For this lab, leave most options as-is. You can revisit boot diagnostics, identities, and extensions later if needed.

4) **Disable Boot Diagnostics (Lab Setting)**  
<img width="1800" height="1010" alt="image" src="https://github.com/user-attachments/assets/b8aeaf3c-3643-4943-828a-969cf996f485" />

*Ref 5:* Turn off boot diagnostics for simplicity/cost in this lab. (Optional; keep enabled if you want serial console logs.)

5) **Review + Create → Deploy VM**  
*Ref 6:* Validate settings, click **Create**, and wait for deployment to finish. Then navigate to the new Resource Group.

6) **Open the VM’s NSG**  
<img width="3813" height="1149" alt="image" src="https://github.com/user-attachments/assets/28de399f-fda5-4281-b0a3-fd011ef9f22c" />
*Ref 7:* In the Resource Group, open the Network Security Group (NSG) tied to the VM NIC or subnet to manage inbound/outbound rules.

7) **Remove Default RDP Rule (Cleanup)**  
<img width="3812" height="1096" alt="image" src="https://github.com/user-attachments/assets/8a030840-6292-4977-b2e7-2a1d9ba85bf0" />
*Ref 8:* Delete the default RDP inbound rule to avoid confusion before adding your lab’s intentional exposure rule.

8) **Add an “Allow All Ports” Inbound Rule (Honeypot Only)**  
<img width="880" height="1919" alt="image" src="https://github.com/user-attachments/assets/d1863aa5-47f8-4f4c-be69-9022a851dd1f" />
*Ref 9:* Add a new inbound rule: Source = Any, Destination = Any, **Port ranges = “*”**, Protocol = Any, Action = Allow.  
⚠️ **Warning:** This is intentionally insecure for honeypot telemetry. Never use in production.

9) **Confirm NSG Rule Order and Effect**  
<img width="3830" height="802" alt="image" src="https://github.com/user-attachments/assets/44cc1e88-8dc8-45b3-8206-6feccc88566a" />
*Ref 10:* Verify the new rule sits above any deny rules so it actually takes effect.

10) **Copy the VM’s Public IP**  
<img width="2967" height="1915" alt="image" src="https://github.com/user-attachments/assets/de9d5f59-c7df-4cc2-b034-ba215ba43a58" />
*Ref 11:* Grab the public IP for RDP/SSH testing and later external scans.

11) **Remote Desktop / SSH to the VM**  
<img width="640" height="395" alt="image" src="https://github.com/user-attachments/assets/2026870c-1aee-4405-985d-ceb8758b08bb" />
*Ref 12:* Open **Remote Desktop Connection** (Windows) or SSH (Linux/macOS). Enter the public IP and authenticate with the admin credentials you set.

12) **Initial OS Setup**  
<img width="2241" height="1561" alt="image" src="https://github.com/user-attachments/assets/31d78797-aea7-48de-8793-c40bc97ff108" />
*Ref 13:* Complete the initial Windows setup (privacy, updates). For honeypot purposes, keep things simple.

13) **Disable Windows Defender & Firewall (Lab Only)**  
<img width="650" height="713" alt="image" src="https://github.com/user-attachments/assets/efdfff6c-7ece-47e6-8d99-c0d393f407c8" />
*Ref 14:* Open **Windows Defender Firewall with Advanced Security** → **Windows Defender Firewall Properties** → Turn off Domain/Private/Public profiles.  
⚠️ **Lab Only:** This is to maximize observable telemetry. Re-enable for any environment you care about.

14) **Verify External Reachability from Host**  
<img width="3835" height="2106" alt="image" src="https://github.com/user-attachments/assets/44110369-4e3f-4166-961b-758f6d482b09" />
*Ref 15:* From your host, test reachability (ping if allowed, or `nmap -Pn -p <port> <public-ip>`). You should see the VM responding.

15) **Generate Failed Logons for Telemetry**  
<img width="716" height="590" alt="image" src="https://github.com/user-attachments/assets/b4b06ea7-2ff4-4bca-8f96-70326f7151ec" />
*Ref 16:* Log off and intentionally enter wrong credentials several times to create **4625** failed logon events.

16) **Check Event Viewer for 4625**  
<img width="2214" height="1387" alt="image" src="https://github.com/user-attachments/assets/ac1d8ddb-7c8b-4aaf-afc3-96dd777037fa" />
*Ref 17:* On the VM: open **Event Viewer → Windows Logs → Security**. Use **Ctrl+F** to find **Event ID 4625** (failed logon). Confirm details match your attempts.

17) **Create Log Analytics Workspace**  
<img width="1363" height="1969" alt="image" src="https://github.com/user-attachments/assets/bc20d038-c4e4-45f7-8a04-cdc68f6350a0" />
*Ref 18:* In Azure, create a **Log Analytics Workspace** in the same Resource Group/region. Click **Review + Create** → **Create**.

18) **Enable Microsoft Sentinel on the Workspace**  
<img width="2543" height="1950" alt="image" src="https://github.com/user-attachments/assets/f6db144c-fb66-47a1-96a0-27a042d5c899" />
*Ref 19:* Open **Microsoft Sentinel**, select your workspace, and click **Enable** to mount SIEM capabilities.

19) **Install “Windows Security Events” via Content Hub**  
<img width="3177" height="1952" alt="image" src="https://github.com/user-attachments/assets/f34c7574-4c12-4989-b05b-cbec905ee2a6" /> 
*Ref 20:* In Sentinel → **Content hub**, search **Windows Security Events** and install the solution.

20) **Open Connector and Create Data Collection Rule (AMA)**  
<img width="3817" height="1900" alt="image" src="https://github.com/user-attachments/assets/5e19d2dc-8bce-40cb-b09a-ee6c10d1f5a9" />
<img width="1796" height="1414" alt="image" src="https://github.com/user-attachments/assets/e4d78ee5-ea25-46c1-86f0-cb57d5350dc9" />

*Ref 21:* In **Data connectors**, open **Windows Security Events via AMA**, click **Open connector page**, then **Create data collection rule (DCR)** to onboard the VM.

21) **Target the VM in the DCR**  
<img width="1274" height="1927" alt="image" src="https://github.com/user-attachments/assets/871ddc4d-3db6-4329-86e0-702511ec3fdd" />
 <img width="1277" height="1911" alt="image" src="https://github.com/user-attachments/assets/dcc6e27f-5f17-44c3-85bd-101de6fb90e6" />

*Ref 22:* In the DCR wizard, select your VM under resources, validate, and create. This attaches the AMA pipeline to your workspace.

22) **Confirm Extension/DCR on the VM**  
<img width="3822" height="1916" alt="image" src="https://github.com/user-attachments/assets/2205c0e8-d69f-46dc-8e31-cc9133f98301" />
 
*Ref 23:* On the VM resource blade → **Extensions**, verify the AMA/DCR shows as **Provisioning succeeded** before proceeding.

23) **Verify Logs in the Workspace**  
<img width="3843" height="1918" alt="image" src="https://github.com/user-attachments/assets/ce74d659-1d20-4bb0-b38c-21b7b45c0791" />
 
*Ref 24:* Open the **Log Analytics Workspace → Logs** and run quick KQL checks:  
`SecurityEvent | take 5` and `Heartbeat | take 5`. Confirm the VM is sending data.

24) **Query Failed Logons with KQL**  
<img width="3828" height="965" alt="image" src="https://github.com/user-attachments/assets/d6453ff3-d92a-461b-9bb2-f6fd28b308e2" />
 
*Ref 25:* Example:  
kql
SecurityEvent
| where EventID == 4625
| summarize count() by Account, IPAddress=coalesce(IpAddress, SourceNetworkAddress), bin(TimeGenerated, 1h)
| order by TimeGenerated desc

25) **Create a GeoIP Watchlist**  
<img width="2021" height="2105" alt="image" src="https://github.com/user-attachments/assets/a897bb1a-42bb-468d-a48c-c1020366fc15" />
<img width="2400" height="1629" alt="image" src="https://github.com/user-attachments/assets/468c06da-b358-4c3b-b279-513f48332fcf" />
<img width="3831" height="1147" alt="image" src="https://github.com/user-attachments/assets/b768245b-38e5-4d13-a7ea-59f0836ad5ea" />
*Ref 26:* In Sentinel → **Watchlists**, click **New**. Name it (e.g., `GeoIP`), set a **SearchKey** column (e.g., `network` or `ip`), and upload your GeoIP CSV.

26) **Confirm Watchlist Upload**  
<img width="1030" height="655" alt="image" src="https://github.com/user-attachments/assets/cdbb99cf-ee4d-44b3-88a8-07642c2c97e4" />
*Ref 27:* Refresh the watchlist blade and open `GeoIP`. It may take a moment to process large CSVs.

27) **Query the Watchlist in KQL**  
<img width="3842" height="1969" alt="image" src="https://github.com/user-attachments/assets/51df0382-2627-446e-9097-c4341f109da9" />

*Ref 28:* Example:  
kql
_GetWatchlist('GeoIP')
| take 10

28) **Join Failed Logons with GeoIP for Mapping**  
<img width="3368" height="1901" alt="image" src="https://github.com/user-attachments/assets/34f84497-a303-4392-9a1a-d867eb90dded" />
 
*Ref 29:* Example:
    let attackers =
        SecurityEvent
        | where EventID == 4625
        | project TimeGenerated, Account, IpAddress = coalesce(IpAddress, SourceNetworkAddress);
    let geo =
        _GetWatchlist('GeoIP')
        | project network, latitude, longitude, country, city;
    attackers
    | join kind=leftouter geo on $left.IpAddress == $right.network
    | summarize attempts = count() by country, city, latitude, longitude

29) **Create a Workbook for the Map**  
<img width="3839" height="1975" alt="image" src="https://github.com/user-attachments/assets/7cfefdac-6d07-4dc7-8cab-4cd03fa1ade2" />
  
*Ref 30:* In Sentinel → **Workbooks**, click **Add workbook** → **Edit** → remove defaults → **Add query**.

30) **Paste JSON/Advanced Editor for Map Visualization**  
<img width="1205" height="1279" alt="image" src="https://github.com/user-attachments/assets/32a360db-308d-40f5-a921-c5c0884b49d6" />
 
*Ref 31:* Open **Advanced editor** and paste your JSON for the map visualization (or configure the query + map view manually).

31) **Finish and Interpret the Map**  
<img width="1896" height="1247" alt="image" src="https://github.com/user-attachments/assets/ff72ff2e-43af-4117-9b58-414a8cdc8139" />
<img width="3833" height="1971" alt="image" src="https://github.com/user-attachments/assets/a2163b24-b2f4-463a-882e-5eaa5ebd6de2" />

*Ref 32:* Click **Done editing** to render the map. Explain color/size encodings (e.g., bubble size = attempts). More runtime = richer data.



---

### Notes
- **Honeypot Warning:** Steps 9 and 14 intentionally reduce security to attract activity. Keep this VM isolated and never reuse credentials.  
- **Costs:** Use small SKUs, budgets, and auto-shutdown to control spend.  
- **Cleanup:** Tear down the Resource Group when finished or keep it only with strict cost guardrails.  

