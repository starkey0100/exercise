# exercise
Install a two node cluster for DSE Analytics workload using Opscenter.
   - node 0 for Opscenter. 2.5GB RAM and 2 Cores
   - node 1 for DSE Analytics 6GB RAM & 3 Cores
   

## Prerequisites:-

Vagrant
Virtual Box

Instructions:-

This Vagrant template sets up DataStax Enterprise (DSE) on a configurable number of VMs:

node[0-n] - DSE nodes (with prerequisites and DSE already installed)
Notes:
## Step1:

To bring up the nodes please export creadential of datastax academy before vagrant up, you can bring up the DSE nodes with the following:

`$ export VAGRANT_DSE_USERNAME=your-username`

`$ export VAGRANT_DSE_PASSWORD=your-password`
`$ # optional: add another row in pocd_nodes Array for more nodes in Vagrantfile (default 2)`
`$ vagrant up`

When the up command is done, you can check the status of the VMs:

`$ vagrant status`

`node0           running (virtualbox)`
`node1           running (virtualbox)`

## Step2: Install the cluster using LCM :-
Create the ssh, config, repository on OpsCenter for installing the cluster.
DSE Analytics needs additional options such as AlwaysOn_SQL & DSE Authentication. Please refer the links below :-

Pro Tip(Read the instruction on following links carefully and follow them)

alwayson_sql: https://docs.datastax.com/en/dse/6.7/dse-dev/datastax_enterprise/spark/alwaysOnSql.html
DSE Authentication: https://docs.datastax.com/en/dse/6.7/dse-admin/datastax_enterprise/security/Auth/secEnableDseAuthenticator.html

Once cluster, DC & Nodes are define, submit the install job for the cluster. 

Check the workload type:-
`root@node1:/home/vagrant# dsetool -l cassandra -p cassandra ring `


`Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns`                ` Token                                        Health [0,1] `
`10.211.55.11     DC1                  rack1        Analytics(SM)        no     Up      Normal   469.39 KiB       ?   `        `         -9223372036854775808                         0.70   `      

`
Cluster Installed :- 
Below is LCM view
![image](https://user-images.githubusercontent.com/50682370/57921809-e6c0c800-7863-11e9-9fb9-e7b313fa8ddc.png)

Below is OpsCenterView 
![image](https://user-images.githubusercontent.com/50682370/57921982-46b76e80-7864-11e9-89ed-6331bdde53a3.png)

## Step3: Prepare the node for enabling analytics. 
Please go through the instructions again. 
Tip: For single node cluster, replication factor of keyspace can be kept as '1' with 'NetworkTopologyStrategy' like below
 `ALTER KEYSPACE "HiveMetaStore"
   WITH REPLICATION = {
   'class': 'NetworkTopologyStrategy', 
   'DC1': '1'};`
   
alwayson_sql instructions: https://docs.datastax.com/en/dse/6.7/dse-dev/datastax_enterprise/spark/alwaysOnSql.html

- create a new user with SUPERUSER login. Disable default Cassandra user.


`anoop@cqlsh> ALTER ROLE cassandra WITH SUPERUSER = false AND LOGIN = false AND password='new_secret_pw';


`anoop@cqlsh> LIST ROLES;`


- grant proxy to alwayson_sql user for the new user. 
- Once all changes are done, restart the dse service and Alwayson_sql comes up itself if everything was setup correctly. 

`root@node1:/home/vagrant# service dse restart
 * Restarting DSE daemon dse                                                                                                                                  [ OK ] 

`DSE daemon starting with Spark enabled (edit /etc/default/dse to disable)
                                                                                                                                                              [ OK ]`
`root@node1:/home/vagrant# dse client-tool -u a#### -p h###y alwayson-sql status
AlwaysOn SQL 10.211.55.11:10001 status: Running
`
!! All set to solve some problems. 

## Step4. Prepare the IDE for running some spark jobs.
used Gradle:-
- Clone the Datastax project template 
https://github.com/datastax/SparkBuildExamples

- place the csv file on DSEFS for spark job to read.

`dsefs dsefs://10.211.55.11:5598/ > mkdir data
dsefs dsefs://10.211.55.11:5598/ > cd data/ 
dsefs dsefs://10.211.55.11:5598/data/ > ls
dsefs dsefs://10.211.55.11:5598/data/ > cp file:/repository/flights_from_pg.csv /data/
dsefs dsefs://10.211.55.11:5598/data/ > ls
flights_from_pg.csv`

- setup gradle 

### Gradle

Task                | Command
--------------------|------------
build               | `gradle shadowJar`
run (Scala, Java)   | `dse -u alwayson_sql spark-submit -u username -p password --class     com.datastax.spark.example.FlightsData /repository/dseexercise-0.1-all.jar`

# Exercise Questions

### please refer FlightsData.scala. 
Create the base data model using the following table definition.
 

`CREATE TABLE flights (

          ID int PRIMARY KEY,

          YEAR int,        

          DAY_OF_MONTH int,

          FL_DATE timestamp,

          AIRLINE_ID int,

          CARRIER varchar,

          FL_NUM int,

          ORIGIN_AIRPORT_ID int,

          ORIGIN varchar,

          ORIGIN_CITY_NAME varchar,

          ORIGIN_STATE_ABR varchar,

          DEST varchar,

          DEST_CITY_NAME varchar,

          DEST_STATE_ABR varchar,

          DEP_TIME timestamp,

          ARR_TIME timestamp,

          ACTUAL_ELAPSED_TIME timestamp,

          AIR_TIME timestamp,

          DISTANCE int)`

Write a program to load the source data from the flights_from_pg.csv file into the flights table.
### please refer FlightsData.scala. 

How many records loaded?

Were any errors returned during data loading?

Answer: couldn`t parse the date to LocalTime. Tried using date type of spark and cassandra which works on open source spark/cassandra but didn`t work on dse cluster. i was getting null on FL_DATE so ingested it as varchar. Solves the current use case.
Had nulls on ARR_TIME , used spark function na.fill to enter "missing arival time" , could have used the cassadra config to drop nulls but for this use case i thought its not needed.

LocalTime & Localdate parsing works fine with another csv file which i downloanded flights.org.

Please provide your code.
Answer: Please refer     `define_flighttable(spark) & load_flighttable(loadFlightDf)` in `FlightsData.scala`

Create and populate a Cassandra table designed to list all flights leaving a particular airport, sorted by time.
Answer: Please refer `departures_table(spark, loadFlightDf)` in `FlightsData.scala`

Create and populate a Cassandra table designed to provide the carrier, origin, and destination airport for a flight based on 10 minute buckets of air_time.

Answer:  Please refer `airports_info(spark, loadFlightDf)` in `FlightsData.scala`

Answer the following queries using either Search or Analytics.

### please refer AnalyticsQueries.scala 

How many flights originated from the ‘HNL’ airport code on 2012-01-25


`Answer: 288`


How many airport codes start with the letter ‘A’


`Answer: 22`

What originating airport had the most flights on 2012-01-23


`Answer: [ATL,2155]  ATLANTA`


Bonus – make a batch update to all records with a ‘BOS’ airport code using Spark and change the airport code to ‘TST’


`Answer: Please refer to AnalyticsQueries.scala`


Bonus – What is the route having most delays?


`Answer: routes having most delays: SFO -> LAX`


Bonus – Is the airport activity a factor of the delay?
`Answer: Yes`


Bonus – Do airports generate delay at arrival and departure the same way?
`Answer: Yes`









