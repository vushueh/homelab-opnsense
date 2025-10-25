# Configuration Management

This directory contains OPNsense configuration files and templates.

## ⚠️ Security Warning

**NEVER commit actual configuration files with passwords, keys, or sensitive data.**

## Directory Structure
```
configs/
├── backups/           # Configuration backups (excluded from git)
├── templates/         # Generic configuration templates
└── production/        # Sanitized production configs (passwords removed)
```

## Backup Procedures

### Manual Backup
```bash
./scripts/backup/backup-config.sh
```

### Automated Backup
```powershell
# Windows Scheduled Task
./scripts/backup/automated-backup.ps1
```

## Configuration Files

Configuration files are XML-based and can be exported from:
**System → Configuration → Backups**

### Sanitization Checklist

Before committing any config:
- [ ] Remove all passwords
- [ ] Remove API keys
- [ ] Remove certificate private keys
- [ ] Remove VPN pre-shared keys
- [ ] Remove RADIUS secrets
- [ ] Replace public IPs with placeholders
- [ ] Remove MAC addresses if sensitive

### Template Usage

Use templates in `templates/` directory as starting points for new deployments.
