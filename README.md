# 404-not-a-problem

Planning notes for turning the Ubuntu desktop **kaviarasu-s-MS-7C89** into a reliable multi-website server.

## Machine

| Item | Value |
|---|---|
| Hostname | `kaviarasu-s-MS-7C89` |
| OS | Ubuntu 26.04.1 LTS (kernel 7.0.0-43-generic) |
| Addresses | `192.168.0.219` (LAN), `100.84.119.80` (Tailscale) |
| User | `kaviarasu-s` |
| CPU / RAM | 8 cores / 15 GB |
| Disk | ~187 GB free on `/` (218 GB total) |
| Swap | 4 GB (`/swap.img`) |

## Current state (assessed Nov 18, 2026)

- No web server, PHP, database, or SSL tooling (certbot) installed.
- No SSH server (`openssh-server` not installed).
- Docker installed; only the team-republic Postgres and SeaweedFS containers exist and are stopped.
- Tailscale running; Tailscale Serve publishes the team-republic app on 443.
- A Cloudflare tunnel (`cloudflared`) is running, launched by T3 Code.
- UFW service active (rules not yet reviewed; needs sudo).
- Connected over Wi-Fi; Ethernet port (`eno1`) available but unused.
- Desktop GNOME install (GDM, CUPS, Bluetooth, Avahi running).
- A dev server on port 4000 listens on all interfaces.
- Unattended upgrades enabled; 10 updates pending.

## Recommended setup path

1. Ethernet with a static IP or DHCP reservation for `192.168.0.219`.
2. Apply updates: `sudo apt update && sudo apt full-upgrade`, then reboot.
3. Install OpenSSH with key-only auth and root login disabled, plus `fail2ban`.
4. Firewall: `ufw default deny incoming`, allow only SSH and web (80/443).
5. Web stack, one of:
   - **Nginx + certbot** (or **Caddy** with automatic HTTPS), `php-fpm` with a pool per site, and MariaDB or Postgres if needed; or
   - **Docker** with one Compose stack per site behind **Caddy** or **Traefik**.
6. Publish sites through a **Cloudflare Tunnel** (no port forwarding, works behind CGNAT).
7. Hardening: boot to `multi-user.target`, disable unneeded desktop services, bind dev servers to `127.0.0.1`.
8. Operations: backups (e.g. `restic`) of sites and database dumps, log rotation, uptime monitoring.

## Power-loss recovery

Findings:

- Boots to `graphical.target` with GDM autologin; user lingering is enabled, so user services start at boot without a login.
- `team-republic-dev.service` (user) and `t3code.service` are enabled at boot.
- The Cloudflare tunnel is launched by T3 Code, not as its own service.
- Wi-Fi reconnects at boot without a login.
- The team-republic Postgres and SeaweedFS containers have `restart=no` and stayed down after the last outage.

To do:

- [ ] BIOS: set **Restore after AC Power Loss** to **Power On**.
- [ ] `docker update --restart unless-stopped team-republic-postgres-1 team-republic-seaweedfs-1` and add `restart: unless-stopped` in the compose file.
- [ ] `team-republic-dev.service`: `Restart=always`.
- [ ] Optional: dedicated `cloudflared` system service for the sites (`sudo cloudflared service install <token>`).
- [ ] Optional: switch default target to `multi-user.target` (note: desktop-session autostart apps would no longer launch).

## Decisions

- Keep Wi-Fi for now.
- Repo name finalized as `404-not-a-problem`.
