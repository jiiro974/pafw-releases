# pafw

CLI tools for Palo Alto Networks firewalls. Single multicall binary (~7MB) with symlinks — archive ~3MB. Connects via SSH, streams output in real-time. Ctrl+C stops interactive commands cleanly.

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

### Shell completion

```bash
# zsh
pafw completion zsh > ~/.zsh/completions/_pafw

# bash
pafw completion bash > ~/.local/share/bash-completion/completions/pafw
```

## Usage

Use `pafw <command>` or standalone symlinks:

```
pafw ping --host fw01 --target 8.8.8.8
paping --host fw01 --target 8.8.8.8      # same thing
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

# Clear by IP or MAC (auto-detects interface)
paarp clear --ip 10.0.0.1 --host fw01
paarp clear --mac 00:1a:2b:3c:4d:5e --host fw01

# Clear on specific interface
paarp clear ethernet1/1 --ip 10.0.0.1 --host fw01
```

## Output formats

```bash
pafw gp --host fw01               # formatted table (default)
pafw gp --host fw01 --raw         # raw PAN-OS output
pafw gp --host fw01 --json        # structured JSON
```

`--json` supported on: gp, if, route, arp, session

## More examples

```bash
pafw ping --host fw01 --target 8.8.8.8 --source 10.0.0.1 --count 10
pafw trace --host fw01 --target 8.8.8.8 --lr MyLR
pafw if --host fw01 --name ethernet1/1
pafw route --host fw01 --vr default
pafw session --host fw01 --src 192.168.1.100 --dport 443
pafw fib --host fw01 --ip 8.8.8.8 --vr default
pafw counter --host fw01 --filter "severity drop"
pafw cap --host fw01 --src 10.0.0.1 --dport 443 --proto 6
```

## Authentication

1. **SSH agent**: auto-detected via `SSH_AUTH_SOCK` (default)
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
--vr         Virtual router (ping, trace, route, fib)
--lr         Logical router (ping, trace, route)
--version    Show version
```
