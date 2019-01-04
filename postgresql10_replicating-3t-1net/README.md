# Create PostgreSQL 10 master with streaming replication on multiple Ubuntu 18.04 VMs

Originally came from: https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fpostgresql-on-ubuntu%2Fazuredeploy.json

This template creates one master PostgreSQL 10 server with streaming-replication to multiple replication servers. Each database server is configured with multiple data disks that are striped into RAID-0 configuration using mdadm.

The template creates the following deployment resources:
* Virtual Network with one vnet and three subnets: "public 172.16.1.0/24" for the Bastion/Jumpbox VM | "private-apps 172.16.2.0/24" for internal services |  "private-data 172.16.3.4/24" for pgsql master and slave VMs
* Storage accounts to store VM data disks
* Public IP address for accessing the jumpbox via ssh
* Network interface card for each VM
* Multiple remotely-hosted Custom Script Extensions to strip the data disks and to install and configure PostgreSQL services

NOTE: To access the PostgreSQL servers, you need to use the externally accessible jumpbox VM and ssh from it into the backend servers.

Assuming your domainName parameter was "mypsqljumpbox" and region was "West US"
* Master PostgreSQL server will be deployed at the first available IP address in the subnet: 172.163.3.4
* Slave PostgreSQL servers will be deployed in the other IP addresses: 172.16.3.5,...etc.
* From your computer, SSH into the Bastion host `ssh mypsqljumpbox.westus.cloudapp.azure.com`
* From Bastion, SSH into App server to conntect to PGSQL cluster
* On the master, use the following code to create table and some test data within your PostgreSQL master database.

```
sudo -u postgres psql
create table table1 (name varchar(100));
insert into table1 (name) values ('name1');
insert into table1 (name) values ('name2');
select * from table1;
```

* From the jumpbox, SSH into one of the slave PostgreSQL servers and use psql to check that the data propaged properly

```
sudo -u postgres psql
select * from table1;
```

The following table outlines the deployment topology characteristics for each supported t-shirt size:

| T-Shirt Size | Database VM Size | CPU Cores | Memory | Data Disks | # of Secondaries | # of Storage Accounts |
|:--- |:---|:---|:---|:---|:---|:---|:---|:---|
| Small | Standard_A1 | 1 |1.75 GB | 2x1023 GB | 1 | 1 |
| Medium | Standard_A3 | 4 | 7 GB | 8x1023 GB | 1 | 2 |
| Large | Standard_A4 | 8 | 14 GB | 16x1023 GB | 2 | 2 |
| XLarge | Standard_A4 | 8 | 14 GB | 16x1023 GB | 3 | 4 |
