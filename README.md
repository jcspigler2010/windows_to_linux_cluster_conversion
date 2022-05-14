<div id="top"></div>
<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Don't forget to give the project a star!
*** Thanks again! Now go create something AMAZING! :D
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->


<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

<h3 align="center">windows_to_linux_cluster_conversion</h3>

  <p align="center">
    <br />
    <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion"><strong>Explore the docs »</strong></a>
    <br />
    <br />
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#warning">WARNING!!</a></li>
      </ul>
      <ul>
        <li><a href="#conf-files-used">conf files used</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prep-windows-and-linux-servers">Prep Windows and Linux servers</a></li>
        <li><a href="#Linux-Servers-SPLUNK-DB">Linux Servers SPLUNK DB</a></li>
        <li><a href="#Windows-Local-Configurations">Windows Local Configurations</a></li>
      </ul>
      <li><a href="#Multisite-clustering-configuration">Multisite clustering configuration</a></li>
        <ul>
        <li><a href="#Cluster-Manager-Configuration">Cluster Manager Configuration</a></li>
        <ul>
        <li><a href="#org-all-cluster-manager-summary-replication">org all cluster manager summary replication</a></li>
        <li><a href="#org-multisite-master-base»">org multisite master base»</a></li>
        <li><a href="#org-all-cluster-managers-assign-primaries-all-sites">org all cluster managers assign primaries all sites</a></li>
        <li><a href="#org-cluster-master-indexerDiscovery-server">org cluster master indexerDiscovery server</a></li>
        </ul>
        <li><a href="#Site-1-windows-indexers">Site 1 Windows Indexers</a></li>
          <ul>
            <li><a href="#multisite-configuration">multisite configuration</a></li>
        </ul>
        <li><a href="#Site-2-linux-indexers">Site 2 Linux Indexers</a></li>
          <ul>
            <li><a href="#org-site-2-indexer-base">org site 2 indexer base</a></li>
        </ul>
        <li><a href="#search-heads">search heads</a></li>
        <li><a href="#Forwarders">Forwarders</a></li>
      </ul>
    </li>
    <li><a href="#Fixing-Buckets">Fixing Buckets</a></li>
    <ul>
      <li><a href="#First-pass">First pass</a></li>
      <ul>
        <li><a href="#Commands">Commands</a></li>
      </ul>
      <ul>
    <li><a href="#Change-Search-head-affinity-to-site-2">Change Search head affinity to site 2</a></li>
    <li><a href="#change-forwarder-affinity-to-site-2">change forwarder affinity to site 2</a></li>
    <li><a href="#Offline-Windows-Indexers-in-site-1">Offline-Windows-Indexers-in-site-1</a></li>
    <li><a href="#Second-pass">Second pass</a></li>
    <ul>
    <li><a href="#Commands">Commands</a></li>
    </ul>
    <li><a href="#Cluster-Manager">Cluster Manager</a></li>
    <ul>
    <li><a href="#Remove-Indexer-peers">Remove Indexer peers</a></li>
    </ul>
    <li><a href="#Revert-Multisite-cluster-back-to-single-site">Revert Multisite cluster back to single site</a></li>
    </ul>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project


<p align="right">(<a href="#top">back to top</a>)</p>

### WARNING!!
This is not an official Splunk supported migration method, strategy or approach to move from Windows Indexers to Linux Indexers.  
- This is completely unsupported and borderline experimental.

- Without a full comprehensive understanding of how  splunk indexer clustering and multisite clustering works along with architecting and administering a distributed splunk environment, unexpected behavior, outages and worse case scenario irreversible data loss is possible.  

- PLEASE PLEASE PLEASE... if there this is a mission critical system, please back up your indexer data just incase.

- Please if possible test before launching in production.

 Please communicate with stakeholders the previous points and risks.

## .conf files used

* [server.conf](https://docs.splunk.com/Documentation/Splunk/8.2.6/Admin/Serverconf)
* [outputs.conf](https://docs.splunk.com/Documentation/Splunk/8.2.6/Admin/Outputsconf)


<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

This is an example of how you may give instructions on setting up your project locally.
To get a local copy up and running follow these simple example steps.

### Prep Windows and Linux servers

You will need to update or set the environment variable SPLUNK_DB in you windows indexers and future linux indexers so your indexes.conf app which is pushed from the cluster manager is homogeneous across the cluster even though directory paths are referenced differently with each OS.

### Linux Servers SPLUNK_DB
This will be done in splunk-launch.conf for linux systems.
* splunk-launch.conf
```#   Version 8.2.5

# Modify the following line to suit the location of your Splunk install.
# If unset, Splunk will use the parent of the directory containing the splunk
# CLI executable.
#
SPLUNK_HOME=/opt/splunk

# By default, Splunk stores its indexes under SPLUNK_HOME in the
# var/lib/splunk subdirectory.  This can be overridden
# here:
#
SPLUNK_DB=/splunkdata
# Splunkd daemon name
SPLUNK_SERVER_NAME=Splunkd

# If SPLUNK_OS_USER is set, then Splunk service will only start
# if the 'splunk [re]start [splunkd]' command is invoked by a user who
# is, or can effectively become via setuid(2), $SPLUNK_OS_USER.
# (This setting can be specified as username or as UID.)
#
# SPLUNK_OS_USER
SPLUNK_OS_USER=splunk
```

### Windows Local Configurations

<img src="images/win_splunk_db.png" alt="Logo" width="400" height="400">
</a>



### Restart Splunk
`$SPLUNK_HOME/bin/splunk restart`



## Multisite clustering configuration
All of the following steps under this section should be done back to back without restarting splunk.  Restarting splunk on any of the servers *(idxrs shs cm)* could result in an outage.  Pelase wait until the end to restart splunk on all servers.

### Cluster Manager Configuration
<p align="left">
  Apps
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_multisite_master_base"><strong>org_multisite_master_base»</strong></a>
  <br />
    <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_cluster_master_indexerDiscovery_server/local"><strong>org_cluster_master_indexerDiscovery_server</strong></a>
  <br />
    <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_all_cluster_manager_summary_replication/local"><strong>org_all_cluster_manager_summary_replication</strong></a>
    <br />
    <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_all_cluster_managers_assign_primaries_all_sites"><strong>org_all_cluster_managers_assign_primaries_all_sites</strong></a>
</p>

#### org all cluster manager summary replication
Place org_all_cluster_manager_summary_replication app in the cluster manager's etc/app directory.  Update values to reflect your enviornment

server.conf
```
[clustering]

summary_replication = true
* Valid for both 'mode=manager' and 'mode=peer'.
* Cluster Manager:
  If set to true, summary replication is enabled.
  If set to false, summary replication is disabled, but can be enabled
  at runtime.
  If set to disabled, summary replication is disabled. Summary replication
  cannot be enabled at runtime.
* Peers:
  If set to true or false, there is no effect. The indexer follows
  whatever setting is on the Cluster Manager.
  If set to disabled, summary replication is disabled. The indexer does
  no scanning of summaries (increased performance during peers joing
  the cluster for large clusters).
* Default: false (for both Cluster Manager and Peers)
```

#### org multisite master base»
Place org_multisite_master_base» app in the cluster manager's etc/app directory.  Update values to reflect your enviornment

server.conf
```
[general]
site = site1

[clustering]
mode = master
constrain_singlesite_buckets = false
multisite = true
available_sites = site1, site2
cluster_label = awsgov_cluster1
site_replication_factor = origin:2, site2:2, total:4
site_search_factor = origin:2, site2:2, total:4

```
#### org all cluster managers assign primaries all sites>>
Place org_all_cluster_managers_assign_primaries_all_sites>> app in the cluster manager's etc/app directory.  


server.conf
```
[clustering]
assign_primaries_to_all_sites=true
```
*OPTIONAL*

#### org cluster master indexerDiscovery server>>
Place org_cluster_master_indexerDiscovery_server>> app in the cluster manager's etc/app directory.  This is optional if you want to use indexer discovery to guide data forwarding

server.conf
```
[indexer_discovery]
pass4SymmKey = clearshark123!
# * Security key shared between master node and forwarders.
# * If specified here, the same value must also be specified on all forwarders
#  connecting to this master.
# * Unencrypted passwords must not begin with "$1$", as this is used by
#  Splunk software to determine if the password is already encrypted.

# polling_rate = <integer>
# * A value between 1 to 10. This value affects the forwarder polling
#  frequency to achieve the desired polling rate. The number of connected
#  forwarders is also taken into consideration.
# * The formula used to determine effective polling interval,
#  in Milliseconds, is:
#  (number_of_forwarders/polling_rate + 30 seconds) * 1000
# * Default: 10

# indexerWeightByDiskCapacity = <boolean>
# * If set to true, it instructs the forwarders to use weighted load
#  balancing. In weighted load balancing, load balancing is based on the
#  total disk capacity  of the target indexers, with the forwarder streaming
#  more data to indexers with larger disks.
# * The traffic sent to each indexer is based on the ratio of:
#   indexer_disk_capacity/total_disk_capacity_of_indexers_combined
# * Default: false
```



### Site 1 Windows Indexers
Apps used in configuring Windows Indexers for Multisite clustering
<p align="left">
  Apps
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_1_indexer_base"><strong>org_site_1_indexer_base</strong></a>
  <br />

Ensure your single site cluster is healthy, operational and setup properly.  Below is an example configuration for an indexer peer.

server.conf
```
[clustering]

mode = slave
manager_uri = https://10.0.1.240:8089
pass4SymmKey = $7$uQsenT0kCyXxb5S2Yb/YwEvkqJap4dnCnuHgMYTcNypZBfEz65zXwKcmwz3yag==


[replication_port://8080]
disabled = false
```

#### multisite configuration
server.conf
```
[general]

site = site1

[clustering]
multisite = true

```

### Site 2 Linux Indexers
Apps used in configuring Linux Indexers for Multisite clustering
<p align="left">
  Apps
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_2_indexer_base"><strong>org_site_2_indexer_base</strong></a>
  <br />

server.conf
```
[clustering]
mode = slave
manager_uri = https://10.0.1.240:8089
pass4SymmKey = $7$u3kOfQCemNmrZwcL5wA07Ld9QtlpZAHoybv9TA4D57ULkGdk+4cCYV5xQDCYKg==


[replication_port://8080]
disabled = false
```

#### org site 2 indexer base
server.conf

```
[general]
site = site2

[clustering]
multisite = true
```


### Search Heads
Apps used in configuring Search Heads for Multisite clustering
<p align="left">
  Apps
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_1_search_base"><strong>org_site_1_search_base</strong></a>
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_cluster_forwarders_indexerDiscovery_outputs"><strong>org_cluster_forwarders_indexerDiscovery_outputs</strong></a>
  <br />

outputs.conf

```
[tcpout]
defaultGroup = indexcluster1
maxQueueSize = 7MB
useACK = true

 forceTimebasedAutoLB = true

[tcpout:indexcluster1]


indexerDiscovery = clustermaster1

[indexer_discovery:clustermaster1]

pass4SymmKey = clearshark123!



manager_uri = https://10.0.1.240:8089
```

server.conf

```

[indexer_discovery]
pass4SymmKey = clearshark123!

```
server.conf

```
[clustering]
mode = searchhead
manager_uri = clustermanager:one

[clustermanager:one]
manager_uri=https://10.0.1.240:8089
pass4SymmKey = clearshark123!

```

server.conf
```
[clustermanager:one]
multisite = true
site = site1
```

### Forwarders
Apps used in configuring forwarders for Multisite clustering site affinity
<p align="left">
  Apps
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_1_forwarder_affinity"><strong>org_site_1_forwarder_affinity</strong></a>
  <br />
  <a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_cluster_forwarders_indexerDiscovery_outputs"><strong>org_cluster_forwarders_indexerDiscovery_outputs</strong></a>
  <br />
outputs.conf

```
[tcpout]
defaultGroup = indexcluster1

maxQueueSize = 7MB
useACK = true

forceTimebasedAutoLB = true

[tcpout:indexcluster1]


indexerDiscovery = clustermanager1

[indexer_discovery:clustermanager1]

pass4SymmKey = clearshark123!


master_uri = https://10.0.1.240:8089

```
server.conf

```
[general]
site = site1
```

## Fixing Buckets
### First pass
#### Commands
```
/opt/splunk/bin/splunk offline
/opt/splunk/bin/splunk fsck scan --all-buckets-all-indexes
/opt/splunk/bin/splunk fsck repair --all-buckets-all-indexes
/opt/splunk/bin/splunk fsck scan --include-rawdata --all-buckets-all-indexes
```

### Change Search head affinity to site 2


server.conf
```
[clustermanager:one]
multisite = true
site = site2
```
`bin/splunk restart`
### Change forwarder affinity to site 2
This will force all forwarders to start sending data to the new linux indexers in mulitsite index cluster site2.

server.conf

```
[general]
site = site2
```
`bin/splunk restart`

### Offline Windows Indexers in site 1
Run the following command on each windows indexer
`splunk.exe offline `


### Second pass
#### Commands
`/opt/splunk/bin/splunk offline`

You may need to doing another iteration of `bin/splunk/fsck repair --all-buckets-all-indexes` or you can run a `bin/splunk/fsck scan --all-buckets-all-indexes` and note which buckets are still corrupted after the first repair pass.  After which you can run `bin/splunk/fsck --one-bucket --bucket-path='/splunkdata/GUID-bucketid-earlestepoc-latestepoc'`

###Cluster Manager
#### Remove Indexer peers
Remove Windows Indexers from cluster peer list
`./splunk remove cluster-peers`

#### Revert Multisite cluster back to single site
Remove the following apps from CM, UFS, SHs and Indexers
<p align="left">
<br />
<a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_1_forwarder_affinity"><strong>org_site_1_forwarder_affinity</strong></a>
<br />
<br />
<a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_1_search_base"><strong>org_site_1_search_base</strong></a>
<br />
<a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_multisite_master_base"><strong>org_multisite_master_base»</strong></a>
<br />
<br />
<a href="https://github.com/jcspigler2010/windows_to_linux_cluster_conversion/tree/master/apps/org_site_2_indexer_base"><strong>org_site_2_indexer_base</strong></a>
<br />

Restart Splunk

For official documentation on how to revert multi site cluster back to single site, see the following

https://docs.splunk.com/Documentation/Splunk/8.2.6/Indexer/Converttosinglesite
