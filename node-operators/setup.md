# Node Setup Guide

This guide will help you set up and run an Enter L2 node to participate in the network, verify transactions, and provide RPC services.

## Overview

Running an Enter L2 node allows you to:
- **Independently verify** all network transactions and state transitions
- **Provide RPC services** for applications and users
- **Contribute to decentralization** by participating in the P2P network
- **Access historical data** for analytics and block exploration
- **Monitor network health** and detect potential issues

## System Requirements

### Minimum Requirements (Light Node)
- **CPU**: 2 cores (2.0 GHz+)
- **RAM**: 4 GB
- **Storage**: 50 GB SSD
- **Network**: 10 Mbps internet connection
- **OS**: Linux (Ubuntu 20.04+), macOS, or Windows

### Recommended Requirements (Full Node)
- **CPU**: 4 cores (3.0 GHz+)
- **RAM**: 8 GB
- **Storage**: 500 GB NVMe SSD
- **Network**: 100 Mbps internet connection
- **OS**: Linux (Ubuntu 22.04 LTS recommended)

### High Performance (RPC/Archive Node)
- **CPU**: 8+ cores (3.5 GHz+)
- **RAM**: 16+ GB
- **Storage**: 1+ TB NVMe SSD
- **Network**: 1+ Gbps internet connection
- **OS**: Linux with optimized kernel

## Installation Methods

### Method 1: Binary Installation (Recommended)

#### Linux/macOS
```bash
# Download latest release
curl -L https://github.com/enter-l2/node/releases/latest/download/enter-l2-node-linux-amd64.tar.gz | tar xz

# Move to system path
sudo mv enter-l2-node /usr/local/bin/

# Verify installation
enter-l2-node --version
```

#### Windows
```powershell
# Download from GitHub releases
Invoke-WebRequest -Uri "https://github.com/enter-l2/node/releases/latest/download/enter-l2-node-windows-amd64.zip" -OutFile "enter-l2-node.zip"

# Extract and add to PATH
Expand-Archive -Path "enter-l2-node.zip" -DestinationPath "C:\enter-l2\"
$env:PATH += ";C:\enter-l2\"
```

### Method 2: Docker Installation

```bash
# Pull the latest image
docker pull enterl2/node:latest

# Run with Docker
docker run -d \
  --name enter-l2-node \
  -p 8545:8545 \
  -p 8546:8546 \
  -p 30303:30303 \
  -v ./data:/data \
  enterl2/node:latest
```

### Method 3: Build from Source

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Clone repository
git clone https://github.com/enter-l2/node.git
cd node

# Build release binary
cargo build --release

# Install binary
sudo cp target/release/enter-l2-node /usr/local/bin/
```

## Initial Configuration

### Create Configuration File

Create `config.toml`:

```toml
[node]
mode = "full"
data_dir = "/var/lib/enter-l2"

[rpc]
enabled = true
host = "0.0.0.0"
port = 8545
cors_origins = ["*"]
max_connections = 100

[websocket]
enabled = true
host = "0.0.0.0"
port = 8546
max_connections = 100

[p2p]
enabled = true
port = 30303
max_peers = 50
discovery_enabled = true
bootstrap_nodes = [
  "/ip4/18.144.24.166/tcp/30303/p2p/12D3KooWBhzAHuBdmJWWvKZZzKKWJKqhQhGhKKqhQhGhKKqhQhGh",
  "/ip4/52.53.195.89/tcp/30303/p2p/12D3KooWCdMKqDBusQhGhKKqhQhGhKKqhQhGhKKqhQhGhKKqhQhGh"
]

[sequencer]
url = "https://sequencer.enterl2.com"
timeout = "30s"

[l1]
rpc_url = "https://mainnet.infura.io/v3/YOUR_INFURA_KEY"
contracts = { 
  state_manager = "0x1234567890123456789012345678901234567890",
  bridge = "0x2345678901234567890123456789012345678901"
}

[database]
url = "postgres://enter_l2:password@localhost/enter_l2"

[metrics]
enabled = true
port = 9090

[logging]
level = "info"
format = "json"
```

### Environment Variables

Set required environment variables:

```bash
# Create environment file
cat > /etc/enter-l2/env << EOF
ENTER_L2_L1_RPC_URL=https://mainnet.infura.io/v3/YOUR_INFURA_KEY
ENTER_L2_DATABASE_URL=postgres://enter_l2:password@localhost/enter_l2
ENTER_L2_LOG_LEVEL=info
EOF
```

## Database Setup

### PostgreSQL Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE enter_l2;
CREATE USER enter_l2 WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE enter_l2 TO enter_l2;
\q
EOF
```

### Database Configuration

```bash
# Update PostgreSQL configuration
sudo nano /etc/postgresql/14/main/postgresql.conf

# Recommended settings for Enter L2
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
```

## Running the Node

### Systemd Service (Linux)

Create service file:

```bash
sudo nano /etc/systemd/system/enter-l2-node.service
```

```ini
[Unit]
Description=Enter L2 Node
After=network.target postgresql.service
Wants=postgresql.service

[Service]
Type=simple
User=enter-l2
Group=enter-l2
WorkingDirectory=/var/lib/enter-l2
ExecStart=/usr/local/bin/enter-l2-node --config /etc/enter-l2/config.toml
Restart=always
RestartSec=5
EnvironmentFile=/etc/enter-l2/env

# Security settings
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/enter-l2 /var/log/enter-l2

[Install]
WantedBy=multi-user.target
```

Start the service:

```bash
# Create user and directories
sudo useradd -r -s /bin/false enter-l2
sudo mkdir -p /var/lib/enter-l2 /var/log/enter-l2 /etc/enter-l2
sudo chown enter-l2:enter-l2 /var/lib/enter-l2 /var/log/enter-l2

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable enter-l2-node
sudo systemctl start enter-l2-node

# Check status
sudo systemctl status enter-l2-node
```

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  enter-l2-node:
    image: enterl2/node:latest
    container_name: enter-l2-node
    restart: unless-stopped
    ports:
      - "8545:8545"
      - "8546:8546"
      - "30303:30303"
      - "9090:9090"
    volumes:
      - ./data:/data
      - ./config.toml:/config.toml
    environment:
      - ENTER_L2_L1_RPC_URL=${L1_RPC_URL}
      - ENTER_L2_DATABASE_URL=postgres://enter_l2:password@postgres:5432/enter_l2
    depends_on:
      - postgres
    command: ["--config", "/config.toml"]

  postgres:
    image: postgres:15
    container_name: enter-l2-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: enter_l2
      POSTGRES_USER: enter_l2
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  prometheus:
    image: prom/prometheus:latest
    container_name: enter-l2-prometheus
    restart: unless-stopped
    ports:
      - "9091:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

volumes:
  postgres_data:
  prometheus_data:
```

Start with Docker Compose:

```bash
# Create environment file
echo "L1_RPC_URL=https://mainnet.infura.io/v3/YOUR_KEY" > .env

# Start services
docker-compose up -d

# View logs
docker-compose logs -f enter-l2-node
```

## Node Modes

### Full Node (Default)
```bash
enter-l2-node --mode full --config config.toml
```
- Complete transaction history
- Full state verification
- All RPC methods available
- Participates in P2P network

### Light Node
```bash
enter-l2-node --mode light --config config.toml
```
- Minimal storage requirements
- Trusts sequencer for most data
- Basic RPC functionality
- Suitable for wallets

### Archive Node
```bash
enter-l2-node --mode archive --config config.toml
```
- Complete historical data
- All state transitions
- Enhanced indexing
- Suitable for explorers

### RPC Node
```bash
enter-l2-node --mode rpc --max-connections 1000 --config config.toml
```
- Optimized for serving requests
- High connection limits
- Enhanced caching
- Public RPC services

## Verification

### Check Node Status

```bash
# Check if node is running
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"enterl2_getNodeStatus","params":[],"id":1}' \
  http://localhost:8545

# Check latest block
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://localhost:8545

# Check peer count
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' \
  http://localhost:8545
```

### Monitor Sync Progress

```bash
# Watch sync status
watch -n 5 'curl -s -X POST -H "Content-Type: application/json" \
  --data "{\"jsonrpc\":\"2.0\",\"method\":\"enterl2_getNodeStatus\",\"params\":[],\"id\":1}" \
  http://localhost:8545 | jq ".result"'
```

### Test RPC Endpoints

```bash
# Test balance query
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x742d35Cc6634C0532925a3b8D4C9db96c4b4d8b","latest"],"id":1}' \
  http://localhost:8545

# Test transaction query
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0x..."],"id":1}' \
  http://localhost:8545
```

## Monitoring Setup

### Prometheus Configuration

Create `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'enter-l2-node'
    static_configs:
      - targets: ['localhost:9090']
    scrape_interval: 10s
    metrics_path: /metrics

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### Grafana Dashboard

Import the Enter L2 dashboard:

1. Open Grafana (http://localhost:3000)
2. Go to Dashboards → Import
3. Use dashboard ID: `15023` (Enter L2 Node Dashboard)
4. Configure Prometheus data source

### Log Monitoring

```bash
# View real-time logs
sudo journalctl -u enter-l2-node -f

# Search for errors
sudo journalctl -u enter-l2-node --since "1 hour ago" | grep ERROR

# Export logs
sudo journalctl -u enter-l2-node --since "2024-01-01" --until "2024-01-02" > node-logs.txt
```

## Maintenance

### Regular Updates

```bash
# Check for updates
enter-l2-node --version

# Download latest version
curl -L https://github.com/enter-l2/node/releases/latest/download/enter-l2-node-linux-amd64.tar.gz | tar xz

# Stop service
sudo systemctl stop enter-l2-node

# Replace binary
sudo mv enter-l2-node /usr/local/bin/

# Start service
sudo systemctl start enter-l2-node
```

### Database Maintenance

```bash
# Vacuum database
sudo -u postgres psql enter_l2 -c "VACUUM ANALYZE;"

# Check database size
sudo -u postgres psql enter_l2 -c "SELECT pg_size_pretty(pg_database_size('enter_l2'));"

# Backup database
sudo -u postgres pg_dump enter_l2 > backup-$(date +%Y%m%d).sql
```

### Performance Tuning

```bash
# Monitor resource usage
htop
iotop
nethogs

# Check disk usage
df -h
du -sh /var/lib/enter-l2/*

# Monitor network connections
ss -tuln | grep :8545
```

## Troubleshooting

### Common Issues

**Node won't start:**
```bash
# Check logs
sudo journalctl -u enter-l2-node -n 50

# Verify configuration
enter-l2-node --config config.toml --check-config

# Test database connection
psql -h localhost -U enter_l2 -d enter_l2 -c "SELECT 1;"
```

**Sync issues:**
```bash
# Reset to specific block
enter-l2-node --reset-to-block 12345

# Clear peer cache
rm -rf /var/lib/enter-l2/p2p/

# Check network connectivity
curl -I https://sequencer.enterl2.com
```

**High resource usage:**
```bash
# Reduce cache size
echo "cache_size = 512MB" >> config.toml

# Limit connections
echo "max_connections = 50" >> config.toml

# Enable pruning
echo "pruning_enabled = true" >> config.toml
```

## Security

### Firewall Configuration

```bash
# Allow required ports
sudo ufw allow 8545/tcp  # RPC
sudo ufw allow 8546/tcp  # WebSocket
sudo ufw allow 30303/tcp # P2P
sudo ufw allow 30303/udp # P2P Discovery

# Restrict RPC access (optional)
sudo ufw allow from 10.0.0.0/8 to any port 8545
```

### SSL/TLS Setup

```bash
# Install certbot
sudo apt install certbot

# Get certificate
sudo certbot certonly --standalone -d node.yourdomain.com

# Configure nginx proxy
sudo nano /etc/nginx/sites-available/enter-l2-node
```

Nginx configuration:
```nginx
server {
    listen 443 ssl;
    server_name node.yourdomain.com;
    
    ssl_certificate /etc/letsencrypt/live/node.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/node.yourdomain.com/privkey.pem;
    
    location / {
        proxy_pass http://localhost:8545;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /ws {
        proxy_pass http://localhost:8546;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## Next Steps

- **[Configuration Guide](configuration.md)** - Detailed configuration options
- **[Monitoring Guide](monitoring.md)** - Advanced monitoring setup
- **[Troubleshooting](troubleshooting.md)** - Common issues and solutions
- **[API Reference](../api/reference.md)** - RPC API documentation
