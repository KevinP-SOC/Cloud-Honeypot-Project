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
  **Full end-to-end video walkthrough: `[Video Link Here soon]`.**


## Steps
`[Images need to be uploaded]`
1) **Create Resource Group, Virtual Network, and Subnet**  
![Ref 1 – RG + VNet + Subnet](LINK_TO_IMAGE_1)  
*Ref 1:* Create a dedicated Resource Group and a Virtual Network with a lab Subnet to keep everything organized and isolated.

2) **Begin VM Creation (Basics Tab)**  
![Ref 2 – VM Basics](LINK_TO_IMAGE_2)  
*Ref 2:* Select the image (Windows/Ubuntu), size, admin username/password (or SSH key), and accept licensing. Use a small SKU to control costs.

3) **Networking: Attach the VM to Your Lab VNet**  
![Ref 3 – VM Networking](LINK_TO_IMAGE_3)  
*Ref 3:* Ensure the the VM is placed on the previously created VNet/Subnet. Leave defaults unless you need a specific NIC name or public IP.

4) **Advanced/Management Tabs (Leave Defaults)**  
![Ref 4 – Defaults Acknowledged](LINK_TO_IMAGE_4)  
*Ref 4:* For this lab, leave most options as-is. You can revisit boot diagnostics, identities, and extensions later if needed.

5) **Disable Boot Diagnostics (Lab Setting)**  
![Ref 5 – Boot Diagnostics Off](LINK_TO_IMAGE_5)  
*Ref 5:* Turn off boot diagnostics for simplicity/cost in this lab. (Optional; keep enabled if you want serial console logs.)

6) **Review + Create → Deploy VM**  
![Ref 6 – Review + Create](LINK_TO_IMAGE_6)  
*Ref 6:* Validate settings, click **Create**, and wait for deployment to finish. Then navigate to the new Resource Group.

7) **Open the VM’s NSG**  
![Ref 7 – NSG in Resource Group](LINK_TO_IMAGE_7)  
*Ref 7:* In the Resource Group, open the Network Security Group (NSG) tied to the VM NIC or subnet to manage inbound/outbound rules.

8) **Remove Default RDP Rule (Cleanup)**  
![Ref 8 – Remove RDP Rule](LINK_TO_IMAGE_8)  
*Ref 8:* Delete the default RDP inbound rule to avoid confusion before adding your lab’s intentional exposure rule.

9) **Add an “Allow All Ports” Inbound Rule (Honeypot Only)**  
![Ref 9 – Allow * Ports](LINK_TO_IMAGE_9)  
*Ref 9:* Add a new inbound rule: Source = Any, Destination = Any, **Port ranges = “*”**, Protocol = Any, Action = Allow.  
⚠️ **Warning:** This is intentionally insecure for honeypot telemetry. Never use in production.

10) **Confirm NSG Rule Order and Effect**  
![Ref 10 – NSG Updated List](LINK_TO_IMAGE_10)  
*Ref 10:* Verify the new rule sits above any deny rules so it actually takes effect.

11) **Copy the VM’s Public IP**  
![Ref 11 – Public IP](LINK_TO_IMAGE_11)  
*Ref 11:* Grab the public IP for RDP/SSH testing and later external scans.

12) **Remote Desktop / SSH to the VM**  
![Ref 12 – RDP Client](LINK_TO_IMAGE_12)  
*Ref 12:* Open **Remote Desktop Connection** (Windows) or SSH (Linux/macOS). Enter the public IP and authenticate with the admin credentials you set.

13) **Initial OS Setup**  
![Ref 13 – First Login](LINK_TO_IMAGE_13)  
*Ref 13:* Complete the initial Windows setup (privacy, updates). For honeypot purposes, keep things simple.

14) **Disable Windows Defender & Firewall (Lab Only)**  
![Ref 14 – Disable Defender/Firewall](LINK_TO_IMAGE_14)  
*Ref 14:* Open **Windows Defender Firewall with Advanced Security** → **Windows Defender Firewall Properties** → Turn off Domain/Private/Public profiles.  
⚠️ **Lab Only:** This is to maximize observable telemetry. Re-enable for any environment you care about.

15) **Verify External Reachability from Host**  
![Ref 15 – Ping/Reachability](LINK_TO_IMAGE_15)  
*Ref 15:* From your host, test reachability (ping if allowed, or `nmap -Pn -p <port> <public-ip>`). You should see the VM responding.

16) **Generate Failed Logons for Telemetry**  
![Ref 16 – Purposely Fail Logon](LINK_TO_IMAGE_16)  
*Ref 16:* Log off and intentionally enter wrong credentials several times to create **4625** failed logon events.

17) **Check Event Viewer for 4625**  
![Ref 17 – Event Viewer 4625](LINK_TO_IMAGE_17)  
*Ref 17:* On the VM: open **Event Viewer → Windows Logs → Security**. Use **Ctrl+F** to find **Event ID 4625** (failed logon). Confirm details match your attempts.

18) **Create Log Analytics Workspace**  
![Ref 18 – Create LAW](LINK_TO_IMAGE_18)  
*Ref 18:* In Azure, create a **Log Analytics Workspace** in the same Resource Group/region. Click **Review + Create** → **Create**.

19) **Enable Microsoft Sentinel on the Workspace**  
![Ref 19 – Enable Sentinel](LINK_TO_IMAGE_19)  
*Ref 19:* Open **Microsoft Sentinel**, select your workspace, and click **Enable** to mount SIEM capabilities.

20) **Install “Windows Security Events” via Content Hub**  
![Ref 20 – Content Hub Install](LINK_TO_IMAGE_20)  
*Ref 20:* In Sentinel → **Content hub**, search **Windows Security Events** and install the solution.

21) **Open Connector and Create Data Collection Rule (AMA)**  
![Ref 21 – AMA Connector](LINK_TO_IMAGE_21)  
*Ref 21:* In **Data connectors**, open **Windows Security Events via AMA**, click **Open connector page**, then **Create data collection rule (DCR)** to onboard the VM.

22) **Target the VM in the DCR**  
![Ref 22 – DCR Targeting VM](LINK_TO_IMAGE_22)  
*Ref 22:* In the DCR wizard, select your VM under resources, validate, and create. This attaches the AMA pipeline to your workspace.

23) **Confirm Extension/DCR on the VM**  
![Ref 23 – VM Extensions](LINK_TO_IMAGE_23)  
*Ref 23:* On the VM resource blade → **Extensions**, verify the AMA/DCR shows as **Provisioning succeeded** before proceeding.

24) **Verify Logs in the Workspace**  
![Ref 24 – LAW Logs Query](LINK_TO_IMAGE_24)  
*Ref 24:* Open the **Log Analytics Workspace → Logs** and run quick KQL checks:  
`SecurityEvent | take 5` and `Heartbeat | take 5`. Confirm the VM is sending data.

25) **Query Failed Logons with KQL**  
![Ref 25 – KQL 4625 Query](LINK_TO_IMAGE_25)  
*Ref 25:* Example:  
kql
SecurityEvent
| where EventID == 4625
| summarize count() by Account, IPAddress=coalesce(IpAddress, SourceNetworkAddress), bin(TimeGenerated, 1h)
| order by TimeGenerated desc

26) **Create a GeoIP Watchlist**  
![Ref 26 – New Watchlist](LINK_TO_IMAGE_26)  
*Ref 26:* In Sentinel → **Watchlists**, click **New**. Name it (e.g., `GeoIP`), set a **SearchKey** column (e.g., `network` or `ip`), and upload your GeoIP CSV.

27) **Confirm Watchlist Upload**  
![Ref 27 – Watchlist Uploaded](LINK_TO_IMAGE_27)  
*Ref 27:* Refresh the watchlist blade and open `GeoIP`. It may take a moment to process large CSVs.

28) **Query the Watchlist in KQL**  
![Ref 28 – Watchlist KQL](LINK_TO_IMAGE_28)  
*Ref 28:* Example:  
kql
_GetWatchlist('GeoIP')
| take 10

29) **Join Failed Logons with GeoIP for Mapping**  
![Ref 29 – KQL Join for Geo](LINK_TO_IMAGE_29)  
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

30) **Create a Workbook for the Map**  
![Ref 30 – New Workbook](LINK_TO_IMAGE_30)  
*Ref 30:* In Sentinel → **Workbooks**, click **Add workbook** → **Edit** → remove defaults → **Add query**.

31) **Paste JSON/Advanced Editor for Map Visualization**  
![Ref 31 – Advanced Editor JSON](LINK_TO_IMAGE_31)  
*Ref 31:* Open **Advanced editor** and paste your JSON for the map visualization (or configure the query + map view manually).

32) **Finish and Interpret the Map**  
![Ref 32 – Map of Attackers](LINK_TO_IMAGE_32)  
*Ref 32:* Click **Done editing** to render the map. Explain color/size encodings (e.g., bubble size = attempts). More runtime = richer data.

33) **(Optional) External Scan Confirmation**  
![Ref 33 – External nmap](LINK_TO_IMAGE_33)  
*Ref 33:* From an external host or Cloud Shell, run: `nmap -Pn -p- <public-ip>` to confirm exposure and observe internet scan/brute-force activity.

---

### Notes
- **Honeypot Warning:** Steps 9 and 14 intentionally reduce security to attract activity. Keep this VM isolated and never reuse credentials.  
- **Costs:** Use small SKUs, budgets, and auto-shutdown to control spend.  
- **Cleanup:** Tear down the Resource Group when finished or keep it only with strict cost guardrails.  
- **Video:** Add your end-to-end walkthrough link here → `[Add Video Link Here]`.

---

### Notes
- This honeypot is **deliberately exposed**—keep it isolated from anything you care about.
- Use strong creds/keys and never reuse passwords.
- Tear down the environment when you’re done, or keep budgets/auto-shutdown enabled.
