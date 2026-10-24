# AGENTS.md — odem-hardening

System-level security hardening — firewall (iptables), apparmor
profiles, auditd config + handler, AIDE intrusion detection, CIS
lockdown (clones the `ansible-lockdown/DEBIAN13-CIS` playbook and
runs it locally).

## Galaxy

- **namespace**: `odem`
- **name**: `hardening`
- **version**: `0.4.0`
- **dependencies**: `odem.base >=0.1.0`, `ansible.posix >=1.0.0`, `community.crypto >=2.0.0`

## Roles

| Role | Description | Complexity |
|---|---|---|
| `odem.hardening.firewall` | iptables rules.v4.j2 with 2 for-loops + 2 if-blocks (port ranges, ICMP, NAT). sysctl reload gated by `firewall_enable_nat_ip_forwarding`. Systemd unit template for the iptables-restore service. `iptables-restore` flushes Docker's `DOCKER-*` chains, so the role restarts docker afterwards (gated by `firewall_enable_docker_compat`, default true) and orders `iptables.service` before `docker.service`. | 3 (accepted) |
| `odem.hardening.apparmor` | Install apparmor-utils, `synchronize` profiles from role `files/`, enforce + restart. | 1 |
| `odem.hardening.auditd` | apt install, 2 dir ensures (rules.d + run), 2 templates (`/etc/audit/auditd.conf` with ~13 vars, systemd override from `files/`). | 2 |
| `odem.hardening.aide` | apt install, `/var/lib/aide` directory, render one small static template. | 1 |
| `odem.hardening.lockdown` | Install `cracklib-runtime` + `ansible`, clone `DEBIAN13-CIS` under `{{ identity_default_repo_dir }}`, generate SSH keypair + authorized_key, 6 `lineinfile` overrides into the CIS defaults — each gated by its own `lockdown_enable_*` toggle (sudoers user, sudoers admin group, AIDE async, X window, firewall package, sugroup) — plus a sub-playbook invocation gated by `lockdown_enable_execute`. | 3 (accepted) |
| `odem.hardening.fail2ban` | Install fail2ban with the default sshd jail (systemd backend, iptables-multiport banaction) to prevent SSH brute-force attacks. Gated by `fail2ban_enable_service`. | 1 |
| `odem.hardening.wazuh` | Install the Wazuh agent (client) from the Wazuh pre-release 5.x apt repo: official GPG key → `/etc/apt/keyrings/wazuh.gpg` (dearmored) + `signed-by` apt_repository, pinned `wazuh-agent=5.0.0-1`, enroll + set the manager endpoint at install time via `WAZUH_*` deployment variables (endpoint port 1517, enrollment port 1515, optional password/CA), enable and start `wazuh-agent`. Enrollment applies on **fresh install only** (deb postinst). Gated by `wazuh_enable_service`. | 1 |

## Conventions

- All roles are system-level (no per-user logic, no `odem_filter_users`).
- `firewall` and `lockdown` complexity is load-bearing — their templates / external-repo orchestration are intrinsically complex. Do not refactor.
- `apparmor` profile files live in `roles/apparmor/files/etc/apparmor.d/` and are `synchronize`'d onto the target. Mode/owner set explicitly.
- `lockdown` runs the CIS playbook as `{{ lockdown_user }}` (default `root`) with `become_user: true`.
- `lockdown`'s CIS-defaults overrides are individually toggleable — `lockdown_enable_<override>` (all default `true`) gates each `lineinfile`; the final `site.yml` execution is gated by `lockdown_enable_execute` (default `false`).
