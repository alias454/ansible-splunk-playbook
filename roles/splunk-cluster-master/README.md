# Cluster Master Overview and Purpose

A Cluster Master coordinates activities involving search peers, search heads, and **buckets** across peer nodes.
When a peer node cluster is setup, app management can only be done from the master node and not a deployment server. This ensures all the peer nodes receive a copy of the app. 
The **master-apps** location is only for peer node files. The master does not use the files in that directory for its own configuration needs.

Custom indexes must be deployed using an app when using an indexer cluster.
Configure the indexes.conf file on the Cluster Master in a custom app and store it in the master-apps directory. ex: **/opt/splunk/etc/master-apps/<app-name>/local/indexes.conf**.
The custom indexes file can be stored on a deployment server and a special deployment client config can be used for moving file to the master and not directly to the peer node.

## Splunk Cluster Master Role Objectives

1. Configure the Splunk Cluster Master after splunk-base has run
    1. Configure deployment client options for indexer app management
    2. Configure cluster (Multisite or Singlesite)
    3. Deploy default apps in **{{ splunk_base }}/etc/apps/**

### The playbook will configure:
  - cluster-master
    - splunk-cluster-master : Copy Splunk deploymentclient.conf for system
    - splunk-cluster-master : Check if clustering is enabled
    - splunk-cluster-master : Setup Cluster Master (Multisite or Singlesite)
    - splunk-cluster-master : Create initial index apps directories
    - splunk-cluster-master : Create Splunk CM server.conf for deployment

### Files included in this role:
 
    splunk-cluster-master/
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
        └── system_local.deploymentclient.j2

## How does this work?

Once **splunk-base** has been applied, nodes can be configured for individual capabilities based on their inventory group.  
The inventory is based on the **hosts.example** file located within the playbook.  
However, inventory can be managed in other ways.

In the **defaults/main.yml**, configure the settings to utilize different options and configurations.  
Options can be overridden in higher level configs or a secrets file.

Define the base directory where splunk is installed.
```yaml
splunk_base: '/opt/splunk'
```

Configure the Cluster Master to connect to a deployment server for indexer app management.
```yaml
# Manage apps and indexes using a deployment server.
# http://docs.splunk.com/Documentation/Splunk/7.1.0/Indexer/Updatepeerconfigurations
use_deployment_server: 1
```

Define whether or not a multisite cluster will be deployed.
```yaml
# Use multisite or single site cluster config
use_multisite_config: "true" #set to false when not using
```

The **Replication Factor** and **Search Factor** configuration options require settings for both singlesite and multisite clusters.

  >Replication Factor. The replication factor determines how many copies of data the cluster maintains. 
The default is 3. Be sure to choose the right replication factor now. 
It is inadvisable to increase the replication factor later, after the cluster contains significant amounts of data.

  >Search Factor. The search factor determines how many immediately searchable copies of data the cluster maintains. 
The default is 2. Be sure to choose the right search factor now. 
It is inadvisable to increase the search factor later, once the cluster has significant amounts of data.

Singlesite cluster config options defined in this role.
```yaml
# Singlesite config options
cluster_replication_factor: 2
cluster_search_factor: 2
```

If using the multisite cluster option these settings define several attributes related to the site config.
```yaml
# Multisite config options http://docs.splunk.com/Documentation/Splunk/7.1.0/Indexer/MultisiteCLI
site: "site1"
cluster_label: "US_Cluster"
available_sites: "site1,site2"
site_replication_factor: "origin:2,total:2"
site_search_factor: "origin:2,total:2"
```