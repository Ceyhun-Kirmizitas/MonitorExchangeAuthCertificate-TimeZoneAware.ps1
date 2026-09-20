# MonitorExchangeAuthCertificate-TimeZoneAware.ps1

Time zone-aware variant of Microsoft's `MonitorExchangeAuthCertificate.ps1` for Exchange Server Auth Certificate validation, renewal, and recovery scenarios.

This repository contains a modified version of the Microsoft CSS-Exchange script. It is **not an official Microsoft release**.

## Download

- [GitHub source](MonitorExchangeAuthCertificate-TimeZoneAware.ps1)
- [GitHub raw download](https://raw.githubusercontent.com/Ceyhun-Kirmizitas/MonitorExchangeAuthCertificate-TimeZoneAware.ps1/main/MonitorExchangeAuthCertificate-TimeZoneAware.ps1)
- [Website mirror](https://ceyhunkirmizitas.net/wp-content/uploads/tools/MonitorExchangeAuthCertificate-TimeZoneAware.ps1)

If GitHub access is restricted in your environment, the same script is also available from the website mirror.

## Why this variant exists

The modification addresses an Auth Certificate activation issue observed on Exchange servers configured in time zones ahead of UTC.

When the script creates a custom-lifetime Auth Certificate, this variant:

- Reads the server's current UTC offset.
- Backdates `NotBefore` only when the server has a positive UTC offset.
- Calculates `NotAfter` from the same adjusted `NotBefore` value so the requested certificate lifetime remains consistent.
- Applies no backdate on UTC or negative UTC offsets.

The issue was reproduced and validated in a UTC+03 environment.

The original Microsoft AuthConfig renewal and recovery logic, Internal Transport Certificate handling, multi-server certificate distribution/import logic, service/app-pool handling, and Hybrid detection logic remain unchanged.

## Important: CustomCertificateLifetimeInDays

The time zone-aware change is implemented in the custom Auth Certificate creation path.

When the script must create, replace, or stage a new Auth Certificate, specify `-CustomCertificateLifetimeInDays` with a value greater than 0.

Without this parameter, the upstream script uses `New-ExchangeCertificate` and the time zone-aware `NotBefore` / `NotAfter` modification is bypassed.

Validation-only runs and certificate import/redistribution-only actions do not require this parameter.

## Examples

Validation only:

```powershell
.\MonitorExchangeAuthCertificate-TimeZoneAware.ps1
```

Validate and renew with a five-year custom lifetime:

```powershell
.\MonitorExchangeAuthCertificate-TimeZoneAware.ps1 `
    -ValidateAndRenewAuthCertificate $true `
    -CustomCertificateLifetimeInDays 1825
```

Force creation of a new next Auth Certificate:

```powershell
.\MonitorExchangeAuthCertificate-TimeZoneAware.ps1 `
    -EnforceNewAuthCertificateCreation `
    -CustomCertificateLifetimeInDays 1825 `
    -Confirm:$false
```

For Hybrid environments, the original script behavior still applies. Use `-IgnoreHybridConfig $true` only when intentionally performing the renewal, and run the Hybrid Configuration Wizard after the new primary Auth Certificate becomes active.

## Upstream

Based on:

- Microsoft CSS-Exchange `MonitorExchangeAuthCertificate.ps1`
- Upstream script version: **26.03.06.1531**
- Upstream commit: [2d8444fb37b77f1b83da2ce352ed683887699daa](https://github.com/microsoft/CSS-Exchange/commit/2d8444fb37b77f1b83da2ce352ed683887699daa)
- [Microsoft CSS-Exchange repository](https://github.com/microsoft/CSS-Exchange)
- [Official MonitorExchangeAuthCertificate documentation](https://github.com/microsoft/CSS-Exchange/blob/main/docs/Admin/MonitorExchangeAuthCertificate.md)

Because this file is modified, the original Microsoft Authenticode signature is not retained. The existing version-check logic skips automatic update checks for unsigned builds, so `-SkipVersionCheck` is not normally required with this variant.

## Full field guide

[Exchange Server Auth Certificate Field Guide: Validation, Rotation, Recovery, and Time Zone Issues](https://ceyhunkirmizitas.net/exchange-server-auth-certificate-renewal-recovery-timezone/)

## License

MIT. The upstream Microsoft copyright and license are preserved. See [LICENSE](LICENSE).
