README — Trend Micro Vision One plugin for Microsoft Security Copilot

Overview

This folder contains a Security Copilot plugin manifest: `Trendmicro-plugin.json` which exposes a small set of skills to query Trend Micro Vision One (Workbench alerts and Endpoint Security endpoints).

This README explains how to upload and configure the plugin, how to test it from Security Copilot, and provides example curl / PowerShell calls you can use to validate connectivity.

Important security note

- Do not store long-lived bearer tokens or secrets in the manifest for production. Use the Security Copilot plugin "Set up" UI to enter tokens, or integrate with a secrets manager.
- Rotate and revoke tokens when no longer needed.

Prerequisites

- Access to Microsoft Security Copilot with permission to upload custom plugins.
- Trend Micro Vision One / XDR API credentials (API key or OAuth client id/secret). The official docs are: https://automation.trendmicro.com/xdr/api-v3/
- (Optional) If Security Copilot cannot fetch the vendor OpenAPI spec, host the OpenAPI spec at a reachable URL and update `Trendmicro-plugin.json` OpenApiSpecUrl field.
  Note: it's common for the Security Copilot uploader to be unable to reach vendor-hosted specs. You have two options:
  1) Host the OpenAPI spec at a public HTTPS URL (GitHub raw URL or your static hosting) and set `api.url` / `OpenApiSpecUrl` to that URL.
  2) Upload the manifest and the OpenAPI spec together; set `OpenApiSpecUrl` to the spec filename (e.g., `trendmicro-openapi.yaml`). The uploader will read the local spec if provided alongside the manifest.

Upload & setup steps (Security Copilot)

1. Open Security Copilot: https://securitycopilot.microsoft.com/ and sign in with an account that can manage plugins.
2. Go to Manage plugins -> Custom -> Add plugin.
3. Choose the JSON file type and upload `Trendmicro-plugin.json`.
4. After upload, click the plugin and then click "Set up" (or similar). Provide the following values when prompted:
   - "Auth Mode": choose `apiKey` (or `oauth` if you're using OAuth flow)
   - "API Key Header": usually `Authorization` (if you will pass `Bearer <token>`) or `x-api-key` if Trend Micro provided a direct API key header name
   - "API Key Value": paste the bearer token or API key here (entering it in the UI is safer than embedding it in the manifest)
   - If using OAuth: enter `ClientId` and `ClientSecret` and follow Trend Micro docs to obtain tokens.
5. Save the plugin configuration.

Testing the plugin via Security Copilot

Once configured:
- Use natural language prompts in Security Copilot such as:
  - "List Trend Micro workbench alerts from the last 24 hours"
  - "Show details for the alert <alertId>"
  - "List endpoints with hostname 'webserver01'"

Direct API test (curl / PowerShell)

If you want to test Trend Micro APIs directly before or while configuring the plugin, the following examples assume you already have a bearer token.

Example curl (replace <TOKEN>):

```
curl -H "Authorization: Bearer <TOKEN>" \
  "https://automation.trendmicro.com/xdr/api-v3/v3.0/workbench/alerts?limit=10"
```

PowerShell (replace <TOKEN>):

```powershell
$token = '<TOKEN>'
Invoke-RestMethod -Method Get -Uri 'https://automation.trendmicro.com/xdr/api-v3/v3.0/workbench/alerts?limit=10' -Headers @{ Authorization = "Bearer $token" }
```

If your tenant uses a different base URL or requires a token exchange endpoint, consult Trend Micro XDR docs and adapt the URL.

Troubleshooting

- 401 Unauthorized: verify token validity, token expiry, correct header name (Authorization vs x-api-key).
- 403 Forbidden: check API permissions in Trend Micro admin console.
- 404 Not Found: confirm the API path matches your Trend Micro XDR API version (some tenants may use /v3 or /v3.0 variations); update OpenApiSpecUrl if needed.
- Plugin not calling API in Security Copilot: ensure the uploaded OpenAPI spec is reachable and the `OperationHint` endpoints in the manifest match the spec paths.

- If you see an error like "The JSON value could not be converted to System.String" during upload, check any `DefaultValue` fields in the manifest. Security Copilot expects string typed default values; change numeric DefaultValue entries (e.g., 50) to strings (e.g., "50").

Follow-ups (recommended)

- Consider hosting the Trend Micro OpenAPI spec yourself if Security Copilot cannot access the vendor-hosted spec.
- Replace manifest-stored secrets with UI entry or secret injection via your secrets manager.
- Add an automated PowerShell test script that reads token from an environment variable instead of embedding it in files.

Contact / Notes

If you want, I can add sample PowerShell scripts and a small test harness that reads the bearer token from an environment variable and performs the same requests used by the skills.
