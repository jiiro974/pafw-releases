# pafw

**Pragma-TIC** (https://www.pragma-tic.org)

CLI tools for Palo Alto Networks firewalls. Single multicall binary (~7MB) with symlinks — archives ~3MB. Connects via SSH, streams output in real-time. Ctrl+C stops interactive commands cleanly.

## Install

### macOS / Linux (Homebrew)

```
brew tap jiiro974/tap
brew install pafw
```

### Windows (MSI installer)

Download `pafw-x.y.z-windows-amd64.msi` from [Releases](https://github.com/jiiro974/pafw-releases/releases) and run it. Installs to `Program Files\pafw` and adds to PATH.

### Windows (portable ZIP)

Download `pafw-windows-amd64.zip`, extract, and add the folder to your PATH.

### Manual install (all platforms)

Download from [Releases](https://github.com/jiiro974/pafw-releases/releases):

```bash
# Unix
tar xzf pafw-<os>-<arch>.tar.gz
sudo mv pa* /usr/local/bin/
```

### Shell completion

```bash
# zsh
pafw completion zsh > ~/.zsh/completions/_pafw

# bash
pafw completion bash > ~/.local/share/bash-completion/completions/pafw
```

## Commands

| Command | Symlink | Description |
|---------|---------|-------------|
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
| `pafw log` | `palog` | Extract activity logs (traffic / GlobalProtect) |

## Examples

```bash
# Ping / traceroute
pafw ping --host fw01 --target 8.8.8.8 --count 10
pafw trace --host fw01 --target 8.8.8.8 --lr MyLR

# Show / JSON
pafw if --host fw01 --json
pafw route --host fw01 --vr default --json
pafw arp --host fw01 --json
pafw session --host fw01 --src 10.0.0.1

# GlobalProtect
pagp --host fw01                                    # list sessions
pagp disconnect user toto --host fw01               # disconnect user
pagp disconnect all --host fw01 --gateway gw01      # disconnect all

# ARP management
paarp clear --ip 10.0.0.1 --host fw01              # auto-detect interface
paarp clear --mac 00:1a:2b:3c:4d:5e --host fw01

# Packet capture (Ctrl+C to stop)
pafw cap --host fw01 --src 10.0.0.1 --dport 443 --proto 6

# FIB lookup / counters
pafw fib --host fw01 --ip 8.8.8.8
pafw counter --host fw01 --filter "severity drop"
```

## Log extraction

Query traffic or GlobalProtect logs over a time period. Output is pure JSON when `--json` is set, ready to pipe into `jq`.

```bash
# Last 24h traffic for a machine
palog --host fw01 --src 10.0.0.42 --last 24h --json | jq '.[] | {time,dst_ip,app}'

# Denied traffic over a date range
palog --host fw01 --action deny --from 2025-05-01 --to 2025-05-07 --json

# All traffic for a user
palog --host fw01 --filter-user jdoe --last 7d --json | jq '.[] | select(.action=="deny")'

# GlobalProtect connections
palog --host fw01 --type globalprotect --filter-user jdoe --last 7d
palog --host fw01 --type globalprotect --src 203.0.113.5 --last 30d --json

# Increase result limit (default 1000)
palog --host fw01 --src 10.0.0.1 --last 24h --limit 5000 --json
```

### Log filters

| Flag | Description |
|------|-------------|
| `--type` | `traffic` (default) or `globalprotect` |
| `--src` | Source IP or subnet |
| `--dst` | Destination IP (traffic only) |
| `--filter-user` | Username (User-ID / GlobalProtect) |
| `--app` | Application: `ssl`, `web-browsing`, … |
| `--action` | `allow` or `deny` |
| `--last` | Time window: `24h`, `7d`, `30d` (default: `24h`) |
| `--from` | Start date `YYYY-MM-DD` |
| `--to` | End date `YYYY-MM-DD` |
| `--limit` | Max entries (default: 1000) |

## Output formats

```bash
pafw gp --host fw01               # formatted table
pafw gp --host fw01 --raw         # raw PAN-OS output
pafw gp --host fw01 --json        # structured JSON
```

`--json` supported on: gp, if, route, arp, session, log

## Authentication

1. **SSH agent** (default)
2. **Keeper Secrets Manager**: `--keeper-secret "fw-admin"`
3. **SSH key file**: `--key ~/.ssh/id_rsa`
4. **Password flag**: `--password secret`
5. **Interactive prompt**: fallback

## Common flags

```
--host       Firewall hostname or IP (required)
--user       Username (default: OS login)
--password   Password
--key        SSH private key file
--json       Structured JSON output
--raw        Raw firewall output
--vr         Virtual router
--lr         Logical router
--version    Show version
```

## License

Proprietary - [Pragma-TIC](https://www.pragma-tic.org)
