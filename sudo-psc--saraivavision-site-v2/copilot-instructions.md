## saraivavision-site-v2

> This document provides instructions for manually updating the Nginx configuration.

# Gemini - Nginx Update Instructions

This document provides instructions for manually updating the Nginx configuration.

**Note:** The preferred method for deploying this application, including Nginx updates, is by using the `./deploy.sh` script. These instructions are for manual updates or for situations where the deploy script is not used.

## When to Update Nginx

You should follow these steps if you have made any changes to the Nginx configuration files, such as:

*   `nginx.conf`
*   `nginx-site-minimal.conf`
*   Files in the `nginx-includes/` directory

## Update Procedure

These commands should be run as root (`sudo`).

1.  **Copy the new configuration file:**

    This project uses a canonical `nginx.conf` or `nginx-site-minimal.conf` in the project root. You must copy the appropriate file to the Nginx configuration directory.

    ```bash
    # Choose the correct source file
    sudo cp nginx-site-minimal.conf /etc/nginx/sites-available/saraivavision
    ```

2.  **Test the Nginx configuration:**

    Before reloading, always test the configuration to ensure there are no syntax errors.

    ```bash
    sudo nginx -t
    ```

    If the test is successful, you will see a message like:
    `nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`
    `nginx: configuration file /etc/nginx/nginx.conf test is successful`

3.  **Reload Nginx:**

    If the configuration test passes, you can safely reload Nginx to apply the changes without downtime.

    ```bash
    sudo systemctl reload nginx
    ```

4.  **Verify the status:**

    Check the status of the Nginx service to ensure it reloaded correctly.

    ```bash
    sudo systemctl status nginx
    ```

---
*This is a guideline. Always refer to the `deploy.sh` script for the most current deployment logic.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sudo-psc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sudo-psc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
