# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.2.2...v1.3.0) (2026-08-18)


### Features

* **logrotate:** add archive directory and date suffix support ([#30](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/30)) ([678a8a0](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/678a8a0bd97cd0a8d37467a6a10f4cfaa30510cb))

## [1.2.2](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.2.1...v1.2.2) (2026-08-14)


### Miscellaneous

* **26:** align workflows, lint config and README with logrotate role ([#27](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/27)) ([ce231a9](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/ce231a9e3108892077c3201290881373045cd2df))

## [1.2.1](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.2.0...v1.2.1) (2026-08-14)


### Bug Fixes

* **23:** stop claiming /var/log/postgresql in the logrotate drop-in ([#24](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/24)) ([3f51e2a](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/3f51e2a1eb674d2f667d7b703fbdb550d5005188))

## [1.2.0](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.1.4...v1.2.0) (2026-07-24)


### Features

* **postgresql:** add parallel worker and query planner tuning variables ([#18](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/18)) ([603e14c](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/603e14c74c49a680d326921a8ecdf4a1757b7d4b))

## [1.1.4](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.1.3...v1.1.4) (2026-07-24)


### Bug Fixes

* **postgresql:** add cache_valid_time to apt update task in repository setup ([#14](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/14)) ([3242801](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/32428011330a9758f3e2be15ccefbe0c611029a5))

## [1.1.3](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.1.2...v1.1.3) (2026-07-24)


### Bug Fixes

* **postgresql:** update postgresql_ping login_port and pre-create postgres remote_tmp directory ([#11](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/11)) ([18ccf68](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/18ccf685ec61252ac7812318bcc6eda31cd272df))

## [1.1.2](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.1.1...v1.1.2) (2026-07-24)


### Bug Fixes

* **postgresql:** add su directive to logrotate configuration template ([#8](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/8)) ([44a3baf](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/44a3baff2195700a182008867fe4acfeb6fdf8dd))

## [1.1.1](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.1.0...v1.1.1) (2026-07-24)


### Bug Fixes

* **postgresql:** allow logrotate configuration without requiring logging_collector ([#5](https://github.com/grzegorzfranus/ansible-role-postgresql/issues/5)) ([33504fa](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/33504fab539485c77047195068bc7a38c1e508b6))

## [1.1.0](https://github.com/grzegorzfranus/ansible-role-postgresql/compare/v1.0.0...v1.1.0) (2026-07-23)


### Features

* **postgresql:** implement standalone postgresql ansible role ([66ec59d](https://github.com/grzegorzfranus/ansible-role-postgresql/commit/66ec59d21dc580e745baf2674f868239425e889e))

## [1.0.0] - 2026-07-23

### Features

- Initial release of standalone PostgreSQL Ansible role supporting Ubuntu 24.04 and Ubuntu 26.04.
- Official PGDG apt repository deb822 integration for PostgreSQL versions 14 through 18.
- Upgrades-safe drop-in `99-ansible.conf` configuration.
- Declarative `pg_hba.conf` authentication with `scram-sha-256` defaults.
- Declarative databases, users (`no_log`), privileges, and extensions via `community.postgresql`.
- Comprehensive Molecule testing scenario with Ubuntu 24.04 and 26.04 image targets.
