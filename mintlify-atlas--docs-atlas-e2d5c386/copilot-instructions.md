## docs-atlas-e2d5c386

> Link your running Reacher server to Claude.ai as a custom MCP connector.


## Prerequisites

- Reacher must be running and publicly reachable over HTTPS. If you haven't done that yet, complete the [Quickstart](/quickstart) first.
- Your server URL must use HTTPS — Claude.ai will not connect to plain HTTP endpoints.
- You'll need your `MCP_SECRET` value from your `.env` file.

---

## Add the connector

<Steps>

<Step title="Open Claude.ai settings">

Go to [Claude.ai](https://claude.ai) and click your profile avatar, then select **Settings**.

</Step>

<Step title="Navigate to Integrations">

In the Settings sidebar, select **Integrations**.

</Step>

<Step title="Add a custom connector">

Click **Add custom connector**.

</Step>

<Step title="Enter your server URL">

Paste your Reacher URL in the following format:

```
https://yourdomain.com/mcp?token=YOUR_MCP_SECRET
```

Replace `yourdomain.com` with your actual domain and `YOUR_MCP_SECRET` with the value from your `.env` file.

<Warning>
The `token` query parameter must exactly match the `MCP_SECRET` in your `.env`. If they don't match, every request returns a 401 Unauthorized error and Claude will show a connection failure.
</Warning>

</Step>

<Step title="Save and verify">

Save the connector. Claude.ai will attempt to contact your server and retrieve the tools list.

If the connection succeeds, you'll see Reacher listed as an active integration. Open a new conversation and you should see the Reacher tools available.

</Step>

</Steps>

---

## Verify Claude can call tools

Start a new conversation and try a prompt that exercises a tool:

> "What devices are on my Tailscale network?"

Claude should call `tailscale_status` and return a list of your devices with their hostnames, IP addresses, OS, and online/offline status.

If Claude responds without calling any tools, the connector may not be active in that conversation. Look for a tools icon or connector indicator in the Claude.ai UI and confirm Reacher is enabled.

---

## Bootstrap Claude with AGENT.MD

The repository includes an `AGENT.MD` file that tells Claude how to use Reacher's tools effectively: how to discover your devices, when to use each tool, how to handle Windows vs. Linux targets, and what to save to the knowledge base.

<Tip>
Drop the contents of `AGENT.MD` into the start of a new conversation. Claude will run the first-time setup checklist automatically: discover your Tailscale devices, probe SSH access on each one, check the `gist_kb` knowledge base for any saved context, and write a device map so future sessions don't start from scratch.
</Tip>

The first-time checklist Claude follows from `AGENT.MD`:

1. Call `tailscale_status` to discover all devices, their hostnames, OS, and online status
2. Run a test command via `ssh_exec` on each online device to confirm SSH access
3. Call `gist_kb` with `action: list` to check for saved notes from previous sessions
4. Write a device map to a gist (`RC: device-map`) so future sessions resume with full context

After this runs once, every new session can pick up where the last one left off.

---

## Troubleshooting

**Claude.ai shows "connection failed" when adding the connector**

- Confirm the server is publicly reachable: `curl "https://yourdomain.com/health?token=YOUR_MCP_SECRET"`
- Check that your reverse proxy is forwarding to port 3000 (or whatever `PORT` you set)
- Verify your SSL certificate is valid and not expired
- Make sure port 443 is open in your VPS firewall

**Tools are listed but Claude gets errors when calling them**

- Check the server logs: `docker compose logs -f reacher`
- Confirm `MCP_SECRET` in the URL matches the one in `.env` exactly (it's case-sensitive)
- Look at `reacher-audit.log` for logged tool calls and their results

**`fetch_external` returns "Domain not allowed"**

The domain isn't in your `PROXY_ALLOWED_DOMAINS` list. Add it to `.env` and restart the server:

```bash
PROXY_ALLOWED_DOMAINS=api.github.com,api.linear.app,api.notion.com
```

**`ssh_exec` fails to connect to a device**

- Run `tailscale_status` to confirm the device shows online (note: status can lag — try SSH anyway)
- Verify Tailscale SSH is enabled on the target device: `sudo tailscale up --ssh`
- Test the connection manually from your Reacher server: `ssh user@device-hostname`

**`gist_kb` or `github_search` returns an authentication error**

Your `GITHUB_TOKEN` is missing or doesn't have the required scopes. The token needs `gist` scope for `gist_kb` and any additional API scopes needed for `fetch_external` targets. Create or update the token at [github.com/settings/tokens](https://github.com/settings/tokens).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mintlify-atlas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mintlify-atlas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
