# Cloudflare Tunnel

This repository contains the configuration for setting up Cloudflare Tunnel to securely expose local services to the internet without opening firewall ports.

## What is Cloudflare Tunnel?

Cloudflare Tunnel creates a secure, outbound-only connection from your local machine to Cloudflare's network. This allows you to:

- Expose local web services to the internet securely
- Connect domains or subdomains without exposing your home IP address
- Bypass NAT and firewall restrictions
- Protect services with Cloudflare's security features

## Prerequisites

- A Cloudflare account (free tier works)
- A domain added to your Cloudflare account
- Docker and Docker Compose installed on your local machine

## Setup Instructions

### Step 1: Create a Cloudflare Tunnel

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Select your domain from the list
3. Navigate to **Zero Trust** > **Access** > **Tunnels**
4. Click **Create a tunnel**
5. Choose **Cloudflared** as the connector type
6. Give your tunnel a name (e.g., "home-tunnel")
7. Click **Save tunnel**

### Step 2: Get Your Tunnel Token

After creating the tunnel:

1. Cloudflare will display your tunnel token
2. Copy the token (it's a long string that looks like `eyJhIjoiX...`)
3. Keep this token secure - you'll need it for the next step

### Step 3: Configure Docker Compose

1. Open `docker-compose.yml` in this repository
2. Replace `<your_token>` with your actual tunnel token:

```yaml
command: >
  tunnel --no-autoupdate run --token eyJhIjoiX...your_actual_token_here
```

### Step 4: Configure Public Hostname (Domain/Subdomain)

Back in the Cloudflare Dashboard:

1. In your tunnel configuration, go to the **Public Hostname** tab
2. Click **Add a public hostname**
3. Configure the hostname:
   - **Subdomain**: Enter your subdomain (e.g., `app`, `api`, `www`)
   - **Domain**: Select your domain from the dropdown
   - **Path**: Leave empty unless you need path-based routing
   - **Type**: Select `HTTP`
   - **URL**: Enter your local service URL (e.g., `http://host.docker.internal:8069` for Odoo)

4. Click **Save hostname**

### Step 5: Start the Tunnel

Run the following command in the repository directory:

```bash
docker-compose up -d
```

### Step 6: Verify the Tunnel

Check if the tunnel is running:

```bash
docker-compose ps
docker-compose logs -f cloudflare-tunnel
```

You should see logs indicating the tunnel has connected successfully.

## Connecting Local Services

The `docker-compose.yml` is configured with `host.docker.internal:host-gateway`, which allows the container to access services running on your host machine.

### Example: Exposing a Local Web Server

If you have a service running on `localhost:8069`:

1. In Cloudflare Dashboard, add a public hostname as described in Step 4
2. Set the URL to: `http://host.docker.internal:8069`
3. Your service will be accessible at `https://your-subdomain.your-domain.com`

### Multiple Subdomains

You can create multiple public hostnames for the same tunnel:

- `app.example.com` → `http://host.docker.internal:8069`
- `api.example.com` → `http://host.docker.internal:3000`
- `db.example.com` → `http://host.docker.internal:5432`

## Managing the Tunnel

### Stop the tunnel
```bash
docker-compose down
```

### Restart the tunnel
```bash
docker-compose restart
```

### View logs
```bash
docker-compose logs -f cloudflare-tunnel
```

### Update the tunnel
```bash
docker-compose pull
docker-compose up -d
```

## Security Notes

- Your tunnel token is sensitive - never commit it to public repositories
- Consider using environment variables or Docker secrets for the token
- Enable Cloudflare Access policies for additional security
- Review Cloudflare's audit logs regularly

## Troubleshooting

### Tunnel won't start
- Verify your token is correct
- Check Docker logs: `docker-compose logs cloudflare-tunnel`
- Ensure Docker can access the internet

### Can't access local service
- Verify the service is running on your local machine
- Check that you're using `host.docker.internal` instead of `localhost`
- Confirm the port number is correct

### 502 Bad Gateway
- The local service might not be running
- Check the URL configuration in Cloudflare Dashboard
- Verify firewall settings on your local machine

## Useful Links

- [Cloudflare Tunnel Documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [Cloudflared GitHub Repository](https://github.com/cloudflare/cloudflared)
- [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com/)

