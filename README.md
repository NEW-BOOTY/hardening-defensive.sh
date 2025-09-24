How this script works (explanations — read after the code)
Safety-first design
Default operation is --dry-run. No system-altering actions occur unless you run with --apply.
All potentially destructive or system-level actions are routed through the run() helper which logs the intent and executes only in non-dry-run mode.
The script creates backups where safe (e.g., pwquality.conf.bak) and logs everything to /var/log/hardening_defensive.log.
DNS Blacklist
Maintains /etc/hardening/blacklist_domains.txt (one domain per line). On apply, it writes 127.0.0.1 domain entries to /etc/hosts for simple host-based blocking.
If dnsmasq exists, it builds address=/domain/127.0.0.1 entries under /etc/dnsmasq.d/ and reloads dnsmasq. This blocks domains at DNS resolution, which helps stop phishing domains and social media blocking as well.
ClamAV
Installs ClamAV (apt/yum supported) and creates a daily quick scan cron job.
Runs freshclam to update signatures.
Scans mail spool/paths if configured.
AIDE (File Integrity Monitoring)
Installs AIDE and initializes an AIDE database (if missing), schedules daily checks to detect tampering of system files.
USB & Social Engineering Protections
Installs a udev rule that hints automounters to ignore new removable USB devices, reducing auto-exec attack vectors.
Provides a simple usb-insert-monitor.sh that logs USB events (can be converted to a systemd service or integrated with SIEM/rsyslog).
Notes in comments to tighten mount options (nosuid,nodev,noexec) for removable mount paths.
Outbound SMTP Monitoring
Uses ss to count established outbound SMTP connections. If the count exceeds a heuristic threshold (here >5), an alert is sent. This helps detect exfiltration or spam-bots.
File Permissions Hardening
Walks sensitive dirs and removes go-w (group/other write) recursively to reduce risky file permissions.
pam_pwquality
Edits /etc/security/pwquality.conf to set stricter password policy (minlen = 14, require character classes). It takes a conservative approach (appends/checks).
Disable Unused Services + Firewall
Attempts to stop/disable/mask common unnecessary services (bluetooth, avahi, cups, nfs, rpcbind). You should customize this list for your environment.
Sets basic UFW defaults (deny incoming, allow outgoing) if ufw exists. For blocking social domains at the network level, DNS blocking (dnsmasq/hosts) is recommended.
Centralized Logging & Rotation
Ensures rsyslog is enabled and creates a logrotate entry for the script log.
The send_alert function supports email (via mail) and webhook (curl) to integrate with SIEM/CSIRT tooling.
Basic Anomaly Detection
Checks for spikes in failed authentication entries in auth logs and top process owners. Alerts if suspicious patterns appear.
Email Header & Attachment Scanning
Greps common mail spool locations for suspicious attachments and header patterns.
Optionally runs clamscan on mail directories.
MOTD Security Awareness
Writes a short message to /etc/motd to remind interactive users about phishing and reporting.
Deployment & Integration Suggestions
Test in staging first. Run --dry-run, review /var/log/hardening_defensive.log, then run --apply on a test VM.
SIEM/CSIRT: Configure WEBHOOK_URL or ALERT_EMAIL to your incident platform. You can also configure rsyslog to forward logs to your SIEM.
Sysadmin customization: Replace domain lists in ${BLACKLIST_FILE} with curated threat intel lists (commercial or community).
Audit & RBAC: Keep root access limited and ensure this script and its config files have strict permissions (root-only editing).
Package management: The script supports apt/yum. Extend for other distros as needed (dnf, zypper, pacman).
Systemd services: Convert monitor scripts (USB monitor) into systemd units for better lifecycle management.
