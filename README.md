# Secure Linux Bastion Host & Process Management

Elevvo Cloud & DevOps Track, Module 1. Environment: Ubuntu 24.04 on WSL2 (systemd enabled), isolated lab distro.

> Note: WSL2 sits behind NAT, so the optional Tailscale bonus was skipped.

## 1. SSH key generation (Ed25519, passphrase-protected)
`ssh-keygen -t ed25519 -C "abdallah@bastion"`, then `ssh-copy-id`.
Output: [outputs/keygen.txt](outputs/keygen.txt) | ![](screenshots/01-keygen.png)

Why: Ed25519 gives small, fast keys. The passphrase encrypts the private key at rest.

## 2. Daemon hardening
Set in `/etc/ssh/sshd_config`:
```
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
```
Checked `sshd_config.d/` for overrides, ran `sshd -t` before restarting, and kept a second session open.
Effective values: [outputs/sshd-effective.txt](outputs/sshd-effective.txt) | ![](screenshots/02-sshd-hardening.png)

## 3. Access verification
- Key login: success
- `ssh -o PubkeyAuthentication=no pc@localhost`: `Permission denied (publickey)`
- `ssh root@localhost`: `Permission denied (publickey)`

![](screenshots/03-access-verification.png)

The single `Accepted password` log line was the one-time `ssh-copy-id` key install, done before hardening.

## 4. systemd service
Custom unit: [configs/webapp.service](configs/webapp.service). It runs a Python HTTP server as the dedicated non-login user `websvc`, with `Restart=on-failure` and sandboxing options (`NoNewPrivileges`, `ProtectSystem`, `ProtectHome`, `PrivateTmp`).
```
sudo systemctl daemon-reload
sudo systemctl enable --now webapp
```
Status: [outputs/webapp-status.txt](outputs/webapp-status.txt) | ![](screenshots/04-webapp-status.png)

Restart policy proof: `kill -9` on the main PID, and systemd restarted it 3 seconds later.
![](screenshots/05-restart-proof.png)

## 5. Firewall (UFW)
Default deny incoming, allow 22/tcp and 8080/tcp: [outputs/ufw-status.txt](outputs/ufw-status.txt) | ![](screenshots/06-ufw.png)

## 6. Logging verification
[outputs/journal-ssh.txt](outputs/journal-ssh.txt), [outputs/journal-webapp.txt](outputs/journal-webapp.txt) | ![](screenshots/07-logs.png)

## 7. Least privilege
All work was done as the standard user `pc` with `sudo`. Root login is disabled, and the web service runs as an unprivileged system user.

## Security note
No private keys are in this repository (`.gitignore` blocks them).
