# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-07-23

### Features

- Initial release of standalone PostgreSQL Ansible role supporting Ubuntu 24.04 and Ubuntu 26.04.
- Official PGDG apt repository deb822 integration for PostgreSQL versions 14 through 18.
- Upgrades-safe drop-in `99-ansible.conf` configuration.
- Declarative `pg_hba.conf` authentication with `scram-sha-256` defaults.
- Declarative databases, users (`no_log`), privileges, and extensions via `community.postgresql`.
- Comprehensive Molecule testing scenario with Ubuntu 24.04 and 26.04 image targets.
