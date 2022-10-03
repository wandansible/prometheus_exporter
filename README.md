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

Recommended include task
(replace `EXPORTER_NAME` with the name of the exporter sub-role appended by `_exporter`):

```yaml
- name: Apply base prometheus exporter role
  ansible.builtin.include_role:
    name: prometheus_exporter
    public: true
  vars:
    exporter_name: EXPORTER_NAME
```

By default, `exporter_vars_prefix` will be set to `exporter_name`.
Certain vars prefixed with `exporter_vars_prefix`
will override the vars in the base role.
When including the following vars in the role, make sure to replace
`EXPORTER_NAME` with the value you have assigned to `exporter_name`
or `exporter_vars_prefix` if you have assigned a value to it.
It is recommended to include these vars in `defaults`.

Required vars:

```yaml
exporter_name: <string>  # Set this in task vars not defaults

EXPORTER_NAME_port: <port>
EXPORTER_NAME_flags: <string or list>
```

Required vars example:

```yaml
# tasks/main.yml:
- name: Apply base prometheus exporter role
  ansible.builtin.include_role:
    name: prometheus_exporter
    public: true
  vars:
    exporter_name: node_exporter

# defaults/main.yml:
node_exporter_port: 9100
node_exporter_flags: "{{ lookup('template', 'flags') }}"
```

Common vars:

```yaml
# These will be necessary for non-official Prometheus exporters
EXPORTER_NAME_git_org: <string>
EXPORTER_NAME_git_repo: <string>
```

Other useful vars:

```yaml
# These may be necessary if format is slightly different to official Prometheus exporters
EXPORTER_NAME_package: <string>  # Tarball package format (use 'exporter_selected_version' to get latest version)
EXPORTER_NAME_checksums_file: <string>  # Checksums filename
EXPORTER_NAME_add_extract_dir: <bool>  # Set to true if the package doesn't extract to a directory

# Add basic auth users for all exporters
caddy_basic_auth_users:
  - username: <string>
    hash: <string>
    hash_scheme: <string>  # E.g. bcrypt

# Control what the base role performs
EXPORTER_NAME_install: <bool>  # Install exporter
EXPORTER_NAME_configure_caddy: <bool>  # Add Caddy reverse proxy for exporter
EXPORTER_NAME_register: <bool>  # Register exporter target with Prometheus servers
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

`prometheus_exporter.caddy` must run before this role.
