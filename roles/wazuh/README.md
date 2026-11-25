---
namespace: odem
collection: hardening
role: wazuh
---

# `odem.hardening.wazuh`

Install the Wazuh agent (client) from the Wazuh pre-release 5.x apt repository
and enroll it with a Wazuh manager.

## Default variables

| Variable | Default | Description |
|---|---|---|
| `wazuh_enable_service` | `true` | Toggle to install the wazuh-agent and run it entirely |
| `wazuh_enable_enrollment` | `true` | Pass the `WAZUH_REGISTRATION_*` variables so the agent auto-enrolls at install time |
| `wazuh_enable_registration_password` | `false` | Pass `WAZUH_REGISTRATION_PASSWORD` (requires the manager to enforce an enrollment password) |
| `wazuh_prereq_packages` | `- gnupg` | Apt packages required to add the Wazuh repository |
| `wazuh_repo_key_url` | `https://packages.wazuh.com/key/GPG-KEY-WAZUH` | URL of the official (ASCII-armored) Wazuh signing key |
| `wazuh_keyring_asc_path` | `/etc/apt/keyrings/wazuh.asc` | Path for the downloaded armored GPG key |
| `wazuh_keyring_path` | `/etc/apt/keyrings/wazuh.gpg` | Path for the dearmored GPG keyring referenced by `signed-by` |
| `wazuh_repo_url` | `https://packages-staging.xdrsiem.wazuh.info/pre-release/5.x/apt` | Base URL of the Wazuh apt repository (pre-release 5.x beta repo by default) |
| `wazuh_repo_suite` | `unstable` | Apt suite of the Wazuh repository |
| `wazuh_repo_component` | `main` | Apt component of the Wazuh repository |
| `wazuh_apt_filename` | `wazuh` | Filename for the apt source list |
| `wazuh_package` | `wazuh-agent` | Wazuh agent package name |
| `wazuh_package_version` | `5.0.0-1` | Pinned wazuh-agent version (empty string tracks the repository latest) |
| `wazuh_agent_manager_host` | `""` | Hostname or IP of the Wazuh manager (required) |
| `wazuh_agent_manager_port` | `1517` | Manager client-comms (HTTPS) port |
| `wazuh_agent_endpoint` | `https://{{ wazuh_agent_manager_host }}:{{ wazuh_agent_manager_port }}` | Full management endpoint passed as `WAZUH_MANAGER_ENDPOINT` (override to add a URL prefix) |
| `wazuh_agent_enrollment_server` | `{{ wazuh_agent_manager_host }}` | Enrollment server passed as `WAZUH_REGISTRATION_SERVER` |
| `wazuh_agent_enrollment_port` | `1515` | Enrollment port passed as `WAZUH_REGISTRATION_PORT` |
| `wazuh_agent_name` | `""` | Agent name passed as `WAZUH_AGENT_NAME` (empty string uses the agent hostname) |
| `wazuh_agent_groups` | `[]` | Agent groups passed comma-joined as `WAZUH_AGENT_GROUP` |
| `wazuh_agent_registration_password` | `""` | Enrollment password passed as `WAZUH_REGISTRATION_PASSWORD` when `wazuh_enable_registration_password` is true |
| `wazuh_agent_server_ca_path` | `""` | Manager CA certificate path passed as `WAZUH_REGISTRATION_CA` (enables TLS verification of the manager) |
| `wazuh_service_name` | `wazuh-agent` | systemd service name for the Wazuh agent |

## Notes

- The agent is installed from the Wazuh **pre-release 5.x** apt repository
  (suite `unstable`), signed by the official Wazuh key. `wazuh_package_version`
  is pinned to `5.0.0-1` (the 5.0.0-beta5 build) to match the manager images
  shipped by `mps-services-monitoring`.
- Enrollment and the manager endpoint are consumed by the deb **postinst** on a
  **fresh install only**. Changing the manager after the agent is installed
  requires re-enrolling manually (or reinstalling the package); re-running this
  role against an already-installed agent will not re-apply the variables.
- On the manager host, `mps-services-monitoring`'s `BIND_IP` must not stay `127.0.0.1`
  for remote agents, and the enrollment (`1515`) and client-comms (`1517`) ports
  must be reachable.
- The beta5 manager generates its own (self-signed) remoted certificate. If TLS
  verification is enforced by the agent, set `wazuh_agent_server_ca_path` to the
  manager CA so enrollment/connection succeeds.

## Dependencies

None.

## Example usage

```yaml
- hosts: all
  vars:
    wazuh_agent_manager_host: 10.0.0.10
  roles:
    - odem.hardening.wazuh
```

## Role metadata

- **Min Ansible version**: `2.16.0`
- **License**: GPL-3.0-or-later
- **Platforms**: Debian (trixie)
- **Tasks file lines**: 106

## Related files

- [`meta/main.yml`](meta/main.yml) — galaxy_info + role dependencies
- [`meta/argument_specs.yml`](meta/argument_specs.yml) — variable spec (the source of the variable table above)
- [`defaults/main.yml`](defaults/main.yml) — variable defaults (the source of the default values above)
