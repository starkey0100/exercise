# exercise
Install a two node cluster for DSE Analytics workload using Opscenter.
   - node 0 for Opscenter. 2.5GB RAM and 2 Cores
   - node 1 for DSE Analytics 6GB RAM & 3 Cores
   

##Prerequisites:-

Vagrant
Virtual Box

Instructions:-

This Vagrant template sets up DataStax Enterprise (DSE) on a configurable number of VMs:

node[0-n] - DSE nodes (with prerequisites and DSE already installed)
Notes:
##Step1:

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

**Step2: Install the cluster using LCM :-
Create the ssh, config, repository on OpsCenter for installing the cluster.
DSE Analytics needs additional options such as AlwaysOn_SQL & DSE Authentication. Please refer the links below :-

Pro Tip(Read the instruction on following links carefully and follow them)

alwayson_sql: https://docs.datastax.com/en/dse/6.7/dse-dev/datastax_enterprise/spark/alwaysOnSql.html
DSE Authentication: https://docs.datastax.com/en/dse/6.7/dse-admin/datastax_enterprise/security/Auth/secEnableDseAuthenticator.html

Once cluster, DC & Nodes are define, submit the install job for the cluster. 

Check the workload type:-
root@node1:/home/vagrant# dsetool -l cassandra -p cassandra ring 
Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1] 
10.211.55.11     DC1                  rack1        Analytics(SM)        no     Up      Normal   469.39 KiB       ?                    -9223372036854775808                         0.70         


Cluster Installed :- 
Below is LCM view
![image](https://user-images.githubusercontent.com/50682370/57921809-e6c0c800-7863-11e9-9fb9-e7b313fa8ddc.png)

Below is OpsCenterView 
![image](https://user-images.githubusercontent.com/50682370/57921982-46b76e80-7864-11e9-89ed-6331bdde53a3.png)

**Step3: Prepare the node for enabling analytics. 
Please go through the instructions again. 
Tip: For single node cluster, replication factor of keyspace can be kept as '1' with 'NetworkTopologyStrategy' like below
 ALTER KEYSPACE "HiveMetaStore"
   WITH REPLICATION = {
   'class': 'NetworkTopologyStrategy', 
   'DC1': '1'};
   
alwayson_sql instructions: https://docs.datastax.com/en/dse/6.7/dse-dev/datastax_enterprise/spark/alwaysOnSql.html

- create a new user with SUPERUSER login. Disable default Cassandra user.
anoop@cqlsh> ALTER ROLE cassandra WITH SUPERUSER = false AND LOGIN = false AND password='new_secret_pw';
anoop@cqlsh> LIST ROLES;

 role         | super | login | options
--------------+-------+-------+---------
 alwayson_sql | False |  True |        {}
       my_name|  True |  True |        {}
    cassandra | False | False |        {}

(3 rows)


- grant proxy to alwayson_sql user for the new user. 
- Once all changes are done, restart the dse service and Alwayson_sql comes up itself if everything was setup correctly. 

root@node1:/home/vagrant# service dse restart
 * Restarting DSE daemon dse                                                                                                                                  [ OK ] 

DSE daemon starting with Spark enabled (edit /etc/default/dse to disable)
                                                                                                                                                              [ OK ]
root@node1:/home/vagrant# dse client-tool -u a#### -p h###y alwayson-sql status
AlwaysOn SQL 10.211.55.11:10001 status: Running

!! All set to solve some problems. 

**Step4. Prepare the IDE for running some spark jobs.
used Gradle:-
- Clone the Datastax project template 
https://github.com/datastax/SparkBuildExamples

- place the csv file on DSEFS for spark job to read.

dsefs dsefs://10.211.55.11:5598/ > mkdir data
dsefs dsefs://10.211.55.11:5598/ > cd data/ 
dsefs dsefs://10.211.55.11:5598/data/ > ls
dsefs dsefs://10.211.55.11:5598/data/ > cp file:/repository/flights_from_pg.csv /data/
dsefs dsefs://10.211.55.11:5598/data/ > ls
flights_from_pg.csv

- setup gradle 
Gradle
Task	Command
build	gradle shadowJar
run (Scala, Java)	dse 10.211.55.11 - u username -p password spark-submit --class com.datastax.spark.example.WriteRead build/libs/writeRead-0.1-all.jar


Answer the following queries using either Search or Analytics.

How many flights originated from the ‘HNL’ airport code on 2012-01-25  
How many airport codes start with the letter ‘A’
What originating airport had the most flights on 2012-01-23
Bonus – make a batch update to all records with a ‘BOS’ airport code using Spark and change the airport code to ‘TST’
** Please refer to BatchUpdate(loadFlightDf) method on AnalyticsQueries.scala
Bonus – What is the route having most delays?
Bonus – Is the airport activity a factor of the delay?
Bonus – Do airports generate delay at arrival and departure the same way?

Please see below output of of AnalyticsQueries.scala for further details:-












