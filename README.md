# WN10-CC-000035
# Windows 10 Vulnerability Management Lab: WN10-CC-000035

## Overview

This lab demonstrates the full vulnerability management lifecycle for **WN10-CC-000035 – The system must be configured to ignore NetBIOS name release requests except from WINS servers**. It covers Azure VM provisioning, vulnerability simulation, Tenable authenticated scanning, PowerShell remediation, and verification through post-remediation scans.

---

## Table of Contents

1. [Lab Setup](#lab-setup)
2. [Vulnerability Simulation](#vulnerability-simulation)
3. [Tenable Scan Configuration](#tenable-scan-configuration)
4. [Vulnerability Discovery](#vulnerability-discovery)
5. [Remediation](#remediation)
6. [Post-Remediation Verification](#post-remediation-verification)
7. [Security Rationale](#security-rationale)
8. [Screenshots](#screenshots)
9. [Cleanup Recommendations](#cleanup-recommendations)

---

## Lab Setup

### Azure VM Provisioning

- **OS**: Windows 10 (latest supported version)
- **VM Size**: Minimum `Standard B2s`
- **Resource Group**: All lab resources grouped for simple cleanup
- **Credentials**: Use complex and unique admin credentials (Do **not** use `labuser/Cyberlab123!`)
- **Network Security Group (NSG)**:
  - Allow RDP on TCP/3389
  - Allow access from Tenable scanner
  - Deny all unnecessary ports

### VM Configuration

#### Disable Windows Firewall (Lab Only)

```shell
wf.msc
```

#### Enable Remote Scanning Access

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
-Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

---

## Vulnerability Simulation

### Vulnerability ID: WN10-CC-000035

**Description**: NetBIOS name release requests should only be accepted from authorized WINS servers. Enabling this prevents malicious release spoofing.

#### Simulating the Vulnerable State

Set the registry key to a non-compliant value:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters" `
-Name "NoNameReleaseOnLogoff" -Value 0
```

📸 *Screenshot Placeholder: `1_simulate_nbt_vulnerability.png`*

---

## Tenable Scan Configuration

### Template: Advanced Network Scan

- **Basic Options**:
  - Enable Remote Registry
  - Enable Admin Shares
  - Enable Server Service

- **Discovery Settings**:
  - Ping remote host
  - Fast network discovery
  - TCP port scanning

- **Assessment Settings**:
  - Enable credentialed scans
  - Use thorough testing

- **Compliance Checks**:
  - Load **DISA STIG – Microsoft Windows 10** benchmark

---

## Vulnerability Discovery

After running the scan:

- Search for: `WN10-CC-000035`
- Expected finding: Tenable will flag the system if `NoNameReleaseOnLogoff` is set to `0` or missing.

📸 *Screenshot Placeholder: `2_discovery_tenable_nbt.png`*

---

## Remediation

### PowerShell Command to Remediate

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters" `
-Name "NoNameReleaseOnLogoff" -Value 1
```

📸 *Screenshot Placeholder: `3_registry_fix_nbt.png`*

---

## Post-Remediation Verification

### Rerun the Tenable Scan

Ensure Tenable no longer reports the vulnerability.

### Manual Registry Validation (Optional)

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters" `
-Name "NoNameReleaseOnLogoff"
```

Expected output:
```
NoNameReleaseOnLogoff : 1
```

📸 *Screenshot Placeholder: `4_verification_nbt_rescan.png`*

---

## Security Rationale

Accepting NetBIOS name release requests from unauthorized systems allows attackers to hijack names on the network, leading to:

- **Man-in-the-middle attacks**
- **Denial-of-service attacks**
- **Network spoofing**

This setting enforces stricter NetBIOS behavior, reducing risk in enterprise environments.

---

## Screenshots

| Step                         | Screenshot Placeholder               |
|------------------------------|---------------------------------------|
| Simulated Vulnerable State   | `1_simulate_nbt_vulnerability.png`    |
| Vulnerability Discovery      | `2_discovery_tenable_nbt.png`         |
| Registry Fix Applied         | `3_registry_fix_nbt.png`              |
| Verification After Remediation | `4_verification_nbt_rescan.png`     |

---

## Cleanup Recommendations

- **Delete Azure resource group** to remove all lab components
- **Clear NSG rules** for exposed RDP or Tenable access
- **Revoke and rotate any scan credentials**
- **Retain scan results if needed for compliance documentation**

---

## Notes

- **Vulnerability ID**: WN10-CC-000035
- **Category**: NetBIOS Security / STIG Compliance
- **Time to Remediate**: <1 minute
- **Restart Required**: No
- **Compliance Impact**: High (network spoofing exposure)

---
