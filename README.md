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

Recommended include task:

```yaml
- name: Apply base prometheus exporter role
  ansible.builtin.include_role:
    name: prometheus_exporter
    public: true
  vars:
    exporter_name: "{{ exporter_name }}"
```

By default, `exporter_vars_prefix` will be set to `exporter_name`.
Certain vars prefixed with `exporter_vars_prefix`
will override the vars in the base role.
It is recommended to include these vars in `defaults`.

Required vars:

```yaml
exporter_name: <string>  # Set this in task vars not defaults

"{{ exporter_vars_prefix }}_port": <port>
"{{ exporter_vars_prefix }}_flags": <string or list>

# For example:
exporter_name: node_exporter
node_exporter_port: 9100
node_exporter_flags: "{{ lookup('template', 'flags') }}"
```

Common vars:

```yaml
# These will be necessary for non-official Prometheus exporters
"{{ exporter_vars_prefix }}_git_org": <string>
"{{ exporter_vars_prefix }}_git_repo": <string>
```

Other useful vars:

```yaml
# These may be necessary if format is slightly different to official Prometheus exporters
"{{ exporter_vars_prefix }}_package": <string>  # Tarball package format (use 'exporter_selected_version' to get latest version)
"{{ exporter_vars_prefix }}_checksums_file": <string>  # Checksums filename
"{{ exporter_vars_prefix }}_add_extract_dir": <bool>  # Set to true if the package doesn't extract to a directory

# Add basic auth users for all exporters
caddy_basic_auth_users:
  - username: <string>
    hash: <string>
    hash_scheme: <string>  # E.g. bcrypt

# Control what the base role performs
"{{ exporter_vars_prefix }}_install": <bool>  # Install exporter
"{{ exporter_vars_prefix }}_configure_caddy": <bool>  # Add Caddy reverse proxy for exporter
"{{ exporter_vars_prefix }}_register": <bool>  # Register exporter target with Prometheus servers
```

Register vars:

```yaml
# Global vars
prometheus_labels: <dict>
prometheus_scrape_servers: <list>

# Exporter specific vars
"{{ exporter_vars_prefix }}_labels": <dict>
"{{ exporter_vars_prefix }}_scrape_servers": <list>
```

`prometheus_exporter.caddy` must run before this role.

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
        default: true

      EXPORTER_NAME_register:
        description: If true, register the exporter with the scrape servers
        type: bool
        default: true

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
        description: Directory for the exporter downloads
        type: str

      EXPORTER_NAME_add_extract_dir:
        description: If true, add an extraction directory for the exporter package
        type: bool
        default: false

      EXPORTER_NAME_git_org:
        description: Name of organisation for exporter git repository
        type: str

      EXPORTER_NAME_git_repo:
        description: Name of exporter git repository
        type: str

      EXPORTER_NAME_latest_url:
        description: URL for the latest version
        type: str

      EXPORTER_NAME_version:
        description: Version to install (use "latest" for the latest version)
        type: str
        default: latest

      EXPORTER_NAME_version_regex:
        description: Regular expression for capturing the version from the latest tag
        type: str

      EXPORTER_NAME_tag:
        description: Version git tag
        type: str

      EXPORTER_NAME_package_name:
        description: Name of the exporter package
        type: str

      EXPORTER_NAME_package:
        description: Filename of the exporter package (without extension)
        type: str

      EXPORTER_NAME_arch_map:
        description: Mapping of the possible values of ansible_architecture to the exporter package architectures
        type: dict

      EXPORTER_NAME_package_url:
        description: URL for the exporter package
        type: str

      EXPORTER_NAME_package_dir:
        description: Directory the exporter package is extracted to
        type: str

      EXPORTER_NAME_binary:
        description: Filename for the exporter executable
        type: str

      EXPORTER_NAME_checksums_url:
        description: URL for the exporter package checksums
        type: str

      EXPORTER_NAME_checksums_file:
        description: Filename for the exporter package checksums
        type: str

      EXPORTER_NAME_checksum_type:
        description: The exporter package checksum type
        type: str

      EXPORTER_NAME_checksum:
        description: The exporter package checksum
        type: str

      EXPORTER_NAME_service:
        description: Name of the exporter systemd service
        type: str

      EXPORTER_NAME_description:
        description: Description for the exporter systemd service
        type: str

      EXPORTER_NAME_flags:
        description: Contents or list of flags to run exporter with
        type: raw

      EXPORTER_NAME_target:
        description: Scrape target hostname and port
        type: str
```
