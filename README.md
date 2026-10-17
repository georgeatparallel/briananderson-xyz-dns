# briananderson-xyz-dns

Modular Terraform project to manage all DNS records for `briananderson.xyz` via Cloudflare.

## What This Project Demonstrates

A **production-grade** Terraform setup featuring:
- Modular DNS record management
- Cloudflare provider integration (v5)
- Zone settings management (HTTPS enforcement, redirect rules)
- Cloudflare Workers proxy for Cloud Run functions
- CI/CD automation with GitHub Actions
- Secure secret management via OIDC (no long-lived credentials)
- GCS remote state storage

## Project Structure

```
briananderson-xyz-dns/
├── modules/
│   ├── dns_web/          # Web DNS records (A, CNAME)
│   ├── dns_mail/         # Mail DNS records (MX, DKIM, SPF, DMARC)
│   ├── dns_homelab/      # Homelab services
│   ├── dns_verification/ # Domain verification (TXT)
│   ├── api_worker/       # Cloudflare Workers proxy to Cloud Run
│   └── zone_settings/    # HTTPS enforcement, www redirect
├── docs/                 # Documentation
├── .github/workflows/    # CI/CD pipeline
├── main.tf               # Module orchestration
├── records.tf            # DNS record and service definitions
├── variables.tf          # Secret variables (API token, zone/account IDs)
├── outputs.tf            # Outputs
├── backend.tf            # GCS remote state
├── provider.tf           # Cloudflare provider
└── terraform.tf          # Version constraints
```

## Quick Start

### 1. Install Prerequisites

```bash
# Terraform (>= 1.5.0)
# See https://developer.hashicorp.com/terraform/install
terraform --version
```

### 2. Setup Authentication

For CI/CD, see [docs/secrets-setup.md](docs/secrets-setup.md) for GitHub Secrets and OIDC setup.

For local development, create a `terraform.tfvars` file (gitignored) with your credentials:

```hcl
cloudflare_api_token  = "<your-api-token>"
cloudflare_zone_id    = "<your-zone-id>"
cloudflare_account_id = "<your-account-id>"
```

### 3. Run

```bash
terraform init
terraform plan
terraform apply
```

## Modules

| Module | Purpose |
|--------|---------|
| **dns_web** | Web DNS records (A, CNAME) for sites and apps |
| **dns_mail** | Email routing (MX, DKIM, SPF, DMARC) via Google Workspace |
| **dns_homelab** | Homelab service records |
| **dns_verification** | Domain verification TXT records |
| **api_worker** | Cloudflare Worker proxying to Cloud Run functions |
| **zone_settings** | Always Use HTTPS, www-to-root redirect |

## CI/CD Pipeline

GitHub Actions workflow automatically:

**On Pull Request:** Format check (enforced), validate, plan, comment on PR

**On Push to main:** All validation + `terraform apply`

## Security

- Secrets (API tokens, zone/account IDs) stored in GitHub Secrets and local `.tfvars` (gitignored)
- DNS record definitions are in committed code (`records.tf`) — they're public data
- OIDC authentication to GCP (no long-lived credentials)
- No secrets in repository history

## Documentation

| Document | Description |
|----------|-------------|
| [docs/architecture.md](docs/architecture.md) | Architecture, modules, and setup guide |
| [docs/secrets-setup.md](docs/secrets-setup.md) | GitHub Secrets and OIDC configuration |
| [docs/dns-records.md](docs/dns-records.md) | DNS records quick reference |
| [docs/mcp-gateway.md](docs/mcp-gateway.md) | Optional web search and page fetching through the MCP gateway |

## License

MIT License - Use and modify freely for your own projects.
