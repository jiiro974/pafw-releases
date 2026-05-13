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
| `pafw session` | `pasession` | Show / filter active sessions |
| `pafw fib` | `pafib` | FIB lookup |
| `pafw counter` | `pacounter` | Show global counters |
| `pafw cap` | `pacap` | Packet capture |
| `pafw gp` | `pagp` | GlobalProtect sessions / disconnect |
| `pafw log` | `palog` | Extract activity logs (traffic / GlobalProtect) |

All commands support `--json` for structured output pipeable to `jq`.

## JSON output examples

```bash
# Ping — RTT stats and per-reply detail
paping --host fw01 --target 8.8.8.8 --json | jq '{target, loss_pct, rtt_avg_ms}'

# Traceroute — per-hop breakdown
patrace --host fw01 --target 8.8.8.8 --json | jq '.hops[] | select(.timeout==false) | {hop, ip, rtt_ms}'

# Active sessions — filter by user, app, state
pasession --host fw01 --filter-user domain\\jdoe --json | jq '.[] | {id, app, state, dst_ip, dst_port}'
pasession --host fw01 --src 10.0.0.42 --json | jq '.[] | select(.state=="ACTIVE")'

# Global counters — non-zero drops only
pacounter --host fw01 --filter "severity drop" --json | jq '.[] | select(.value > 0) | {name, value}'

# FIB lookup
pafib --host fw01 --ip 8.8.8.8 --json | jq '.entries[] | {nexthop, interface, metric}'

# Traffic logs
palog --host fw01 --src 10.0.0.42 --last 24h --json | jq '.[] | {time, dst_ip, app, action}'
palog --host fw01 --action deny --last 7d --json | jq '.[] | select(.app=="ssl")'

# GlobalProtect logs
palog --host fw01 --type globalprotect --filter-user jdoe --last 7d --json
```

## Session filters

```bash
pasession --host fw01 --src 10.0.0.42
pasession --host fw01 --filter-user domain\\jdoe
pasession --host fw01 --proto 6 --state ACTIVE
pasession --host fw01 --from LAN --to Internet
```

| Flag | Description |
|------|-------------|
| `--src` / `--dst` | Source / destination IP |
| `--sport` / `--dport` | Source / destination port |
| `--proto` | IP protocol (6=TCP, 17=UDP, 1=ICMP) |
| `--application` | Application name |
| `--filter-user` | Source user (User-ID) |
| `--state` | ACTIVE, INIT, OPENING, CLOSING |
| `--from` / `--to` | Source / destination security zone |

## Log extraction filters

| Flag | Description |
|------|-------------|
| `--type` | `traffic` (default) or `globalprotect` |
| `--src` / `--dst` | Source / destination IP |
| `--filter-user` | Username (User-ID / GlobalProtect) |
| `--app` | Application name |
| `--action` | `allow` or `deny` |
| `--last` | `24h`, `7d`, `30d` (default: `24h`) |
| `--from` / `--to` | Date range `YYYY-MM-DD` |
| `--limit` | Max entries (default: 1000) |

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
--json       Structured JSON output (all commands)
--raw        Raw firewall output
--vr         Virtual router
--lr         Logical router
--version    Show version
```

## License

Proprietary - [Pragma-TIC](https://www.pragma-tic.org)
