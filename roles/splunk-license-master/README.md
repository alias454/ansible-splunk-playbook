# License Master Overview and Purpose

With Splunk there are two licensing options. One is a standalone license master, for a deployment consisting of a single indexer. 
The other is using a central license master, for a distributed deployment with multiple indexers.
In a distributed deployment of multiple Splunk Enterprise instances, one of the instances must be configured as a license master.
The license master can be installed on another Splunk server and does not require setup of a dedicated licensing server.

Licensing:  
For event data, data volume is based on the amount of raw external data that the indexer ingests into its indexing pipeline, after any filtering. It is not based on the amount of compressed data that gets written to disk.

For metrics data, each metric event counts as a fixed 150 bytes. Metrics data does not have a separate license. Ingested metrics data draws from the same license quota as event data.

## Splunk License Master Role Objectives

1. Configure the Splunk License Master after splunk-base has run
    1. Configure server to act as license server for Splunk Enterprise

### The playbook will configure:
  - license-master
    - splunk-license-master : Create Enterprise licenses dir
    - splunk-license-master : Copy over Splunk license file
    - splunk-license-master : Check if license already added
    - splunk-license-master : Install Splunk Enterprise license key

### Files included in this role:

    splunk-license-master/
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

Define name of license file, which is stored in **group_vars/secrets/Splunk.License.lic**.
```yaml
splunk_license_file: "Splunk.License.lic"
```