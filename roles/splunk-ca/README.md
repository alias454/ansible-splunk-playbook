# CA Role Overview and Purpose

By default Splunk has existing certs but those all use the same root key and can be prone to MITM attacks.  
I decided as a preparation for running a Splunk cluster, I needed a way to manage certificates and distribute them between Splunk nodes.  
This role is a heavily modified version of what can be found at the below links
  - https://www.tikalk.com/posts/2016/10/30/Running-Your-Own-Ansible-Driven-CA/
  - https://github.com/shelleg/ansible-role-ca

This role can be used to manage CA services for other purposes as well.  

The general process for manually handling certs and certificate signing requests(CSR), looks like this:
1. Generate private key on client node
2. Generate CSR on client node using clients private key
3. Send CSR to certificate authority(CA)
4. Generate client cert from CSR using CA key pair
5. Return CA public key and client cert

Assuming the client already trusts the CA, it is pretty simple. However, when the CA is untrusted, its cert also has to be installed on the client.  
This role attempts to automate the above process, making it easier to manage internal CA requests.

## CA Role Objectives

1. Configure the CA server
    1. Install OpenSSL on the CA server host
    2. Generate CA private key
    3. Generate CA public key
    4. Generate CA certificate using private key
2. Generate private keys, csr, and certs for each node (including the CA server itself)
3. Generate a wildcard cert (optionally configured)
4. Distribute the CA cert and the Host certificates to clients (optionally configured)
5. Configure services to use the generated certs and keys (happens in splunk-base)

### The playbook will configure:
  - splunk-base
    - splunk-ca : Check if ca.key and ca.pem certificate exists
    - splunk-ca : Ensure existing certificate is still valid 2 weeks from now
    - splunk-ca : Generate an OpenSSL CA private key.
    - splunk-ca : Generate a Self Signed OpenSSL CA certificate.
    - splunk-ca : Generate an OpenSSL private key for client nodes.
    - splunk-ca : Generate an OpenSSL CSR for client nodes.
    - splunk-ca : Generate a Signed OpenSSL certificate for client nodes.
    - splunk-ca : Copy keys and certs from CA to ansible machine for distribution
    - splunk-ca : Distribute keys and certs to hosts

### Files included in this role:

    splunk-ca/
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   ├── certify_nodes.yml
    │   ├── check-ca-certs.yml
    │   ├── deploy-ca-certs.yml
    │   ├── deploy-ca-config.yml
    │   ├── distribute_keys.yml
    │   ├── fetch_keys.yml
    │   ├── main.yml
    │   └── wildcard_cert.yml
    └── templates
        ├── ca.conf.j2
        └── serial.j2

## How does this work?

The inventory is based on the **hosts.example** file located within the playbook.  
However, inventory can be managed in other ways as long as a server is identified to act as the CA and there are other client nodes

In the **defaults/main.yml**, configure the settings to utilize different options and configurations.  
To enable this role set the following **use_internal_ca** to true.
```yaml
# defaults file for splunk-ca
use_internal_ca: 'true'
```

Each of the below settings toggle some feature on or off depending on the bool setting.
```yaml
# ca_force_create and ca_force_certify_nodes are designed to prevent overriding of existing certificates !
ca_init: true                    # Install and configure the root CA (from scratch)
ca_force_create: false           # Force creating even if files exist on the node
ca_force_certify_nodes: false    # Force creating of node certificates
ca_certify_nodes: true           # Generate certs for nodes
ca_wildcard: true                # Generate wildcard cert
ca_fetch_keys: true              # Copy key to control machine
ca_distribute_keys: false        # Copy host keys and certs from control machine to nodes
ca_distribute_ca_cert: true      # Copy CA cert from control machine to nodes
ca_distribute_root_keys: true    # Copy CA cert and key to default locations on CA server
```

The below settings designate the default locations for certs and key files. 
```yaml
# Where the certs go
ca_certs_dir: "/usr/local/share/ca-certificates"
ca_default_ssl_certs_dir: "/etc/ssl/certs"
ca_default_ssl_key_dir: "/etc/ssl/private"
ca_conf_dir: "/etc/ssl/conf"
ca_generated_certs_dir: "/etc/ssl/ca-certs"
```

The below settings designate the location for generated certs on the ansible control machine. 
```yaml
# Where to copy the generated cert files to locally
ca_distribution_dir: "{{ playbook_dir }}/group_vars/secrets/certs"
```

Configure a panic threshold were the CA certs automatically get regenerated and a default days until expired (5 years).
```yaml
# how long should remain before we generate a new CA cert ?
ca_cert_expiration_panic_threshold: 1209600 # 2 weeks
ca_default_days: "1825"
```

The names for generated certs and keys on the CA server.
```yaml
# CA filenames
ca_cert: ca-cert.pem
ca_key: ca-key.pem
ca_pub_key: ca-pubkey.pem
```

These settings are overridden by default as part of the overall splunk deployment.  
The variable overrides are located in the group_vars/secrets.yml file. 
```yaml
# Should match the unique part of the CA's hostname
stage.ca.ca_descriptor: "CA-servername"

# Set under the appropriate environment variable in the secrets file
default_stage:
  ca:
    ca_domain: "domain.tld"
    ca_descriptor: "CA-servername"
    ca_country: "US"
    ca_state: "Illinois"
    ca_locality: "Chicago"
    ca_organization: "myOrg"
    ca_organizational_unit: "Security"
    ca_email: "splunk-ca-admin@domain.tld"
    ca_sslPassword: 'password'
```