# exercise
Install a two node cluster for DSE Analytics workload using Opscenter.
   - node 0 for Opscenter. 2.5GB RAM and 2 Cores
   - node 1 for DSE Analytics 6GB RAM & 3 Cores
   

Prerequisites:-

Vagrant
Virtual Box

Instructions:-

This Vagrant template sets up DataStax Enterprise (DSE) on a configurable number of VMs:

node[0-n] - DSE nodes (with prerequisites and DSE already installed)
Notes:
Step1:

To bring up the nodes please export creadential of datastax academy before vagrant up, you can bring up the DSE nodes with the following:

$ export VAGRANT_DSE_USERNAME=your-username
$ export VAGRANT_DSE_PASSWORD=your-password
$ # optional: adjust DSE_NODES value in Vagrantfile (default 3)
$ vagrant up
When the up command is done, you can check the status of the VMs:

$ vagrant status
Configured for 3 DSE node(s)
Current machine states:

node0           running (virtualbox)
node1           running (virtualbox)

Step2: Create the ssh, config, repository on OpsCenter for installing the cluster.
DSE Analytics needs additional options such as AlwaysOn_SQL & DSE Authentication. Please refer the links below :-

Pro Tip(Read the instruction on following links carefully and follow them)

alwayson_sql: https://docs.datastax.com/en/dse/6.7/dse-dev/datastax_enterprise/spark/alwaysOnSql.html
DSE Authentication: https://docs.datastax.com/en/dse/6.7/dse-admin/datastax_enterprise/security/Auth/secEnableDseAuthenticator.html

Once cluster, DC & Nodes are define, submit the install job for the cluster. 

Cluster Installed :- 






