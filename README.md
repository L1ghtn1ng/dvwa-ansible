# DVWA Ansible

Deploy a pinned revision of [Damn Vulnerable Web Application](https://github.com/digininja/DVWA)
to a dedicated Ubuntu 24.04 LTS host. The playbook installs Apache, PHP 8.3,
and MariaDB, configures DVWA, and initializes its training database.

> [!CAUTION]
> DVWA is intentionally vulnerable. Run it only in an isolated lab VM on a
> private network. Do not expose it to the public internet or install it on a
> machine that contains sensitive data.

## Requirements

- An Ubuntu 24.04 LTS target with an `ansible` user that has passwordless sudo
- SSH key authentication from the controller to the target
- [uv](https://docs.astral.sh/uv/getting-started/installation/) 0.11.x on the
  controller

Python and Ansible are managed by uv. The project pins `ansible-core` and all
Ansible collections, so no global Python packages are required.

## Deploy

Update the address in `hosts.yml`, then install the locked dependencies and
collections:

```shell
uv sync --locked
uv run ansible-galaxy collection install --requirements-file requirements.yml
```

Confirm SSH and privilege escalation work before changing the target:

```shell
uv run ansible dvwa --module-name ansible.builtin.ping
```

Deploy DVWA:

```shell
uv run ansible-playbook main.yml
```

The first run initializes the DVWA database. Later runs check for the seeded
`users` table and do not reset lab data. Preview later changes with check mode:

```shell
uv run ansible-playbook main.yml --check --diff
```

Open `http://TARGET_ADDRESS/` and sign in with DVWA's default credentials:
`admin` / `password`.

## Configuration

Deployment defaults live in `group_vars/all.yml`.

| Variable | Purpose |
| --- | --- |
| `dvwa_source_version` | Exact upstream Git revision to deploy |
| `dvwa_db_name` | MariaDB database name |
| `dvwa_db_user` | Restricted application database user |
| `dvwa_db_password` | Application database password |
| `dvwa_install_root` | Git checkout and Apache document root |
| `dvwa_default_security_level` | Default DVWA exercise difficulty |

The checked-in database password is a convenience value for an isolated
training lab, not a secret. Override it through inventory or Ansible Vault if
the lab's risk model requires a unique credential.

## Validate

Run the same static checks used by CI:

```shell
uv lock --check
uv run ansible-lint
uv run ansible-playbook --syntax-check main.yml
```

## Molecule VM test

The default Molecule scenario creates an Ubuntu 24.04 cloud VM with libvirt.
On an Ubuntu controller, install the host dependencies:

```shell
sudo apt-get update
sudo apt-get install --yes \
  build-essential cloud-image-utils libvirt-clients libvirt-daemon-system \
  libvirt-dev pkg-config qemu-system-x86 qemu-utils virtinst
sudo systemctl enable --now libvirtd
```

Ensure libvirt's default network exists and is active, then run the scenario:

```shell
sudo virsh net-define /usr/share/libvirt/networks/default.xml  # only if absent
sudo virsh net-start default                                  # only if inactive
uv sync --locked --group molecule
uv run ansible-galaxy collection install --requirements-file requirements.yml
sudo -v
uv run molecule test
```

Molecule uses KVM when `/dev/kvm` is available and falls back to QEMU software
emulation otherwise. The scenario verifies the OS and PHP versions, service
health, Apache configuration, pinned DVWA revision, permissions, database
contents, HTTP response, and playbook idempotence. It destroys the VM after the
test, including when verification fails.
