# Splunk playbook Objectives
This playbook deploys configuration changes to setup a Splunk cluster with independent components.

  - Configure splunk nodes with common configuration
  - Setup certificates on Splunk nodes
  - Install a license on the License Master
  - Configure a Deployment Server
  - Install basic custom apps
  - Configure a Cluster Master
  - Add Search Heads to a cluster
  - Add Peer Nodes to a cluster
  - Configure universal forwarder (standalone)

## Additional Role README info
- [Role Overview and Purpose splunk-ca](./roles/splunk-ca/README.md)
- [Role Overview and Purpose splunk-base](./roles/splunk-base/README.md)
- [Role Overview and Purpose splunk-license-master](./roles/splunk-license-master/README.md)
- [Role Overview and Purpose splunk-deployment-server](./roles/splunk-deployment-server/README.md)
- [Role Overview and Purpose splunk-cluster-master](./roles/splunk-cluster-master/README.md)
- [Role Overview and Purpose splunk-search-heads](./roles/splunk-search-heads/README.md)
- [Role Overview and Purpose splunk-peer-nodes](./roles/splunk-peer-nodes/README.md)
- [Role Overview and Purpose splunk-universal-forwarder](./roles/splunk-universal-forwarder/README.md)

## Usage Requirements

Prior to using this playbook there are a few things that should be handled.  
  - Make sure you have completed all the steps to [Setup Ansible on your control machine](../README.md)  
  - ansible > **v2.5** needs to be installed on the control machine (**you might need to upgrade**)  
  - The servers you intend to deploy Splunk on need to be built and running
  - Configure the inventory file like the **hosts.example** provided with this playbook
  - Clone this repo to your local machine
  - Read the Additional Role README info (all of them)
  - Downloaded Splunk package files and placed them in the correct folders (**use latest versions if new install**)
    - Splunk_TA_nix add-on
      - **roles/splunk-deployment-server/files/packages/splunk-add-on-for-unix-and-linux_xxx.tgz**
    - Grab any additional Splunk add-ons that may be in use and store them in the appropriate place
      - **roles/splunk-deployment-server/files/packages/**
  - Setup specific configurations in the secrets folder
    - Create the secrets file at **group_vars/secrets/secrets.yml** from the example secrets.example
    - Create the Splunk license file at **group_vars/secrets/Splunk.License.lic** using your Splunk license information
    - Update the custom_packages file at **group_vars/custom_files/custom_packages.yml** with additional add-ons/apps info

Run this in a test environment first **do not run in production without testing**

## Run the playbook like this:  
  >ansible-playbook /path/to/splunk-staging.yml -u username --become --ask-become-pass (--check)

If you have additional groups setup in your **/etc/ansible/hosts** file, you can specify different host groups by passing
**--extra-vars="servers=HOSTGROUP"**. Multiple vars are supported.
  >ansible-playbook /path/to/splunk-staging.yml --extra-vars="servers=HOSTGROUP" -u username --become --ask-become-pass (--check)

Extra vars:
  >servers  
  >lm_servers  
  >ds_servers  
  >cm_servers  
  >search_servers  
  >idx_servers  
  >uf_servers - Only for Universal Forwarder role  

Apply individual roles as needed using **tags** to run specific portions of the playbook.  
  >ansible-playbook /path/to/splunk-staging.yml --tags splunk-staging -u username --become --ask-become-pass (--check)  

### The playbook will configure:
- Setup base config on Splunk servers
  - common:
    - Install required packages using apt and pip modules
  - splunk-ca:
    - Check if CA key and certificate exist
    - Ensure existing certificate is still valid 2 weeks from now
    - Deploy openssl configuration items
    - Generate an OpenSSL CA private and public key
    - Generate a Self Signed OpenSSL CA certificate.
    - Generate an OpenSSL private key and CSR for client nodes
    - Generate a Signed OpenSSL certificate for client nodes
    - Generate an OpenSSL private key and CSR for wildcard cert
    - Generate a Signed OpenSSL certificate for wildcard cert
    - Copy keys and certs from CA to ansible machine for distribution
    - Distribute keys and certs to hosts
  - splunk-kernel:
    - Setup disable-transparent-hugepages service
    - Set swappiness to 10 in /etc/sysctl.conf
    - Set ulimits for splunk user
  - splunk-base:
    - Install Splunk from .deb package
    - Fix up perms after splunk user created
    - Sets default acl for splunk on /var/log
    - Distribute priv-key.pem, cert.pem, and ca-pubkey.pem to host
    - Remove a cert bundle if it exists to allow creation of new
    - Generate a cert bundle from artifacts or wildcard cert
    - Generate an OpenSSL private key for SplunkWeb
    - Generate an OpenSSL CSR for SplunkWeb
    - Generate a Self Signed OpenSSL certificate for SplunkWeb
    - Copy Splunk inputs.conf, web.conf, authentication.conf, and alert_actions.conf for system local
    - Create default admin user seed file
    - Accept Splunk license and set Splunk to start at boot using splunk user
    - Check passwd file contains current admin hash value
    - Reset Splunk admin password if changed
    - Set sslPassword using splunk_cluster_sslPassword
    - Add serverCert and sslRootCAPath to system/local server.conf
    - Add nodes to license master
    - TODO: Copy custom certs for Splunk Web
    - TODO: Configure LDAP for Splunk (optional)
- Setup config on Splunk License Master
  - splunk-license-master:
    - Create Enterprise licenses dir
    - Copy Splunk license file
    - Check if license already added
    - Install Splunk Enterprise license key
- Setup config on Splunk Deployment Server
  - splunk-deployment-server:
    - Enable Deployment Server
    - Create serverclass.conf to local system
    - Create initial org index apps directories
    - Create org_all_indexes app for deployment
    - Add custom mappings to indexes file for deployment
    - Create org_indexer_volume_indexes app for deployment
    - Create initial org_cluster_indexer_base apps directories
    - Copy Splunk org_cluster_indexer_base app for deployment
    - Create initial org_cluster_forwarder_outputs apps directories
    - Copy Splunk org_cluster_forwarder_outputs app.conf for deployment
    - Copy Splunk org_cluster_forwarder_outputs outputs.conf for deployment
    - Copy Splunk org_cluster_forwarder_outputs server.conf for deployment
    - Create initial org multisite apps directories
    - Create org_site_X_base apps for deployment
    - Create Splunkbase app directories
    - Configure deployer for search head cluster (optional)
    - TODO: Additional config for Deployment Server (more apps, serverclass settings etc)
- Setup config on Splunk Cluster Master
  - splunk-cluster-master:
    - Copy Splunk deploymentclient.conf for system
    - Check if clustering is enabled
    - Setup Cluster Master (Multisite or Singlesite)
    - Create initial index apps directories
    - Create Splunk CM server.conf for deployment
- Setup config on Splunk Search Heads
  - splunk-search-heads:
    - Check for custom roles file
    - Include roles from custom_roles.yml
    - Copy Splunk authorize.conf for system
    - Check if clustering is enabled
    - Add Search Heads to cluster (Multisite or Singlesite)
    - Configure search head clustering (optional)
    - TODO: Configure distributed search (optional)
- Setup config on Splunk Peer Nodes
  - splunk-peer-nodes:
    - Check if clustering is enabled
    - Add Peer Nodes to cluster (Multisite or Singlesite)

## Run the UF playbook like this:  
  >ansible-playbook /path/to/splunk-staging-UF.yml -u username --become --ask-become-pass (--check)

Specify a different **deploy_stage** value by passing 
**--extra-vars="deploy_stage_value=VALUE"**. depending on the environment the UF is being deployed to.
  >ansible-playbook /path/to/splunk-staging-UF.yml --extra-vars="deploy_stage_value=staging" -u username --become --ask-become-pass (--check)

### The UF playbook will configure:
- Setup Splunk Universal Forwarder
  - common:
    - Install required packages using apt and pip modules
  - splunk-universal-forwarder:
    - Install Splunk from .deb package
    - Fix up perms after splunk user created
    - Sets default acl for splunk on /var/log
    - Ensure directory exists for local self-signed TLS certs.
    - Remove a cert bundle if it exists to allow creation of new
    - Distribute wildcard-priv-key.pem, wildcard-cert.pem to host
    - Generate a cert bundle from wildcard artifacts
    - Copy Splunk inputs.conf for system local
    - Copy Splunk web.conf for system
    - Use splunk hash-passwd to return a password hash
    - Check for user seed file
    - Check for passwd file
    - Create default admin user seed file
    - Accept Splunk license
    - Setup Splunk to start at boot with splunk user
    - Copy Splunk init for service
    - Check passwd file contains current admin hash value
    - Reset Splunk admin password if changed
    - Set sslPassword using splunk_cluster_sslPassword
    - Add serverCert to system/local server.conf
    - Add sslRootCAPath to system/local server.conf
    - Copy Splunk deploymentclient.conf for system
