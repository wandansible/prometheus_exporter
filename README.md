### Prometheus exporter base role

This role handles the common tasks associated with installing Prometheus exporters.
Other Prometheus exporter roles can be built on top of this for less duplicate Ansible configuration.

**What this role handles**

* Adding the exporter user
* Retrieving the latest version of the exporter
* Installing the exporter binary
* Adding the exporter systemd unit
* Configures Caddy to reverse proxy requests to the exporter
* Registering the exporter with Prometheus servers

**How to use this role**

Inside your own prometheus exporter role you will need to include this role:

```yaml
# tasks/main.yml:
- name: Apply base prometheus exporter role
  ansible.builtin.import_role:
    name: prometheus_exporter

# vars/main.yml:
exporter_name: EXPORTER_NAME
```

Note: replace `EXPORTER_NAME` with the name of the exporter sub-role appended by `_exporter`.

By default, `exporter_vars_prefix` will be set to `exporter_name`.
Certain vars prefixed with `exporter_vars_prefix`
will override the vars in the base role.
When including the following vars in the role, make sure to replace
`EXPORTER_NAME` with the value you have assigned to `exporter_name`
or `exporter_vars_prefix` if you have assigned a value to it.
It is recommended to include these vars in `defaults`.

Required vars:

```yaml
# vars/main.yml:
exporter_name: <string>  # Set this in task vars not defaults

# defaults/main.yml:
EXPORTER_NAME_port: <port>
EXPORTER_NAME_flags: <string or list>
```

You must also include a handler for restarting the exporter:

```yaml
# handlers/main.yml
- name: restart EXPORTER_SERVICE
  ansible.builtin.service:
    name: EXPORTER_SERVICE
    state: restarted
```

Note: replace `EXPORTER_SERVICE` with the value you have assigned to `exporter_name`
or `EXPORTER_NAME_service` if you have assigned a value to it.

Example of the required vars and handler:

```yaml
# tasks/main.yml:
- name: Apply base prometheus exporter role
  ansible.builtin.import_role:
    name: prometheus_exporter

# vars/main.yml:
exporter_name: node_exporter

# defaults/main.yml:
node_exporter_port: 9100
node_exporter_flags: "{{ lookup('template', 'flags') }}"

# handlers/main.yml
- name: restart node_exporter
  ansible.builtin.service:
    name: node_exporter
    state: restarted
```

Common vars:

```yaml
# These will be necessary for non-official Prometheus exporters
EXPORTER_NAME_github_org: <string>
EXPORTER_NAME_github_repo: <string>
```

Other useful vars:

```yaml
# These may be necessary if format is slightly different to official Prometheus exporters
EXPORTER_NAME_binary: <string>              # Name of exporter binary
EXPORTER_NAME_checksums_filename: <string>  # Checksums filename
EXPORTER_NAME_strip_components: <int>       # May need to change this if src tarball doesn't contain a directory

# Add basic auth users for all exporters
caddy_basic_auth_users:
  - username: <string>
    hash: <string>
    hash_scheme: <string>  # E.g. bcrypt

# Control what the base role performs
EXPORTER_NAME_install: <bool>          # Install exporter
EXPORTER_NAME_configure_caddy: <bool>  # Add Caddy reverse proxy for exporter
EXPORTER_NAME_register: <bool>         # Register exporter target with Prometheus servers
```

Register vars:

```yaml
# Global vars
prometheus_labels: <dict>
prometheus_scrape_servers: <list>

# Exporter specific vars
EXPORTER_NAME_labels: <dict>
EXPORTER_NAME_scrape_servers: <list>
```

Each exporter role must add the following definitions
to their argument specs (`meta/argument_specs.yml`)
and replace `EXPORTER_NAME` with the value you have assigned to `exporter_name`
or `exporter_vars_prefix` if you have assigned a value to it:

```yaml
      EXPORTER_NAME_port:
        description: Listen port
        type: int
        # default: <port>  # Set a default port

      EXPORTER_NAME_file_sd_dir:
        description: Directory, on scrape servers, for the file service discovery target
        type: str
        # default: <dir>  # Set a default directory

      EXPORTER_NAME_scrape_servers:
        description: |
          List of servers that scrape exporter metrics from the host,
          overrides prometheus_scrape_servers
        type: list
        elements: str

      EXPORTER_NAME_labels:
        description: |
          Labels added to exporter metrics,
          overrides prometheus_labels
        type: dict

      EXPORTER_NAME_install:
        description: If true, install exporter
        type: bool
        default: true

      EXPORTER_NAME_configure_caddy:
        description: If true, configure caddy to add a TLS endpoint for the exporter
        type: bool
        default: false

      EXPORTER_NAME_register:
        description: If true, register the exporter with the scrape servers
        type: bool
        default: false

      EXPORTER_NAME_user:
        description: Name of the exporter unix user
        type: str

      EXPORTER_NAME_group:
        description: Name of the exporter unix group
        type: str

      EXPORTER_NAME_groups:
        description: Unix groups added to exporter unix user
        type: list
        elements: str

      EXPORTER_NAME_manage_user:
        description: If true, add exporter unix user and group
        type: bool
        default: true

      EXPORTER_NAME_bin_dir:
        description: Directory for the exporter executable
        type: str

      EXPORTER_NAME_src_dir:
        description: Directory for the downloaded exporter src archive
        type: str

      EXPORTER_NAME_strip_components:
        description: Strip NUMBER leading components from file names on extraction
        type: int
        default: 1

      EXPORTER_NAME_clean_src_dir:
        description: Remove old downloaded archive files from exporter src directory
        type: bool
        default: true

      EXPORTER_NAME_github_org:
        description: Name of organisation for exporter github repository
        type: str

      EXPORTER_NAME_github_repo:
        description: Name of exporter github repository
        type: str

      EXPORTER_NAME_github_checksum_filename:
        description: Filename for the exporter package checksums file on github
        type: str

      EXPORTER_NAME_archive_urls:
        description: Override the list of exporter archive urls for different platforms and architectures
        type: list
        elements: str

      EXPORTER_NAME_version:
        description: Version to install (use "latest" for the latest version)
        type: str
        default: latest

      EXPORTER_NAME_arch_map:
        description: Mapping of the possible values of ansible_architecture to the exporter package architectures
        type: dict

      EXPORTER_NAME_binary:
        description: Filename for the exporter executable
        type: str

      EXPORTER_NAME_checksum_url:
        description: Override the URL for the exporter checksum file
        type: str

      EXPORTER_NAME_checksum_type:
        description: The exporter package checksum type
        type: str

      EXPORTER_NAME_checksums:
        description: Override exporter archive checksums file contents
        type: str

      EXPORTER_NAME_service:
        description: Name of the exporter systemd service
        type: str

      EXPORTER_NAME_handler:
        description: Name of the exporter handler to notify
        type: str

      EXPORTER_NAME_description:
        description: Description for the exporter systemd service
        type: str

      EXPORTER_NAME_flags:
        description: List of flags to run exporter with, as string or list
        type: raw

      EXPORTER_NAME_target:
        description: Scrape target hostname and port
        type: str
```
