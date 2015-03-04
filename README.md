[] Introduction
=====

This is a fork from the original https://github.com/myaaaaa-chan/zabbix-couchbase-template

It initially contains the translation of instructions to English, plus tips and tricks on configuration.


[] Step 1: Import the Zabbix Couchbase template
=====

The Zabbix Couchbase template is the Couchbase monitoring model for Zabbix.

All you need to do is download the zbx_couchbase_templates.xml and point your browser to the Zabbix web interface.

Go to Configurations > Templates > Import (top right side). Choose the file and click the Import buttom.

[] Step 2: Configure the Zabbix Client
=====

On the Couchbase server (the one that will be monitored by Zabbix), install the zabbix-agent (if not done yet).

    # #On Ubuntu:
    # apt-get install zabbix-agent

Edit /etc/zabbix/zabbix_agentd.conf and make sure you have the following line uncommented:

    EnableRemoteCommands=1
    
In order to make the checks, you will also need this line on the /etc/sudoers file:

    # Enabling Zabbix Couchbase monitoring
    #Defaults    requiretty
    zabbix ALL=(root) NOPASSWD: /opt/couchbase/bin/couchbase-cli

This Zabbix Couchbase monitoring depends on jq binary (https://stedolan.github.io/jq/), so you need to install it:

    # apt-get install jq

Now copy the userparameter_couchbase.conf file under /etc/zabbix/zabbix_agentd.d

And restart the zabbix-agent

    # service zabbix-agent restart
    


[] Step 3: Add Couchbase monitoring to the Host
=====

First of all, make sure you have Zabbix ports set up between client and server. You need port 10050 open on client for incoming connections from server. And you need port 10051 open on server for incoming connections from client.

On Zabbix server web interface, go to Configuration > Hosts and click on the Name of the Couchbase instance you want to monitor (this guide doesn't cover basic addition of servers to Couchbase).

Enter the Templates tab and select the "Template App Couchbase". Click Add. Then Save.

Go to Macros tab, and add the following entries:

    {$USERNAME}     -> Your Couchbase admin user
    {$PASSWORD}     -> Your Couchbase admin user's password
    {$BUCKET}       -> Put 'defaults'
    {$HDD_INDEX}    -> The ID of the hard disk (HDD) you want to monitor (Ex: 0)
    {$SSD_INDEX}    -> The ID of the SSD disk you want to monitor (Ex: 0)
    
[] Items you can monitor with this template
=====

| Name | Key | Content |
|:-----------|:------------|:------------|
| Bucket item count | cb.bucket.items[{$USERNAME},{$PASSWORD},{$BUCKET}] | The number of items in the bucket |
| Bucket name | cb.bucket.name[{$USERNAME},{$PASSWORD},{$BUCKET}] | Name of the monitored bucket |
| Bucket ops | cb.bucket.ops[{$USERNAME},{$PASSWORD},{$BUCKET}] | OPS to the monitored bucket |
| Bucket used(percent) | cb.bucket.quotapercent[{$USERNAME},{$PASSWORD},{$BUCKET}] | Bucket usage (%) |
| Bucket size | cb.bucket.size[{$USERNAME},{$PASSWORD},{$BUCKET}] | Bucket size |
| Bucket type | cb.bucket.type[{$USERNAME},{$PASSWORD},{$BUCKET}] | Bucket tipe (couchbase or memcache) |
| Cluster status | cb.cluster.membership[{$USERNAME},{$PASSWORD}] | Cluster status (active or not) |
| Hdd free | cb.hdd.free[{$USERNAME},{$PASSWORD},{$HDD_INDEX}] | HDD free space |
| Hdd quota total | cb.hdd.quotatotal[{$USERNAME},{$PASSWORD},{$HDD_INDEX}] | HDD Quota |
| Hdd total | cb.hdd.total[{$USERNAME},{$PASSWORD},{$HDD_INDEX}] | HDD capacity |
| Hdd used by data | cb.hdd.usedbydata[{$USERNAME},{$PASSWORD}] | HDD usage |
| Ram quota total | cb.ram.quotatotal[{$USERNAME},{$PASSWORD}] | Memory (quota) |
| Ram quota used | cb.ram.quotaused[{$USERNAME},{$PASSWORD}] | Memory (quota) |
| Ram total | cb.ram.total[{$USERNAME},{$PASSWORD}] | Memory capacity |
| Ram used | cb.ram.used[{$USERNAME},{$PASSWORD}] | Memory usage |
| Data index path on HDD | cb.strage.hdd.indexpath[{$USERNAME},{$PASSWORD},{$HDD_INDEX}] | Database storage path |
| Data index path on HDD | cb.strage.hdd.indexpath[{$USERNAME},{$PASSWORD},{$HDD_INDEX}] | Database index storage path |
| Data path on SSD | cb.strage.ssd.datapath[{$USERNAME},{$PASSWORD},{$SSD_INDEX}] | Database storage path |
| Data index path on SSD | cb.strage.ssd.indexpath[{$USERNAME},{$PASSWORD},{$SSD_INDEX}] | Database index storage path |
| Version of Couchbase running | cb.version[{$USERNAME},{$PASSWORD}] | Couchbase version |
| Couchbase running status | net.tcp.listen[11210] | Couchbase status (1:OK,2:NG) |

　
[] Triggers defined by this template
=====

| Name | Severity | Content |
|:-----------|:------------|:------------|
| Lack of free bucket quota warning on server {HOST.NAME} | Warning | Warn if the free space of the bucket is less than 30% |
| Lack of free bucket quota on server {HOST.NAME} | Warning | Warn If the free space of the bucket is less than 10% |
| Couchbase status is not healthy | Critical | Alert when the status is other than helthy |
| Couchbase process is not running. | Critical | Alert when TCP port 11210 is no longer accessible |
　
