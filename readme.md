# Scaling up an existing service
To meet the requirements I propose a migration to an EKS cluster on AWS. This will provide the following benifits:

1. Availability and resilience:

EKS: Amazon EKS runs the Kubernetes control plane across multiple availability zones, ensuring high availability. It automatically detects and replaces unhealthy control plane nodes.
Docker Compose: It is mainly designed for defining and running multi-container applications on a single host, and doesn't inherently support high availability or failover capabilities.

2. Performance and scalability:

EKS: Comes with the Cluster Autoscaler which can automatically adjust the number of nodes in your cluster based on demand, ensuring efficient use of resources.
Docker Compose: Scaling services is manual and limited to the host's resources.

3. Security:

EKS: Benefits from AWS's security features. It integrates with IAM for access control and provides automated security patches.
Docker Compose: Security configurations are manual, and it doesn't have integrated tools for access control.

4. Observability:

EKS: Easily integrates with AWS monitoring tools like CloudWatch and X-Ray. In this repo I use Prometheus, Grafana and Fluent bit.
Docker Compose: Basic logging and monitoring capabilities, but lacks deep integration with advanced monitoring tools.

5. Pricing and Costs:

EKS: Users pay for AWS resources (e.g., EC2 instances) they create to run Kubernetes worker nodes. Costs can be optimized based on demand.
Docker Compose: No inherent costs, but scaling might require more expensive infrastructure upgrades, and there is no built-in way to optimize costs based on usage.

6. Testing, migration, and roll-out:

EKS: Kubernetes provides sophisticated deployment strategies like rolling updates, blue-green deployments, and canaries.
Docker Compose: Basic support for rolling out changes, but lacks advanced deployment strategies.

## Helm charts and files in repo
To deploy the pods to EKS and to have some I propose the following structure in the Github repo for this application:

```/users-info/
|-- .github/                # GitHub Actions configuration and scripts
|   |-- workflows/         # GitHub Actions CI/CD workflows
|       |-- release.yml    # Workflow for releasing/updating charts
|-- templates/             # Kubernetes YAML templates
|   |-- alert-rules.yml
|   |-- deployment.yaml
|   |-- prometheus-config.yaml
|   |-- hpa.yaml        
|   |-- service.yaml       
|-- .helmignore            # Patterns to ignore when packaging
|-- Chart.yaml             # Information about the Helm chart
|-- values.yaml            # Default configuration file 
|-- README.md              # README with information about your chart and its values
|-- LICENSE                # License of your Helm chart 

```

This is boilerplate code and has not been tested on an actual EKS cluster. Also the files are example files to give a broad idea. For instance in the release.yaml there are plain secrets now which is a red flaf off course. Hashicorp vault operator or github actions secrets would allready be a much better idea.

## Github link:
[GitHub Repository](https://github.com/AlbanBarbosa1982/users-info)

### Performance and scalabillity in depth solution
To make sure the infra and the application can handle the load to the application I have include a service.yaml file which creates a loadbalancer that can distribute external traffic to the pods running behind the service.

To increase performance and to make this application more scalable I have added a HorizontalPodAutoscaler (HPA) resource that based upon the CPU treshold of 80% will spin up another pod untill a maximum of ten. By scaling out spikes in traffic can be handled better and single pods will not be overloaded and this will benefit performance. The CPU metric here uses is just and example and should be tweaked according to the traffic and load.

### Observability in depth solution
To have observability for this application and the cluster "shuttle" as a whole I propose to have Prometheus, Grafana and Fluent Operator installed clusterwide. I would install the tools as operators with default helm charts. 

#### Prometheus
Prometheus can be used to monitor metrics of the EKS nodes the cluster is running of but also to monitor the resources and traffic to the pods. In this repo I have added a prometheus-config.yaml file to monitor out of the box metrics as CPU/Mem?HTTP requests etc.

#### Fluent bit
Cluster wide fluent operator will be installed and configured to collect logs from the different pods and also this application. In this repo in the deployment.yaml code is added to create a sidecar pod that stores the logs and also some fluent-bit config is in there. the logs can then be sent to CloudWatch logs where they can be access in a central place. 

#### Alerting
I have also added a alert-rules.yml to have alerting setup upon basic examples of memory and cpu here that will be use Alertmanager which is a separate resource that also needs to be installed cluster wide.

#### Grafana
In the cluster wide Grafana instance setup prometheus as the datasource via the url in prometheus. Then a default dashboard can be created for these default metrics that are scraped by Prometheus. 