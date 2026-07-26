# Security Best Practices

This document provides security guidance for deploying and using legacy GeoIP databases.

## File Security

### Unix/Linux Permissions

```bash
# Optimal permissions for GeoIP database
sudo chmod 644 /usr/share/GeoIP/GeoIP.dat
sudo chown root:root /usr/share/GeoIP/GeoIP.dat

# Restrict directory access
sudo chmod 755 /usr/share/GeoIP/
```

### Windows Permissions

```powershell
# Remove inheritance
$path = "C:\GeoIP\GeoIP.dat"
$acl = Get-Acl $path
$acl.SetAccessRuleProtection($true, $false)

# Grant read-only to SYSTEM
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "SYSTEM",
    "Read",
    "Allow"
)
$acl.AddAccessRule($rule)

# Grant read-only to application user
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "IIS AppPool\DefaultAppPool",
    "Read",
    "Allow"
)
$acl.AddAccessRule($rule)

Set-Acl -Path $path -AclObject $acl
```

## Access Control

### Least Privilege Access

```bash
# Create dedicated user for GeoIP lookups (Linux)
sudo useradd -r -s /bin/false geoip

# Assign permissions
sudo chown geoip:geoip /usr/share/GeoIP/GeoIP.dat
sudo chmod 640 /usr/share/GeoIP/GeoIP.dat
```

### Application Isolation

```apache
# Apache - Run GeoIP in separate process
<IfModule mod_geoip.c>
    # Isolate GeoIP module
    GeoIPEnable On
    GeoIPDBFile /usr/share/GeoIP/GeoIP.dat
</IfModule>
```

## Data Validation

### Input Validation

Always validate IP addresses before lookup:

```python
import ipaddress
import geoip2.database

def safe_geoip_lookup(ip_string):
    try:
        # Validate IP address format
        ipaddress.ip_address(ip_string)
        
        with geoip2.database.Reader('GeoIP.dat') as reader:
            return reader.city(ip_string)
    except ValueError:
        print(f"Invalid IP address: {ip_string}")
        return None
    except Exception as e:
        print(f"Error: {e}")
        return None
```

### Rate Limiting

Prevent abuse of GeoIP service:

```python
from flask import Flask
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)
limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/geoip/<ip>')
@limiter.limit("10/minute")
def get_geoip(ip):
    # Your GeoIP lookup logic
    pass
```

## Network Security

### Secure Access

```apache
# Restrict API access by network
<Location /api/geoip>
    Require ip 192.168.1.0/24
    Require ip 10.0.0.0/8
</Location>
```

### HTTPS/TLS

```nginx
# Force HTTPS for GeoIP API
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name api.example.com;
    
    ssl_certificate /etc/ssl/certs/certificate.crt;
    ssl_certificate_key /etc/ssl/private/key.key;
    
    location /geoip {
        # GeoIP API endpoint
    }
}
```

## Monitoring & Logging

### Audit Logging

```bash
# Enable GeoIP access logging on Apache
<VirtualHost *:80>
    CustomLog logs/geoip_audit.log "%h %l %u %t \"%r\" %>s %b %{GEOIP_COUNTRY_CODE}e"
</VirtualHost>

# Monitor access
tail -f logs/geoip_audit.log
```

### Detect Anomalies

```bash
#!/bin/bash
# geoip-anomaly.sh - Detect suspicious GeoIP patterns

# Alert on excessive lookups from same IP
tail -1000 /var/log/apache2/access.log | \
    awk '{print $1}' | \
    sort | uniq -c | \
    sort -rn | \
    head -20 | \
    awk '$1 > 100 {print "Alert: " $2 " made " $1 " requests"}'
```

## Database Integrity

### Verify File Integrity

```bash
# Create checksum
sha256sum /usr/share/GeoIP/GeoIP.dat > /usr/share/GeoIP/GeoIP.dat.sha256

# Verify later
sha256sum -c /usr/share/GeoIP/GeoIP.dat.sha256
```

### Regular Backups

```bash
#!/bin/bash
# backup-geoip.sh

BACKUP_DIR="/var/backups/geoip"
GEOIP_DIR="/usr/share/GeoIP"

mkdir -p "$BACKUP_DIR"

# Daily backup
cp "$GEOIP_DIR/GeoIP.dat" "$BACKUP_DIR/GeoIP.dat.$(date +%Y%m%d)"

# Keep only last 30 days
find "$BACKUP_DIR" -name "GeoIP.dat.*" -mtime +30 -delete

echo "GeoIP backup completed"
```

### Cron Backup Job

```bash
# Add to crontab
0 2 * * * /usr/local/bin/backup-geoip.sh
```

## Compliance & Privacy

### Privacy Considerations

- GeoIP lookups may involve processing location data
- Comply with GDPR, CCPA, and other privacy regulations
- Implement data retention policies
- Log only necessary information

### Privacy-Aware Logging

```python
# Hash IP addresses in logs
import hashlib

def log_geoip_access(ip_address):
    # Hash IP for privacy
    hashed_ip = hashlib.sha256(ip_address.encode()).hexdigest()[:16]
    # Log hashed IP instead
    print(f"GeoIP lookup: {hashed_ip}")
```

### GDPR Compliance

```python
# Implement data deletion
def delete_geoip_logs(days=90):
    """Delete GeoIP logs older than specified days"""
    import os
    from datetime import datetime, timedelta
    
    cutoff_date = datetime.now() - timedelta(days=days)
    
    for filename in os.listdir('/var/log/geoip/'):
        file_path = os.path.join('/var/log/geoip/', filename)
        file_time = datetime.fromtimestamp(os.path.getmtime(file_path))
        
        if file_time < cutoff_date:
            os.remove(file_path)
```

## Vulnerability Management

### Keep Dependencies Updated

```bash
# Update system
sudo apt-get update
sudo apt-get upgrade

# Update GeoIP tools
sudo apt-get install --only-upgrade geoip-bin

# Update Python packages
pip install --upgrade geoip2
```

### Monitor Advisories

- Subscribe to security mailing lists
- Follow GitHub security advisories
- Monitor MaxMind security notices
- Keep database versions current

## Incident Response

### Database Compromise

If database is compromised:

```bash
# 1. Restore from backup
sudo cp /var/backups/geoip/GeoIP.dat.20260701 /usr/share/GeoIP/GeoIP.dat

# 2. Verify integrity
sha256sum -c /usr/share/GeoIP/GeoIP.dat.sha256

# 3. Review logs for suspicious activity
grep "geoip" /var/log/apache2/access.log | tail -1000

# 4. Restart affected services
sudo systemctl restart apache2

# 5. Audit file permissions
ls -la /usr/share/GeoIP/
```

### Unauthorized Access

```bash
# Check access logs
sudo tail -1000 /var/log/apache2/access.log

# Identify suspicious IPs
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -10

# Block suspicious IPs
sudo iptables -A INPUT -s SUSPICIOUS_IP -j DROP
```

## Security Checklist

- [ ] File permissions set correctly
- [ ] Access control configured
- [ ] Input validation implemented
- [ ] Rate limiting enabled
- [ ] HTTPS/TLS enabled for APIs
- [ ] Audit logging configured
- [ ] Backups scheduled
- [ ] Integrity checks in place
- [ ] Privacy policies compliant
- [ ] Emergency response plan ready

## Additional Resources

- OWASP Security Guidelines: https://owasp.org/
- CIS Benchmarks: https://www.cisecurity.org/
- NIST Cybersecurity Framework: https://www.nist.gov/

