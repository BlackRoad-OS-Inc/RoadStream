# INTEGRATION — RoadStream

> How this fork connects to the rest of BlackRoad OS

## Node Assignment

| Property | Value |
|----------|-------|
| **Primary Node** | Lucidia (.38) |
| **Fork Of** | Jellyfin |
| **RoundTrip Agent** | RoadStream Agent |
| **NLP Intents** | 'play media' / 'stream' |
| **NATS Subject** | `blackroad.RoadStream.>` |
| **GuardRail Monitor** | `https://guard.blackroad.io/status/RoadStream` |

## Deployment

Deploy via blackroad-operator:

```bash
# From blackroad-operator
cd ~/blackroad-operator
./scripts/deploy/deploy-RoadStream.sh

# Or via fleet coordinator
./fleet-coordinator.sh deploy RoadStream

# Manual deploy to Lucidia (.38)
ssh blackroad@$(echo "Lucidia (.38)" | grep -oP '[0-9.]+' || echo "Lucidia (.38)") \
  "cd /opt/blackroad/RoadStream && git pull && sudo systemctl restart RoadStream"
```

## Systemd Service

```ini
[Unit]
Description=BlackRoad RoadStream (Jellyfin fork)
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=blackroad
WorkingDirectory=/opt/blackroad/RoadStream
ExecStart=/opt/blackroad/RoadStream/start.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## NATS Integration (CarPool)

```bash
# Subscribe to RoadStream events
nats sub "blackroad.RoadStream.>" --server nats://192.168.4.101:4222

# Publish status
nats pub "blackroad.RoadStream.status" '{"node":"Lucidia (.38)","status":"running"}' \
  --server nats://192.168.4.101:4222
```

## RoundTrip Agent

The **RoadStream Agent** manages this service via RoundTrip:

```bash
# Check agent status
curl -s https://roundtrip.blackroad.io/api/agents | jq '.[] | select(.name=="RoadStream Agent")'

# Send command to agent
curl -X POST https://roundtrip.blackroad.io/api/chat \
  -H 'Content-Type: application/json' \
  -d '{"agent":"RoadStream Agent","message":"status","channel":"fleet"}'
```

## GuardRail Monitoring

Add to Uptime Kuma (Alice :3001):

| Check | URL/Command | Interval |
|-------|------------|----------|
| HTTP Health | `http://Lucidia (.38):PORT/health` | 30s |
| Process | `systemctl is-active RoadStream` | 60s |
| NATS Heartbeat | `blackroad.RoadStream.heartbeat` | 60s |

## Memory System Integration

```bash
# Log actions
~/blackroad-operator/scripts/memory/memory-system.sh log deploy RoadStream "Deployed to Lucidia (.38)"

# Add solutions to Codex
~/blackroad-operator/scripts/memory/memory-codex.sh add-solution "RoadStream" "How to restart" \
  "sudo systemctl restart RoadStream"

# Broadcast learnings
~/blackroad-operator/scripts/memory/memory-til-broadcast.sh broadcast "RoadStream" "Config change: ..."
```

## Related Components

| Component | Role | Connection |
|-----------|------|-----------|
| **TollBooth** (WireGuard) | VPN mesh | All traffic between nodes |
| **CarPool** (NATS) | Messaging | Event pub/sub on `blackroad.RoadStream.>` |
| **GuardRail** (Uptime Kuma) | Monitoring | Health checks every 30s |
| **RoadMem** (Mem0) | Memory | Persistent agent state |
| **OneWay** (Caddy) | TLS Edge | HTTPS termination on Gematria |
| **RearView** (Qdrant) | Vector Search | Semantic search over RoadStream logs |
| **BackRoad** (Portainer) | Containers | Docker management if containerized |
| **PitStop** (Pi-hole) | DNS | Internal `RoadStream.blackroad.local` resolution |
