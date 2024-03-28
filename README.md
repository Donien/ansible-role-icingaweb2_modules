# icingaweb2_modules

This role is meant to **manage Icinga Web 2 modules** which are not managed by the [**official Ansible Collection by Icinga**](https://github.com/Icinga/ansible-collection-icinga/).  
It tries to follow the same general structure as its inspiration but also adds to it.

> You have to ensure compatibility between module versions you install and the rest of your system. The module "grafana" needs a specific version of PHP for example.

Tasks it can do:

* Install Icinga Web 2 modules from source, git or tar ball
* Enable/Disable modules
* Configure modules

Table of contents:

* [Variables](#variables)
  * [General](#general)
  * [Grafana](#grafana)
* [Example Playbooks](#example-playbooks)
  * [Grafana Playbooks](#grafana-playbooks)

# Variables

## General

- `icingaweb2_modules_config_path`: `string`  
  The path where your Icinga Web 2 configuration is stored.  
  Default: `/etc/icingaweb2`

- `icingaweb2_modules_path`: `string`  
  The path where your Icinga Web 2 modules are located.  
  Default: `/usr/share/icingaweb2/modules`

- `icingaweb2_modules`: `dictionary`  
  This variable holds keys for all modules to be managed. Each key is a dictionary itself and holds information regarding that specific module.  
  Each module might provide a specific set of options to be used. The handling of those options happens within that module's task file (e.g. "*tasks/configure_`module`*").  
  All module keys share a common set of options that handle the installation of the modules.  
  Default: `none`  
  Common options:
    - `source`: `string`  
      The installation method (`package | git | archive`).  
      **Required**
    - `package_name`: `string`  
      The name of the package to be installed if different from the module's name.  
      **Important:** you **should** set this key regardless of `source` to avoid accidentally removing a package with the same name as your module (e.g. "*grafana*").
    - `url`: `string`  
      The URL to either the Git repository or the tar ball.  
      **Required if `source: git | archive`**
    - `version`: `string`  
      The version / tag for Git installation (e.g. `version: "v2.0.2"`)
    - `enabled`: `boolean`  
      If `true`, enables the module. If `false`, disables the module. If not set, does nothing.

## Grafana

- `icingaweb2_modules.grafana`: `dictionary`  
  - `backend`: `string`  
    The backend used in Grafana (`influxdb | graphite `)  
    Default: `influxdb`
  - `local_dashboards`: `string`  
    The directory where your local dashboards are located on your Ansible Controller (uses `lookup`).  
    Default: `none`
  - `data_source`: `string`  
    The name of your data source within Grafana.  
    Default: `none`
  - `grafana_host`: `string`  
    The Ansible inventory hostname of your Grafana server. It is used to delegate tasks for dashboard imports.  
    Default: `icingaweb2_modules.grafana.config.grafana.host`
  - `config.grafana`: `dictionary`  
    Manages *config.ini* to configure the module. Its keys are equal to the modules [configuration file](https://github.com/Mikesch-mp/icingaweb2-module-grafana/blob/master/doc/03-module-configuration.md#example-configini-etcicingaweb2modulesgrafanaconfigini), though not necessarily complete yet.  
    For possible default values have look at [templates/grafana/config.ini.j2](templates/grafana/config.ini.j2).
  - `config.graphs`
    Manages *graphs.ini* to configure the module. Its keys are equal to the modules [graphs file](https://github.com/Mikesch-mp/icingaweb2-module-grafana/blob/master/doc/04-graph-configuration.md#options), though not necessarily complete yet.  
    Here you can set specialized dashboards for specific Icinga Services or CheckCommands.  
    For possible default values have look at [templates/grafana/graphs.ini.j2](templates/grafana/graphs.ini.j2).

- `icingaweb2_modules_grafana_import_dashboards`: `boolean`  
  The path where your Icinga Web 2 modules are located.  
  Default: `/usr/share/icingaweb2/modules`

- `icingaweb2_modules_grafana_server_config_path`: `string`  
  The path where your Icinga Web 2 modules are located.  
  Default: `/usr/share/icingaweb2/modules`

- `icingaweb2_modules_grafana_server_dashboard_path`: `string`  
  The path where your Icinga Web 2 modules are located.  
  Default: `/usr/share/icingaweb2/modules`

# Example Playbooks

## Grafana Playbooks

Install grafana module via package management. Use basic authentication for connection to the Grafana server. Use existing dashboard.

```yaml
- name: Manage Grafana Module
  hosts: 
    - icingaweb2

  vars:
    icingaweb2_modules:
      grafana:
        # Avoid removing Grafana as in "the Grafana server"
        package_name: "icinga-grafana"
        source: package
        package_name: icinga-grafana
        data_source: "influxdb_icinga"
        config:
          grafana:
            host: "grafana.localdomain:3000"
            defaultdashboard: "icinga2-default"
            defaultdashboarduid: "a515ad9f-20e2-4b4f-b068-24f3eb3e30ba"
            authentication: "basic"
            username: "admin"
            password: "12345678"

  roles:
    - icingaweb2_modules
```

---

Install grafana module via Git and configure connection to Grafana server. Also import dashboards to Grafana server.

```yaml
- name: Manage Grafana Module
  hosts: 
    - icingaweb2

  vars:
    icingaweb2_modules_grafana_import_dashboards: true
    icingaweb2_modules:
      grafana:
        # Avoid removing Grafana as in "the Grafana server"
        package_name: "icinga-grafana"
        source: git
        url: "https://github.com/Mikesch-mp/icingaweb2-module-grafana.git"
        version: "v2.0.2"
        enabled: true
        data_source: "influxdb_icinga"
        config:
          grafana:
            host: "localhost:3000"
            defaultdashboard: "icinga2-default"
            authentication: "token"
            token: "glsa_VBTDQJ998i6fHPxeeYrvFrKBXfjHjaGx_4e52ed40"
            enableLink: true

  roles:
    - icingaweb2_modules
```
