# n8n with PostgreSQL on Docker

A complete Docker Compose setup for running [n8n](https://n8n.io/) (workflow automation tool) with a PostgreSQL database. All data is persisted to disk using Docker volumes, ensuring your workflows, credentials, and settings are preserved across container restarts.

## Features

- 🐳 **Docker Compose setup** - Easy deployment with a single command
- 🔒 **Secure PostgreSQL database** - All n8n data stored in a dedicated PostgreSQL database
- 💾 **Persistent storage** - Data and workflows saved outside containers
- 🔗 **Internal networking** - Database communication happens within Docker network
- 🏥 **Health checks** - Automatic container health monitoring
- 📦 **Synology NAS ready** - Detailed instructions for Synology Container Manager

## Prerequisites

- Docker Engine 20.10 or later
- Docker Compose V2 or later
- For Synology NAS: DSM 7.0 or later with Container Manager installed

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/mrswadge/docker-n8n-postgres.git
cd docker-n8n-postgres
```

### 2. Configure Environment Variables

Copy the example environment file and edit it with your settings:

```bash
cp .env.example .env
```

Edit `.env` and update at minimum:
- `POSTGRES_PASSWORD` - Set a strong password for the database
- `N8N_ENCRYPTION_KEY` - Set a random encryption key (used for encrypting credentials)

**Important:** Keep your `.env` file secure and never commit it to version control!

To generate a secure encryption key, you can use:

```bash
# Linux/Mac
openssl rand -base64 32

# Or use this online (for convenience)
# Visit: https://generate-random.org/encryption-key-generator
```

### 3. Start the Services

```bash
docker compose up -d
```

This will:
- Pull the required Docker images (n8n and PostgreSQL)
- Create a dedicated Docker network for internal communication
- Start PostgreSQL and wait for it to be healthy
- Start n8n and connect it to PostgreSQL
- Create persistent volumes for data storage

### 4. Access n8n

Open your browser and navigate to:
- **Local access:** http://localhost:5678
- **Remote access:** http://YOUR_SERVER_IP:5678

On first access, you'll be prompted to create an admin account.

## Synology NAS Setup

### Prerequisites on Synology

1. **Install Container Manager**
   - Open **Package Center**
   - Search for "Container Manager"
   - Click **Install**
   - Wait for installation to complete

2. **Enable SSH (optional, for command-line setup)**
   - Go to **Control Panel** > **Terminal & SNMP**
   - Enable **SSH service**
   - Note: You can also use File Station to upload files

### Method 1: Using Container Manager GUI (Recommended)

#### Step 1: Upload Files to Synology

1. Open **File Station**
2. Create a new folder, e.g., `/docker/n8n` in your main storage volume
3. Upload these files to the folder:
   - `docker-compose.yml`
   - `.env.example`
4. Rename `.env.example` to `.env`
5. Right-click on `.env` and select **Edit** to modify the settings:
   - Set `POSTGRES_PASSWORD` to a strong password
   - Set `N8N_ENCRYPTION_KEY` to a random string (32+ characters)
   - Update `N8N_HOST` to your NAS IP address or domain name
   - Update `WEBHOOK_URL` if you'll use webhooks (e.g., `http://192.168.1.100:5678/`)

#### Step 2: Import to Container Manager

1. Open **Container Manager**
2. Go to **Project** tab
3. Click **Create**
4. Set **Project Name**: `n8n-postgres`
5. Set **Path**: Navigate to `/docker/n8n` (or your chosen folder)
6. Set **Source**: `docker-compose.yml`
7. Click **Next**
8. Review the configuration and click **Done**

The containers will start automatically. Container Manager will:
- Download the required images
- Create the containers
- Create volumes in `/volume1/@docker/volumes/`
- Start the services

#### Step 3: Access n8n

1. In Container Manager, you should see two containers running:
   - `n8n`
   - `n8n-postgres`

2. Access n8n:
   - Click on the `n8n` container
   - You'll see the port mapping (default: 5678)
   - Open browser to: `http://YOUR_NAS_IP:5678`

3. Create your admin account when prompted

### Method 2: Using SSH/Command Line

If you prefer command-line setup:

1. SSH into your Synology NAS:
   ```bash
   ssh admin@YOUR_NAS_IP
   ```

2. Navigate to your docker directory:
   ```bash
   cd /volume1/docker
   mkdir n8n
   cd n8n
   ```

3. Create/upload the required files (`docker-compose.yml`, `.env`)

4. Start the containers:
   ```bash
   sudo docker compose up -d
   ```

### Synology-Specific Notes

**Volume Storage:**
- Docker volumes are stored in `/volume1/@docker/volumes/`
- You can find your n8n data at: `/volume1/@docker/volumes/n8n-postgres_n8n_data/`
- PostgreSQL data at: `/volume1/@docker/volumes/n8n-postgres_postgres_data/`

**Port Configuration:**
- Default port is 5678
- If port 5678 is already in use, change `N8N_PORT` in your `.env` file
- Update the port mapping in docker-compose.yml if needed

**Accessing Externally:**
- To access n8n from outside your network, set up port forwarding on your router
- Forward external port (e.g., 5678) to your NAS IP on port 5678
- Consider using a reverse proxy with SSL for production use

**Auto-start:**
- Container Manager projects start automatically on NAS boot
- The `restart: unless-stopped` policy ensures containers restart after crashes

**Updates:**
- To update n8n or PostgreSQL, stop the project in Container Manager
- Edit the docker-compose.yml to change image versions if needed
- Start the project again - Container Manager will pull new images

## Configuration

### Environment Variables

All configuration is done through the `.env` file:

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `POSTGRES_USER` | PostgreSQL username | `n8n` | No |
| `POSTGRES_PASSWORD` | PostgreSQL password | - | **Yes** |
| `POSTGRES_DB` | PostgreSQL database name | `n8n` | No |
| `N8N_PORT` | Port for accessing n8n | `5678` | No |
| `N8N_HOST` | Hostname for n8n | `localhost` | No |
| `N8N_PROTOCOL` | Protocol (http/https) | `http` | No |
| `N8N_ENCRYPTION_KEY` | Key for encrypting credentials | - | **Yes** |
| `WEBHOOK_URL` | Base URL for webhooks | `http://localhost:5678/` | No |
| `TIMEZONE` | Timezone for n8n | `UTC` | No |

### Volume Management

This setup uses Docker named volumes for data persistence:

- **postgres_data**: Stores PostgreSQL database files
- **n8n_data**: Stores n8n workflows, credentials, and settings

To backup your data:

```bash
# Stop containers
docker compose down

# Backup volumes (example using tar)
docker run --rm -v n8n-postgres_n8n_data:/data -v $(pwd):/backup ubuntu tar czf /backup/n8n-backup.tar.gz /data
docker run --rm -v n8n-postgres_postgres_data:/data -v $(pwd):/backup ubuntu tar czf /backup/postgres-backup.tar.gz /data

# Start containers
docker compose up -d
```

## Common Operations

### View Logs

```bash
# All services
docker compose logs -f

# Just n8n
docker compose logs -f n8n

# Just PostgreSQL
docker compose logs -f postgres
```

### Stop Services

```bash
docker compose down
```

### Restart Services

```bash
docker compose restart
```

### Update n8n

```bash
docker compose pull
docker compose up -d
```

### Access PostgreSQL Database

```bash
docker compose exec postgres psql -U n8n -d n8n
```

## Troubleshooting

### n8n Won't Start

1. **Check if PostgreSQL is healthy:**
   ```bash
   docker compose ps
   ```
   The postgres container should show as "healthy"

2. **Check logs:**
   ```bash
   docker compose logs postgres
   docker compose logs n8n
   ```

3. **Verify environment variables:**
   - Ensure `.env` file exists and has the required variables
   - Check that `POSTGRES_PASSWORD` and `N8N_ENCRYPTION_KEY` are set

### Can't Access n8n UI

1. **Verify containers are running:**
   ```bash
   docker compose ps
   ```

2. **Check port binding:**
   ```bash
   docker compose ps n8n
   ```
   Should show: `0.0.0.0:5678->5678/tcp`

3. **Test local access:**
   ```bash
   curl http://localhost:5678
   ```

4. **Firewall issues:**
   - On Synology: Check if port 5678 is allowed in firewall rules
   - On router: Check if port forwarding is configured correctly

### Database Connection Errors

1. **Wait for PostgreSQL to be ready:**
   The healthcheck ensures n8n waits for PostgreSQL, but first startup can take 30-60 seconds

2. **Check database credentials:**
   - Ensure `POSTGRES_PASSWORD` in `.env` matches between services
   - Verify database name and user are correct

3. **Reset database (⚠️ destroys all data):**
   ```bash
   docker compose down -v
   docker compose up -d
   ```

### Synology-Specific Issues

**Container Manager shows error:**
- Check logs in Container Manager UI (click container > Details > Logs)
- Verify the `.env` file is in the same directory as `docker-compose.yml`
- Ensure no port conflicts with other services

**Can't edit .env file:**
- Use Text Editor app from Package Center
- Or use SSH with nano/vi: `sudo nano .env`

**Volumes not accessible:**
- Volumes are in `/volume1/@docker/volumes/`
- May need sudo to access: `sudo ls /volume1/@docker/volumes/`

## Security Recommendations

1. **Use strong passwords:**
   - Generate random passwords for `POSTGRES_PASSWORD`
   - Use a secure random string for `N8N_ENCRYPTION_KEY`

2. **Enable HTTPS:**
   - Use a reverse proxy (nginx, Traefik, Caddy)
   - Obtain SSL certificate (Let's Encrypt)
   - Update `N8N_PROTOCOL` to `https`

3. **Restrict access:**
   - Use firewall rules to limit access to port 5678
   - Consider VPN for remote access
   - Enable n8n's built-in user management

4. **Regular backups:**
   - Backup Docker volumes regularly
   - Test restore procedures
   - Store backups securely off-site

5. **Keep updated:**
   - Regularly update n8n and PostgreSQL images
   - Monitor n8n security advisories

## Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community Forum](https://community.n8n.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Synology Docker Guide](https://www.synology.com/en-us/dsm/packages/ContainerManager)

## License

This project is released under the MIT License. See individual component licenses:
- n8n: [Fair-code License](https://docs.n8n.io/reference/license/)
- PostgreSQL: [PostgreSQL License](https://www.postgresql.org/about/licence/)

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Support

If you encounter issues:
1. Check the Troubleshooting section above
2. Review logs with `docker compose logs`
3. Search existing issues on GitHub
4. Create a new issue with detailed information
