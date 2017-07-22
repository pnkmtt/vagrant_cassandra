This will use vagrant and digital ocean

1. install vagrant on your system
2. run 'vagrant plugin install vagrant-digitalocean' to have vagrant use digitalocean an not local box
3. sign up for a digitalocean account here: https://www.digitalocean.com
4. once you have an account create a api key
5. setup key in your vagrant file
6. create a ssh key and populate it in digital ocesn
7. vagrant up

note:

if you use this you need to set your key and check the java and cassandra versions as they may have drifted

once the cluster is up:

example output for review:

# vagrant ssh node 1
root@node1:~# cd /opt/apache-cassandra-3.0.14/bin/
root@node1:/opt/apache-cassandra-3.0.14/bin# ./nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address        Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.132.92.149  121.9 KB   256          67.4%             0997722f-f90d-45d5-aeff-55dc59d56156  rack1
UN  10.132.18.94   103.14 KB  256          66.8%             19ffa67f-1d31-4ba4-8fed-40146e6aca50  rack1
UN  10.132.92.223  84.59 KB   256          65.8%             13f24196-4e14-4b8f-b355-e81db0be419c  rack1

# tail -n100 /opt/apache-cassandra-3.0.14/logs/system.log 


check data within cassandra:

root@node1:/opt/apache-cassandra-3.0.14/bin# pwd
/opt/apache-cassandra-3.0.14/bin
root@node1:/opt/apache-cassandra-3.0.14/bin# ./cqlsh
Connected to Training Cluster PXFLQEIH at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.14 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> 
cqlsh> create KEYSPACE test1 WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3}
   ... ;
cqlsh> use test1;
cqlsh:test1> create table customers ( id int primary key, firstname varchar, lastname varchar);
cqlsh:test1> INSERT into customers (id, firstname, lastname) values (1, 'John', 'Smith');
cqlsh:test1> INSERT into customers (id, firstname, lastname) values (2, 'Alice', 'Jones');
cqlsh:test1> INSERT into customers (id, firstname, lastname) values (3, 'Bob', 'Jones');
cqlsh:test1> select * from customers; 

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test1> select * from customers where id = 3; 

 id | firstname | lastname
----+-----------+----------
  3 |       Bob |    Jones

(1 rows)
cqlsh:test1> select * from customers where firstname = 'Jones';
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"
cqlsh:test1> 
cqlsh:test1> quit
root@node1:/opt/apache-cassandra-3.0.14/bin# ./nodetool getendpoints test1 customers 1
10.132.18.94
10.132.92.149
10.132.92.223
root@node1:/opt/apache-cassandra-3.0.14/bin# ./nodetool getendpoints test1 customers 2
10.132.92.223
10.132.92.149
10.132.18.94
root@node1:/opt/apache-cassandra-3.0.14/bin# ./nodetool getendpoints test1 customers 3
10.132.92.149
10.132.18.94
10.132.92.223


differnt replication:

root@node1:/opt/apache-cassandra-3.0.14/bin# ./cqlsh
Connected to Training Cluster PXFLQEIH at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.14 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> create KEYSPACE test2 WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 2};
cqlsh> use test2;
cqlsh:test2> create table customers ( id int primary key, firstname varchar, lastname varchar);
cqlsh:test2> INSERT into customers (id, firstname, lastname) values (1, 'John', 'Smith');
cqlsh:test2> INSERT into customers (id, firstname, lastname) values (2, 'Alice', 'Jones');
cqlsh:test2> INSERT into customers (id, firstname, lastname) values (3, 'Bob', 'Jones');
cqlsh:test2> select * from customers; 

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test2> quit
root@node1:/opt/apache-cassandra-3.0.14/bin# ./nodetool getendpoints test2 customers 2
10.132.92.223
10.132.92.149
root@node1:/opt/apache-cassandra-3.0.14/bin# exit
logout
Connection to 174.138.56.119 closed.
Manshoon:cassandra_vagrant mpanik000$ vagrant destroy node2
    node2: Are you sure you want to destroy the 'node2' VM? [y/N] y
WARNING: Unexpected middleware set after the adapter. This won't be supported from Faraday 1.0.
==> node2: Destroying the droplet...
Manshoon:cassandra_vagrant mpanik000$ vagrant ssh node1
root@node1:/opt/apache-cassandra-3.0.14/bin# ./cqlsh
Connected to Training Cluster PXFLQEIH at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.14 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> use test1;
cqlsh:test1> select * from customers;

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test1> use test2;
cqlsh:test2> select * from customers;

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test2> 
cqlsh:test2> CONSISTENCY ;
Current consistency level is ONE.
cqlsh:test2> 
cqlsh:test2> CONSISTENCY THREE;
Consistency level set to THREE.
cqlsh:test2> use test;1
         ... 
cqlsh:test2> use test1;
cqlsh:test1> select * from customers;
NoHostAvailable: 
cqlsh:test1> CONSISTENCY TWO;
Consistency level set to TWO.
cqlsh:test1> select * from customers;

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test1> CONSISTENCY QUORUM;
Consistency level set to QUORUM.
cqlsh:test1> select * from customers;

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test1> use test2;
cqlsh:test2> select * from customers;
NoHostAvailable: 
cqlsh:test2> 
cqlsh:test2> CONSISTENCY ONE;
Consistency level set to ONE.
cqlsh:test2> select * from customers;

 id | firstname | lastname
----+-----------+----------
  1 |      John |    Smith
  2 |     Alice |    Jones
  3 |       Bob |    Jones

(3 rows)
cqlsh:test2> 
root@node1:/opt/apache-cassandra-3.0.14/bin# ./nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address        Load       Tokens       Owns    Host ID                               Rack
UN  10.132.92.149  74.8 KB    256          ?       0997722f-f90d-45d5-aeff-55dc59d56156  rack1
UN  10.132.18.94   81.53 KB   256          ?       19ffa67f-1d31-4ba4-8fed-40146e6aca50  rack1
DN  10.132.92.223  93.13 KB   256          ?       13f24196-4e14-4b8f-b355-e81db0be419c  rack1

Note: Non-system keyspaces don't have the same replication settings, effective ownership information is meaningless
root@node1:/opt/apache-cassandra-3.0.14/bin# 






