# Initial Evaluation

##### Environment #####
The machine we used for the tests is a PC with an Intel i7 CPU (4 physical cores + 4 virtual cores using Hyperthreading technology and 8GB of RAM. 
We created 5 virtual machines: one using 2 cores and 2 GB of RAM (main-node), while the remaining four using 2 cores and 1GB of RAM (gru-nodes).

##### Deployment #####
We deployed in the main-node the etcd server, as well as influxdb and Grafana. We deployed in the main-node also a load-balancer (using the [MuSim](https://github.com/elleFlorio/mu-sim) tool) that act as the endpoint of the application: it receives the requests for the microservice and send them to an instance of the microserviece that is chosen randomly. The main node also runs Apache Jmeter that is used to send requests to the application. Gru-Agents are deployed in each gru-node (gru-nodes are part of the same cluster). Gru-Agents are configured to run the autonomic loop every 2 minutes. The communication is limited to 2 peers randomly chosen at each iteration of the autonomic loop.

##### Test Application #####
We used the [MuSim](https://github.com/elleFlorio/mu-sim) tool (modified to add the correct log entry) to create a microservice that runs in a Docker container and simulates the work with a function that keeps busy the CPU according to a time that is computed at the arrival of each request. The time for the computation is chosen according to the exponential distribution with $\lambda = 5000ms$. The maximum response time of the service has been set as 3 times the $\lambda$, i.e. 15000ms.

##### Goal #####
The system starts with one active instance of the microservice in a gru-node and Gru should scale the microservice according to the variation of the workload. The metric considered for the auto-scaling are the CPU utilization of the container and its load (response time over maximum response time). The workload has been generated using Apache Jmeter and goes from 0.1 to 2 RPS (Requests Per Second) and last one hour. The workload has been chosen taking into account the available resources of the nodes. The execution time of the experiment has been set to one hour in order to have a time interval between each autonomic iteration that was not too short, letting Gru-Agents to collect an adequate amount of data to analyze.

##### Results summary #####
We executed 5 runs of the test obtaining similar results. The results show that Gru is able to scale containers in order to keep the response time under the threshold defined in the Gru-Service configuration of the microservice, i.e. 15 seconds, with an average response time of 5 seconds. The number of active instances does not follow exactly the workload, but may present variations; this is due to the probabilistic approach adopted by Gru. However, since Docker containers can be started and stopped in a time of seconds, this behavior does not represent a relevant problem in the usage of resources.
