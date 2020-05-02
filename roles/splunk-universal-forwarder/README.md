# Universal Forwarder Overview and Purpose

Forwarders are Splunk Enterprise instances that receive data inputs and then consolidate the data prior to sending it on to a Splunk Enterprise indexer.

Forwarders come in two flavors **Universal Forwarders** and **Heavy Forwarders** but this role only manages the former.
Universal Forwarders have a separate installation package that maintains a small footprint on a host machine.
Universal Forwarders perform minimal processing on incoming data streams before forwarding data on to an indexer, also known as the receiver.

Forwarders tag data with metadata such as **host**, **source**, and **sourcetype** before forwarding data to an indexer.
They also allow efficient resource utilization when processing large amounts or different types of data coming from remote sources.
Forwarders support capabilities for **load balancing**, **data filtering**, and **routing** too.

Heavy forwarders are a full installation of Splunk Enterprise and retain most of the basic functionality.
They can parse data before forwarding it to the receiving indexer, store indexed data locally, and/or forward the parsed data to a receiver for final indexing on that machine as well.

## Splunk Universal Forwarder Role Objectives

1. Configure the Splunk Universal Forwarder
    1. Configure the Splunk Universal Forwarder
    2. Connect the Universal Forwarder to a Deployment Server

### The playbook will configure:
  - universal-forwarder
    - splunk-universal-forwarder : Install Splunk from .deb package
    - splunk-universal-forwarder : Fix up perms after splunk user created
    - splunk-universal-forwarder : Sets default acl for splunk on /var/log
    - splunk-universal-forwarder : Ensure directory exists for local self-signed TLS certs.
    - splunk-universal-forwarder : Remove a cert bundle if it exists to allow creation of new
    - splunk-universal-forwarder : Distribute wildcard-priv-key.pem, wildcard-cert.pem to host
    - splunk-universal-forwarder : Generate a cert bundle from wildcard artifacts
    - splunk-universal-forwarder : Copy Splunk inputs.conf for system local
    - splunk-universal-forwarder : Copy Splunk web.conf for system
    - splunk-universal-forwarder : Use splunk hash-passwd to return a password hash
    - splunk-universal-forwarder : Check for user seed file
    - splunk-universal-forwarder : Check for passwd file
    - splunk-universal-forwarder : Create default admin user seed file
    - splunk-universal-forwarder : Accept Splunk license
    - splunk-universal-forwarder : Setup Splunk to start at boot with splunk user
    - splunk-universal-forwarder : Copy Splunk init for service
    - splunk-universal-forwarder : Check passwd file contains current admin hash value
    - splunk-universal-forwarder : Reset Splunk admin password if changed
    - splunk-universal-forwarder : Set sslPassword using splunk_cluster_sslPassword
    - splunk-universal-forwarder : Add serverCert to system/local server.conf
    - splunk-universal-forwarder : Add sslRootCAPath to system/local server.conf
    - splunk-universal-forwarder : Copy Splunk deploymentclient.conf for system

### Files included in this role:

    splunk-universal-forwarder/
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    │   └── splunkforwarder-7.1.0-2e75b3406c5b-linux-2.6-amd64.deb
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   ├── certs.yml
    │   ├── config.yml
    │   ├── deploy.yml
    │   ├── ds_client.yml
    │   ├── main.yml
    │   └── service.yml
    └── templates
        ├── splunk.init.j2
        ├── splunk.service.j2
        ├── system_local.deploymentclient.j2
        └── system_local.web.j2

## How does this work?

Nodes can be configured for individual capabilities based on their inventory group.  
The inventory is based on the **hosts.example** file located within the playbook.  
However, inventory can be managed in other ways.

In the **defaults/main.yml**, configure the settings to utilize different options and configurations.  
Options can be overridden in higher level configs or a secrets file.

Define the base directory where splunkforwarder is installed.
```yaml
splunk_uf_base: '/opt/splunkforwarder'
```

Use wget to grab the latest package from splunk.com and place the file under -> ***splunk-universal-forwarder/files***.
```yaml
# Use wget to grab the latest package from splunk.com
splunk_uf_package_file: 'splunkforwarder-7.3.3-7af3758d0d5e-linux-2.6-amd64.deb'
splunk_uf_version: '7.3.3'
splunk_uf_base: '/opt/splunkforwarder'
splunk_uf_product: 'universalforwarder'
```

Configure the Universal Forwarder to connect to a deployment server for app management.  
NOTE: If using custom certs the ca-cert and the wildcard cert will need to be deployed as part of the setup.
```yaml
# Manage apps using a deployment server.
# http://docs.splunk.com/Documentation/Splunk/7.1.0/Indexer/Updatepeerconfigurations
use_deployment_server: 1
```

To enable using internal certs set **use_internal_ca** to true.
```yaml
# Use Cluster Master as CA
use_internal_ca: 'true'
```

When toggling use_internal_ca setting you may want to use splunk defaults.
```yaml
# When toggling use_internal_ca setting you may want to use splunk defaults
use_stage_spec: false
```

Each of the below settings toggle some feature on or off depending on the bool setting.
```yaml
# ca_force_create and ca_force_certify_nodes are designed to prevent overriding of existing certificates !
ca_force_create: false           # Force creating even if files exist on the node
ca_force_certify_nodes: false    # Force creating of node certificates
```

Where to copy the generated cert files from on the local control machine.
```yaml
# Where to copy the generated cert files from on the local control machine
ca_distribution_dir: "{{ playbook_dir }}/group_vars/secrets/certs"
```
Disable splunk web by default.
```yaml
# Disable splunk web by default
use_splunk_web: 0
```