# Development and Test Variants

This directory contains historical test variants created while investigating Exchange Server Auth Certificate renewal and time-zone behavior. They are based on Microsoft CSS-Exchange `MonitorExchangeAuthCertificate.ps1`, upstream version **26.03.06.1531**.

**These scripts are for development and isolated lab testing only. Neither is recommended for production use.** For the published version, see [MonitorExchangeAuthCertificate-TimeZoneAware.ps1](../MonitorExchangeAuthCertificate-TimeZoneAware.ps1) and the [main README](../README.md).

## Scripts and intended use

| Script | Why it exists | Key differences |
| --- | --- | --- |
| [MonitorExchangeAuthCertificate-UTC3.ps1](MonitorExchangeAuthCertificate-UTC3.ps1) | Reproduce and compare the Auth Certificate `NotBefore` issue on UTC+03 Exchange servers (A/B experiment). | Uses a **fixed three-hour backdate** (`UtcNow.AddHours(-3)`) when creating a custom-lifetime certificate; `NotAfter` uses that adjusted start time. Not suitable for other time zones. |
| [MonitorExchangeAuthCertificate-Lab.ps1](MonitorExchangeAuthCertificate-Lab.ps1) | Observe certificate creation, staging, rollover, expiry and recovery without waiting days or years. | Adds minute-scale certificate lifetime / activation controls and explicit past validity windows; requires `-LabMode`, disables Scheduled Task/AD automation setup and skips upstream version checking. |

### UTC+03 comparison

The fixed three-hour adjustment is **only used in the custom certificate creation path**. Supply `-CustomCertificateLifetimeInDays` greater than zero when testing that behavior. The standard `New-ExchangeCertificate` path does not apply the UTC3 correction.

This experiment helped isolate the issue. The published TimeZoneAware script supersedes it by calculating the server's positive UTC offset dynamically.

### Accelerated LAB tests

The LAB variant supports these additional options:

- `-CertificateLifetimeInMinutes`: generate a short-lived Auth Certificate.
- `-NextCertificateEffectiveInMinutes`: set the next certificate's effective time to now (0) or a future number of minutes; -1 retains the original day-based behavior.
- `-CertificateNotBeforeHoursAgo` and `-CertificateNotAfterHoursAgo`: set a historical validity window together, including one that has already expired.
- `-LabMode`: mandatory acknowledgement; **not** an isolation or safety mechanism.

The LAB version deliberately changes the normal certificate-creation and Internal Transport certificate handling path for test scenarios, while retaining the underlying renewal/recovery workflow. A newly staged certificate becoming eligible does not force the Exchange AuthAdmin servicelet to run immediately.

**Use a disposable Exchange lab organization only.** This script can change AuthConfig and related certificate state, trigger service restarts/app-pool recycles, and create already-expired certificates. It is not appropriate for production, even if the `-LabMode` switch is present.

## Example commands

Run from the Exchange Management Shell with the required Exchange permissions. The first examples are validation-only; the creation examples intentionally change certificates and must only be used in an isolated lab.

**UTC3 — validation only:**

```powershell
.\MonitorExchangeAuthCertificate-UTC3.ps1
```

**UTC3 — force a custom-lifetime test certificate (UTC+03 test only):**

```powershell
.\MonitorExchangeAuthCertificate-UTC3.ps1 -EnforceNewAuthCertificateCreation -CustomCertificateLifetimeInDays 365 -Confirm:$false
```

**Lab — validation only:**

```powershell
.\MonitorExchangeAuthCertificate-Lab.ps1 -LabMode
```

**Lab — stage a certificate with a 10-minute lifetime and immediate effective time:**

```powershell
.\MonitorExchangeAuthCertificate-Lab.ps1 -LabMode -EnforceNewAuthCertificateCreation -CertificateLifetimeInMinutes 10 -NextCertificateEffectiveInMinutes 0 -Confirm:$false
```

**Lab — create a test certificate with an already-expired validity window:**

```powershell
.\MonitorExchangeAuthCertificate-Lab.ps1 -LabMode -ValidateAndRenewAuthCertificate $true -CertificateNotBeforeHoursAgo 4 -CertificateNotAfterHoursAgo 3 -Confirm:$false
```

The expired-certificate example invokes the script's renewal decision logic, so it only creates a replacement if renewal is actually required. These examples are **not** automated test assertions or proof of successful execution.

## Authenticode signatures and version checks

The original Microsoft `# SIG # Begin signature block` through `# SIG # End signature block` sections have been **removed** from both modified files. Leaving the upstream signature attached to changed code resulted in `HashMismatch`, rather than a valid signature.

Both files are now **unsigned (`NotSigned`)**; neither has been signed or endorsed by Microsoft. They will still be blocked by an `AllSigned` execution policy unless separately signed with a trusted code-signing certificate.

With the obsolete signature removed, the UTC3 variant's inherited version-check logic can recognize the file as unsigned and normally skip automatic update checks. The Lab variant disables version checking internally.

No functional code changes were made as part of this archive cleanup; only the obsolete signature blocks were removed.

## Microsoft upstream

A duplicate copy of Microsoft's source is intentionally **not** stored in `Dev/`.

- Source: [Microsoft CSS-Exchange](https://github.com/microsoft/CSS-Exchange)
- Upstream script: `MonitorExchangeAuthCertificate.ps1`
- Based on version: **26.03.06.1531**
- Referenced upstream commit: [2d8444fb37b77f1b83da2ce352ed683887699daa](https://github.com/microsoft/CSS-Exchange/commit/2d8444fb37b77f1b83da2ce352ed683887699daa)
- [Official documentation](https://github.com/microsoft/CSS-Exchange/blob/main/docs/Admin/MonitorExchangeAuthCertificate.md)

The original Microsoft MIT copyright notices are retained in the test files. See the repository's [LICENSE](../LICENSE).
