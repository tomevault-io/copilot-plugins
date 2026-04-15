## east-african-intellectual-property-portal-website

> Secure SSH connection and remote management for eastafricanip.com


# Remote Server Management (SSH)

Use these standards for all remote operations on the cPanel production server:

## **Connection Details**
- **Host**: `eastafricanip.com`
- **Port**: `7822`
- **User**: `falolega`
- **Authentication**: Uses the pre-configured SSH key on `tpms-elitebook`.

## **Core Commands**
### **1. Interactive Shell**
```bash
ssh -p 7822 falolega@eastafricanip.com
```

### **2. Database Operations (via SSH)**
To run SQL queries or updates directly:
```bash
ssh -p 7822 falolega@eastafricanip.com "mysql -u falolega_admin -p'eastafricanip1q2w3e4r5t' -D falolega_tpms -e \"[SQL_QUERY_HERE]\""
```

### **3. Application Management**
- **Read Passenger Logs**: `ssh -p 7822 falolega@eastafricanip.com "tail -n 50 ~/logs/passenger.log"`
- **Restart Backend**: `ssh -p 7822 falolega@eastafricanip.com "touch ~/eastafricanip.com/api/tmp/restart.txt"`
- **List Files**: `ssh -p 7822 falolega@eastafricanip.com "ls -la ~/eastafricanip.com"`

## **Workflow**
1. Always run `node server/generate_tables.js` locally after making remote DB changes.
2. Use the `restart.txt` method to apply backend code changes instantly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/israelseleshi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
