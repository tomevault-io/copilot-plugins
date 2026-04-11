## pihole-email-tracker-blacklist

> This repository contains a curated list of domains used by various email tracking services. It is primarily intended for use with network-level DNS blockers like Pi-hole to enhance privacy by preventing email trackers from reporting user activity.

# Pi-hole Email Tracker Blacklist

This repository contains a curated list of domains used by various email tracking services. It is primarily intended for use with network-level DNS blockers like Pi-hole to enhance privacy by preventing email trackers from reporting user activity.

## Directory Overview

This directory serves as a centralized source for tracking and blocking email-related telemetry and tracking domains. It is a non-code project consisting mainly of domain lists.

## Key Files

- **email-trackers.txt**: The primary blacklist file. It contains a list of domains (one per line) associated with email tracking services (e.g., Mixmax, Yesware, HubSpot, Mailchimp, etc.).

## Future Scope

Based on repository branches (e.g., `add-spam-domains`), this project may expand to include blacklists for spam and other unwanted telemetry domains.

## Usage

To use this blacklist with Pi-hole or a similar DNS-based blocker:

1.  Open your Pi-hole admin interface.
2.  Navigate to **Adlists**.
3.  Add the URL of the `email-trackers.txt` file (e.g., the "raw" link from GitHub) to your adlists.
4.  Update your gravity database (`pihole -g` in the terminal or **Tools > Update Gravity** in the web interface).

Alternatively, you can manually append the contents of `email-trackers.txt` to your Pi-hole's local blacklist.

## Contributing

If you encounter an email tracker domain that is not currently blocked, feel free to suggest its addition to `email-trackers.txt`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaybocc2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jaybocc2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
