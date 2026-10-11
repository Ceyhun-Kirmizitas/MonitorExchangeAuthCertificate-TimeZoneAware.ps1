# Development and Lab Variants

These files document the development and testing work behind the published [MonitorExchangeAuthCertificate-TimeZoneAware.ps1](../MonitorExchangeAuthCertificate-TimeZoneAware.ps1).

**The published TimeZoneAware script remains in the repository root.** Nothing under `Dev/` is a separate production release. These are historical source snapshots (September 2026).

## File guide

| Location | Purpose | Status |
| --- | --- | --- |
| `Upstream/MonitorExchangeAuthCertificate.ps1` | Baseline Microsoft CSS-Exchange script, version `26.03.06.1531`, before custom changes | Reference copy, not maintained here |
| `Experiments/MonitorExchangeAuthCertificate-UTC3.ps1` | Isolated UTC+03 experiment using `UtcNow.AddHours(-3)` for custom certificate `NotBefore`; `NotAfter` is based on the adjusted start time | A/B test only; not recommended for general use |
| `Lab/MonitorExchangeAuthCertificate-Lab.ps1` | Accelerated Auth Certificate lifecycle and recovery tests using minutes, immediate/future effective dates, and intentionally expired validity windows | **LAB ONLY – NOT FOR PRODUCTION** |

## Background

### UTC+03 A/B variant

The original custom certificate creation path uses `[System.DateTimeOffset]::UtcNow`. The UTC3 variant changes this to `UtcNow.AddHours(-3)` to isolate an observed UTC+03 activation issue. This hard-coded change only applies to the *custom-lifetime certificate generation path*. The default `New-ExchangeCertificate` path bypasses it when `-CustomCertificateLifetimeInDays` is not specified. The published TimeZoneAware variant supersedes this experiment by calculating the positive UTC offset dynamically and checking that the custom lifetime parameter is supplied whenever certificate creation is required.

### Accelerated LAB variant

The LAB variant adds `-LabMode`, `-CertificateLifetimeInMinutes`, `-NextCertificateEffectiveInMinutes`, `-CertificateNotBeforeHoursAgo` and `-CertificateNotAfterHoursAgo`. It supports rollover and expiry observations on a minutes-scale, and even an explicit already-expired certificate validity window. It **blocks Scheduled Task and AD automation setup**, forces version checking off, and bypasses the upstream Internal Transport certificate rebinding path by always using its custom self-signed generator for Auth Certificates. The upstream expiry recovery actions remain available (including AuthConfig publication and service/app-pool restarts). These functional changes make it unsuitable as a production substitute. `-LabMode` is acknowledgement, **not** a test sandbox.

## Provenance, safety and signatures

- All three files are preserved *as supplied*, including original encoding/line endings and the upstream Microsoft MIT notice.
- Derived from [Microsoft CSS-Exchange](https://github.com/microsoft/CSS-Exchange), upstream version `26.03.06.1531`.
- The Upstream file is a historical snapshot, not a replacement for the current Microsoft source.
- Both **UTC3 and Lab still contain copied Microsoft `# SIG #` blocks despite code changes**. Those signatures are **invalid for the modified scripts** and are preserved solely for archival fidelity. The released TimeZoneAware script removed the outdated signature block.
- A date or version in a file header is not a declaration that it has been validated on every Exchange build.
- **Do not test forced expiry or Auth Certificate recovery scenarios in production.** Use an isolated disposable Exchange lab.
- The root [LICENSE](../LICENSE) covers the MIT-licensed source.

## Published script

Refer to the root [MonitorExchangeAuthCertificate-TimeZoneAware.ps1](../MonitorExchangeAuthCertificate-TimeZoneAware.ps1), [README](../README.md) and Microsoft's current Exchange Auth Certificate / Hybrid guidance for supported production administration.
