# Day 05 — Linux Log Review and Incident Triage

## Objective

Review local system logs for warning and error events, failed services, and signs of unexpected authentication or privilege activity.

## Scope

This is a defensive review of a personally administered Linux workstation. The work is limited to reading local system state and logs. No log entries are changed or removed. During the baseline collection, the distribution-provided `wtmpdb` utility was installed to make the `last` login-history command available.

## Evidence collected

### System baseline

- The host had been running for approximately one hour with low load and one interactive user.
- `systemctl --failed` reported zero failed units.
- Journald occupied approximately 395 MB, which is a modest local log footprint.

### Login-history review

- The initial `last` command was unavailable because the system did not have the `wtmpdb` utility installed.
- After installation, the reviewed login history showed local console sessions only; no remote source addresses were present in the displayed records.
- Multiple historical records displayed as "still logged in." This is common with graphical-session or reboot accounting and is not, by itself, evidence of concurrent active sessions.

## Preliminary assessment

**Status: No immediate incident indicator observed.** The captured baseline showed no failed services and no remote login source in the reviewed history. Current-boot warnings, authentication events, and service-specific logs still require review.

## Current-boot warning triage

### Expected or low-priority desktop and firmware warnings

- ACPI, PCI routing, USB-C, NVMe, Bluetooth, and Wi-Fi-driver warnings were recorded during boot. These are generally firmware or driver compatibility issues, not evidence of compromise.
- GNOME, desktop-extension, media-key, display, and application-autostart warnings were also present. They may affect user experience but do not indicate unauthorized access.
- DNS temporarily fell back from EDNS0 to plain UDP for one resolver. This suggests network or resolver compatibility behavior and should be monitored only if name-resolution problems occur.

### Configuration follow-up items

- Several SSSD responder sockets reported dependency failures during boot. Inspection showed that the SSSD service was intentionally skipped because its start condition was unmet; the responder sockets were inactive only because their parent service was inactive. This is expected when SSSD is installed but not configured for centralized identity, directory authentication, or SSSD-backed sudo rules.
- Polkit logged a reference to a nonexistent `admin` group. The source was Ubuntu's standard `49-ubuntu-admin.rules`, which authorizes both the `sudo` group and the legacy `admin` group. Because the existing `sudo` group is present, the missing legacy group is a benign compatibility warning rather than an authorization failure.
- A user-level telemetry service reported that an IP-firewall sandbox setting could not be applied because the unit was not running as root. This is a hardening limitation for that unit, not a host-firewall finding.

## Updated assessment

**Security incident review: No direct compromise indicator observed.** The reviewed warnings are dominated by boot firmware, driver, and desktop integration messages.

**Configuration hygiene: Resolved.** SSSD is dormant rather than failed, and the Polkit message comes from a standard compatibility rule. No change is required unless centralized authentication is needed; in that case, configure SSSD deliberately.

## Privilege-use audit

- All reviewed current-boot `sudo` events were initiated from the local terminal by the expected local account.
- The recorded commands matched the Day 04 and Day 05 activities: socket inspection, firewall inspection, log review, service-status checks, policy-file inspection, and installation of the `wtmpdb` login-history utility.
- No unexpected command, account, remote terminal, or failed sudo authentication event appeared in the reviewed sample.

## Final assessment

**Status: No security incident indicator observed in the reviewed evidence.** Local login history, service health, current-boot warnings, and the sampled sudo audit all aligned with known administrative and lab activity.

## Recommendations

1. Review warning-level journal events after operating-system, firmware, or driver updates.
2. Keep the system and firmware current, particularly to address recurring Wi-Fi and ACPI compatibility warnings.
3. Retain only the authentication components needed for the workstation's intended use; configure SSSD only if centralized identity is required.
4. Review privileged-command logs periodically and investigate activity that does not match planned administration.

## Assessment method

1. Record the system uptime and recent interactive-login history.
2. Review warnings and errors from the current boot.
3. Identify failed systemd units.
4. Check log retention and storage use.
5. Review local sudo and remote-access service events.
6. Distinguish expected application warnings from actionable security events.

## Success criteria

- No unexplained failed services or repeated critical system errors.
- No unexpected login or privilege-use patterns in the reviewed window.
- Findings identify evidence, impact, and a proportionate follow-up action.
