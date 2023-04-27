---
title: Benchmark Cassandra & CockroachDB with YCSB and HAProxy
date: 2018-07-30 07:49
author: selcuk
image: 
  thumbnail: /assets/images/msft.png
---

The group I was in at Microsoft Research, Asia, was on database, distributed database in particular. <a href="http://github.com/Microsoft/GraphView">GraphView</a>, which is a middleware within Microsoft's <a href="https://docs.microsoft.com/en-us/azure/cosmos-db/introduction">CosmosDB</a> that accepts graph operations and translates them to T-SQL executed in SQL Server or Azure SQL Database, is our focus. In order to  provide a Transaction-as-a-service at a global and persistent states, our team was formulating a new concurrency control protocol, which is based on a protocol from MIT, called <a href="https://people.csail.mit.edu/sanchez/papers/2016.tictoc.sigmod.pdf">TicToc</a> and several KV backends. At memory level, <a href="https://redis.io/">Redis</a> is picked and at disk level Facebook's <a href="http://cassandra.apache.org">CassandraDB</a> is picked.

Hence, a thorough study on Cassandra's performance on cluster is in need. <a href="https://github.com/cockroachdb/cockroach">CockroachDB</a>, as  the flagship product of a startup by some ex-Googlers, is a distributed SQL database built on a transactional and strongly-consistent key-value store, which is a perfect match against our own datastore. So I will also put down the guide to benchmark CockroachDB on our own cluster with <a href="https://github.com/brianfrankcooper/YCSB">YCSB</a>.
<ol>
 	<li style="list-style-type: none;">
<ol>
 	<li>Environment:
<ul>
 	<li>10 Ubuntu server nodes w. 1 Ubuntu client node</li>
 	<li>Cassandra 3.11.2 binaries</li>
 	<li>CockroachDB 2.0.2 binaries</li>
</ul>
</li>
 	<li>Cassandra Benchmark with Cassandra-stress tool
<ul>
 	<li>Install Java8 via:
<pre>sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer</pre>
</li>
 	<li>Download Cassandra via:
<pre>wget http://www-us.apache.org/dist/cassandra/3.11.2/apache-cassandra-3.11.2-bin.tar.gz
</pre>
</li>
 	<li>Make the following changes to conf/cassandra.yaml to setup the cluster
<ul>
 	<li><em>seeds</em>: Interneal IP address of each seed node. DO NOT MAKE ALL NODES SEED NODES. It is a best practice to have more than one seed node per datacenter. In our 10-node cluster, there is one seed node - 10.6.0.4</li>
 	<li><em>listen_address</em>: The IP address or hostname that Cassandra binds to for conencting to other Cassandra nodes. A.K.A. the private IP address of the currect node</li>
 	<li>[Optional] <em>rpc_address</em>: Private IP address of the current node.</li>
</ul>
</li>
 	<li>Start, Stop and Monitor Cassandra cluster
<pre>./bin/cassandra &amp;
pkill -9 java
./bin/nodetool status
</pre>
</li>
 	<li>Run Cassandra-stress tool
<code>Populate the cluster with 1 million record (default) and 500 threads</code>
<pre>./tools/bin/cassandra-stress write -node 10.6.0.4,10.6.0.5,10.6.0.6,10.6.0.12,10.6.0.13,10.6.0.14,10.6.0.15,10.6.0.16,10.6.0.17,10.6.0.18 -rate threads=500</pre>
<code>Test the cluster with 1 million read-requests and 500 threads </code>
<pre>./tools/bin/cassandra-stress read -node 10.6.0.4,10.6.0.5,10.6.0.6,10.6.0.12,10.6.0.13,10.6.0.14,10.6.0.15,10.6.0.16,10.6.0.17,10.6.0.18 -rate threads=500</pre>
<code>Test the cluster with a mixed workload (R/W: 50%:50%) and 500 threads</code>
<pre>./tools/bin/cassandra-stress mixed ratio\(write=1,read=1\) -node 10.6.0.4,10.6.0.5,10.6.0.6,10.6.0.12,10.6.0.13,10.6.0.14,10.6.0.15,10.6.0.16,10.6.0.17,10.6.0.18 -rate threads=500</pre>
</li>
</ul>
</li>
 	<li>Cockroach Benchmark via YCSB
<ul>
 	<li>Install Cockroach cluster
<code>Download Cockroach binaries</code>
<pre>wget -qO- https://binaries.cockroachdb.com/cockroach-v2.0.2.linux-amd64.tgz | tar  xvz</pre>
<code>Expand the limit on the number of file descriptors a process may have</code>
<pre>ulimit -HSn 51200</pre>
<code>Launch the root node</code>
<pre>nohup cockroach start --cache=0.4 --max-sql-memory=0.4 --insecure --host=10.6.0.4 &amp;</pre>
<code>Launch other nodes to join this cluster</code>
<pre>nohup cockroach start --cache=0.4 --max-sql-memory=0.4 --insecure --host=10.6.0.5 --join=10.6.0.4:26257 &amp;</pre>
<code>Verify the cluster is successfully launched by visiting on client node: </code>
<pre>https://10.6.0.4:8080</pre>
<code>Kill Cockroach process on current node</code>
<pre>pkill -9 cockroach</pre>
</li>
 	<li>Create YCSB database and usertable
<code>Open SQL shell from any node within our cluster</code>
<pre>./cockroach sql --insecure --host=10.6.0.4</pre>
<code>Create database "ycsb" if not exists</code>
<pre>create database ycsb;</pre>
<code>Create single table "usertable" if not exists</code>
<pre>CREATE TABLE usertable ( YCSB_KEY VARCHAR(255) PRIMARY KEY, 
	FIELD0 TEXT, FIELD1 TEXT, 
	FIELD2 TEXT, FIELD3 TEXT, 
	FIELD4 TEXT, FIELD5 TEXT, 
	FIELD6 TEXT, FIELD7 TEXT,
	FIELD8 TEXT, FIELD9 TEXT );</pre>
</li>
 	<li>Launch HA proxy as load balancer
<code>Synchronize time on each node since Cockroach node fails easily if out of sync</code>
<pre>sudo apt-get install ntp
sudo service stop ntp</pre>
<code>Edit /etc/ntp.conf by commenting any lines start with "server" or "pool" and add:</code>
<pre>server ntp.int.scaleway.com iburst
server ntp.ubuntu.com iburst</pre>
<code>Start ntp and verify if the server has been changed</code>
<pre>sudo service start ntp
sudo ntpq -p
</pre>
<code>Output should be something like this</code>
<pre>     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 10.1.31.39      .INIT.          16 u    - 1024    0    0.000    0.000   0.000
*alphyn.canonica 192.53.103.108   2 u  832 1024  377  237.330   -0.197   0.648
</pre>
<code>Generate HAproxy configuration file from root(primary) node</code>
<pre>./cockroach gen haproxy --insecure --host=10.6.0.4 --port=26257</pre>
<code>Copy it to the client node, install HAproxy and run it with this .cfg file</code>
<pre>scp haproxy.cfg elastas@10.6.0.9:~/
sudo apt-get install haproxy
ulimit -HSn 51200
haproxy -f haproxy.cfg</pre>
</li>
 	<li>Run YCSB script
<code>Download YCSB binaries to client node</code>
<pre>https://github.com/brianfrankcooper/YCSB/releases</pre>
<code>Load data from workloads</code>
<pre>python bin/ycsb load jdbc -P workloads/workloadc -p db.driver=org.postgresql.Driver -p db.url=jdbc:postgresql://10.6.0.9:26257/ycsb -p db.user=root -p db.passwd=root -p recordcount=200000 -threads 32 -p operationcount=500000 -p db.batchsize=100 -s</pre>
<code>Run workloads</code>
<pre>python bin/ycsb run jdbc -P workloads/workloadRO -p db.driver=org.postgresql.Driver -p db.url=jdbc:postgresql://10.6.0.9:26257/ycsb -p db.user=root -p db.passwd=root -p recordcount=200000 -threads 1024 -p operationcount=500000 -p db.batchsize=100 -s</pre>
</li>
 	<li>Since I have done this benchmarks a couple times, in order to simplify this process, I put a start.sh file in each cockroach folder. With our own servers, since all binaries are downloaded with cluster &amp; haproxy configured, you only need to change to cockroach directory and run start.sh (start-haproxy.sh for client node).</li>
 	<li>For Read-only &amp; Write-only workloads, change directory to ycsb-ro/ycsb-wo on ubuntu client node and run start.sh</li>
</ul>
</li>
</ol>
</li>
</ol>
Useful links:
<ol>
 	<li><a href="https://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsCStress.html">Cassandra-stress tool</a></li>
 	<li><a href="https://docs.datastax.com/en/cassandra/2.1/cassandra/tools/toolsCStressOutput_c.html">Interpreting the output of cassandra-stress</a></li>
 	<li><a href="http://cassandra.apache.org/doc/latest/tools/cassandra_stress.html">Cassandra-stress docs</a></li>
 	<li><a href="https://www.cockroachlabs.com/docs/v2.0/learn-cockroachdb-sql.html#main-content">Learn CockroachDB SQL</a></li>
 	<li><a href="https://www.scaleway.com/docs/how-to-configure-a-cockroachdb-cluster/">Install a multi-node Cockroach Database with HA proxy</a></li>
 	<li><a href="https://www.cockroachlabs.com/docs/stable/cluster-setup-troubleshooting.html">Troubleshoot Cluster Setup</a></li>
</ol>
