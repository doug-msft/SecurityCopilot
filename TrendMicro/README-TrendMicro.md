README — Trend Micro Vision One plugin for Microsoft Security Copilot

Overview

This folder contains a Security Copilot plugin manifest `trendmicro-plugin.yaml` (YAML) and an OpenAPI spec `trendmicro-openapi.yaml` which expose a small set of skills to query Trend Micro Vision One (Workbench alerts and Endpoint Security endpoints).

This README explains how to upload and configure the plugin, how to test it from Security Copilot, and provides example curl / PowerShell calls you can use to validate connectivity.

Important security note

- Do not store long-lived bearer tokens or secrets in the manifest for production. Use the Security Copilot plugin "Set up" UI to enter tokens, or integrate with a secrets manager.
- Rotate and revoke tokens when no longer needed.

Prerequisites

- Access to Microsoft Security Copilot with permission to upload custom plugins.
- Trend Micro Vision One / XDR API key. The official docs are: 
   --https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-platform-api-keys
   --https://automation.trendmicro.com/xdr/api-v3/
 - (Optional) If Security Copilot cannot fetch the vendor OpenAPI spec, host the OpenAPI spec at a reachable URL and update the plugin manifest `OpenApiSpecUrl` field (for example, `trendmicro-openapi.yaml`).
  Note: For this sample plugin, you may use the existing OpenAPI spec path stored on this repo, which is already configured in the sample manifest file, as documented below:
  OpenApiSpecUrl: https://raw.githubusercontent.com/doug-msft/SecurityCopilot/main/TrendMicro/trendmicro-openapi.yaml 

Upload & setup steps (Security Copilot)

1. Open Security Copilot: https://securitycopilot.microsoft.com/ and sign in with an account that can manage plugins.
2. Go to Manage plugins -> Custom -> Add plugin.
3. Choose the YAML file type and upload `trendmicro-plugin.yaml` 
4. After upload, click the plugin and then click "Set up" (or similar). Provide the following API Key when prompted:
   - "API Key Value": paste the bearer token or API key here (entering it in the UI is safer than embedding it in the manifest)  
5. Save the plugin configuration.

Testing the plugin via Security Copilot

Once configured:
- Use natural language prompts in Security Copilot such as:
  - "List Trend Micro alerts from the last 24 hours"
  - "Show details for the alert <alertId>"
  - "List endpoints with hostname 'webserver01'"
  - "Get endpoint details"

Direct API test (curl / PowerShell)

If you want to test Trend Micro APIs directly before or while configuring the plugin, the following examples assume you already have a bearer token.

Example curl (replace <TOKEN>):

```
curl -H "Authorization: Bearer <TOKEN>" \
  "https://api.xdr.trendmicro.com/v3.0/workbench/alerts?limit=10"
```

PowerShell (replace <TOKEN>):

```powershell
$token = '<TOKEN>'
Invoke-RestMethod -Method Get -Uri 'https://api.xdr.trendmicro.com/v3.0/workbench/alerts?limit=10' -Headers @{ Authorization = "Bearer $token" }
```

If your tenant uses a different base URL or requires a token exchange endpoint, consult Trend Micro XDR docs and adapt the URL.

Troubleshooting

- 401 Unauthorized: verify token validity, token expiry, correct header name (Authorization vs x-api-key).
- 403 Forbidden: check API permissions in Trend Micro admin console.
- 404 Not Found: confirm the API path matches your Trend Micro XDR API version (some tenants may use /v3 or /v3.0 variations); update OpenApiSpecUrl if needed. The sample OpenAPI in this folder targets the base URL `https://api.xdr.trendmicro.com`.



