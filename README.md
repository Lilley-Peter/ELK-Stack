# ELK-Stack
Heres how to get ELK going in a Yugabyte Cluster

Fundamental Knowledge Base
Kibana depends on Elasticsearch, this means that Elasticsearch has to be up and running for you to start Kibana
Node Tokens are only valid for 30 minutes 
On the initial startup of elastic search you be be provided with a password, please take note of it, it CAN be changed later in the GUI
After using the command <filebeat setup> you might get a licence issue error. If this happens stop all your services and start them again
Elasticsearch Cluster Node
Important links:
https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rpm.html 	

Prerequisites:
Java installed
CentOS
yum update
sudo yum install java-1.8.0-openjdk.x86_64
This should install java
Verify
java -V OR java –version

Open Ports: 
9200

Installation on the HOST Node Elasticsearch, Kibana, & Filebeat:
CentOS/RHEL

$ sudo su

$ yum install java-1.8.0-openjdk.x86_64

$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

Now create a file called elasticsearch.repo in the /etc/yum.repos.d/ directory for RedHat based distributions, or in the /etc/zypp/repos.d/ directory for OpenSuSE based distributions, containing:

$ cd /etc/yum.repos.d/

$ vi elasticsearch.repo

elasticsearch.repo


[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md

$ sudo yum install --enablerepo=elasticsearch elasticsearch

$ systemctl enable elasticsearch.service

elasticsearch.yml configuration 

$ sudo su

$ cd /etc/elatiscsearch/

$ cp elasticsearch.yml elasticsearch.yml.DEFAULT

$ vi elasticsearch.yml

The cluster.name has to be the same on every node in the cluster
 
node.name needs to be unique for each node

network.host needs to be the IP address we got above

discovery.seed_hosts: ["same IP as network.host"]

cluster.initial_master_node: ["same IP as network.host"]

Screenshots on the Elasticsearch Node 
IP verification

Elasticsearch.yml






Starting elasticsearch:

$ systemctl start elasticsearch.service
$ systemctl status elasticsearch.service
Tips and tricks:
Check to see it the “Begin Security Auto Configuration” is present
This will be the last section in the  elasticsearch.yml file version 8.6+
Open and other terminal on the same node and run:
$ tail -f /var/log/elasticsearch/my-application.log
If you have errors:
journalctl -ru elasticsearch.service -n 100
Kibana on Cluster Node
Important Links: 
https://www.elastic.co/guide/en/kibana/7.17/rpm.html 	

Open Port:
5601


Create a file called kibana.repo in the /etc/yum.repos.d/ directory for RedHat based distributions, or in the /etc/zypp/repos.d/ directory for OpenSuSE based distributions, containing:

$ cd cd /etc/yum.repos.d/

$ vi kibana.repo
Kibana.repo

[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md

$ yum install kibana

$ systemctl enable kibana.service

Kibana.yml
vi /etc/kibana/kibana.yml 

Change the following 
server.port 5601
Server.host 10.0.0.52
elasticsearch.hosts: ["http://10.0.0.52:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "password"






Starting Kibana:
$ systemctl daemon-reload
$ systemctl enable kibana.service
$ systemctl start kibana.service
$ systemctl status kibana.service

If you have errors-open a new terminal 
$ journalctl -ru kibana.service -n 100
$ tail -f /var/log/kibana/kibana.yml

Adding a node to your elasticsearch cluster
You can repeat this process above on each node in the cluster if you want

Go to the node that Elasticsearch and Kibana is installed on, if you are following along that would be a node in your universe.

Tips and Tricks:
The cluster.name needs to be the same in each elasticsearch.yml file so they know its the same cluster
The node.name should be unique for each node
Your network.host is your private IP
To check to see if a node was added to the cluster
https://<Your Univers Node>:9200/_cluster/health?pretty
Run <yum update> to start

Run:
$ /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
##### NOTE THE ENROLLMENT TOKEN IS ONLY VALID FOR 30 MINUTES ####

Now go to node 2 in the cluster run: 
$ /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>

$ cd /etc/elasticsearch/
$ ip addr (Copy the ent2 private IP  for configuring the elasticsearch.yml file)

$ vi elasticsearch.yml 

Elasticsearch.yml (On an cluster node)








$ systemctl daemon-reload
$ systemctl start elasticsearch.service
$ systemctl status elasticsearch.service

If you have errors-open a new terminal 
$ journalctl -ru elasticsearch.service -n 100
$ tail -f /var/log/elasticsearch/my-aaplication.yml


Filebeat Universe Node
Important Links:
https://www.elastic.co/guide/en/beats/filebeat/7.17/setup-repositories.html#_yum 

Install Filebeat
$ sudo rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

Create a file with a .repo extension (for example, filebeat.repo) in your /etc/yum.repos.d/ directory and add the following lines:

$ cd /etc/yum.repos.d/

$ vi filebeat.repo

Filebeat.repo



[elastic-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md

Filebeat.yml
/etc/filebeat/filebeat.yml
Things to change:
setup.dashboards.enabled: true
setup.kibana:
	host: “10.0.0.50:5601”
output.elasticsearch:
	host: [“10.0.0.50:5601”]











Starting filebeat
$ yum install filebeat

$ systemctl enable filebeat.service

$ filebeat modules enable postgresql

$ filebeat test output

$ systemctl start filebeat.service

$ filebeat setup


Enable Postgresql
Postgresql.conf
/mnt/d0/pg_data/postgresql.conf

















Here at #log_statment = ‘none’ can be changed to ‘all’








Postgeresql.yml
vi /etc/filebeat/modules.d/postgresql.yml
vi /mnt/d0/yb-data/tserver/logs/postgresql-2023-01-31_182204.log
/mnt/d0/yb-data/tserver/logs/*.log


For Postgresql Logs:



For Yugaware logs:


The container is the container containing the version of postgres installed by replicated: 
You can navigate to it by finding the same container in /var/lib/docker/containers/



Now you can save all your yml files and start all your services.

Adding Logstash To Filebeat
Important Links:
https://www.elastic.co/guide/en/logstash/7.17/installing-logstash.html#_apt 
https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns 

Install Logstash:
Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo

$ cd /etc/yum.repos.d/

$ vi logstash.repo 

[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md


$ yum install logstash

Yugabyte.conf
To customise your search follow this documentation: https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns 

$ cd /etc/logstash/conf.d/
$ vi yugabyte.conf 

Tip: the <index> name has to be all lowercase

$ systemctl restart logstash.service
$ tail -f /var/log/logstash/logstash-plain.log


You should see that logstash has started on port 5044

Now we need to change the filebeat.yml file to see the output from logstash instead of elasticsearch

$ cd /etc/filebeat/
$ vi filebeat.yml

TO DO
Comment out <output.elasticsearch> and <host> under <Elasticsearch Output> and uncomment <output.logstash> and <hosts> under <Logstash Output> (Use the private IP of your Universe Node)


$ systemctl restart filebeat.service
$ systemctl restart logstash.service

Adding ELK to nodes in the Cluster
At this point you can continue to these steps on the nodes in the cluster, now the nodes in the cluster will be on CentOS so the commands are a little different.
SSH into the first node by following these steps:
Nodes -> Actions -> Connect 

Copy the command and paste it into your terminal and change the user from yugabyte to centos


Install on 1 CLUSTER Node:
Node 1:
Link: 
https://www.elastic.co/guide/en/elasticsearch/reference/7.17/deb.html	

Example:
$sudo ssh -i /opt/yugabyte/yugaware/data/keys/ddbc9202-8054-4f62-8a5f-b7164eea013b/yb-dev-elk_ddbc9202-8054-4f62-8a5f-b7164eea013b-key.pem -ostricthostkeychecking=no -p 54422 centos@10.0.0.45

$ sudo su
$ yum clean all
$ yum makecache
$ yum update
$ yum install java
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
$ cd /etc/yum.repos.d/
$ vi elasticsearch.repo
#### ADD THIS TO THE elasticsearch.repo FILE JUST CREATED ####
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md

$ yum install --enablerepo=elasticsearch elasticsearch
$ systemctl daemon-reload
$ systemctl enable elasticsearch.service

Screenshot of elasticsaerch.repo




$ systemctl daemon-reload
$ systenctl enable elasticsearch.service


You can repeat this process on each node in the cluster if you want
Adding a node to your elasticsearch cluster
Go to the node that Elasticsearch and Kibana is installed on, if you are following along that would be your YBA Node.

Tips and Tricks:
The cluster.name needs to be the same in each elasticsearch.yml file so they know its the same cluster
The node.name should be unique for each node
Your network.host is your private IP
To check to see if a node was added to the cluster
https://<Your YBA Node>:9200/_cluster/health?pretty

Run:
$ /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
##### NOTE THE ENROLLMENT TOKEN IS ONLY VALID FOR 30 MINUTES ####

Now go to node 1 in the cluster run: 
$ /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>

$ cd /etc/elasticsearch/
$ ip addr (Copy the ent2 private IP  for configuring the elasticsearch.yml file)

$ vi elasticsearch.yml 

Elasticsearch.yml (On a cluster node)









$ systemctl daemon-reload
$ systemctl start elasticsearch.service
$ systemctl status elasticsearch.service

If you have errors-open a new terminal 
$ journalctl -ru elasticsearch.service -n 100
$ tail -f /var/log/elasticsearch/my-aaplication.yml
Kibana GUI for filebeats
Now we can go to the Kibana GUI but http://<Universe Node IP>:5601
Now click the hamburger menu on the left
Go to Observability -> Logs 



Kibana GUI for logstash


