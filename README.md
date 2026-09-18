# Ansible Role: Puppet

Ansible Galaxy role for installing and configuring the open-source Puppet Server 7 and Puppet Agent 7 on supported Ubuntu systems.

## Requirements

- Ansible >= 2.14
- Ubuntu 20.04 (Focal)
- Ubuntu 22.04 (Jammy)
- Puppet 7 repository

Molecule tests are executed in GitHub Actions.

## Supported platforms

| Ubuntu | Codename | Puppet 7 |
|--------|----------|----------|
| 20.04  | Focal    | Supported |
| 22.04  | Jammy    | Supported |

Ubuntu 24.04 is not currently supported by this role with Puppet Server 7.

## Role variables

### Puppet version

```yaml
puppet_version: "7"
```

### Puppet Server

```yaml
puppetserver_package_name: "puppetserver"
puppetserver_service_name: "puppetserver"

puppetserver_certname: "puppet"
puppet_environment: "production"
puppetserver_port: 8140
```

### Repository

```yaml
puppet_manage_repository: true
puppet_apt_update: true

puppet_repository_url: "https://apt.puppet.com"
puppet_repository_component: "puppet7"
```

The repository release package is selected automatically according to the Ubuntu distribution release.

### JVM memory

By default, the role keeps the Puppet Server package defaults:

```yaml
puppetserver_java_min_heap: ""
puppetserver_java_max_heap: ""
```

To override the JVM heap:

```yaml
puppetserver_java_min_heap: "512m"
puppetserver_java_max_heap: "1024m"
```

Additional JVM arguments:

```yaml
puppetserver_java_extra_args:
  - "-Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

The role modifies only `JAVA_ARGS` in `/etc/default/puppetserver` and preserves the remaining package-managed configuration.

### Puppet command symlink

The role can create a convenient symlink:

```yaml
puppet_manage_command_symlink: true
puppet_command_symlink: "/usr/local/bin/puppet"
puppet_command_path: "/opt/puppetlabs/bin/puppet"
```

To disable it:

```yaml
puppet_manage_command_symlink: false
```

## Example

```yaml
---
- name: Install Puppet Server
  hosts: puppet_servers
  become: true

  roles:
    - role: ildar.puppet
      vars:
        puppetserver_certname: "puppet"
        puppet_environment: "production"
        puppetserver_java_min_heap: "512m"
        puppetserver_java_max_heap: "1024m"
```

## What the role does

The role:

1. Validates the target Ubuntu release.
2. Installs the required APT packages.
3. Configures the official Puppet APT repository.
4. Installs Puppet Server.
5. Installs Puppet Agent.
6. Configures `/etc/puppetlabs/puppet/puppet.conf`.
7. Configures Puppet Server JVM arguments.
8. Enables and starts the Puppet Server systemd service.
9. Optionally creates `/usr/local/bin/puppet`.

## Puppet configuration

The generated `puppet.conf` contains:

```ini
[main]
certname = puppet
server = puppet
environment = production

[server]
certname = puppet
port = 8140
```

Values can be customized through role variables.

## Testing

Molecule is used for integration testing.

Molecule is intentionally not required for local development. Tests are executed in GitHub Actions.

The CI pipeline runs:

```text
Lint
  ├── yamllint
  └── ansible-lint

Molecule
  ├── syntax
  ├── create
  ├── converge
  ├── verify
  └── destroy
```

The Molecule scenario uses Docker with Ubuntu 20.04 or Ubuntu 22.04.

## Local linting

Install the required tools:

```bash
python -m pip install --upgrade pip
pip install ansible-core ansible-lint yamllint
```

Run YAML linting:

```bash
yamllint -c .config/yamllint.yml .
```

Run Ansible linting:

```bash
ansible-lint -c .config/ansible-lint.yml .
```

Molecule itself is intended to run in GitHub Actions.

## Repository structure

```text
.
├── .config/
│   ├── ansible-lint.yml
│   └── yamllint.yml
├── .github/
│   └── workflows/
│       └── ci.yml
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── molecule/
│   └── default/
│       ├── Dockerfile
│       ├── molecule.yml
│       ├── converge.yml
│       └── verify.yml
├── tasks/
│   ├── main.yml
│   ├── repository.yml
│   ├── install.yml
│   ├── configure.yml
│   └── service.yml
├── templates/
│   └── puppet.conf.j2
├── .gitignore
├── LICENSE
└── README.md
```

## License

MIT
