# <div align='left'>VPN Performance Optimization using ASUS VPN Fusion + Wireguard (ProtonVPN)</div>

<div align='left'>Author: Nic Betancourt</div>
<div align='left'>Date & Time: 6/23/2025 12:43pm CDT</div>
<div align='left'>Version 1.0</div>

# Overview

In this case study, I walk through how I identified and fixed a major speed issue on my home network — specifically when using ProtonVPN’s WireGuard protocol through an ASUS router’s VPN Fusion feature. By digging into routing and network configuration, I managed to increase my encrypted download speed from just **69 Mbps to over 810 Mbps** resulting in a 60-80% change.

## Problem

I first noticed a delayed response time from applications and web browsers during system start-up. Web pages would take several seconds to load, and some apps with online components (e.g., Spotify, Google) appeared to hang briefly or fail to connect on the first attempt.

The issue was isolated to my desktop, which is connected via Ethernet and configured to route traffic through ProtonVPN using ASUS VPN Fusion. Other devices on the network — whether using the VPN or not — were unaffected. Speed tests on the desktop confirmed consistently degraded performance, particularly for download speeds, which ranged from **~60–200 Mbps** despite a Gigabit internet connection.

This prompted a deeper investigation into the VPN tunnel’s routing configuration and potential MTU-related inefficiencies.
</br>
</br>

# Phase 1: Optimizing VPN Routing and Server Selection

### What I Observed:

- Checked router speed: **~936 Mbps down / ~845 Mbps up**

- Desktop speed test without VPN: **~970 Mbps down / ~845 Mbps up**

- Desktop routed through VPN showed poor performance: **~60–200 Mbps down**

### Actions I Took:

1. [X] Ran baseline ping tests to assess response time and DNS resolution using these commands below:

```powershell
ping 8.8.8.8
ping google.com
nslookup google.com
```
   ![image](https://github.com/user-attachments/assets/6098309c-4807-41c2-bdc0-46caafdd7d4e)

2. [X] Verified that my desktop was properly assigned to the ProtonVPN client under the VPN Fusion configuration in the router dashboard, with NAT enabled.
   - Router dashboard > general > VPN > VPN Fusion
   ![image](https://github.com/user-attachments/assets/06b087ce-4e8e-41e7-aa4a-e358dd6f3340)
   
4. [X] Identified the active VPN endpoint and switched to a lower-latency regional WireGuard server by generating a new configuration file through ProtonVPN’s dashboard.
   ![image](https://github.com/user-attachments/assets/704b6fb5-4f68-431d-b675-603936bb7111)

5. [X] Disabled lower-priority devices from the VPN Fusion device list to reduce client load
   </br>![image](https://github.com/user-attachments/assets/b4b5d7fc-4ca6-48d2-9a62-5c0193a16156)
   
## Result

Running ping tests revealed that IP-based pinging (e.g., `ping 8.8.8.8`) returned successfully, while domain-based pinging (e.g., `ping google.com`) timed out. The `nslookup` command returned correct IP addresses but showed the DNS server as "unknown." While this didn’t appear to cause immediate service issues, it helped confirm that DNS resolution over the VPN was inconsistent or slow.

After the VPN configuration and server switch, desktop download speed **improved to ~350 Mbps, with upload speeds around ~290 Mbps**. This confirmed that server endpoint selection, routing, and client priority were significant contributors to the slowdown.
</br>
</br>

# Phase 2: Resolving MTU Fragmentation for Better Throughput

### What I Observed:

- After changing the VPN endpoint, download speeds remained low and online services showed delays.

- Router’s configuration file for VPN Fusion did not have a limit on MTU.

### How I Tested It:

1. [X] As administrator, I used PowerShell to perform MTU discovery.
2. [X] Determined that 1380 was the largest non-fragmenting payload, implying a usable MTU of 1408.

```powershell
ping -f -l 1380 8.8.8.8
```
  ![image](https://github.com/user-attachments/assets/67be640e-f856-434c-b6d7-cd74b67db6aa)


### What I Did:

1. [X] Manually set the MTU to 1400 in the WireGuard configuration on the router.
2. [X] Restarted the VPN tunnel and retested network performance

   ![image](https://github.com/user-attachments/assets/109a7705-f4d5-4620-9fed-c8d05a865193)


## Result
**The final result was a speed test with the download increased to ~810 Mbps and the upload steady at ~350 Mbps!**

This confirmed that packet fragmentation inside the tunnel was responsible for the remaining performance bottleneck. The results also demonstrated that a properly tuned MTU can significantly reduce latency and packet loss, especially in encrypted tunnels like WireGuard. Since this adjustment aligned the MTU across both the local network and VPN tunnel, it allowed for more efficient packet delivery and improved consistency in application response times.

![Result-speed-test](https://github.com/user-attachments/assets/ad8b8f7b-2529-4e9d-b31a-5bc35cb83e76)


## Takeaway

Small configuration issues — such as a mismatched MTU or inefficient VPN routing — can significantly degrade performance on otherwise fast home networks. In this case, I used a two-phase approach to resolve these issues. In Phase 1, I optimized VPN routing by assigning the desktop to the correct VPN client, switching to a better regional WireGuard server, and adjusting device priorities, which boosted download speed from ~60–200 Mbps to ~350 Mbps. In Phase 2, I tuned the MTU based on packet fragmentation tests and adjusted the WireGuard configuration, raising download speeds to ~810 Mbps and stabilizing application performance.

A properly tuned configuration file and MTU can significantly reduce latency and packet loss, especially in encrypted tunnels such as WireGuard. Since this adjustment aligned the MTU across the local network and VPN tunnel, it allowed for more efficient packet delivery and improved consistency in application response times. Understanding routing logic, client prioritization, and packet-level tuning can significantly improve performance in VPN-secured environments. Further tuning and testing could potentially maximize more performance, but utilizing a VPN could generally be limiting as it tunnels encrypted data.
