# Changelog

## 1.0 - 2026-09-20

- Added time zone-aware handling to the custom Auth Certificate validity window.
- For positive UTC offsets, `NotBefore` is backdated by the server's current UTC offset.
- `NotAfter` is calculated from the adjusted `NotBefore` value so the configured certificate lifetime remains consistent.
- No backdate is applied on UTC or negative UTC offsets.
- Preserved the upstream AuthConfig renewal and recovery logic.
- Preserved the upstream Internal Transport Certificate handling.
- Preserved the upstream multi-server certificate distribution/import logic.
- Preserved the upstream service/app-pool and Hybrid detection logic.
- Documented that `-CustomCertificateLifetimeInDays` must be greater than 0 whenever this variant creates, replaces, or stages a new Auth Certificate.

Based on Microsoft CSS-Exchange `MonitorExchangeAuthCertificate.ps1`, upstream version **26.03.06.1531**.
