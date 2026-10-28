# Windows Event Logs for SOC

Today, I will compile information from multiple sources (Facebook groups, Microsoft, etc.) and list important and commonly used Event IDs for Blue Team operations. This article is for reference purposes only. If anything is incorrect or missing, please feel free to contribute additional information.

> Most of these Event IDs are from the **Security Log**, with some from the **System** and **Application** logs.

## 1. Logon / Logoff Activity

**Log Source:** Security

### Event ID(s)

- **4624**: Successful Logon
- **4625**: Failed Logon
- **4634 / 4647**: Logoff
- **4672**: Special Logon — an account with administrative or equivalent privileges logged on

### Description

Records logon and logoff sessions on the system. Event 4624 contains detailed information such as logon type (Interactive, RemoteInteractive, Network, Batch, Service), the account used, the source IP address, and more.

### SOC Relevance

- **Detect unauthorized logons:** Event 4625, especially when occurring in large volumes from a single source IP or across multiple accounts, can indicate brute-force or password-guessing attacks.
- **Track user activity:** Event 4624 helps identify who logged on to which system, when, and how. Event 4672 highlights privileged logon sessions.
- **Identify anomalous logons:** Watch for logons at unusual times, from unusual locations, or with abnormal logon types, such as a standard user account using a `Service` or `Batch` logon type.

### Potential Impact

- Unauthorized access to systems or data.
- Establishment of initial access.
- Privilege escalation when suspicious privileged logons are detected.
- Detection of attack attempts.

## 2. User and Group Account Management

**Log Source:** Security

### Event ID(s)

- **4720**: A user account was created.
- **4726**: A user account was deleted.
- **4722**: A user account was enabled.
- **4723**: An attempt was made to change an account's password.
- **4724**: An attempt was made to reset an account's password.
- **4738**: A user account was changed. Check changes to name, status, description, and other attributes.
- **4756**: A member was added to a security-enabled global group. Example: adding a user to an Administrators group.
- **4757**: A member was removed from a security-enabled global group.
- **4767**: A user account was unlocked.

### Description

Records changes to user accounts and security groups in an Active Directory environment or on a local machine.

### SOC Relevance

- **Detect rogue or backdoor accounts:** Creating or enabling a new account, especially one with a suspicious name or one added to a privileged group, may indicate persistence or unauthorized access.
- **Track privilege changes:** Adding users to administrative groups such as `Administrators` or `Domain Admins` is a highly important signal of possible privilege escalation.
- **Monitor administrative activity:** Verify that account changes are performed by authorized users and follow approved procedures.

### Potential Impact

- Creation of persistent accounts.
- Unauthorized privilege escalation.
- Concealment of malicious activity through newly created accounts.

## 3. Process Execution

**Log Source:** Security

### Event ID(s)

- **4688**: A new process has been created. **VERY IMPORTANT** — requires Audit Process Creation to be enabled.
- **4689**: A process has exited.

### Description

Event 4688 records information about newly created processes, including parent and child process names, the full executable path, command-line arguments (**extremely important**), and process IDs.

### SOC Relevance

- **Detect malware or exploit activity:** Malware often creates suspicious processes, executes from unusual locations such as temporary directories, or uses abnormal command lines.
- **Detect attack techniques:** Techniques such as Living Off the Land Binaries and Scripts (LOLBAS), including abuse of `powershell.exe`, `cmd.exe`, or `certutil.exe`, in-memory execution, and malicious scripts can often be identified through command-line logging.
- **Reconstruct activity chains:** Helps build an event sequence leading to an incident. For example, a browser downloads a malicious file, then spawns `cmd.exe` or `powershell.exe`, which then launches malware.

### Potential Impact

- Malicious code execution.
- Discovery activity.
- Actions on objectives.
- Persistence establishment.

## 4. Audit Policy / Security Policy Changes

**Log Source:** Security

### Event ID(s)

- **4719**: System audit policy was changed.
- **4816**: The Security object logon cache was cleared. This may be associated with Kerberos ticket-related activity or attacks.

### Description

Records changes to the system's Audit Policy configuration.

### SOC Relevance

- **Detect attempts to hide activity:** Attackers may try to disable or modify Audit Policy settings to prevent their actions from being logged.

### Potential Impact

- Allows malicious activity to occur without being fully logged.
- Conceals evidence of an attack.

## 5. Security Log Clearing

**Log Source:** Security

### Event ID

- **1102**: The audit log was cleared.

### Description

Records when the Security Log is cleared.

### SOC Relevance

- **Strong indicator of attempted evidence removal:** Clearing logs is a common technique used by attackers or malicious insiders to erase traces of their activity.

### Potential Impact

- Loss of critical incident evidence.
- Difficulty or inability to perform digital forensics.

## 6. Important System Configuration Changes

**Log Source:** System, Security

### Event ID(s)

#### System Log

- **7036**: The service entered the running state / stopped state. Track important services or newly created services.
- **7045**: A service was installed in the system. Potential persistence indicator.
- **1074**: The process ... has initiated the shutdown/restart of the computer.

#### Security Log

- **4697**: A service was installed in the system. Security Log equivalent of 7045; requires the relevant Audit Policy.
- **4698**: A scheduled task was created. Potential persistence indicator; requires the relevant Audit Policy.
- **4702**: A scheduled task was updated.
- **4896**: A scheduled task was deleted.
- **4946**: A rule has been added to the Windows Firewall exception list, or a rule has been changed.
- **4947**: A rule has been deleted from the Windows Firewall exception list.

### Description

Records events related to service state changes, installation of new services, creation or modification of scheduled tasks, and Windows Firewall configuration changes.

### SOC Relevance

- **Detect persistence techniques:** Attackers often install services or create scheduled tasks to make malicious code execute again after a system restart.
- **Detect changes to security controls:** Firewall changes may allow attackers to communicate externally or move laterally within the network.
- **Monitor abnormal system activity:** Unexpected stopping or restarting of important services, or unplanned shutdown/restart activity, can be suspicious.

### Potential Impact

- Persistence.
- Weakening of defensive controls such as the firewall.
- Service disruption or denial of service by stopping critical services.

## 7. Object Access — Files, Registry, etc.

**Log Source:** Security

### Event ID(s)

- **4656**: A handle to an object was requested.
- **4663**: An attempt was made to access an object. **VERY IMPORTANT** — requires appropriate SACLs on specific files or registry keys and the relevant Audit Policy.

### Description

Records attempts to read, write, delete, or otherwise access system objects such as files, folders, registry keys, printers, and more.

### SOC Relevance

- **Detect unauthorized access to sensitive data:** Track who attempts to read, write, or delete important files and directories.
- **Detect malicious Registry changes:** Monitor Registry keys associated with persistence, such as Run keys, as well as changes to security settings or other critical system configuration.
- **Trace post-compromise activity:** Identify which files or Registry keys were accessed or modified by an attacker.

### Potential Impact

- Data exfiltration.
- Data tampering.
- Persistence or malicious configuration changes through the Registry.

## 8. External Device Usage

**Log Source:** Security

### Event ID(s)

- **4660**: An object was deleted. This may sometimes relate to device removal after data has been copied and requires the appropriate Audit Policy.
- Other Event IDs may be relevant depending on the specific configuration used to audit USB device insertion and removal.

### Description

Records events related to connecting and disconnecting removable storage devices such as USB drives.

### SOC Relevance

- **Detect data exfiltration:** Connecting USB drives to sensitive servers or workstations may indicate unauthorized data copying.
- **Detect malware infection through USB devices:** Monitor the use of unapproved removable devices.

### Potential Impact

- Data theft.
- Malware infection within the internal network.

## 9. Critical System Errors and Warnings

**Log Source:** System

### Event ID(s)

Event IDs vary significantly depending on the root cause.

- **Bugcheck events:** Event IDs associated with Blue Screen of Death (BSOD) conditions.
- **Disk / hardware errors:** Warnings or errors related to disks, RAM, power supply, and other hardware components.
- **Service Control Manager errors:** Errors that occur when starting or running important services.

### Description

Records critical errors, warnings, and hardware or software failures that affect system availability or stability.

### SOC Relevance

- **Detect system stability issues:** Repeated hardware or software errors may indicate a problem that requires remediation or may be the result of an attack, such as denial of service.
- **Identify clues of malicious activity:** Some malware or attack techniques can cause system errors or crash important services.

### Potential Impact

- Denial of service.
- Data loss, particularly in the case of disk failures.
- System unavailability.

## 10. Application Errors

**Log Source:** Application

### Event ID(s)

Event IDs vary depending on the application.

- **1000**: Application Error — a general application error, commonly associated with application crashes.
- Other application-specific Event IDs, such as errors from databases, IIS, antivirus software, and other applications.

### Description

Records errors that occur while applications are running.

### SOC Relevance

- **Detect exploitation attempts:** Some attacks, such as buffer overflows, may cause an application to crash.
- **Monitor security software activity:** Antivirus or EDR logs are often stored in Application-related logs and may provide important alerts about malware detection or prevention.
- **Identify configuration or compatibility problems:** Helps diagnose issues that may disrupt business operations.

### Potential Impact

- Application service disruption.

## 11. Sysmon — Process / Network / File

**Log Source:** `Microsoft-Windows-Sysmon/Operational`

### Event ID(s)

- **1**: Process Create.
- **2**: File creation time changed.
- **3**: Network connection.
- **5**: Process terminated.
- **6**: Driver loaded.
- **7**: Image loaded.
- **8**: CreateRemoteThread.
- **10**: Process accessed.
- **11**: File created.
- **22**: DNS query.
- **23**: File deleted (archived).
- **25**: Process tampering.

### Description

Sysmon provides more detailed telemetry than the standard Security Log for processes, network activity, drivers and modules, files, DNS, and process tampering.

### SOC Relevance

- Event **1** provides a strong foundation for process-tree analysis.
- Events **3** and **22** provide useful network and DNS telemetry.
- Events **8**, **10**, and **25** support hunting for process injection, credential theft, and tampering.
- Events **11** and **23** support analysis of artifact creation and deletion.
- Some Sysmon events can be very noisy and require filtering.

### Potential Impact

- Execution.
- Command and Control (C2).
- Credential Access.
- Defense Evasion.
- Persistence.

## 12. Sysmon — Registry / WMI / Named Pipe

**Log Source:** `Microsoft-Windows-Sysmon/Operational`

### Event ID(s)

- **12**: Registry object create/delete.
- **13**: Registry value set.
- **14**: Registry object rename.
- **15**: FileCreateStreamHash (ADS).
- **17**: Named pipe created.
- **18**: Named pipe connected.
- **19**: WMI event filter.
- **20**: WMI event consumer.
- **21**: WMI event consumer binding.
- **24**: Clipboard change.

### Description

Records telemetry related to Registry activity, WMI, named pipes, Alternate Data Streams (ADS), clipboard activity, and several persistence-related behaviors.

### SOC Relevance

- Events **12–14** are particularly useful for hunting Registry-based persistence.
- Events **19–21** support detection of WMI-based persistence.
- Events **17–18** help analyze inter-process communication (IPC), which is commonly used by malware.
- Filtering should be tuned against the environment baseline.

### Potential Impact

- Persistence.
- Execution.
- Defense Evasion.
- Collection.

## 13. Windows Firewall / Windows Filtering Platform (WFP)

**Log Source:** Security

### Event ID(s)

- **5031**: The Windows Firewall Service blocked an application from accepting incoming connections on the network.
- **5152**: The Windows Filtering Platform blocked a packet.
- **5153**: A more restrictive Windows Filtering Platform filter has blocked a packet.
- **5154**: The Windows Filtering Platform has permitted an application or service to listen on a port for incoming connections.
- **5155**: The Windows Filtering Platform has blocked an application or service from listening on a port for incoming connections.
- **5156**: The Windows Filtering Platform has allowed a connection.
- **5157**: The Windows Filtering Platform has blocked a connection.
- **5158**: The Windows Filtering Platform has permitted a bind to a local port.
- **5159**: The Windows Filtering Platform has blocked a bind to a local port.
- **5440**: Persistent callout added.
- **5441**: Boot-time or persistent filter added.
- **5442**: Persistent provider added.
- **5443**: Persistent provider context added.
- **5444**: Persistent sub-layer added.
- **5446**: Run-time callout added or removed.
- **5447**: Run-time filter added or removed.
- **5448**: Run-time provider added or removed.
- **5449**: Run-time provider context added or removed.
- **5450**: Run-time sub-layer added or removed.

### Description

Windows Filtering Platform (WFP) provides audit telemetry for Windows Firewall activity, packet drops, network connections, listen/bind operations, and changes to WFP filters and providers. These events are recorded in the Security Log.

### SOC Relevance

- Events **5152** and **5157** help identify blocked packets or connections.
- Events **5154** and **5155** help track applications or services attempting to listen on network ports.
- Event **5156** helps track allowed network connections.
- Events **5158** and **5159** help track local port binding activity.
- Events **5440–5450** are useful when hunting for suspicious additions or modifications to WFP objects.
- Connection auditing can generate a very high volume of events, so filtering and baselining are important.

### Potential Impact

- Network Intrusion.
- Command and Control (C2).
- Lateral Movement.
- Defense Evasion.
- Persistence.

## 14. Malware Detection / Microsoft Defender Antivirus

**Log Source:** `Microsoft-Windows-Windows Defender/Operational`

### Event ID(s)

- **1000**: An antimalware scan started.
- **1001**: An antimalware scan finished.
- **1002**: An antimalware scan was stopped before it finished.
- **1006**: The antimalware engine found malware or other potentially unwanted software.
- **1007**: The antimalware platform performed an action to protect the system from malware or other potentially unwanted software.
- **1008**: The antimalware platform attempted to perform an action, but the action failed.
- **1009**: The antimalware platform restored an item from quarantine.
- **1010**: The antimalware platform could not restore an item from quarantine.
- **1011**: The antimalware platform deleted an item from quarantine.
- **1012**: The antimalware platform could not delete an item from quarantine.
- **1013**: The antimalware platform deleted malware history.
- **1014**: The antimalware platform could not delete malware history.
- **1015**: The antimalware platform detected suspicious behavior.
- **1116**: The antimalware platform detected malware or other potentially unwanted software.
- **1117**: The antimalware platform performed an action to protect the system from malware or other potentially unwanted software.
- **5001**: Microsoft Defender Antivirus real-time protection is disabled.
- **5007**: Microsoft Defender Antivirus configuration has changed.
- **5010**: Microsoft Defender Antivirus scanning for spyware and other potentially unwanted software is disabled.
- **1125**: Network Protection detected a network connection in audit mode.
- **1126**: Network Protection blocked a network connection.

### Description

Microsoft Defender Antivirus records scan activity, malware and potentially unwanted application (PUA) detections, remediation actions, remediation failures, quarantine operations, suspicious behavior, and changes to protection settings. Network Protection adds telemetry when network connections are audited or blocked.

### SOC Relevance

- **Direct malware detection:** Events **1006** and **1116** indicate that Defender detected malware or potentially unwanted software. Investigators should collect fields such as Threat Name, Severity, Path, Process Name, Detection Source, and User.
- **Track remediation actions:** Events **1007** and **1117** indicate that Defender performed an action such as Clean, Quarantine, Remove, Block, or Allow. Event **1008** indicates that remediation failed and requires further investigation.
- **Monitor suspicious behavior:** Event **1015** may provide indicators of suspicious activity even when there is no explicit malware detection.
- **Detect attempts to disable protection:** Event **5001** indicates that real-time protection was disabled. Event **5007** records Defender configuration changes. Event **5010** indicates that scanning for spyware and potentially unwanted software was disabled. Unplanned changes should be investigated because they may indicate malware activity or Defense Evasion.
- **Monitor Network Protection:** Events **1125** and **1126** help detect connections to sites or resources considered suspicious or malicious, depending on whether Network Protection is configured in audit or block mode.
- **Correlate with Sysmon:** Events **1116**, **1117**, and **1015** can be correlated with Sysmon Event **1** (Process Create), Event **3** (Network Connection), Event **11** (File Create), Event **22** (DNS Query), and Events **12–14** (Registry) to build a timeline from execution to network activity, payload drop, and persistence.

### Potential Impact

- Malware Execution.
- Initial Access.
- Command and Control (C2).
- Persistence.
- Credential Access.
- Defense Evasion.
- Impact when remediation fails or endpoint protection is disabled.
