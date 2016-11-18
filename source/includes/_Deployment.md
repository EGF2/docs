# Deployment

## Config Management and Service Discovery

These two topics are pretty broad. There are different systems that are available in this area which can be utilized with EGF2 with some additional effort. Out of the box, config management and service discovery are organized as follows.

There is a single required parameter for each service to start - "config". This parameter should hold a URL pointing to a reachable location with a JSON config file for the service. When service is started it downloads config file. In case config file is not reachable the service will die with an error message. 

Config files for services can be stored in S3 bucket or any other convenient location reachable by services. Services are not checking whether config has changed or not. In order to apply changes in config to a set of services sysops person will have to restart a service or restart an instance that runs the service.

We strive to minimize inter service dependencies as a conscious architectural choice. There are some dependencies though, listed in the table below.

<table>
	<thead>
		<tr>
			<th>Service</th>
			<th>Talks To</th>
			<th>Config Parameters</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>client-api</td>
			<td>Data Storage, QueueSolution</td>
			<td></td>
		</tr>
		<tr>
			<td>client-data</td>
			<td>required</td>
			<td></td>
		</tr>
		<tr>
			<td>auth</td>
			<td>client-data</td>
			<td></td>
		</tr>
		<tr>
			<td>sync</td>
			<td>QueueSolution, client-data</td>
			<td></td>
		</tr>
		<tr>
			<td>pusher</td>
			<td>QueueSolution, client-data</td>
			<td></td>
		</tr>
		<tr>
			<td>scheduler</td>
			<td>QueueSolution, client-data</td>
			<td></td>
		</tr>
		<tr>
			<td>logic</td>
			<td>QueueSolution, client-data</td>
			<td></td>
		</tr>
		<tr>
			<td>file</td>
			<td>QueueSolution, client-data, S3</td>
			<td></td>
		</tr>
		<tr>
			<td>job</td>
			<td>QueueSolution, client-data</td>
			<td></td>
		</tr>
	</tbody>
</table>

In case a service should talk to another service (e.g. **client-api** talks to **auth** and almost all services talk to **client-data**) it will support an option in config file that will point to a URL for the required service. We don’t support multiple URLs pointing to a single service at the moment. 

In order to make a particular service fault tolerant and scalable we recommend the following AWS based setup:

1. Create an ELB for the service.
* Create ASG for the service and link it with the ELB.
* Create a Route53 DNS record for the ELB.
* Use Route53 DNS address to connect a service to another.

Advantages of the setup:

1. ELB will load balance requests to the service.
* ASG will provide auto scaling for the service - no need for sysops intervention in case of load spikes.


## Small Mode
This part will be described as step by step instructions how to deploy working infrastructure in AWS. So, if you have no AWS account, please create one.

### Deploy Data storage
In this step RethinkDB and Elasticsearch clusters/instances should be created. In small deployment it is not necessary to have separate instances for RethinkDB and Elasticsearch. General purpose instance of small size is enough for testing and at least medium-sized instance should be used for real data.  
Please consult original documentation of service/OS for proper configuration.

#### Deploy RethinkDB
For service configuration please consult original documentation.

#### Deploy single instance
This is the simplest option. No sharding, no replication and no fault-tolerance as a result. Enough for testing purposes.

#### Deploy cluster
As RethinkDB have no tools for cluster configuration all steps should be made manually. One of the instances should be chosen as `"master"` and the rest must be instructed to `"join"` it.

#### Deploy Elasticsearch
For service configuration please consult original documentation.  
EC2-discovery using cloud-aws plugin is a preferable way for inter-cluster nodes discovery.


### Deploy Subsystems
All subsystems can be deployed on the same instance. RAM memory of this instance should be more then 1Gb. In low load deployment it can be even micro instance with swap. Micro instance itself have only 1Gb of RAM and it is not enough to successfully install subsystem dependencies using `"npm install"` procedure.
For subsystems configuration simplicity and to achieve fault tolerance smart proxy should be used for data requests forwarding. So called client-node in case of elasticsearch and rethinkdb-proxy in case of RethinkDB. Both of them should be deployed on the same instance with subsystems.

### Deploy Single Entry Point
This step will finalize deployment process. In this example Nginx will be used.
As Nginx will be used as simple request router it is possible to run it on the same instance with subsystems.
Sample configuration can be found in commons repository.


## Scalable Mode
TODO

## Big Data Mode
TODO



