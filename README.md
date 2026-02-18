# tempo-ansible

Ansible playbook for deploying and managing Tempo RPC nodes. We recommend going through the playbook and adjusting to your use case. This playbook is provided as-is. It is more of a starting point than a ready to use playbook.

## Roles

| Role | Tag | Description |
|------|-----|-------------|
| **setup** | `setup` | Full node setup: installs Tempo, configures systemd, UFW firewall, Promtail log shipping, and optional Caddy reverse proxy |
| **download** | `download` | Downloads chain data via a background systemd oneshot service |
| **upgrade** | `upgrade` | Upgrades Tempo to a specified version using `tempoup` |

## Prerequisites

- Ubuntu target hosts
- Tailscale installed and configured (metrics bind to `ansible_tailscale0` interface)
- A Loki instance for log aggregation (if using Promtail)

## Variables

### Defaults (`group_vars/all.yml`)

| Variable | Default | Description |
|----------|---------|-------------|
| `tempo_http_addr` | `127.0.0.1` | HTTP RPC bind address |
| `tempo_http_port` | `8545` | HTTP RPC port |
| `tempo_p2p_port` | `30303` | P2P port (TCP + UDP) |
| `tempo_chain` | `moderato` | Chain name |
| `tempo_metrics_port` | `9000` | Metrics port |
| `tempo_data_dir` | `/var/lib/tempo` | Data directory path |
| `tempo_log_dir` | `/var/log/tempo` | Log directory path |
| `tempo_install_url` | `https://tempo.xyz/install` | URL for Tempo install script |
| `promtail_version` | `3.3.2` | Promtail version to install |

### Host variables (`inventory.yaml`)

| Variable | Required | Description |
|----------|----------|-------------|
| `ansible_user` | yes | SSH user |
| `ansible_host` | yes | Host IP address |
| `deploy_user` | yes | User to run Tempo as |
| `deploy_group` | yes | Group for Tempo process |
| `tempo_version` | yes | Tempo version (e.g. `v1.1.4`) |
| `loki_server` | no | Loki server hostname/IP (omit to skip Promtail setup) |
| `tempo_caddy_domain` | no | Domain for Caddy reverse proxy (omit to skip Caddy setup) |
| `tempo_trusted_peers` | no | List of trusted peer addresses |
| `tempo_bootnodes` | no | List of bootnode addresses |

## Usage

```bash
# Full setup
ansible-playbook -i inventory.yaml site.yaml

# Run specific roles
ansible-playbook -i inventory.yaml site.yaml --tags setup
ansible-playbook -i inventory.yaml site.yaml --tags download
ansible-playbook -i inventory.yaml site.yaml --tags upgrade

# Monitor chain download progress
journalctl -u tempo-download -f
```
