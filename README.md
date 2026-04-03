# pafw

CLI tools for Palo Alto Networks firewalls. Each command connects via SSH, runs the equivalent PAN-OS CLI command, and streams output to your terminal. Ctrl+C stops interactive commands cleanly.

## Install

### Homebrew (macOS / Linux)

```
brew tap jiiro974/tap
brew install pafw
```

### Download binaries

Download from [Releases](https://github.com/jiiro974/pafw-releases/releases), then:

```
tar xzf pafw-<os>-<arch>.tar.gz
sudo mv pa* /usr/local/bin/
```

### From source

```
go install github.com/jiiro974/pafw/cmd/...@latest
```

## Usage

Use `pafw <command>` or standalone binaries:

```
pafw ping --host fw01 --target 8.8.8.8
paping --host fw01 --target 8.8.8.8      # equivalent
```

## Commands

| Command | Standalone | Description |
|---------|------------|-------------|
| `pafw ping` | `paping` | Ping from firewall |
| `pafw trace` | `patrace` | Traceroute from firewall |
| `pafw if` | `paif` | Show interfaces |
| `pafw route` | `paroute` | Show routing table |
| `pafw arp` | `paarp` | Show ARP table |
| `pafw session` | `pasession` | Show sessions |
| `pafw fib` | `pafib` | FIB lookup |
| `pafw counter` | `pacounter` | Show global counters |
| `pafw cap` | `pacap` | Packet capture (start/stop/download pcap) |
| `pafw gp` | `pagp` | GlobalProtect sessions / disconnect |

## Examples

```bash
# Ping from firewall
pafw ping --host fw01 --target 8.8.8.8 --source 10.0.0.1 --count 10

# Traceroute with logical router
pafw trace --host fw01 --target 8.8.8.8 --lr MyLR

# Show interfaces
pafw if --host fw01
pafw if --host fw01 --name ethernet1/1

# Routing table
pafw route --host fw01 --vr default

# ARP table
pafw arp --host fw01 --interface ethernet1/1

# Sessions
pafw session --host fw01 --src 192.168.1.100 --dport 443

# FIB lookup
pafw fib --host fw01 --ip 8.8.8.8 --vr default

# Global counters (drops only)
pafw counter --host fw01 --filter "severity drop"

# Packet capture (Ctrl+C to stop and download)
pafw cap --host fw01 --src 10.0.0.1 --dport 443 --proto 6

# GlobalProtect: list sessions
pafw gp --host fw01

# GlobalProtect: disconnect a user
pafw gp --host fw01 --gateway gw01 --gp-user joe --computer PC01
```

## Authentication

Tried in order:

1. **SSH agent**: auto-detected via `SSH_AUTH_SOCK` (default)
2. **Keeper Secrets Manager**: `--keeper-secret "fw-admin"` (set `KEEPER_TOKEN` env or `--keeper-token`)
3. **SSH key file**: `--key ~/.ssh/id_rsa`
4. **Password flag**: `--password secret`
5. **Interactive prompt**: asks for password if nothing else works

## Common flags

```
--host       Firewall hostname or IP (required)
--user       Username (default: OS login)
--password   Password
--key        SSH private key file
--insecure   Skip TLS verification (default: true)
--vr         Virtual router (ping, trace, route, fib)
--lr         Logical router (ping, trace, route)
--version    Show version
```
