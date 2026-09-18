# CARC GPU queue dashboard (portable)

Self-contained local web UI for USC CARC / Discovery Slurm queues (`gpu` / `debug`): your jobs, recent finished jobs, node types, and top pending priority.

**Requirements:** Python 3.9+ (stdlib only — no `pip install`). Live mode needs USC VPN + SSH to Discovery.

## Quick start (no cluster — verify the UI)

```bash
cd carc-gpu-dashboard
./run.sh --demo
# open http://127.0.0.1:8767
```

## Live start (real queue)

1. Connect to **USC VPN**.
2. Add an SSH Host alias (see `ssh_config.example`):

```bash
# ~/.ssh/config
Host discovery
  HostName discovery.usc.edu
  User YOUR_NETID
  IdentityFile ~/.ssh/id_ed25519
```

3. Confirm non-interactive SSH works:

```bash
ssh -o BatchMode=yes discovery 'echo ok && whoami'
```

4. Run:

```bash
cd carc-gpu-dashboard
./run.sh
# or: python3 server.py
# open http://127.0.0.1:8767
```

Optional display label (queries always use the remote `$USER` from SSH):

```bash
export CARC_NETID=YOUR_NETID
./run.sh
```

## Environment knobs

| Variable | Default | Meaning |
|----------|---------|---------|
| `CARC_SSH_HOST` | `discovery` | SSH Host alias / hostname |
| `CARC_NETID` | *(auto)* | Label only; live queries use remote `$USER` |
| `CARC_DASH_HOST` | `127.0.0.1` | Bind address |
| `CARC_DASH_PORT` | `8767` | HTTP port |
| `CARC_DASH_TTL` | `25` | Cache seconds between SSH pulls |
| `CARC_DASH_HISTORY_HOURS` | `48` | `sacct` lookback window |
| `CARC_DASH_PARTITIONS` | `gpu,debug` | `sinfo -p` list |
| `CARC_DASH_DEMO` | unset | `1` = fixture mode |

CLI mirrors the same: `python3 server.py --demo --port 8767 --ssh-host discovery`.

## Layout

```
carc-gpu-dashboard/
  server.py              # HTTP + SSH collector
  run.sh                 # thin launcher
  README.md
  ssh_config.example
  env.example
  static/
    index.html
    style.css
  fixtures/
    demo_status.json     # offline UI smoke data
```

## API

| Path | Notes |
|------|--------|
| `GET /` | UI |
| `GET /api/status` | JSON snapshot (`?force=1` bypasses cache) |
| `GET /api/health` | Bind / mode probe |

Auto-refresh is every 30s in the browser; **Refresh** forces a new SSH pull.

## Sharing

Zip the folder (exclude nothing required):

```bash
cd "$(dirname "$0")/.."   # parent of this package if needed
zip -r carc-gpu-dashboard.zip carc-gpu-dashboard \
  -x '*/__pycache__/*' '*.pyc' '.*'
```

Teammate: unzip → `./run.sh --demo` first → then configure SSH and drop `--demo`.

## Notes

- Uses `BatchMode=yes` SSH (no password prompts). Prefer key auth + VPN.
- Stale snapshots are kept if a refresh fails mid-session.
- Not tied to any one research repo; rename the Host alias if your lab uses a different one.
