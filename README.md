# Ansible Role: PostgreSQL

|Source|Version|CI|License|
|------|-------|--|-------|
|[![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-postgresql)|[![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-postgresql)](https://github.com/grzegorzfranus/ansible-role-postgresql/releases)|[![CI](https://github.com/grzegorzfranus/ansible-role-postgresql/actions/workflows/ci.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-postgresql/actions/workflows/ci.yml)|[![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE)|

This Ansible role installs and configures a standalone PostgreSQL database server on Ubuntu and Debian systems. It provides a production-grade, secure, and upgrade-safe database setup featuring PGDG repository selection, conf.d drop-in configuration management, pg_hba.conf authentication control, declarative database objects (DBs, users, privileges, extensions), optional logrotate, and post-config readiness validation.

## ✨ Features

- 🐘 **PostgreSQL Major Versioning**: Supports PostgreSQL major versions 14 through 18 (default 18) via PGDG apt repository or OS distribution packages.
- 🔧 **conf.d Drop-in Configuration**: Upgrades-safe drop-in `99-ansible.conf` rendering without replacing system default `postgresql.conf`.
- 🛡️ **Security & SCRAM Authentication**: Default `scram-sha-256` password encryption and strict `pg_hba.conf` rules with peer unix socket access.
- 👥 **Declarative Database Objects**: Declarative databases, users (`no_log`), privileges, and extensions managed via `community.postgresql`.
- 📊 **Log Rotation**: Optional system logrotate integration with configuration syntax dry-run validation.
- 🧪 **Readiness Validation**: Built-in readiness checks using `pg_isready` and `community.postgresql.postgresql_ping`.
- 🔄 **Service Lifecycle**: Systemd unit management with configurable reload or restart triggers.
- 🧪 **Container Integration Testing**: Molecule test scenarios on Ubuntu 24.04 and Ubuntu 26.04.

## 🎯 Architecture

The role configures a standalone PostgreSQL cluster with drop-in configuration layering:

```
/etc/postgresql/<ver>/main/
├── postgresql.conf          (system stock)
├── conf.d/
│   └── 99-ansible.conf      (managed drop-in configuration)
└── pg_hba.conf              (managed authentication rules)
```

## 📋 Requirements

- **Ansible**: 2.15 or higher
- **Python**: 3.9 or higher on target hosts
- **Collections**: `community.postgresql >= 3.4.0`, `community.general`
- **Python Packages**: `python3-psycopg` (psycopg3, installed automatically by prerequisites)
- **Privileges**: sudo/root access on target hosts

### Supported operating systems

| OS Family | Version | Status |
|-----------|---------|---------|
| Ubuntu | 26.04 (Resolute) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 24.04 (Noble) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 13 (Trixie)   | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |

### Setup module
The role uses facts gathered by Ansible on the remote host. If you disable the Setup module in your playbook, the role will not work properly.

### Root access
This role requires root access for package installation and service management. Make sure you are using a user with root privileges.

## 🚀 Quick Start

### 1. Basic PostgreSQL Setup (PGDG Version 18)

```yaml
---
- name: Configure Standalone PostgreSQL Server
  hosts: db_servers
  become: true
  roles:
    - role: grzegorzfranus.postgresql
      vars:
        postgresql_version: "18"
        postgresql_service_enabled: true
```

### 2. Declarative Databases, Users & Extensions

```yaml
---
- name: Configure PostgreSQL with Databases and Users
  hosts: db_servers
  become: true
  roles:
    - role: grzegorzfranus.postgresql
      vars:
        postgresql_version: "18"
        postgresql_databases:
          - name: "app_db"
            encoding: "UTF8"
        postgresql_users:
          - name: "app_user"
            password: "{{ vault_app_user_password }}"
        postgresql_privs:
          - database: "app_db"
            roles: ["app_user"]
            privs: "ALL"
            type: "database"
        postgresql_extensions:
          - name: "pgcrypto"
            db: "app_db"
```

## ⚙️ Configuration

### Default Configuration Summary

```yaml
postgresql_role_action: "all"
postgresql_service_enabled: true
postgresql_use_pgdg_repo: true
postgresql_version: "18"
postgresql_listen_addresses: ["localhost"]
postgresql_port: 5432
postgresql_max_connections: 100
postgresql_shared_buffers: "128MB"
postgresql_work_mem: "4MB"
postgresql_maintenance_work_mem: "64MB"
postgresql_effective_cache_size: "4GB"
postgresql_wal_level: "replica"
postgresql_auth_method: "scram-sha-256"
postgresql_password_encryption: "scram-sha-256"
postgresql_ssl_enabled: true
postgresql_users_no_log: true
```

## 📊 Variables

### General Options

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_role_action` | Action phase (`all`, `prerequisites`, `install`, `configure`, `databases`, `upgrade`) | `"all"` |
| `postgresql_service_enabled` | Enable service at boot | `true` |
| `postgresql_run_test` | Enable post-config readiness ping tests | `false` |
| `postgresql_configure_logrotate` | Render logrotate configuration | `false` |
| `postgresql_config_change_action` | Handler action on config change (`reload` or `restart`) | `"reload"` |

### Installation Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_use_pgdg_repo` | Enable official PGDG apt repository | `true` |
| `postgresql_version` | PostgreSQL major version string (`"14"`, `"15"`, `"16"`, `"17"`, `"18"`) | `"18"` |
| `postgresql_install_extra_packages` | Additional apt packages to install | `[]` |
| `postgresql_locales` | System locales to generate | `["en_US.UTF-8"]` |

### Connection & Performance Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_listen_addresses` | Interface listen addresses | `["localhost"]` |
| `postgresql_port` | PostgreSQL listen port (1–65535) | `5432` |
| `postgresql_max_connections` | Max concurrent connections | `100` |
| `postgresql_unix_socket_directories` | Unix domain socket directory paths | `["/var/run/postgresql"]` |
| `postgresql_shared_buffers` | Shared buffer memory | `"128MB"` |
| `postgresql_work_mem` | Sort/hash work memory | `"4MB"` |
| `postgresql_maintenance_work_mem` | Maintenance work memory | `"64MB"` |
| `postgresql_effective_cache_size` | Planner cache estimation | `"4GB"` |

### Write-Ahead Log (WAL) & Checkpoints

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_wal_level` | Level of detail written to WAL (`minimal`, `replica`, `logical`) | `"replica"` |
| `postgresql_max_wal_size` | Max size to let WAL grow between checkpoints | `"1GB"` |
| `postgresql_min_wal_size` | Minimum size of WAL files retained | `"80MB"` |
| `postgresql_checkpoint_completion_target` | Completion target for WAL checkpoints (0.0–1.0) | `0.9` |

### Logging Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_logging_collector` | Enable background log collector | `false` |
| `postgresql_log_directory` | Directory where log files are stored | `"log"` |
| `postgresql_log_filename` | Log file naming pattern | `"postgresql-%Y-%m-%d.log"` |
| `postgresql_log_rotation_age` | Automatic rotation age for log files | `"1d"` |
| `postgresql_log_line_prefix` | Format string for log line prefixes | `"%m [%p] %q%u@%d "` |
| `postgresql_log_min_duration_statement` | Minimum statement duration (ms) to trigger log (-1 to disable) | `-1` |

### Authentication & Security Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_auth_method` | Default authentication method (`scram-sha-256`, `trust`, `reject`, `md5`) | `"scram-sha-256"` |
| `postgresql_password_encryption` | Password encryption algorithm (`scram-sha-256`, `md5`) | `"scram-sha-256"` |
| `postgresql_ssl_enabled` | Enable SSL connections | `true` |
| `postgresql_ssl_cert_file` | Path to SSL certificate file | `"/etc/ssl/certs/ssl-cert-snakeoil.pem"` |
| `postgresql_ssl_key_file` | Path to SSL private key file | `"/etc/ssl/private/ssl-cert-snakeoil.key"` |
| `postgresql_hba_entries` | List of pg_hba.conf rules | *See defaults/main.yml* |
| `postgresql_extra_config_options` | Free-form key/value dictionary rendered into 99-ansible.conf | `{}` |
| `postgresql_users_no_log` | Suppress output logging on user password tasks | `true` |

### Database Objects

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_databases` | List of databases to create | `[]` |
| `postgresql_users` | List of database users to create | `[]` |
| `postgresql_privs` | List of database privileges to grant | `[]` |
| `postgresql_extensions` | List of database extensions to enable | `[]` |

### Logrotate Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_logrotate_options` | Logrotate options dictionary for PostgreSQL log files | *See defaults/main.yml* |



## 📌 Role Properties

| Property | Value | Description |
|----------|-------|-------------|
| **Idempotent** | ✅ Yes | Running the role multiple times produces zero changes after initial convergence. |
| **Check Mode** | ✅ Supported | Configuration rendering supports dry-run preview. |
| **Diff Mode** | ✅ Supported | Template tasks show exact configuration line diffs. |

## 🚫 Scope Limits & Roadmap

The following topics are explicitly out of scope for this standalone role:
- **Backups**: Covered by `ansible-role-pg-backup`.
- **Replication / High Availability**: Patroni, repmgr, and streaming replication.
- **PgBouncer / Exporters**: Separate middleware roles.
- **Major Upgrades**: Single cluster lifecycle only; `pg_upgrade` is handled out of band.

## 🔍 Verification

Verify PostgreSQL cluster health:

```bash
# Check systemd status
sudo systemctl status postgresql

# Run pg_isready
pg_isready -p 5432

# Verify catalog access
sudo -u postgres psql -c "\l"
```

## 📁 File Structure

```
ansible-role-postgresql/
├── .github/
│   └── workflows/
│       ├── ci.yml                     # CI pipeline
│       └── release.yml                # Release Please + Galaxy publish
├── defaults/
│   └── main.yml                       # Default configuration variables
├── handlers/
│   └── main.yml                       # Service restart and reload handlers
├── meta/
│   ├── main.yml                       # Role metadata
│   └── argument_specs.yml             # Native argument specification validation
├── molecule/
│   └── default/                       # Molecule testing scenario (Ubuntu 24.04/26.04)
├── tasks/
│   ├── main.yml                       # Main task orchestration
│   ├── assert.yml                     # Input assertion validation
│   ├── prerequisites.yml              # Python & locale setup
│   ├── repository.yml                 # PGDG deb822 repo setup
│   ├── install.yml                    # Package installation
│   ├── configure.yml                  # conf.d & pg_hba.conf rendering
│   ├── databases.yml                  # DBs, users, privs, extensions
│   ├── logrotate.yml                  # Logrotate setup
│   ├── upgrade.yml                    # Minor package upgrade
│   └── test.yml                       # Readiness checks
├── templates/
│   ├── postgresql/
│   │   ├── 99-ansible.conf.j2         # conf.d drop-in template
│   │   └── pg_hba.conf.j2             # pg_hba.conf template
│   └── logrotate/
│       └── postgresql.j2              # Logrotate template
└── vars/
    ├── main.yml                       # Internal variables
    ├── debian.yml                     # Debian OS variables
    ├── ubuntu_24.04.yml               # Ubuntu 24.04 default version
    └── ubuntu_26.04.yml               # Ubuntu 26.04 default version
```

## CI/CD Pipeline

Uses centralized GitHub Actions workflows from `grzegorzfranus/github-workflows@v3.0.1`:
- Branch naming lint (`feature/`, `bugfix/`, `fix/`, etc.)
- PR title Conventional Commits lint (`feat:`, `fix:`, etc.)
- Yamllint + Ansible Lint
- Molecule testing across Ubuntu 24.04 and 26.04

## 📝 License

Licensed under the Apache-2.0 License - see the [LICENSE](LICENSE) file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).
