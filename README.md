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
"{{ exporter_vars_prefix }}_package": <string>  # Tarball package format (use '_version' to get latest version)
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
