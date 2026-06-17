## mailman3-gemini-skill

> Guide for moderating the Mailman 3 system, querying its active SQLite database directly, and adding or removing members without using sudo. Use this when the user needs to in


# Mailman 3 Moderation & Administration

## Core Principles & Permissions
- **Avoid `sudo` for files:** Do not use `sudo` to bypass file permission restrictions when reading logs or files.
- **Permissions:** If you encounter "permission denied" messages when trying to read files or logs, use `setfacl` to grant yourself (`gemini-sysadminhelper`) read permission to the specif
ic files or directories instead of elevating to root.

## Querying the Database
It is **much better and more reliable to directly query the active `sqlite3` database** than to try using the Mailman CLI commands (like `mailman members` or `mailman info`) to read data.
 The Mailman CLI often throws exceptions or lacks proper output formatting in this environment.

**Active Database Location:** `/opt/mailman/mm/var/data/mailman.db`

### Example Queries
To check the role distribution of members in a list:
```bash
sqlite3 /opt/mailman/mm/var/data/mailman.db "SELECT role, count(*) FROM member WHERE list_id = 'discuss.example.com' GROUP BY role;"
```
*Note: Role 1 = member, Role 2 = owner, Role 3 = moderator, Role 4 = nonmember.*

To find held/pending messages, query the `pended` and `pendedkeyvalue` tables.

## Adding and Removing Members
When you need to add or remove members, use the `mailman` executable inside the virtual environment by running the command as the `mailman` user (using `sudo -u mailman`).

The correct syntax requires passing the emails via a text file.

**To ADD a member:**
Write the email to a temporary file, then use `-W` to suppress welcome messages:
```bash
sudo -u mailman -H /opt/mailman/venv/bin/mailman -C /etc/mailman3/mailman.cfg addmembers -W /path/to/emails.txt list@domain.com
```

**To REMOVE a member:**
Write the email to a temporary file, then use `-f` to read the file:
```bash
sudo -u mailman -H /opt/mailman/venv/bin/mailman -C /etc/mailman3/mailman.cfg delmembers -G -N -f /path/to/emails.txt -l list@domain.com

---
> Source: [foofusdotcom/mailman3-gemini-skill](https://github.com/foofusdotcom/mailman3-gemini-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
