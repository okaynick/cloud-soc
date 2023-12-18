# Creation of a SOC/Cloud Network in Azure (with live traffic)

## Introduction

The purpose of this project is to showcase the creation and securing of a cloud network in Azure. The project includes three parts:  network/SOC creation, exposure of the network to the public internet, and finally incident response and hardening using NIST 800-53 and NIST 800-61. The network includes two virtual machines, a SQL server, blob storage, and a key vault--with a log to capture events from the rest of the network. From there, Microsoft Sentinel (SIEM) is used to scan the event log for security incidents, trigger alerts, and create attack maps.

## Architecture Before Hardening & Security Controls
![Azure-SOC-Insecure](https://github.com/okaynick/cloud-soc/assets/10458811/98d417b6-20fd-4cd0-aa09-006d683efed6)

The network consists of the following:

- Virtual Machines (windows-vm, linux-vm)
- Network Security Group (NSG)
- Log Analytics Workspace
- Azure Key Vault
- Azure Storage Account (Blob)
- Microsoft Sentinel (SIEM)

As well as a handful of users created in Entra ID (formerly Active Directory) given different levels of permissions and access.

All resources were originally deployed without any network protections and exposed to the public internet. Built-in firewalls were disabled and NSG permissions allowed all traffic in-flows. All endpoints were visible to anyone on the internet.

## Metrics Before Hardening & Security Controls

Metrics taken for our insecure environment are as follows. Some of the incidents were intentionally triggered by me using my personal machine or a third attack-vm, but most of the events are from real entities (humans or bots).

Start Time 2023-12-09 9:18:29:02 AM
Stop Time 2023-12-10 9:18:29:02 AM

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 28895
| Syslog                   | 12880
| SecurityAlert            | 7
| SecurityIncident         | 288
| AzureNetworkAnalytics_CL | 3976

## Attack Maps Before Hardening & Security Controls

I created workbooks in Sentinel to map the attackers based on their IP address. The following attack maps correspond to the same time ranges as above.

![BEFORE-nsg-malicious-flows](https://github.com/okaynick/cloud-soc/assets/10458811/c0ffe1b3-303c-4ab4-8d67-ea39f5967628)<br>
![BEFORE-linux-ssh-auth-fail](https://github.com/okaynick/cloud-soc/assets/10458811/721a7769-8360-4493-a5b0-ab77445cec4e)<br>
![BEFORE-windows-rdp-failed-auth](https://github.com/okaynick/cloud-soc/assets/10458811/ad6cea20-d5d6-4e06-ba70-96beeb8ee40a)<br>

## Architecture After Hardening & Security Controls
At the conclusion of the 24 hours of running an insecure network, I handled the security incidents in Sentinel, noting specific vulnerabilities and closing the incidents. Other than the incidents I triggered, most of the incidents were brute force attempts.

Additionally, I added NIST 800-53 controls to Microsoft Defender for Cloud, specifically SC-7 "Boundary Protection." I hardened the environment using these guidelines and the learnings gained from the security incidents. The Network Security Groups were hardening by allowing access to only my personal IP. All other resources were protected by resorting the default settings of the built-in firewalls.

## Metrics After Hardening & Security Controls

The following metrics were recording for a 24-hour period following the hardening of the network environment.
Start Time 2023-12-15 8:45:15 AM
Stop Time	2023-12-16 8:45:15 AM

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 9073
| Syslog                   | 1
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0

The result was a significant drop (-68%) in events as access to resources were protected from the public internet. Attempts to connect to resources within the NSGs were denied access if their IP address did not match my personal IP. No security incidents were triggered.

## Attack Map After Hardening & Security Controls

With no security incidents, all map queries returned no results due to no instances of malicious activity.

![AFTER-security-incidents](https://github.com/okaynick/cloud-soc/assets/10458811/de8a975a-6279-4c4e-b956-06e916be078f)

## Conclusion

In this project, a mini network was created in Microsoft Azure and intentionally left open (honeynet) for attackers to generate security events and incidents. Logs from the various resources were forwarded to a Log Analytics Workspace, and Microsoft Sentinel combed through the logs to look for possible security incidents. Metrics were measured two separate 24-hour periods before and after applying network hardening measures.

In a real life setting, many more security events and incidents would be triggered before and possibly after network hardening. The results of the measures employed demonstrate the effectiveness of such controls recommended by NIST 800-53.
