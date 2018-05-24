# Peer Node Overview and Purpose

Splunk supports utilizing multiple peer nodes(indexers) in an **index cluster** to increase indexing capability as well as allowing replication of data across multiple nodes and potential different sites.
Multiple, redundant copies of rawdata files live across multiple indexers so in the event of an unexpected indexer failure or planned maintenance, data is not lost.
Having multiple copies of data also has an added benefit of increased search performance because data is distributed across indexers.

## Splunk Peer Node Role Objectives

1. Configure the Splunk Peer Nodes after splunk-base has run
    1. Configure cluster (Multisite or Singlesite)

### The playbook will configure:
  - peer-nodes
    - splunk-peer-nodes : Check if clustering is enabled
    - splunk-peer-nodes : Add Peer Nodes to Cluster (Multisite or Singlesite)

### Files included in this role:

    splunk-peer-nodes/
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    └── tasks
        └── main.yml

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

Define whether or not a multisite cluster will be deployed.
```yaml
# Use multisite or single site cluster config
use_multisite_config: "true" #set to false when not using
```

If using the multisite cluster option this setting defines the default site for nodes.
```yaml
# Multisite config options http://docs.splunk.com/Documentation/Splunk/7.1.0/Indexer/MultisiteCLI
site: "site1"
```