# CVMFS setup

This repository contains ansible roles for setting up a computer as a part of a CVMFSnetwork. The supported roles are:

- stratum0 (the main repository server for all software)
- stratum1 (a distribution node to offload stratum0)
- proxy (offloads stratum1 servers)
- client (a computer that actually uses the CVMFS software for actual work)

# Using these roles

# Requirements / dependencies

These roles have the following external dependencies:
  - ansible v2.12.0 or higher
  - jinja v3.0.0 or higher

First create or update a requirements file as such:

#### **`requirements.yaml`**
``` yaml
roles:
  - name: cvmfs.roles
    src: https://github.com/bedroge/ansible-cvmfs-roles
    version: develop
```

Now fetch dependencies by doing `ansible-galaxy install -r requirements.yml`. Add `--force` to guarantee updating the requirements. This is useful if your version is a branch like `develop`. 

Then create a playbook for your host type, following one of the examples below. Create (or reuse) a suitable inventory file. Then run as `ansible-playbook playbook.yaml -b -i inventory`.

# Playbook examples
## Stratum0

#### **`playbook.yaml`**
``` yaml
- hosts: stratum0
  roles:
  - cvmfs.stratum0

  vars:
    # Storage device
    cvmfs_srv_device: /dev/sdb
```

## Stratum1
#### **`playbook.yaml`**
``` yaml
- hosts: stratum1
  roles:
  - cvmfs.stratum1

  vars:
    # The license key for the Geo API:
    # https://cvmfs.readthedocs.io/en/stable/cpt-replica.html#geo-api-setup
    cvmfs_geo_license_key: INSERT_YOUR_KEY

    # Everything after this is optional. The defaults should provide a working stratum1
    # with local monitoring enabled.

    # Set the node as a public node. If the node is set as public the prometheus
    # instance will allow other prometheus instances to connect to scrape data.
    cvmfs_public: false

    # Enable monitoring on nodes, defaults to true.
    # This will install grafana, prometheus, node_exporter, and cvmfs_exporter
    # on the monitored node.
    cvmfs_monitoring: true

    # All services bind to localhost, unless cvmfs_public is set to
    # true, in that case grafana and prometheus binds to "*". 
    cvmfs_service_ports:
      grafana: 3000
      prometheus: 9090
      node_exporter: 9100
      cvmfs_exporter: 9101

    # If you want to use your own Squid configuration template for the Stratum 1
    local_stratum1_cvmfs_squid_conf_src: "/path/to/your/template"
```

## Proxy
#### **`playbook.yaml`**
``` yaml
- hosts: squid_proxy
  roles:
  - cvmfs.proxy

  vars:
    # List of clients allowed to access your local proxies.
    # Add individual IPs and/or use CIDR notation.
    local_cvmfs_http_proxies_allowed_clients:
      - 192.168.0.0/12
      - 10.0.0.15

    # If you want to use your own Squid configuration template for the local proxies
    local_proxies_cvmfs_squid_conf_src: "/path/to/your/template"
```

## Clients

#### **`playbook.yaml`**
``` yaml
- hosts: client
  roles:
  - cvmfs.client

  vars:
    # List of all http proxies that should be configured for the clients.
    # Remove or comment the line if you do not want to use a proxy; in this
    # case it will be set to "DIRECT" in the client configuration.
    local_cvmfs_http_proxies:
      - your-proxy-1:3128
      - your-proxy-2:3128
```

# Running the playbooks

`ansible-playbook -i inventory playbook.yaml` 

Avoid using `-b`, the roles will sudo if required.
