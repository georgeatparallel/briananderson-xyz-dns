# Web Search Through the MCP Gateway

The `mcp_gateway` module can proxy [Parallel Search MCP](https://docs.parallel.ai/integrations/mcp/search-mcp)
alongside your existing MCP backends. It provides `web_search` for current information
and `web_fetch` for reading pages. Anonymous search uses Fast mode and requires no
Parallel account or API key. The free endpoint is intended for exploration and light
use, with rate limits.

This setup is optional. The default `mcp_gateway_routes` map is empty, and adding
this route does not require a new tunnel or a self-hosted search server.

## Configure the Route

Complete the [Cloudflare and Terraform setup](../README.md#quick-start) first. In your
gitignored `terraform.tfvars`, add a `parallel` entry to `mcp_gateway_routes`:

```hcl
mcp_gateway_routes = {
  # Keep your existing route entries here too.
  parallel = {
    url = "https://search.parallel.ai"
    headers = {
      "User-Agent" = "briananderson-xyz-dns/mcp-gateway"
    }
  }
}
```

Terraform replaces the whole map when you set this variable, so retain any existing
entries, including their backend headers. Keep your existing
`mcp_gateway_bearer_token`; do not commit it or any backend credentials.

Use the host-only backend URL above, without a trailing slash or `/mcp`. The Worker
removes the first path segment: `/parallel/mcp` becomes `/mcp` at
`https://search.parallel.ai`. Setting the backend URL to the full MCP endpoint would
duplicate that path.

Review the plan before applying the route:

```bash
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

The current CI workflow does not set `TF_VAR_mcp_gateway_routes`. A local `.tfvars`
file is not available in GitHub Actions, so a later CI apply can reset routes to the
empty default. For a persistent CI-managed setup, supply the **complete** route map
through `TF_VAR_mcp_gateway_routes` in both plan and apply, using GitHub Secrets if
it includes credentials. Coordinate this with the existing deployment workflow.

## Connect an MCP Client

Configure an MCP client that supports Streamable HTTP and custom headers with:

| Setting | Value |
|---------|-------|
| Server URL | `https://mcp.<your-domain>/parallel/mcp` |
| Transport | Streamable HTTP |
| Authorization header | `Bearer <your-gateway-token>` |

Use your configured gateway hostname and domain if they differ from the defaults.
The token is the gateway's `mcp_gateway_bearer_token`, not a Parallel API key. Store
it using your client's secret configuration. The Worker strips that Authorization
header before forwarding to Parallel; leave Authorization out of this route's
backend `headers` to use the anonymous endpoint.

After connecting, list the server's tools. You should see `web_search` and
`web_fetch`. Try asking the client to find the current Terraform GCS backend
documentation, then read the relevant page. The client handles MCP initialization
and tool calls.

## Troubleshooting

| Response | Check |
|----------|-------|
| `401` from the gateway | Send an Authorization header starting with `Bearer `, including the space. |
| `403` from the gateway | Use the configured gateway bearer token. |
| `404` with `Unknown route` | Confirm the deployed map contains `parallel` and the client URL includes `/parallel/mcp`. |
| Upstream `404` | Confirm the backend URL is `https://search.parallel.ai` and the client path ends in `/parallel/mcp`. |
| Upstream `429` or a quota tool error | Respect any retry guidance; the anonymous service has usage limits. |

For higher limits, see [Parallel's authentication options](https://docs.parallel.ai/integrations/mcp/search-mcp#installation).
Gateway authentication and upstream authentication are separate credentials.
