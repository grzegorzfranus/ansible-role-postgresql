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
- 🧪 **Container Integration Testing**: Molecule test scenarios on Ubuntu 24.04, Ubuntu 26.04, and Debian 13.

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

> [!IMPORTANT]
> **Configuration Change Behavior & Service Restarts**
>
> Modifying `postgresql_port`, `postgresql_listen_addresses`, `postgresql_shared_buffers`, `postgresql_wal_level`, or `postgresql_max_worker_processes` requires a full PostgreSQL service restart. Because `postgresql_config_change_action` defaults to `"reload"`, PostgreSQL will NOT apply changes to these parameters until a restart occurs.
> To automatically restart PostgreSQL when configuration changes, set `postgresql_config_change_action: "restart"`.

> [!NOTE]
> **Mandatory Size & Time Unit Suffixes**
>
> All memory size variables (`postgresql_shared_buffers`, `postgresql_work_mem`, `postgresql_maintenance_work_mem`, `postgresql_effective_cache_size`, `postgresql_max_wal_size`, `postgresql_min_wal_size`) and time duration variables (`postgresql_checkpoint_timeout`, `postgresql_log_rotation_age`) require explicit mandatory unit suffixes without spaces (e.g., `B`, `kB`, `MB`, `GB`, `TB` for memory; `ms`, `s`, `min`, `h`, `d` for time). Bare integers without explicit unit suffixes are rejected by assertion validation.

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
| `postgresql_max_worker_processes` | Maximum background worker processes | `8` |
| `postgresql_max_parallel_workers_per_gather` | Parallel workers per Gather node | `2` |
| `postgresql_max_parallel_maintenance_workers` | Parallel workers per maintenance command | `2` |
| `postgresql_max_parallel_workers` | Maximum total parallel query workers | `8` |
| `postgresql_random_page_cost` | Estimate of non-sequential disk fetch cost (1.1 for SSD) | `1.1` |
| `postgresql_effective_io_concurrency` | Expected simultaneous disk I/O operations for prefetching | `200` |

### Write-Ahead Log (WAL) & Checkpoints

| Variable | Description | Default |
|----------|-------------|---------|
| `postgresql_wal_level` | Level of detail written to WAL (`minimal`, `replica`, `logical`) | `"replica"` |
| `postgresql_checkpoint_timeout` | Maximum time between automatic WAL checkpoints | `"15min"` |
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
| `postgresql_logrotate_options.rotate` | Number of rotated log files to retain | `7` |
| `postgresql_logrotate_options.frequency` | Rotation frequency (`daily`, `weekly`, `monthly`, `yearly`) | `"daily"` |
| `postgresql_logrotate_options.compress` | Compress rotated log files with gzip | `true` |
| `postgresql_logrotate_options.delaycompress` | Postpone compression of previous log file until next cycle | `true` |
| `postgresql_logrotate_options.missingok` | Do not issue error if log file is missing | `true` |
| `postgresql_logrotate_options.notifempty` | Do not rotate log file if it is empty | `true` |
| `postgresql_logrotate_options.su` | User and group credentials used for running logrotate | `"postgres postgres"` |
| `postgresql_logrotate_options.archive_directory_path` | Dedicated directory path to move rotated log archives into (`olddir`). Empty string disables `olddir` | `""` |
| `postgresql_logrotate_options.dateext` | Append date extension suffix to rotated log files | `true` |
| `postgresql_logrotate_options.dateformat` | Date extension format string when `dateext` is true | `"-%Y%m%d"` |

#### Log Rotation Ownership

PostgreSQL provides its own internal log rotation mechanism via `logging_collector` (configured through `postgresql_logging_collector`, `postgresql_log_rotation_age`, and the timestamp pattern in `postgresql_log_filename`). When `postgresql_configure_logrotate: true` is enabled, both PostgreSQL internal rotation and system logrotate may operate simultaneously on the cluster log directory.

When system logrotate is enabled, the recommended configuration to avoid dual rotation conflicts is:
- Set `postgresql_log_rotation_age: "0"` to disable PostgreSQL internal time-based automatic rotation.
- Set `postgresql_log_filename` to a static filename without date substitution patterns (for example, `"postgresql.log"`), delegating log versioning and date extension handling entirely to logrotate.

> [!NOTE]
> The role intentionally does not enforce these settings automatically. The choice of whether to rely solely on logrotate, solely on PostgreSQL's internal rotation, or a hybrid combination is left to the user's discretion.

> [!NOTE]
> **Logrotate Scope & Ownership**:
> - This role manages log rotation exclusively for the cluster log directory PostgreSQL writes to (via `postgresql_log_directory`, defaulting to `/var/lib/postgresql/<version>/main/log/*.log`).
> - Cluster startup logs in `/var/log/postgresql/*.log` (written by `pg_ctlcluster`) belong to the OS package's `postgresql-common` drop-in (`/etc/logrotate.d/postgresql-common`), which retains those files for 10 weekly rotations.
> - **Path Assertion Constraint**: `postgresql_log_directory` must not resolve to `/var/log/postgresql` when `postgresql_configure_logrotate` is enabled. Setting it to `/var/log/postgresql` collides with `postgresql-common` and is blocked by validation assertions in `tasks/assert.yml`.

## 📌 Role Properties

| Property | Value | Description |
|----------|-------|-------------|
| **Idempotent** | Yes | Running the role multiple times with identical inputs produces no further changes. |
| **Check Mode** | Supported | Tasks run safely without mutating state when check mode (`--check`) is enabled. |
| **Diff Mode** | Supported | Template changes show inline diffs when diff mode (`--diff`) is enabled. |

## 📤 Role Output

This role does not set any public output facts. Task-level registered variables use the `__postgresql_` prefix; internal defaults and constants are defined in `vars/`.

## 🚫 Scope Limits & Roadmap

The following topics are explicitly out of scope for this standalone role:
- **Backups**: Covered by `ansible-role-pg-backup`.
- **Replication / High Availability**: Patroni, repmgr, and streaming replication.
- **PgBouncer / Exporters**: Separate middleware roles.
- **Major Upgrades**: Single cluster lifecycle only; `pg_upgrade` is handled out of band.

## 🔍 Verification

### Check PostgreSQL Cluster Health

Verify service and database cluster accessibility:

```bash
# Check systemd status
sudo systemctl status postgresql

# Run pg_isready connection check
pg_isready -p 5432

# Verify catalog database access
sudo -u postgres psql -c "\l"
```

### Validate Logrotate Configuration Syntax

Verify logrotate configuration syntax by executing a dry-run check:

```bash
sudo logrotate -d -s /var/lib/logrotate/status /etc/logrotate.conf
```

## 🛡️ Security Features

- **Authentication**: `postgresql_auth_method` defaults to `scram-sha-256`. Legacy methods (`trust`, `md5`) are avoided for secure production operation.
- **Log Masking**: Database user creation tasks enforce `no_log: true` by default (`postgresql_users_no_log: true`) to prevent password exposure in execution logs.
- **File Permissions**: Cluster configuration drop-ins in `/etc/postgresql/<version>/main/conf.d/` are created with mode `0640` owned by `postgres:postgres`.
- **SSL Encryption**: SSL connection support is enabled by default (`postgresql_ssl_enabled: true`).

## 🧪 Check mode behavior

- Declarative argument specifications (`meta/argument_specs.yml`) and input validation tasks (`tasks/assert.yml`) run normally in Check Mode (`--check`).
- Package installation (`ansible.builtin.apt`) and configuration rendering (`ansible.builtin.template`) simulate changes without mutating remote state or writing files to disk.
- Template tasks show exact line diffs when diff mode (`--diff`) is enabled.

## 🌐 Network resilience

This role relies on external package repositories and network downloads for PostgreSQL installation and repository keys. Key retrieval (`ansible.builtin.get_url`) and package installation tasks include automatic retry logic to withstand transient network failures during updates and package downloads.

## 🧰 Repository management

The role manages the official PostgreSQL Global Development Group (PGDG) apt repository at `apt.postgresql.org` (`https://apt.postgresql.org/pub/repos/apt`).
- Keyring location: `/usr/share/keyrings/postgresql-archive-keyring.gpg`
- Repository source file: `/etc/apt/sources.list.d/pgdg.list`
- Repository setup can be disabled by setting `postgresql_use_pgdg_repo: false` to use standard OS distribution packages.

## 🔧 Troubleshooting

### Check Cluster Status
```bash
sudo systemctl status postgresql
pg_lsclusters
```

### Inspect Log Files
```bash
# Cluster data log directory
ls -la /var/lib/postgresql/<version>/main/log/

# Cluster startup logs
ls -la /var/log/postgresql/
```

### Test Connection & Catalog Access
```bash
pg_isready -p 5432
sudo -u postgres psql -c "\l"
```

### Validate Logrotate Configuration Syntax
```bash
sudo logrotate -d -s /var/lib/logrotate/status /etc/logrotate.conf
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
│   └── default/                       # Molecule testing scenario
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

## 🏷️ Tags

| Tag | Description |
|---|---|
| `postgresql_setup` | Python prerequisites & locale generation |
| `postgresql_install` | PGDG apt repository and package installation |
| `postgresql_configure` | `99-ansible.conf` & `pg_hba.conf` rendering |
| `postgresql_databases` | Declarative databases, users, privileges, extensions |
| `postgresql_logrotate` | Log rotation configuration |
| `postgresql_test` | Readiness checks |

## CI/CD Pipeline

This repository uses centralized, reusable GitHub Actions workflows from [github-workflows](https://github.com/grzegorzfranus/github-workflows) (`@main`) for quality assurance, security scanning, and release automation.

### CI Pipeline (`ansible-ci.yml`)

Runs on every Pull Request in a two-tier gate pattern:

1. **Branch Name Lint** — enforces naming conventions (`feature/`, `bugfix/`, `fix/`, `hotfix/`, `release/`, `chore/`, `docs/`, `refactor/`, `test/`, `build/`, `ci/`, `perf/`, `revert/`)
2. **PR Title Lint** — enforces [Conventional Commits](https://www.conventionalcommits.org/) format (`feat:`, `fix:`, `ci:`, etc.)
3. **YAML Syntax Lint** — validates YAML formatting via `yamllint`
4. **Ansible Lint** — checks Ansible best practices and role standards
5. **Galaxy Metadata Validation** — verifies `meta/main.yml` schema and requirements (`ansible-meta-validate.yml`)
6. **Security Scanning** — TruffleHog secret detection and Trivy IaC scanning (`ansible-security.yml`)
7. **Molecule Integration Tests** — executes Molecule test matrix (`default` scenario) across supported distros (`ansible-molecule.yml`)
8. **Merge Check Gate** — single authoritative status check aggregating all results for branch protection

### Release & Publish Pipeline (`ansible-publish.yml`)

Automated via [Release Please](https://github.com/googleapis/release-please):

1. **Push to `main`** → Release Please creates or updates a Release PR with automated changelog generation
2. **Release PR Validation** → validates YAML syntax and actions schema before setting `Merge Check` status
3. **Merge Release PR** → creates Git version tag and GitHub Release automatically
4. **Ansible Galaxy Publish** → publishes tagged release to Ansible Galaxy via `ansible-publish.yml`

## Example Playbooks

```yaml
---
- name: Deploy Standalone PostgreSQL 18 Server
  hosts: db_servers
  become: true
  roles:
    - role: grzegorzfranus.postgresql
      vars:
        postgresql_version: "18"
        postgresql_shared_buffers: "1330MB"
        postgresql_effective_cache_size: "3975MB"
        postgresql_work_mem: "13MB"
        postgresql_maintenance_work_mem: "332MB"
        postgresql_auth_method: "scram-sha-256"
        postgresql_run_test: true
```

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`
- Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:
  - `feat:` — new features
  - `fix:` — bug fixes
  - `refactor:` — code refactoring
  - `docs:` — documentation changes
  - `ci:` — CI/CD pipeline updates
  - `build:` — dependency and build configuration updates
  - `chore:` — maintenance tasks
  - `test:` — test additions or corrections
  - `perf:` — performance improvements
  - `revert:` — code reverts
  - `style:` — code formatting and style
- Use branch naming convention: `feature/`, `bugfix/`, `fix/`, `hotfix/`, `release/`, `chore/`, `docs/`, `refactor/`, `test/`, `build/`, `ci/`, `perf/`, `revert/`
- Ensure your code passes all CI checks (YAML lint, Ansible lint, Molecule tests)
- Centralized workflows from [github-workflows](https://github.com/grzegorzfranus/github-workflows) are used to run CI/CD pipelines
- Submit a pull request describing your changes (a template is available under `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md` to help structure your PR description)
- For major changes, please open an issue first to discuss what you would like to change (issue templates for bug reports, feature requests, and tasks are available under `.github/ISSUE_TEMPLATE/`)

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).
