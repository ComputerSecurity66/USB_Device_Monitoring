# Enhanced USB Threat Detection & Behavior Analysis Tool

A Windows-based C# cybersecurity assessment utility designed to monitor USB devices, analyze files for suspicious indicators, correlate USB-launched processes with network connections, inspect Windows security events, assess potential BadUSB/HID anomalies, control USB devices, and perform additional malware assessment using **ML.NET** and **YARA**.

> **Security assessment tool:** This project provides indicators and observations for security analysis. It is not intended to replace Microsoft Defender, enterprise EDR/XDR platforms, endpoint security products, or professional malware-analysis environments.

---

## 🛡️ Overview

USB devices are commonly used for data transfer, software installation, system maintenance, and offline file exchange. They can also introduce potentially unwanted or malicious files into a Windows environment.

**Enhanced USB Threat Detection & Behavior Analysis Tool** provides a collection of Windows security assessment functions from a single administrative console.

The tool combines:

* USB device enumeration
* Continuous USB monitoring
* Real-time USB connection monitoring
* USB file analysis
* File extension and filename heuristics
* File entropy analysis
* Suspicious file identification
* Optional file quarantine
* USB process monitoring
* Process/network correlation
* Windows Event Log analysis
* Security event monitoring
* BadUSB/HID assessment
* USB storage activity monitoring
* USB device disable/enable operations
* ML.NET-based file assessment
* YARA rule scanning
* Security findings collection
* Threat assessment reporting
* Security and diagnostic logging

---

# 🚀 Features

## 1. USB Device Continuous Monitoring

Continuously queries Windows USB device information and tracks devices over time.

The monitoring system can display:

* Device name
* Manufacturer
* USB Vendor ID (VID)
* USB Product ID (PID)
* Device status
* First observed connection time
* Last observed time
* Internal risk/review level

The device scanner periodically refreshes the USB device inventory and maintains tracked device profiles.

### Example

```text
USB Device Continuous Monitoring

[Device 1]
  VID:PID: 0781:5581
  Name: USB Mass Storage Device
  Manufacturer: SanDisk
  Status: OK
```

---

## 2. Real-Time USB Monitoring

Monitors the current USB device inventory and reports changes.

The tool can identify when a USB device:

* Appears
* Is removed
* Remains connected

Example:

```text
[21:30:15] USB device detected: USBSTOR\...
[21:30:18] USB devices present: 2
[21:31:02] USB device no longer detected: USB\...
```

Press:

```text
S
```

to stop monitoring.

---

# 3. USB File Threat Analysis

The file-analysis module recursively scans a selected USB storage drive.

The tool examines file characteristics including:

* Executable files
* Dynamic-link libraries
* Drivers
* Screensavers
* Control Panel extensions
* Scripts
* Installers
* Archives
* Suspicious filenames
* File entropy
* Large executable files

### Executable extensions

The current implementation reviews extensions such as:

```text
.exe
.dll
.scr
.com
.cpl
.ocx
.sys
```

### Script extensions

```text
.vbs
.vbe
.bat
.cmd
.ps1
.psm1
.js
.jse
.wsf
.wsh
```

### Installer extensions

```text
.msi
.msp
```

### Archive extensions

```text
.zip
.rar
.7z
```

---

# 4. Suspicious Filename Detection

The tool checks filenames against a collection of suspicious keyword patterns.

Current examples include:

```text
malware
trojan
virus
ransom
worm
backdoor
exploit
payload
keylog
stealer
```

A filename match is treated as a **review indicator**, not automatic proof that the file is malicious.

For example:

```text
[REVIEW] Suspicious filename:
E:\Downloads\possible_payload.exe
```

---

# 5. File Entropy Analysis

The tool can calculate Shannon entropy for supported files.

Entropy can help identify files that contain highly compressed, encrypted, or packed data.

Example:

```text
Entropy: 7.82
```

High entropy may be useful during malware triage, particularly for packed or encrypted executables.

However:

> **High entropy does not prove that a file is malware.**

Legitimate software, installers, compressed files, and encrypted data can also have high entropy.

The implementation currently limits entropy processing to files up to:

```text
100 MB
```

and applies a timeout to prevent excessively long analysis.

---

# 6. File Quarantine

Files identified by the current heuristic review process can optionally be moved into an application quarantine directory.

The quarantine location is:

```text
%LOCALAPPDATA%\EnhancedUSBThreatDetection\Quarantine
```

Files are renamed using a timestamp and GUID to reduce filename collisions.

Example:

```text
20260924_213045123_8f1..._suspicious.exe
```

The tool asks for confirmation before moving files:

```text
Quarantine these files? (Y/N):
```

> **Important:** The quarantine feature is an application-level file move. It is not equivalent to Microsoft Defender quarantine or an enterprise endpoint security quarantine mechanism.

---

# 7. USB Process & Network Activity Monitoring

The tool identifies currently running processes whose executable paths originate from removable USB storage.

It uses Windows process information to examine:

```text
Process Name
Process ID
Executable Path
```

It can then correlate those process IDs with established TCP connections.

Example output:

```text
--- PROCESSES RUNNING FROM USB STORAGE ---

Name       ProcessId    ExecutablePath
----       ---------    --------------
tool.exe   4216         E:\Tools\tool.exe
```

Network correlation can include:

```text
Local Address
Local Port
Remote Address
Remote Port
Owning Process
```

This can help with basic incident-response investigation when an executable is launched directly from removable storage.

---

# 8. USB Behavior & Windows Event Activity

The tool queries available Windows event logs for USB-related activity.

The current implementation can inspect events including:

```text
2003
2004
2100
2102
20001
20003
```

Depending on Windows configuration, these events may provide information related to:

* USB driver activity
* Device activity
* Plug and Play activity
* Driver installation
* Device installation

Event availability varies by Windows version and configuration.

---

# 9. Windows Security Event Monitoring

The security-event module queries existing Windows Security logs.

The current implementation examines:

### Event ID 4688

Process creation.

### Event ID 5156

Windows Filtering Platform allowed network connection.

### Event ID 4663

Object access activity.

### Event ID 4657

Registry value modification.

It also queries USB-related Driver Framework events.

The tool does **not** silently modify Windows audit policy.

If the required auditing is not enabled, the corresponding events may not exist.

---

# 10. BadUSB / HID Anomaly Assessment

The tool provides an observational assessment of USB and HID devices.

It examines information such as:

* USB device status
* Device class
* Friendly name
* Manufacturer
* Instance ID
* Vendor IDs
* HID-class devices

It can identify review indicators such as:

```text
Duplicate Vendor ID groups
Unknown manufacturer information
USB/HID device observations
```

Example:

```text
[REVIEW] 1 Vendor ID group(s) contain multiple devices.
```

The application intentionally does not treat these indicators as proof of malicious hardware.

> A shared Vendor ID or missing manufacturer information alone is not sufficient to establish that a device is a BadUSB device.

---

# 11. USB Storage Activity Monitoring

The storage activity monitor measures changes in used storage space on a selected removable drive.

It records:

```text
Initial used storage
Final used storage
Net storage change
Net size-change rate
```

Example:

```text
Initial used storage: 1024.50 MB
Final used storage:   1100.20 MB

Net size change:      79,364,096 bytes
Net size-change rate: 2.52 MB/s
```

### Important limitation

This feature measures **net storage-size change**.

It does **not** claim to measure:

* USB bus throughput
* Actual read bandwidth
* Actual write bandwidth
* USB controller throughput
* Network transfer rate

---

# 12. USB Device Blocking

Administrators can disable supported USB devices using Windows Plug and Play management.

The tool uses:

```text
pnputil.exe
```

The workflow requires confirmation before disabling the selected device.

Example:

```text
Enter Device Instance ID to BLOCK:
```

The tool then attempts to verify that Windows reports the device as:

```text
Disabled
```

This feature requires Administrator privileges.

---

# 13. USB Device Enabling

Previously disabled USB devices can also be enabled through the tool.

The tool searches for USB devices reported as:

```text
Error
Unknown
Disabled
```

After confirmation, it uses Windows Plug and Play management to attempt to enable the selected device.

The tool verifies whether the resulting status is reported as:

```text
OK
```

---

# 14. ML.NET Malware Assessment

The project includes an optional **ML.NET** assessment component.

The current model uses file-oriented features such as:

```text
File Size
Entropy
Executable Count
```

The application creates or loads:

```text
malware_model_v2.zip
```

The model uses binary classification through ML.NET.

The current training dataset is intentionally small and embedded in the application for demonstration and development purposes.

### Example output

```text
[ML.NET] Assessment Results

File: example.exe
Size: 245,760 bytes
Entropy: 7.31
Executable count: 1
Probability: 82.15%
Score: 1.5264
```

### Important ML limitation

The included model should **not** be considered a production-grade malware detection model.

A real malware classification system requires a substantially larger and carefully validated dataset containing representative benign and malicious samples, appropriate feature engineering, evaluation, validation, and ongoing model maintenance.

The ML.NET component is intended primarily for:

* Research
* Experimentation
* Security assessment
* Demonstration
* Local malware-analysis workflows

---

# 15. YARA Malware Assessment

The project supports external **YARA** rule scanning.

The application searches trusted locations for:

```text
yara.exe
```

Supported locations include:

```text
ApplicationDirectory\Tools\yara.exe
ApplicationDirectory\yara.exe
C:\YARA\yara.exe
C:\Program Files\YARA\yara.exe
C:\Program Files (x86)\YARA\yara.exe
```

YARA rules should be stored under:

```text
YaraRules\
```

Supported rule extensions:

```text
.yar
.yara
```

The tool can scan:

* Individual files
* Directories
* Recursive directory contents

### YARA exit codes

The application interprets YARA results using the standard convention:

```text
0 = Match
1 = No match
Other = Error
```

Example:

```text
[YARA] ⚠ Match: malware_rules.yar
```

---

# 16. Security Findings

The application maintains a collection of security findings generated during analysis.

Findings can originate from:

* USB file scanning
* Quarantine operations
* BadUSB assessment
* USB storage monitoring
* USB device control
* Windows event monitoring
* ML.NET assessment
* YARA assessment

Each finding contains:

```text
Timestamp
Category
Threat level
Details
```

The application maintains up to:

```text
500
```

recent findings in memory.

---

# 17. Threat Levels

The application defines the following assessment levels:

```text
Safe
Low
Medium
High
Critical
```

These levels represent the application's assessment logic and should not be interpreted as definitive malware classifications.

For example, the USB file scoring system considers:

* Number of executables
* Number of scripts/installers
* Suspicious filename indicators

The resulting score is converted into a review level.

---

# 18. Threat Reporting

The tool can generate a text-based security report.

Reports are saved under:

```text
%USERPROFILE%\Documents\EnhancedUSBThreatDetection\Reports
```

Example:

```text
USBThreatReport_20260924_213045.txt
```

Reports can contain:

* Computer information
* Windows version
* USB devices
* Vendor/Product IDs
* Manufacturer information
* Connection timestamps
* USB drive scan results
* Executable counts
* Review levels
* Recent security findings
* Assessment notes

---

# 19. Security Logging

The application maintains application logs under:

```text
%LOCALAPPDATA%\EnhancedUSBThreatDetection\Logs
```

### Security log

```text
USBSecurityLog.txt
```

Used for security-related administrative actions such as device enable/disable operations.

### Debug log

```text
USBDebug.log
```

Used for application diagnostics and operational errors.

---

# 🖥️ Requirements

## Operating System

The application currently requires:

```text
Windows 10 / Windows 11
```

Administrator privileges are required for functionality involving device management and certain system-level queries.

---

## .NET

The project is designed for a modern .NET runtime supporting the APIs used by the application.

Recommended:

```text
.NET 8 or later
```

depending on the project's `.csproj` configuration.

---

## NuGet Packages

The project requires the appropriate ML.NET packages used by the source code.

For example:

```text
Microsoft.ML
```

The project also uses Windows Management Instrumentation APIs through:

```text
System.Management
```

Make sure the corresponding package/reference is included in the project configuration when required by the target framework.

---

# 🧰 Optional YARA Installation

YARA is optional.

If YARA is installed, place the executable in one of the supported locations.

Recommended application structure:

```text
EnhancedUSBThreatDetection/
│
├── EnhancedUSBThreatDetection.exe
├── malware_model_v2.zip
│
├── Tools/
│   └── yara.exe
│
└── YaraRules/
    ├── rules.yar
    ├── malware.yar
    └── custom_rules.yara
```

Only use YARA rules that you trust and understand.

---

# 📁 Suggested Project Structure

```text
EnhancedUSBThreatDetection/
│
├── Program.cs
├── EnhancedUSBThreatDetection.csproj
├── malware_model_v2.zip
│
├── Tools/
│   └── yara.exe
│
├── YaraRules/
│   ├── example.yar
│   └── custom.yara
│
└── README.md
```

Runtime-created data:

```text
%LOCALAPPDATA%\EnhancedUSBThreatDetection\
│
├── Logs/
│   ├── USBSecurityLog.txt
│   └── USBDebug.log
│
└── Quarantine/
```

Reports:

```text
%USERPROFILE%\Documents\EnhancedUSBThreatDetection\
└── Reports/
```

---

# ▶️ Usage

Run the application as Administrator.

The main menu provides:

```text
================================================================
 ENHANCED USB THREAT DETECTION & BEHAVIOR ANALYSIS TOOL
================================================================

1. List USB Devices with Continuous Monitoring
2. Start Real-time USB Monitoring
3. Analyze USB Files for Threat Indicators
4. Monitor USB Processes & Correlate Network Activity
5. Check USB Behavior & Windows Event Activity
6. Generate Threat Report
7. BadUSB / HID Anomaly Assessment
8. USB Storage Activity Monitoring
9. Block USB Device
10. Enable USB Device
11. Windows Security Event Monitoring
12. ML.NET & YARA File Analysis
0. Exit
```

---

# 🔍 Example Workflow

A basic USB security assessment can be performed in the following order:

### Step 1 — Inventory

Run:

```text
1. List USB Devices with Continuous Monitoring
```

Record:

* Device name
* Manufacturer
* VID
* PID
* Device status

### Step 2 — Inspect Files

Run:

```text
3. Analyze USB Files for Threat Indicators
```

Review:

* Executables
* Scripts
* Installers
* Suspicious filenames
* Entropy indicators

### Step 3 — Check Running Processes

Run:

```text
4. Monitor USB Processes & Correlate Network Activity
```

Look for processes executing directly from removable storage.

### Step 4 — Inspect Events

Run:

```text
5. Check USB Behavior & Windows Event Activity
```

and:

```text
11. Windows Security Event Monitoring
```

### Step 5 — Advanced File Assessment

Run:

```text
12. ML.NET & YARA File Analysis
```

Use both ML.NET and YARA results as additional assessment indicators.

### Step 6 — Generate Report

Run:

```text
6. Generate Threat Report
```

to preserve the application's collected findings.

---

# ⚠️ Security & Safety Notes

This project is designed for defensive security assessment.

The tool should be used only on:

* Computers you own
* Systems you are authorized to administer
* USB devices you are authorized to inspect

Do not use the device-control functionality on systems where you do not have administrative authorization.

Before quarantining or disabling a device, verify that it is not required for:

* System operation
* Authentication
* Backup
* Data recovery
* Business-critical hardware
* Accessibility equipment
* Security controls

---

# ⚠️ Detection Limitations

This project should not be described as a replacement for a commercial antivirus or EDR solution.

Several assessment methods are heuristic or observational.

For example:

### File extension

A `.exe` file is not automatically malicious.

### Filename

A filename containing `trojan` or `payload` does not prove malicious behavior.

### Entropy

High entropy can occur in legitimate compressed or encrypted files.

### Duplicate VID

Multiple devices can legitimately share a Vendor ID.

### Unknown manufacturer

Missing manufacturer information does not establish that hardware is malicious.

### ML.NET

The included demonstration model is trained on a very small dataset and should not be used as a production malware classifier.

### YARA

YARA results depend on the quality and coverage of the installed rules.

### Windows Event Logs

Event availability depends on Windows configuration, audit policies, logging state, and operating-system version.

---

# 🔐 Recommended Defensive Architecture

For a production security solution, this project should be used alongside established security controls such as:

* Microsoft Defender Antivirus
* Microsoft Defender for Endpoint
* Windows Firewall
* Attack Surface Reduction rules
* Device Control policies
* Application Control
* WDAC
* AppLocker
* Enterprise EDR/XDR
* Centralized Windows Event Collection
* SIEM monitoring
* Network security monitoring

The application can serve as an additional local assessment and investigation utility.

---

# 🧪 Development Roadmap

Potential future improvements include:

* [ ] Improved USB device persistence
* [ ] USB device fingerprinting
* [ ] Digital signature verification
* [ ] Authenticode certificate inspection
* [ ] PE header analysis
* [ ] Import/export analysis
* [ ] Hash calculation
* [ ] SHA-256 reputation workflow
* [ ] Improved ML.NET training dataset
* [ ] Model evaluation metrics
* [ ] YARA rule management
* [ ] Automatic rule categorization
* [ ] Windows Defender integration
* [ ] Windows Defender scan integration
* [ ] Process tree visualization
* [ ] Improved network process correlation
* [ ] USB event timeline
* [ ] JSON report output
* [ ] CSV report output
* [ ] HTML security reports
* [ ] Centralized logging
* [ ] Configurable detection thresholds
* [ ] Configurable quarantine policy
* [ ] USB allowlist/blocklist
* [ ] Device fingerprint database
* [ ] Improved BadUSB/HID behavioral analysis
* [ ] Digital forensic timeline generation

---

# 📊 Technology Stack

| Component           | Technology                         |
| ------------------- | ---------------------------------- |
| Language            | C#                                 |
| Runtime             | .NET                               |
| Machine Learning    | ML.NET                             |
| Malware Rules       | YARA                               |
| USB Discovery       | Windows Management Instrumentation |
| Device Management   | Windows Plug and Play / PnPUtil    |
| Process Analysis    | Windows Management / PowerShell    |
| Network Correlation | Windows TCP connection information |
| Event Analysis      | Windows Event Log                  |
| Reporting           | Text files                         |
| Logging             | Local application logs             |
| Interface           | Windows Console                    |

---

# 🛡️ Threat Assessment Philosophy

The application follows a layered assessment approach.

Instead of relying on one indicator, it can combine multiple observations:

```text
USB Device
    │
    ├── Device Information
    │
    ├── File Inventory
    │       ├── File Type
    │       ├── Filename
    │       ├── Entropy
    │       └── Size
    │
    ├── Process Activity
    │       └── Network Correlation
    │
    ├── Windows Events
    │
    ├── HID / BadUSB Assessment
    │
    └── ML.NET + YARA
             │
             ▼
       Security Assessment
             │
             ▼
       Findings / Report
```

The purpose is to provide **multiple sources of evidence** that can assist with security investigation.

---

# 📄 License

Add your preferred license before publishing the repository.

For example:

```text
MIT License
```

or another license appropriate for your project.

Do not claim a license that has not actually been added to the repository.

---

# 🤝 Contributing

Contributions are welcome.

Potential contribution areas include:

* Detection improvements
* Windows compatibility improvements
* USB device analysis
* Malware-analysis features
* YARA rule integration
* ML.NET model improvements
* Reporting
* Performance optimization
* Documentation
* Bug fixes

When contributing security-related detection logic, include information explaining:

1. What is being detected
2. Why the indicator is relevant
3. Known false positives
4. Windows versions tested
5. How the feature was validated

---

# ⚖️ Disclaimer

This software is provided for defensive cybersecurity research, system administration, security assessment, and educational purposes.

Detection results are indicators and observations generated by the application. A `Safe`, `Low`, `Medium`, `High`, or `Critical` result does not constitute a definitive determination that a system or file is clean or malicious.

The authors are not responsible for data loss, system interruption, hardware disruption, or other consequences resulting from the use of this software.

Always verify security findings using additional trusted security controls before taking destructive or disruptive action.

---

# 👨‍💻 Project

**Enhanced USB Threat Detection & Behavior Analysis Tool**

A defensive Windows security utility combining:

```text
USB Monitoring
+
File Threat Assessment
+
Process Analysis
+
Network Correlation
+
Windows Event Monitoring
+
BadUSB/HID Assessment
+
Device Control
+
ML.NET
+
YARA
+
Threat Reporting
```

Built with **C# and .NET for Windows security assessment and research.**
