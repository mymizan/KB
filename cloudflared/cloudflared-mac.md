# Cloudflare Tunnel on macOS for Local n8n

This guide explains how to expose a local n8n instance running on macOS using Cloudflare Tunnel. The resulting URL is suitable for OAuth callback URLs (e.g., Google OAuth), webhooks, and other public integrations.

---

# Prerequisites

- A Cloudflare account
- A domain managed by Cloudflare
- Homebrew installed
- n8n running locally (e.g. `http://localhost:5678`)

---

# 1. Install cloudflared

```bash
brew install cloudflared
```

Verify the installation:

```bash
cloudflared --version
```

---

# 2. Authenticate with Cloudflare

Login to your Cloudflare account:

```bash
cloudflared tunnel login
```

This will:

- Open your browser
- Ask you to log into Cloudflare
- Ask you to select your domain
- Download an origin certificate (`cert.pem`) into:

```text
~/.cloudflared/cert.pem
```

This certificate is only required for creating and managing tunnels.

---

# 3. Create a Named Tunnel

Choose any name you like.

Example:

```bash
cloudflared tunnel create my-n8n
```

Example output:

```
Created tunnel my-n8n with id:
4fd3b71d-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Cloudflare also creates:

```
~/.cloudflared/<TUNNEL_UUID>.json
```

This JSON file contains the tunnel credentials.

---

# 4. Create a DNS Route

Suppose you want:

```
n8n.example.com
```

Create the DNS record:

```bash
cloudflared tunnel route dns my-n8n n8n.example.com
```

Cloudflare creates the required CNAME automatically.

Verify:

```bash
dig n8n.example.com
```

---

# 5. Create the Tunnel Configuration

Create:

```text
~/.cloudflared/config.yml
```

Example:

```yaml
tunnel: 4fd3b71d-xxxx-xxxx-xxxx-xxxxxxxxxxxx

credentials-file: /Users/YOUR_USERNAME/.cloudflared/4fd3b71d-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: n8n.example.com
    service: http://localhost:5678

  - service: http_status:404
```

Replace:

- `YOUR_USERNAME`
- Tunnel UUID
- Domain name

---

# 6. Run the Tunnel

```bash
cloudflared tunnel run my-n8n
```

Or explicitly specify the configuration:

```bash
cloudflared --config ~/.cloudflared/config.yml tunnel run
```

Your local n8n instance is now accessible at:

```
https://n8n.example.com
```

---

# 7. Configure n8n

Configure n8n to use the public URL.

Example environment variables:

```env
N8N_HOST=n8n.example.com
N8N_PROTOCOL=https
N8N_PORT=5678
WEBHOOK_URL=https://n8n.example.com/
```

Restart n8n after changing the configuration.

---

# Managing Multiple Applications

A single tunnel can expose multiple local services.

Example configuration:

```yaml
tunnel: 4fd3b71d-xxxx-xxxx-xxxx-xxxxxxxxxxxx

credentials-file: /Users/YOUR_USERNAME/.cloudflared/4fd3b71d-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: n8n.example.com
    service: http://localhost:5678

  - hostname: openclaw.example.com
    service: http://localhost:18789

  - hostname: grafana.example.com
    service: http://localhost:3000

  - service: http_status:404
```

Create the DNS entries:

```bash
cloudflared tunnel route dns my-n8n n8n.example.com
cloudflared tunnel route dns my-n8n openclaw.example.com
cloudflared tunnel route dns my-n8n grafana.example.com
```

Restart the tunnel after editing `config.yml`.

---

# Useful Commands

List tunnels:

```bash
cloudflared tunnel list
```

View tunnel information:

```bash
cloudflared tunnel info my-n8n
```

List DNS routes:

```bash
cloudflared tunnel route ip show
```

Delete a DNS route:

```bash
cloudflared tunnel route dns delete my-n8n n8n.example.com
```

Delete a tunnel:

```bash
cloudflared tunnel delete my-n8n
```

---

# Running as a Background Service

Install as a macOS Launch Agent:

```bash
cloudflared service install
```

Start:

```bash
brew services start cloudflared
```

Stop:

```bash
brew services stop cloudflared
```

Alternatively, for development, simply run:

```bash
cloudflared tunnel run my-n8n
```

---

# Using Docker Later

If n8n and cloudflared run in the same Docker Compose network, replace:

```yaml
service: http://localhost:5678
```

with:

```yaml
service: http://n8n:5678
```

where `n8n` is the Docker Compose service name.

This allows the same tunnel to work entirely inside Docker without exposing localhost.

---

# File Locations

```
~/.cloudflared/
├── cert.pem
├── config.yml
└── <TUNNEL_UUID>.json
```

- **cert.pem** — Used to authenticate with Cloudflare and manage tunnels.
- **config.yml** — Tunnel configuration and ingress rules.
- **<TUNNEL_UUID>.json** — Tunnel credentials required to run the tunnel.
