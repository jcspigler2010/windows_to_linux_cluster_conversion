


To Migrate from Windows Index cluster to Linux Index cluster


Use Volumes and SPLUNK_DB

#################################
Linux Local Configurations#######
#################################
splunk-launch.conf
#   Version 8.2.5

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



###################################
Windows Local Configurations#######
###################################

path = $SPLUNK_DB
Convert Single site cluster to multi site cluster

{screen shot}

[volume:primary]
path = $SPLUNK_DB
# Note: The *only* reason to use a volume is to set a cumulative size-based
# limit across several indexes stored on the same partition. There are *not*
# time-based volume limits.
# ~5 TB
maxVolumeDataSizeMB = 50000

restart splunk



Multisite clustering

############################
Cluster Manager ###########
############################

# MULTI-SITE VERSION

server.conf
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


server.conf

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


server.conf
[clustering]
assign_primaries_to_all_sites=true


##############################
Site 1 Windows Indexers#######
###############################
server.conf ###Single site configuration all sites
[clustering]

mode = slave
manager_uri = https://10.0.1.240:8089
pass4SymmKey = $7$uQsenT0kCyXxb5S2Yb/YwEvkqJap4dnCnuHgMYTcNypZBfEz65zXwKcmwz3yag==


[replication_port://8080]
disabled = false


server.conf ###multisite configuration
[general]

site = site1

[clustering]
multisite = true


############################
Site 2 Linux Indexers#######
############################

server.conf ###single site all sites
[clustering]
mode = slave
manager_uri = https://10.0.1.240:8089
pass4SymmKey = $7$u3kOfQCemNmrZwcL5wA07Ld9QtlpZAHoybv9TA4D57ULkGdk+4cCYV5xQDCYKg==


[replication_port://8080]
disabled = false

server.conf ###multi site

[general]
site = site2

[clustering]
multisite = true

################
Search Heads#######
#################

outputs.conf ### all sites

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

server.conf ### all sites

[indexer_discovery]
pass4SymmKey = clearshark123!


server.conf ### all sites

[clustering]
mode = searchhead
manager_uri = clustermanager:one

[clustermanager:one]
manager_uri=https://10.0.1.240:8089
pass4SymmKey = clearshark123!


server.conf ### multisite configuration

server.conf #### multisite configuration

[clustermanager:one]
multisite = true
site = site1

################
Forwarders#######
#################
outputs.conf ### multisite configuration

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


server.conf ### multisite configuration

[general]
site = site1


Commands

/opt/splunk/bin/splunk offline
/opt/splunk/bin/splunk fsck scan --all-buckets-all-indexes
/opt/splunk/bin/splunk fsck repair --all-buckets-all-indexes
/opt/splunk/bin/splunk fsck scan --include-rawdata --all-buckets-all-indexes

Change Search head affinity to site 2


server.conf #### multisite configuration

[clustermanager:one]
multisite = true
site = site2

You may need to doing another iteration of fsck repair --all-buckets-all-indexes

server.conf ####

#####Cluster Manager #####
###########################

[general]
site = site2

[clustering]
mode = master
constrain_singlesite_buckets = false
multisite = true
available_sites = site2
cluster_label = awsgov_cluster1
site_replication_factor = origin:2, total:2
site_search_factor = origin:2, total:2

site_mappings = site1:site2

restart cluster manager

stop splunk on windows indexers

fsck any remaining corrupted buckets

https://community.splunk.com/t5/Deployment-Architecture/Question-How-does-Cluster-Master-decide-Primary-searchable-copy/m-p/311222

In non muli-site clustering, its either 0x0, or 0xFFFFFFFF , basically primary or not primary.

The individual bits are only relevant in multi-site clustering.
The flags is a 64 bit bitmask, with the smallest bit corresponding to Primary for site0. The second smallest would be primary for site1, the third for site2, and so on....so 0x0 = primary for nothing (searches from any site will not get results for this bucket on this peer)

Bucket marked 0x1 = searches that come from searchheads with site=0 will get results! (primary for site0)
Bucket marked 0x2 = primary for site1, all site=site1 searches will get results from these buckets
Bucket marked 0x3 == (0x1+0x2) primary for site0 + site1, so searches from site0 SearchHeads and site1 searchheads will get results for this bucket.

On the cluster/master/buckets/BID endpoint, the masks should add up to whatever the mutl-site config is
for example, if We have available_sites=site2,site3, then the mask will need to have 0x1 (site0), 0x4(site2), 0x8 (site3) distributed among its indexers. in this example, if search_factor=1, then only 1 bucket will be searchable and should get assigned all the flags (0x13)
