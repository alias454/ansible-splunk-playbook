# Deployment Server Overview and Purpose

The Deployment server acts as a central repository for Splunk apps.

To set up a deployment server, configure both the deployment server and the deployment clients.
Most configuration occurs on the deployment server side. However, when dealing with certificate based connections some additional configuration may be required on the clients.

Deployment apps consist of any arbitrary content needing to be copied to a set of deployment clients.

**Content that can be deployed via an app:**  
  - A Splunk Enterprise app (such as those on Splunkbase)
  - A set of Splunk Enterprise configurations (inputs.conf, etc.)
  - Other content, such as scripts, images, and supporting files

Because of high CPU and memory usage during app downloads, it is recommended that the deployment server instance reside on a dedicated machine.

**Create Deployment apps:**  
To prepare deployment apps, create separate directories for each deployment app in a special location on the deployment server.
The default location is **$SPLUNK_HOME/etc/deployment-apps**, but this is configurable through the **repositoryLocation** attribute in serverclass.conf.
Underneath this location, each app must have its own subdirectory. The name of the subdirectory serves as the app name in the forwarder management interface.
Creating an app directory, effectively creates the app itself, even if the directory does not yet contain any content.

**Define a server class:**  
A server class is a mapping between deployment clients and apps. It tells the deployment server which apps to send to which clients. 
When a servername is added to a server class, it is associated with one or more apps that will be automatically deployed to its deployment groups.

**Create the server class:**  
Specify one or more deployment apps for the server class. Then specify the clients that belong to the server class.
This info gets saved in serverclass.conf, which is located in **/opt/splunk/etc/system/local/**.

Configure the deployment clients to connect to a deployment server (Universal Forwarders, etc.)

Notes: You can use a deployment server to distribute updates to search heads in indexer clusters, as long as they are standalone search heads. 
You **cannot use the deployment server to distribute updates to members of a search head cluster - you must use a Deployer**.

## Splunk Deployment Server Role Objectives

1. Configure the Splunk **Deployment** Server after splunk-base has run
    1. Create a custom serverclass.conf to map specific apps to specific clients
    2. Create indexes file for deployment to peer nodes via the Cluster Master
    3. Create custom apps to stand up a basic cluster
        - org_all_indexes
        - org_indexer_volume_indexes 
        - org_cluster_forwarder_outputs
        - org_forwarder_outputs
        - org_site_X_base
    4. Deploy Splunkbase apps and manage them
        - Splunk_TA_nix
        - Others
2. Setup **Deployer** Server(used for deploying apps to search heads in a search head cluster)

### The playbook will configure:
  - deployment-server
    - splunk-deployment-server : Enable Deployment Server
    - splunk-deployment-server : Create serverclass.conf to local system
    - splunk-deployment-server : Create initial app directories
    - splunk-deployment-server : Create org_all_indexes app for deployment
    - splunk-deployment-server : Check for custom indexes file
    - splunk-deployment-server : Add custom mappings to indexes file for deployment
    - splunk-deployment-server : Create org_indexer_volume_indexes app for deployment
    - splunk-deployment-server : Create org_cluster_indexer_base app for deployment
    - splunk-deployment-server : Create forwarder_outputs app for deployment
    - splunk-deployment-server : Create org_site_X_base apps for deployment
    - splunk-deployment-server : Create Splunkbase app directories
    - splunk-deployment-server : Config for Search Deployer
    - TODO: Additional config for Deployment Server (more apps, serverclass settings etc)

### Files included in this role:

    splunk-deployment-server/
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    │   ├── customer_indexes.conf
    │   ├── deployer-apps
    │   │   └── slack_alerts
    │   │       └── local
    │   │           ├── alert_actions.conf
    │   │           └── app.conf
    │   ├── deployment-apps
    │   │   ├── Splunk_TA_nix
    │   │   │   └── local
    │   │   │       ├── app.conf
    │   │   │       └── inputs.conf
    │   │   ├── org_all_indexes
    │   │   │   └── local
    │   │   │       ├── app.conf
    │   │   │       ├── indexes.conf
    │   │   │       └── props.conf
    │   │   ├── org_site_1_base
    │   │   │   └── local
    │   │   │       ├── app.conf
    │   │   │       └── server.conf
    │   │   └── org_site_2_base
    │   │       └── local
    │   │           ├── app.conf
    │   │           └── server.conf
    │   └── packages
    │       ├── slack-notification-alert_203.tgz
    │       └── splunk-add-on-for-unix-and-linux_524.tgz
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   ├── deploy_Splunk_Packages.yml
    │   ├── deploy_Splunk_TA_Template.yml
    │   ├── deploy_all_indexes.yml
    │   ├── deploy_cluster_indexer_base.yml
    │   ├── deploy_forwarder_outputs.yml
    │   ├── deploy_indexer_volume_indexes.yml
    │   ├── deploy_search_head_volume_indexes.yml
    │   ├── deploy_sites.yml
    │   ├── include_deployment_tasks.yml
    │   ├── main.yml
    │   └── shcluster_setup.yml
    └── templates
        └── system_local.serverclass.j2

### Deployed apps for Deployment Server

As part of the default initial setup, some basic necessary apps will be deployed with sane defaults.  
All default apps are saved with the prefix of **org** but the value set in **org_prefix** will override it on disk.  

**org_all_indexes**  
Manage config settings for:
  > indexes.conf  
  > props.conf

Set index values that will be deployed to the clustered indexers. Values are custom for each install but an example entry is given in the custom_indexes.conf file.  
In addition, the **props.conf** is managed using this app as well. [example props.conf.spec](https://docs.splunk.com/Documentation/Splunk/7.1.0/Admin/Propsconf)

---
**org_indexer_volume_indexes**  
Manage config settings for:
  > indexes.conf  

Used to set primary and secondary volumes path values and maxVolumeDataSizeMB for each.  
The value is derived on a per indexer basis so make sure all indexers have the same disk configuration.

An example with a 5TB primary volume and a 16TB secondary volume. 
```properties
[volume:primary]
maxVolumeDataSizeMB = 5832300
path = /storage/splunk

[volume:secondary]
maxVolumeDataSizeMB = 15562832
path = /coldstorage/splunk
```

---
**org_search_head_volume_indexes**  
Manage config settings for:
  > indexes.conf  

Used to set primary and secondary volumes path values and maxVolumeDataSizeMB for each.  
The value is derived on a per node basis so make sure all search heads have the same disk configuration.

An example with a 32GB filesystem setting primary volume and secondary volume values. 
```properties
[volume:primary]
maxVolumeDataSizeMB = 8187
path = /storage/splunk

[volume:secondary]
maxVolumeDataSizeMB = 21288
path = /storage/splunk
```

---
**org_site_X_base**  
Manage config settings for:
  > server.conf  

Stores the site value setting found in server.conf. Use this to set values like **site = site1** or **site = site2** as examples. 

---
**org_forwarder_outputs**  
Manage config settings for:
  > server.conf  
  > outputs.conf

Setup values for deployment client nodes that forward data into the peer nodes. Sets up the appropriate **tcpout** parameters and **indexer_discovery**.

---
**org_cluster_indexer_base**  
Manage config settings for:
  > server.conf  
  > indexes.conf

Set **repFactor = auto** in indexes.conf and push sslConfig settings for **sslVersions**, **sslVersionsForClient**, and **cipherSuite** for server.conf

---
**Splunk_TA_nix**
Splunkbase apps need to be downloaded prior to using them. Download the tgz file from Splunkbase and save it to the files/ directory.  
  
Sets custom values to override the **default/inputs.conf** configuration shipped with the app.

---
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

Define the app prefix and org name.
```yaml
org_prefix: "org"
org_name: "myOrg"
```

Configure the Deployment Server for app management.
```yaml
# Manage apps and indexes using a deployment server.
# http://docs.splunk.com/Documentation/Splunk/7.1.0/Indexer/Updatepeerconfigurations
use_deployment_server: 1
```

Configure usage of a search head cluster.
```yaml
# Enable Search Head Clustering
use_shclustering: false
```

To use a custom indexes file, create a file in the secrets folder.
```yaml
# To use a custom indexes file, create a file in the custom_files folder
custom_indexes_file: "custom_indexes.conf"
```

Define whether or not a multisite cluster will be used.
```yaml
# Use multisite or single site cluster config
use_multisite_config: "true" #set to false when not using
```

Configure storage volume sizes and paths
```yaml
# Set storage volume data size for indexers
idx_primary_maxVolumeDataSizeMB: "5832300"
idx_primary_path: "/storage/splunk"
idx_secondary_maxVolumeDataSizeMB: "15562832"
idx_secondary_path: "/coldstorage/splunk"

# Set storage volume data size for search heads
search_primary_maxVolumeDataSizeMB: "8187"
search_primary_path: "/storage/splunk"
search_secondary_maxVolumeDataSizeMB: "21288"
search_secondary_path: "/storage/splunk"
```

Defined default stage environment variables that are applicable to this role.
```yaml
# Set under the appropriate environment variable in the secrets file
# splunk certificate paths
default_stage:
  splunk_cluster_pass4SymmKey: 'someRandomKey'
  ssl:
    splunk_cluster_sslPassword: 'password'
    splunk_cluster_wildcard_sslPassword: 'password'
    splunk_clientCert: "$SPLUNK_HOME/etc/auth/server.pem"
    splunk_sslRootCAPath: "/opt/splunk/etc/auth/cacert.pem"
    splunk_serverCert: "$SPLUNK_HOME/etc/auth/server.pem"
    splunkweb_privKeyPath: "etc/auth/splunkweb/privkey.pem"
    splunkweb_serverCert: "etc/auth/splunkweb/cert.pem"
    splunkforwarder_clientCert: "$SPLUNK_HOME/etc/auth/server.pem"
    splunkforwarder_sslRootCAPath: "/opt/splunkforwarder/etc/auth/cacert.pem"
```