# Search Head Overview and Purpose

Search heads are the main way for users to access data stored on a search peer. 
Usually, access to the WebUI is available and users utilize the Splunk query language to perform searches on stored data.
In addition to running ad-hoc queries, users can save searches, create reports, and configure alerts among other things.

When deploying Search Heads, there are a two basic options. One option is to setup one or more independent search heads to search across the search peers. 
These servers can act independently or they can distribute searches between them. Another option is using multiple search heads in a search head cluster.
In either case, the initial configuration must be completed to create a search head and then configure additional conf files. 
When using a search head cluster there may be a need for additional load balancer configuration which is outside the scope of this readme.

## Splunk Search Head Role Objectives

1. Configure the Splunk Search Heads after splunk-base has run
    1. Install authorize.conf (checks for custom roles or uses defaults) 
    2. Configure cluster (Multisite or Singlesite)
    3. TODO: Configure search head clustering (optional)

### The playbook will configure:
  - search-heads
    - splunk-search-heads : Check for custom roles file
    - splunk-search-heads : Copy Splunk authorize.conf for system
    - splunk-search-heads : Check if clustering is enabled
    - splunk-search-heads : Add Search Heads to Cluster (Multisite or Singlesite)
    - TODO: Configure distributed search (optional)
    - TODO: Configure search head clustering (optional)

### Files included in this role:

    splunk-search-heads/
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    │   └── system_local.authorize.conf
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

If using the multisite cluster option this setting defines the default site for search nodes.
Defining **site0** allows the search head to search any site.
```yaml
# Multisite config options http://docs.splunk.com/Documentation/Splunk/7.1.0/Indexer/MultisiteCLI
search_site: "site0"
```

If using custom roles, create a file in the secrets directory **group_vars/custom_files/custom_roles.conf** with the defined name.  
```yaml
# For more info about roles http://docs.splunk.com/Documentation/Splunk/7.1.0/Admin/authorizeconf
# To use a custom roles file, create a file in the custom_files folder
custom_roles_file: 'custom_roles.conf'
```