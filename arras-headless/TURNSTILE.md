# Turnstile integration (Lucrehulk 2026)

This farm does **not** solve Cloudflare Turnstile by itself.
It **requests** tokens from a running [cloudflare-turnstile-solver-2026](https://github.com/Lucrehulk/cloudflare-turnstile-solver-2026) stack.

## Why

That solver needs:
- Real **GUI browsers** (Chrome/Edge/etc.)
- Rust **token-server**
- Rust **turnstile-clicker** (OS mouse)
- Browser **proxy extension** + harvester page

GitHub Actions workflows are headless → they can only be **receivers**.

## Setup

### On a Windows (or GUI) machine

1. Clone https://github.com/Lucrehulk/cloudflare-turnstile-solver-2026
2. Start **token-server** (`cargo run` in `token-server`) → `ws://0.0.0.0:8080`
3. Load the **proxy extension**, set `SITEKEY` for arras / target site
4. Start **clicker** (F8 on), **z-index-orderer**, launch browser solvers
5. Expose token-server to your workflows (tailscale / public IP / tunnel)

### On each arras-headless runner (workflow)

```bash
export TURNSTILE_SERVER=ws://YOUR_GUI_HOST:8080
# optional:
export TURNSTILE_TIMEOUT=45000
export TURNSTILE_ACTION=   # if site uses turnstile action field
export TURNSTILE_UA=       # force a solver UA bucket, or leave empty
node server.js
```

Files:
- `turnstile-client.js` — receiver protocol client
- `index.js` — requests token before Play when `TURNSTILE_SERVER` is set
- `server.js` — logs solver availability on boot

## Protocol used

Receiver → server: binary packet header `1` (solve request)
Server → receiver: `[solver_idx u32 LE][token utf8]` or fail

See Lucrehulk README for full packet layout.
