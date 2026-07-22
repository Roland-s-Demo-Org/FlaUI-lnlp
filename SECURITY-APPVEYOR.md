# AppVeyor Security: dotnet-install.ps1 Hash Pinning

## Overview

The AppVeyor CI pipeline downloads and executes the official .NET SDK installer script from Microsoft. To prevent supply-chain attacks, the pipeline verifies the script's SHA256 hash before execution.

## Initial Setup

Before the first build after this security fix, you must obtain and set the actual hash of the `dotnet-install.ps1` script:

1. Download the script manually:
   ```powershell
   Invoke-WebRequest -Uri 'https://dot.net/v1/dotnet-install.ps1' -UseBasicParsing -OutFile dotnet-install.ps1
   ```

2. Calculate its SHA256 hash:
   ```powershell
   Get-FileHash -Path dotnet-install.ps1 -Algorithm SHA256
   ```

3. Update the `$expectedHash` variable in `appveyor.yml` (line 12) with the actual hash value.

4. Commit the change to the repository.

## Updating the Script

When Microsoft updates the `dotnet-install.ps1` script, or when you need to use a newer version:

1. Download the new version of the script
2. **Manually review the script content** to ensure it hasn't been compromised
3. Calculate the new SHA256 hash using the command above
4. Update the `$expectedHash` variable in `appveyor.yml`
5. Update this documentation with the date and reason for the hash change
6. Commit the changes

## Hash Change Log

| Date | Hash | Reason | Updated By |
|------|------|--------|------------|
| YYYY-MM-DD | (initial hash) | Initial security fix implementation | (your name) |

## Security Rationale

This hash pinning mechanism protects against:
- Compromise of the upstream Microsoft CDN
- Man-in-the-middle attacks that bypass HTTPS
- Accidental or malicious changes to the script

The build will fail if the downloaded script's hash doesn't match the expected value, preventing execution of potentially malicious code.

## Alternative: Vendoring the Script

For maximum security, consider vendoring (copying) the `dotnet-install.ps1` script into the repository:

1. Download and review the script
2. Add it to the repository (e.g., in a `build-tools/` directory)
3. Update `appveyor.yml` to execute the local copy instead of downloading
4. Update the script manually when needed, with proper review

This eliminates the network dependency entirely but requires manual updates.
