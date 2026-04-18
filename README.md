# Linux RBAC Lab — BagmatiCyber Ltd.

A hands-on Linux practice lab covering Role-Based Access Control (RBAC) configuration using real sysadmin tools and commands.

---

## What This Lab Covers

| Part | Topic |
|------|-------|
| 1 | User & group creation, primary/secondary group assignment |
| 2 | Directory structure and file creation |
| 3 | File ownership (`chown`) and permissions (`chmod`), SetGID bit |
| 4 | `sudo` configuration — allow/deny specific commands |
| 5 | Testing & validation — expected vs actual behavior |
| 6 | Group reassignment and full cleanup |
| 7 | Cron jobs, aliases, PAM login auditing, file integrity with `md5sum` |

---

## Skills Demonstrated

- Linux user and group management (`useradd`, `groupadd`, `usermod`, `gpasswd`)
- File permission model — octal notation, `chmod`, `chown`
- Special permission bits — **SetGID** for group inheritance on directories
- Sudoers configuration — fine-grained allow/deny rules via `/etc/sudoers.d/`
- Scheduled tasks — `cron` for automated log backups
- Shell customization — persistent aliases via `.bashrc`
- Login auditing — `pam_exec` to log user logins
- File integrity checking — `md5sum` to detect unauthorized changes

---

## Lab Environment

- OS: Ubuntu / Debian Linux
- Shell: Bash
- Tools: `useradd`, `chmod`, `chown`, `visudo`, `crontab`, `md5sum`, `pam_exec`

---

## Scenario

BagmatiCyber Ltd. has three departments — **Development**, **Audit & Compliance**, and **Operations**. The goal is to configure a secure, access-controlled workspace at `/opt/Bagmaticyber/` where:

- Each team can only access their own files
- New files automatically inherit the correct group ownership (SetGID)
- The Operations user (`Suchana`) can restart networking without a password but cannot open a root shell
- All activity is logged and files can be integrity-checked

---

## Directory Structure

```
/opt/Bagmaticyber/
├── projects/          # dev_team only (SetGID, chmod 2770)
│   └── dev_notes.txt  # owned by Jatil:dev_team
├── logs/              # audit_team only (SetGID, chmod 2770)
│   ├── audit.log      # owned by Mahesh:audit_team
│   └── archive/       # timestamped backups go here
└── README.md          # read-only for everyone (chmod 444)
```

---

## Users & Groups

| User | Primary Group | Secondary Groups | Role |
|------|--------------|-----------------|------|
| Jatil | dev_team | jatil | Developer |
| Mahesh | audit_team | mahesh | Auditor |
| Lok | dev_team | audit_team, lok | Dev + Audit access |
| Suchana | ops_team | sudo, suchana | Operations (limited sudo) |

---

## Key Commands Used

```bash
# Create groups
sudo groupadd dev_team

# Create user with primary and secondary groups
sudo useradd -m -g dev_team -G audit_team,lok -s /bin/bash Lok

# Set ownership
sudo chown Mahesh:audit_team /opt/Bagmaticyber/logs/audit.log

# SetGID on directory (group inheritance)
sudo chmod 2770 /opt/Bagmaticyber/projects

# Fine-grained sudo rule
echo "Suchana ALL=(ALL) NOPASSWD: /bin/systemctl restart networking" \
  | sudo tee /etc/sudoers.d/suchana

# File integrity check
sudo md5sum /opt/Bagmaticyber/README.md | sudo tee /opt/Bagmaticyber/README.md.md5
sudo md5sum -c /opt/Bagmaticyber/README.md.md5
```

---

## Test Results Summary

| Test | Result | Reason |
|------|--------|--------|
| Lok appends to `audit.log` | ❌ Denied | File is `640` — group has read only, not write |
| Mahesh lists `/projects` | ❌ Denied | Not in `dev_team`, others permission is `0` |
| Suchana restarts networking | ✅ Allowed | Explicit `NOPASSWD` rule in sudoers |
| Suchana runs `sudo bash` | ❌ Denied | Explicitly blocked with `!` in sudoers |
| Jatil deletes `README.md` | ❌ Denied | `chmod 444` — no write for anyone |
| Lok creates file in `/logs` | ✅ File owned by `Lok:audit_team` | SetGID forces group inheritance |

---

## Full Documentation

See [`BagmatiCyber_RBAC_Guide.docx`](./BagmatiCyber_RBAC_Guide.pdf) for the complete step-by-step guide with all commands, explanations, and expected outputs.

---

*Practice lab — Linux systems administration | April 2026*
