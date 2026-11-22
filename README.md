# PFS Free Domains for Cursor

Find and register free PFS domains, manage DNS and WHOIS records, issue SSL certificates, and deploy to SEUS free hosting — directly from Cursor or any MCP-compatible client.

This repository is the [Cursor Plugin](https://cursor.com/docs/reference/plugins) package. It contains no code to install: the MCP server is a hosted remote service, so you only add a URL and sign in with your SEUS account.

## Install in Cursor

The fastest way is the one-click install from the Cursor Marketplace. If you have this repository link instead, open it in Cursor via **Customize > Plugins > Install from GitHub** and paste:

```
https://github.com/PublicFreeSuffix/pfs-cursor-plugin
```

Then enable the plugin under **Customize > Plugins** and complete the SEUS sign-in when prompted.

To configure the server manually, add this to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (this project only):

```json
{
  "mcpServers": {
    "pfs": {
      "url": "https://s.eus/api/mcp/pfs",
      "auth": {
        "CLIENT_ID": "nsc_KY87pj5df041ShWTuuN2R_ZR"
      }
    }
  }
}
```

Restart Cursor, enable `pfs` under **Customize > MCPs**, and complete the SEUS sign-in. The client ID is public and safe to commit; no client secret or `Authorization` header is needed. A fixed `Authorization` header prevents the OAuth flow.

## Connection

| Setting | Value |
| --- | --- |
| Transport | Streamable HTTP |
| URL | `https://s.eus/api/mcp/pfs` |
| Authentication | SEUS OAuth (PKCE, S256) |
| Plugin name | `pfs-free-domains` |
| Version | `0.6.0` |

No local process, package, or API key is required. The endpoint is hosted by SEUS and publicly reachable.

## Authorization

You never copy a key, token, or client ID into the client. Authorization takes two steps: click **Connect** (or install the plugin), then sign in to SEUS and approve the request in the browser that opens. The client handles everything else — it probes the endpoint, follows the `401` challenge to SEUS metadata, opens the authorization URL with a public client ID and a fresh PKCE challenge, receives a code on a registered callback, exchanges it for tokens, reconnects, and the tools appear. No client secret is involved.

The grant is persistent: the client stores tokens securely and renews them automatically, so you normally sign in only once. You will be asked again if you disconnect and reconnect, if you revoke the grant, or if you switch to a client that does not reuse stored tokens. For clients without a browser callback, SEUS also supports the device-code grant.

The client ID belongs to the SEUS OAuth application, not to Cursor. One app serves every MCP client. Browser clients use the callbacks registered on that app, device-code clients need no callback, and clients that cannot auto-register can take the same client ID as a pre-registered value. A second app is only needed for separate revocation control or a different callback allowlist.

Scopes advertised for this resource are `domains:read`, `domains:write`, `dns:read`, `dns:write`, `ssl:read`, `ssl:write`, `hosting:read`, and `hosting:write`. Read-only questions such as availability checks and WHOIS work with the read scopes; registration and DNS changes need the matching write scopes. To revoke access, disconnect the plugin in Cursor and remove the application grant from your SEUS account settings.

## Other clients

The same endpoint works in any client that supports remote MCP over Streamable HTTP — Claude, Claude Code, VS Code, Cline, Windsurf / Devin, and ChatGPT. See the [full installation guide](https://github.com/PublicFreeSuffix/PublicFreeSuffix/blob/main/mcp/README.md#install-in-your-mcp-client) for client-specific steps. Clients that only support local stdio servers can bridge through `mcp-remote`.

The service is also listed in the [official MCP Registry](https://registry.modelcontextprotocol.io/?q=PublicFreeSuffix) as `io.github.PublicFreeSuffix/free-domains`.

## Troubleshooting

- **Connected but no tools appear.** Fully quit and restart Cursor, then start a new chat. Clients cache the tool list at startup.
- **No sign-in prompt, or a 401 loop.** Check that the URL is exactly `https://s.eus/api/mcp/pfs`, uses HTTPS, and has no `Authorization` header.
- **403 with `insufficient_scope`.** Disconnect and reconnect, then approve the write scopes you actually need.
- **Sign-in fails with a registration error.** Make sure the `auth.CLIENT_ID` field is present; SEUS does not auto-register OAuth clients.
- **Another client says "the app is misconfigured".** SEUS rejects unregistered loopback ports. Use the device-code bridge configuration in the [full installation guide](https://github.com/PublicFreeSuffix/PublicFreeSuffix/blob/main/mcp/README.md#other-clients).

Try a read-only prompt first: "Use PFS Free Domains to check whether `example.no.kg` is available."

## License

MIT — see [LICENSE](LICENSE). Learn more at [Public Free Suffix](https://publicfreesuffix.org/) and the [SEUS MCP documentation](https://s.eus/docs/integrations/mcp).
