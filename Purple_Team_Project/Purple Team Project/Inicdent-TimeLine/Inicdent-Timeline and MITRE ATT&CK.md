# Purple Team Attack Timeline and MITRE ATT&CK Mapping

## 1. Overview

This report documents offensive security tests and defensive monitoring activities conducted in a controlled lab environment on October 16, 2026.

The objective was to evaluate network reconnaissance detection, SSH authentication security, Windows endpoint telemetry, DNS event visibility, and file integrity monitoring using Kali Linux, a Windows endpoint, Wazuh, and Suricata.

The report correlates the activities performed during testing with the available defensive evidence and maps relevant behaviors to MITRE ATT&CK where appropriate.

## 2. Lab Environment

|Component|Description|
|---|---|
|Attacker Machine|Kali Linux|
|Target Machine|Windows Endpoint|
|Target IP|`192.168.255.148`|
|SIEM and Endpoint Monitoring|Wazuh|
|Network Monitoring|Suricata|
|Testing Tools|Nmap, Hydra, Metasploit Framework, PowerShell|
|Assessment Date|October 16, 2026|

## 3. Attack Timeline and Investigation Findings

### Phase 1 — Network Reconnaissance

**Activity**

Nmap was used to scan the lab target and identify available network services.

**MITRE ATT&CK**

- **Technique:** T1046 — Network Service Discovery
    
- **Tactic:** Discovery
    

**Defensive Evidence**

A local Suricata rule was configured to identify potential TCP SYN port-scanning activity using signature ID `1001001`.

Wazuh recorded a corresponding Suricata alert with the following details:

- **Wazuh Rule ID:** `86601`
    
- **Alert Description:** `Suricata: Alert - LOCAL RECON Possible TCP SYN Port Scan`
    

**Analysis**

The alert demonstrates that Suricata-generated network detection telemetry was forwarded to and recorded by Wazuh. It provides evidence of an alert associated with potential TCP SYN port-scanning activity.

The alert should be correlated with the original network traffic to establish the scope and accuracy of the detection.

### Phase 2 — SSH Authentication Testing

**Activity**

Hydra was used to perform SSH password-guessing attempts against the lab target.

**MITRE ATT&CK**

- **Technique:** T1110.001 — Password Guessing
    
- **Tactic:** Credential Access
    

**Observed Result**

Hydra reported finding valid credentials during the test.

**Defensive Analysis**

The reported result indicates that the password-guessing test identified credentials accepted by the target service. Authentication logs are needed to establish the corresponding login events and determine whether a successful session was created.

Relevant evidence includes repeated authentication failures, successful SSH logins, source IP addresses, targeted accounts, and timestamps.

**Security Relevance**

The result highlights the importance of strong passwords, restricting SSH access to trusted sources, enforcing appropriate authentication controls, and monitoring failed and successful login attempts.

The available results do not independently establish whether Wazuh generated a corresponding authentication alert.

### Phase 3 — Windows Payload Execution Testing

**Activity**

A payload execution test was conducted on the Windows endpoint as part of the controlled lab assessment.

**Potential MITRE ATT&CK Mapping**

- **Technique:** T1204.002 — Malicious File
    
- **Tactic:** Execution
    
- **Mapping limitation:** This technique is applicable when execution depends on a user opening or running a malicious file. The event evidence described below does not independently establish all of these conditions.
    

**Defensive Evidence**

Wazuh recorded Windows Error Reporting events associated with the tested executable.

One event referenced `payload(2).exe` and included the report name `RADAR_PRE_LEAK_64`.

A subsequent application error record referenced `payload.exe` and identified `mswsock.dll` as the faulting module. The recorded exception code was `c00000fd`.

**Analysis**

These events provide endpoint telemetry that can support an investigation into application execution and errors.

However, Windows Error Reporting events alone do not establish that the executable was malicious, that it successfully performed its intended actions, or that a reverse connection was established.

Further correlation with process creation events, file activity, and network connections would be required to determine the execution outcome and any associated activity.

### Phase 4 — DNS Event Investigation

**Activity**

DNS resolution activity involving `sdawx.shop` was observed in the Windows endpoint logs.

**Defensive Evidence**

Windows generated a DNS Client event indicating that name resolution timed out.

**MITRE ATT&CK Mapping**

No specific technique is assigned based on the available evidence. A DNS timeout alone does not establish command-and-control activity or prove that the domain was malicious.

**Analysis**

DNS monitoring provides visibility into name-resolution failures and potentially suspicious domain activity.

The timeout indicates that the recorded name-resolution request did not complete successfully within the applicable time limit. It does not, by itself, identify the cause of the failure.

Additional DNS logs, process telemetry, network traffic, and domain reputation information would be needed to investigate the origin and purpose of the query.

### Phase 5 — File Integrity Monitoring

**Activity**

A controlled file modification was performed on the Windows endpoint to validate real-time File Integrity Monitoring (FIM).

**Monitored File**

`C:\FIM-Test\test.txt`

The file was modified using PowerShell:

```
Set-Content -Path "C:\FIM-Test\test.txt" -Value "Realtime test $(Get-Date -Format o)"
```

**MITRE ATT&CK**

- **Technique:** T1565.001 — Stored Data Manipulation
    
- **Tactic:** Impact
    
- **Mapping limitation:** The technique describes adversarial manipulation of stored data. In this assessment, the file change was a controlled test used to validate detection; the mapping does not imply malicious intent.
    

**Defensive Evidence**

Wazuh generated a file integrity alert with the following details:

|Field|Value|
|---|---|
|Alert Description|`Integrity checksum changed.`|
|Rule ID|`550`|
|Severity Level|`7`|
|Event Type|`modified`|
|Monitoring Mode|`realtime`|
|Agent Name|`Windows-Endpoint`|
|Agent IP|`192.168.255.148`|
|File|`c:\fim-test\test.txt`|
|Event ID|`1792133692.66799318`|
|Event Timestamp|`2026-10-16T06:54:52.824+0000`|

Wazuh recorded changes to the file's size, modification time, MD5, SHA-1, and SHA-256 hashes.

The file size increased from 18 bytes to 49 bytes. The recorded modification time changed from `2026-10-16 06:37:21` to `2026-10-16 06:54:34`.

The cryptographic hashes also changed:

|Hash|Before|After|
|---|---|---|
|MD5|`da62c9b67d61447e673dde5bf5ba7fd2`|`01efe633ecedcdc0a664576b49cc95bb`|
|SHA-1|`0dbd01defe20814eddd402cc2be79b12868c6cb7`|`95bdc286ae17b85ccf3f01151244e368b2ca4d15`|
|SHA-256|`0b180c2b599687adcc28253ddda5b1c8d341e07dcf1eb5175c946677f0c70b45`|`cdd542da867083809cfa6e8b459d5955244f1c8c6e09239f115ea0f7e936b278`|

**Analysis**

The alert and before-and-after integrity metadata confirm that Wazuh detected the controlled modification to the monitored file and recorded the resulting attribute changes.

This is the clearest confirmed detection outcome in the assessment. The event does not identify the process or user responsible for the modification, nor does it establish malicious intent.

## 4. MITRE ATT&CK Mapping Summary

|Activity|Technique|Tactic|Evidence and Limitations|
|---|---|---|---|
|Network reconnaissance|T1046 — Network Service Discovery|Discovery|Suricata alert recorded by Wazuh under rule `86601`.|
|SSH password guessing|T1110.001 — Password Guessing|Credential Access|Hydra reported valid credentials; corresponding authentication logs are needed to confirm the login outcome.|
|Windows payload testing|T1204.002 — Malicious File|Execution|Windows Error Reporting events referenced the tested executable; these records alone do not establish malicious behavior or successful payload activity.|
|DNS investigation|No technique assigned|Not established|A DNS resolution timeout was recorded; command-and-control activity was not established.|
|Controlled file modification|T1565.001 — Stored Data Manipulation|Impact|Wazuh rule `550`, severity `7`, recorded the file modification and integrity changes. The activity was a controlled validation test.|

MITRE ATT&CK mappings provide a common language for describing relevant behaviors. A mapping is not proof that an adversary performed the behavior, and it does not establish that a detection was successful unless supporting evidence is available.

## 5. Defensive Monitoring Summary

The assessment produced several types of security telemetry:

- **Network monitoring:** Suricata generated a potential TCP SYN port-scanning alert that was recorded by Wazuh.
    
- **Authentication security testing:** Hydra reported valid credentials during SSH password-guessing activity. Authentication logs are needed to confirm the resulting login activity.
    
- **Endpoint telemetry:** Windows Error Reporting records associated with the tested executable were available for investigation.
    
- **DNS monitoring:** A Windows DNS Client event recorded a name-resolution timeout involving `sdawx.shop`.
    
- **File integrity monitoring:** Wazuh detected the controlled file modification and recorded changes to the file's size, modification time, and cryptographic hashes.
    

These findings demonstrate how network and endpoint telemetry can contribute to a Purple Team assessment. The degree of detection coverage varies by activity, and not every test has a confirmed detection outcome.

## 6. Key Findings

1. **Network detection:** Suricata alerts were available through Wazuh for investigating potential port-scanning activity.
    
2. **Authentication security:** The SSH password-guessing test identified valid credentials, highlighting the importance of authentication controls and monitoring.
    
3. **Endpoint visibility:** Windows Error Reporting provided events associated with the tested executable, but additional telemetry is required to establish the execution outcome.
    
4. **DNS visibility:** Windows DNS Client logs provided evidence of a resolution timeout, without establishing malicious DNS activity.
    
5. **File integrity detection:** Wazuh successfully detected the controlled modification of a monitored file and generated a level 7 alert.
    
6. **Evidence correlation:** Correlating process, authentication, network, DNS, and file integrity events would improve the ability to reconstruct activity and determine its impact.
    

## 7. Conclusion

This Purple Team assessment combined network reconnaissance, SSH authentication testing, Windows endpoint event investigation, DNS monitoring, and file integrity validation in a controlled lab environment.

Wazuh and Suricata provided complementary visibility into network and endpoint activity. The assessment confirmed that Wazuh detected the controlled file modification and retained before-and-after integrity metadata. Suricata also generated an alert associated with potential TCP SYN port-scanning activity.

Other findings, including the SSH credential result, Windows application errors, and DNS timeout, provide useful investigation leads but require additional evidence before stronger conclusions can be drawn.

The results provide a foundation for evaluating monitoring coverage, improving event correlation, and identifying opportunities to strengthen security controls.

## 8. MITRE ATT&CK References

- **T1046 — Network Service Discovery:** [https://attack.mitre.org/techniques/T1046/](https://attack.mitre.org/techniques/T1046/)
    
- **T1110.001 — Password Guessing:** [https://attack.mitre.org/techniques/T1110/001/](https://attack.mitre.org/techniques/T1110/001/)
    
- **T1204.002 — Malicious File:** [https://attack.mitre.org/techniques/T1204/002/](https://attack.mitre.org/techniques/T1204/002/)
    
- **T1565.001 — Stored Data Manipulation:** [https://attack.mitre.org/techniques/T1565/001/](https://attack.mitre.org/techniques/T1565/001/)