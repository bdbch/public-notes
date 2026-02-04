# Guide: Self-Hosted AT Protocol PDS

This guide covers the installation of the official Bluesky PDS (Personal Data Server) distribution on a Linux server. It uses the `pdsadmin` tool which manages the underlying Docker containers and Caddy reverse proxy.

## Prerequisites

- **Domain Name:** You need full control over a domain (e.g., `example.com`).
    
- **Linux VPS:** Ubuntu 22.04 LTS (recommended) or Debian 11/12.
    
    - **CPU/RAM:** 2 vCPU / 4GB RAM is recommended for stability, though it can run on less.
        
    - **Storage:** 40GB+ SSD (storage grows with usage).
        
- **Root Access:** You must have `sudo` privileges.
    

---

## 1. DNS Configuration

You must configure your DNS records **before** installing the software so SSL certificates can generate correctly during setup.

Go to your domain registrar (Namecheap, Cloudflare, Route53, etc.) and set the following records. Replace `1.2.3.4` with your server's public IP address.

|**Type**|**Name**|**Value**|**TTL**|
|---|---|---|---|
|**A**|`pds`|`1.2.3.4`|Auto / 300|
|**A**|`*.pds`|`1.2.3.4`|Auto / 300|

- **Note:** This sets up your PDS at `pds.example.com`.
    
- The wildcard (`*.pds`) is required because the PDS assigns a subdomain to every user DID (Decentralized Identifier) and handle hosted on the server.
    

---

## 2. Server Preparation

Connect to your server via SSH.

### Update System

Ensure the system is fresh to avoid dependency conflicts.

Bash

```
sudo apt update && sudo apt upgrade -y
```

### Configure Firewall

The PDS requires ports 80 and 443 to be open for web traffic and SSL verification. Port 3000 is used internally by the PDS but does not need to be exposed publicly unless you are debugging behind a custom proxy.

Using `ufw` (Uncomplicated Firewall):

Bash

```
# Deny incoming by default
sudo ufw default deny incoming
# Allow outgoing by default
sudo ufw default allow outgoing
# Allow SSH (CRITICAL: Do not skip this or you will lock yourself out)
sudo ufw allow ssh
# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
# Enable firewall
sudo ufw enable
```

---

## 3. Installation

The official installer script handles Docker installation, storage volume creation, and service configuration.

### Run the Installer

Bash

```
curl https://raw.githubusercontent.com/bluesky-social/pds/main/installer.sh | bash
```

### Installation Prompts

1. **Hostname:** Enter the full domain you configured in step 1 (e.g., `pds.example.com`).
    
2. **Email:** Enter a valid email address for Let's Encrypt SSL expiry notifications.
    

The script will download the Docker images and start the service. This may take a few minutes.

### Verify Status

Once the script finishes, check that the services are healthy.

Bash

```
sudo systemctl status pds
```

You can also run a health check via `curl`:

Bash

```
curl https://pds.example.com/xrpc/_health
```

**Expected Output:** `{"version":"0.x.x"}`

---

## 4. Account Creation

By default, a new PDS does not allow public signups. You must generate an invite code and use the CLI to create your account.

### Generate Invite Code

Bash

```
sudo pdsadmin account create-invite-code
```

- Copy the code generated (e.g., `12345-abcde`).
    

### Create Account

Replace the values below with your desired details.

Bash

```
sudo pdsadmin account create \
  --email "your-email@example.com" \
  --handle "yourhandle.pds.example.com" \
  --password "YourStrongPassword!" \
  --invite-code "12345-abcde"
```

- **Note on Handles:** Your handle will default to `username.pds.example.com`. You can change this to your root domain (e.g., `username.com`) later via DNS TXT records if you own that domain.
    

---

## 5. Federation (Going Public)

At this stage, your PDS exists but the rest of the network (the Relay) may not know about it.

### Sandbox vs. Production

Ensure you are connecting to the production network if you want to interact with users on `bsky.app`. The default installer configures for production.

### Request a Crawl

For your data to appear in the global feed, the Relay must crawl your PDS. This often happens automatically when you federate, but you can force a check or debug connectivity issues here:

1. Go to a WebSocket tester (or use a tool like `wscat`).
    
2. Connect to your PDS: `wss://pds.example.com/xrpc/com.atproto.sync.subscribeRepos`
    
3. If this connection opens successfully, your PDS is ready to emit events to the network.
    

---

## 6. Maintenance & Backups

The PDS installation includes the `pdsadmin` utility to simplify maintenance.

### Updating

Run this periodically to get the latest protocol updates.

Bash

```
sudo pdsadmin update
```

### Backups

The critical data is stored in `/pds`.

- **Database:** SQLite or Postgres (embedded).
    
- **Blob Storage:** User images and avatars.
    

**To Create a Backup:**

The tool does not have a built-in "backup to S3" command yet, so you must back up the directory safely.

Bash

```
# Stop services to ensure database consistency
sudo systemctl stop pds

# Create a tarball
sudo tar -czvf pds-backup-$(date +%F).tar.gz /pds

# Restart services
sudo systemctl start pds
```

_Download this tarball to a secure location off-server immediately._

---

## Troubleshooting

- **Domain Verification Failed:** Ensure your DNS has propagated. It can take up to 24 hours, though usually takes minutes.
    
- **Port Conflicts:** Ensure no other web server (Apache/Nginx) is binding port 80/443. The PDS uses Caddy internally and expects those ports free.
    
- **Permission Denied:** Always run `pdsadmin` with `sudo`.
    

Would you like me to detail how to verify your PDS implementation against the AT Protocol specs using the compliance test suite?