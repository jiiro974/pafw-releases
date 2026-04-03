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
| `pafw arp` | `paarp` | Show / clear ARP table |
| `pafw session` | `pasession` | Show sessions |
| `pafw fib` | `pafib` | FIB lookup |
| `pafw counter` | `pacounter` | Show global counters |
| `pafw cap` | `pacap` | Packet capture |
| `pafw gp` | `pagp` | GlobalProtect sessions / disconnect |

## GlobalProtect

```bash
# List sessions (formatted table)
pagp --host fw01
pagp --host fw01 --json

# Disconnect a user (auto-detects gateway/computer)
pagp disconnect user toto --host fw01
pagp disconnect toto --host fw01              # shorthand

# Disconnect everyone
pagp disconnect all --host fw01
pagp disconnect all --host fw01 --gateway gw01
```

## ARP management

```bash
# Show ARP table
paarp --host fw01
paarp --host fw01 --json

# Clear all ARP
paarp clear all --host fw01

# Clear by interface
paarp clear ethernet1/1 --host fw01

# Clear by IP or MAC (auto-detects interface)
paarp clear --ip 10.0.0.1 --host fw01
paarp clear --mac 00:1a:2b:3c:4d:5e --host fw01

# Clear by IP/MAC on specific interface
paarp clear ethernet1/1 --ip 10.0.0.1 --host fw01
```

## Output formats

All commands support `--raw` (raw firewall output) and most support `--json`:

```bash
pafw gp --host fw01               # formatted table
pafw gp --host fw01 --raw         # raw PAN-OS output
pafw gp --host fw01 --json        # structured JSON
pafw route --host fw01 --json     # routes as JSON
pafw arp --host fw01 --json       # ARP as JSON
```

## More examples

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

# Sessions
pafw session --host fw01 --src 192.168.1.100 --dport 443

# FIB lookup
pafw fib --host fw01 --ip 8.8.8.8 --vr default

# Global counters (drops only)
pafw counter --host fw01 --filter "severity drop"

# Packet capture (Ctrl+C to stop and download)
pafw cap --host fw01 --src 10.0.0.1 --dport 443 --proto 6
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
--json       Structured JSON output
--raw        Raw firewall output
--vr         Virtual router (ping, trace, route, fib)
--lr         Logical router (ping, trace, route)
--version    Show version
```
