---

copyright:
  years: 2024, 2025
lastupdated: "2025-02-27"

keywords: HPC, resource pools, HPC compute, Hybrid Cloud, Hybrid HPC, high performance computing

subcollection: hybrid-hpc-cloud-pool

authors:
  - name: John Easton
    url: https://linkedin.com/in/johnpeaston
  - name: "John Easton"
    url: "linkedIn profile URL"

version: 1.3

deployment-url: url

use-case: HPConIBMCloud

industry: FinancialSector, Electronics, Healthcare

compliance: ISOIEC27001

content-type: reference-architecture

production: false

---

{{site.data.keyword.attribute-definition-list}}

# Hybrid HPC with dynamic cloud resource pools
{: #hybrid-hpc-dynamic-cloud-pool}
{: toc-content-type="reference-architecture"}
{: toc-industry="FinancialSector, Electronics, Healthcare"}
{: toc-use-case="HPConIBMCloud"}
{: toc-compliance="ISOIEC27001"}
{: toc-version="1.3"}

This reference architecture summarizes the best practices for deploying a Hybrid High Performance Computing (HPC) environment connecting an on-premises HPC environment to a dynamically provisioned pool of HPC compute resources on {{site.data.keyword.Bluemix}}. An organization with an existing HPC on-premises facility might decide to augment this facility with these dynamic cloud-based resources.

![Hybrid HPC high level architecture](images/hybrid-hpc-cloud-pool-hla.drawio.svg "Hybrid HPC high level architecture"){: caption="High level Hybrid HPC architecture" caption-side="bottom"}

In the diagram, an existing HPC environment on-premises is connected to an HPC environment in {{site.data.keyword.Bluemix_notm}}. Jobs can be submitted to and run on either compute environment. There is a communications link between the on-premises data center and the {{site.data.keyword.Bluemix_notm}} data center, for example, using a Direct Link. The Hybrid HPC environment is enabled by a hybrid HPC capability that hides the complexity of the environment from the user and automates decisions about which compute jobs are processed where through a set of rules and policies. Hybrid HPC environments where data needs to be transferred or kept synchronized between on-premises and the cloud will also require a capability to enable this data movement.

## Architecture diagrams
{: #dyn-rchitecture-diagrams}

The architecture for the Hybrid HPC with dynamic cloud resource pools has multiple constituent layers:

1. Infrastructure layer
1. Software layer
1. Data layer

### Infrastructure layer
{: #dyn-infrastructure-layer}

Review the following architecture diagram for the infrastructure and cloud services that are used to deliver the Hybrid HPC with dynamic cloud resource pools pattern:

![Infrastructure architecture diagram for Hybrid HPC with dynamic cloud resource pools.](/images/hybrid-hpc-cloud-dynamic-pool-infra.drawio.svg "Infrastructure architecture diagram for Hybrid HPC with dynamic cloud resource pools"){: caption="Infrastructure architecture diagram for Hybrid HPC with dynamic cloud resource pools" caption-side="bottom"}

From an infrastructure perspective, the HPC environment in {{site.data.keyword.Bluemix_notm}} consists of one or more HPC Cluster Management Nodes. Multiple nodes are deployed to provide resilience of the environment. These HPC Cluster Management Nodes distribute workloads across a pool of Compute Nodes. The number and type of Compute Nodes depends on the characteristics of the workload(s) needing to be run within the environment. Both HPC Cluster Management Nodes and Compute Nodes are deployed as Virtual Server Instances (VSIs) within a Virtual Private Cloud (VPC).

The Autoscaler component is responsible for provisioning and deprovisioning compute nodes based on the resource needs of the workload(s) to be processed within the cloud HPC environment.

Many HPC applications process data. A number of Storage Nodes are deployed to deliver this data to the Compute Nodes. The Storage Nodes typically use attached Block Storage and present a Shared File system within which the data is stored. The Compute Nodes read and write data to the Shared File system. There is also a requirement for a Shared File system to hold the metadata used by the HPC Cluster Management Nodes. This is separate to the file system used to store the application data.

The HPC environment in the cloud will likely need its own DNS service to provide name resolution and Virtual Private Endpoints for secure access to other cloud services such as Monitoring, Logging, Identity and Access management (IAM), and so on.

Network connectivity between the enterprise data center and {{site.data.keyword.Bluemix_notm}} is likely to use a Direct Link private network or a Virtual Private Network (VPN).

### Software layer
{: #dyn-software-layer}

Review the following architecture diagram for the software that is used to deliver the Hybrid HPC with dynamic cloud resource pools pattern:

![Software architecture diagram for Hybrid HPC with dynamic cloud resource pools.](/images/hybrid-hpc-cloud-dynamic-pool-sw.drawio.svg "Software architecture diagram for Hybrid HPC with dynamic cloud resource pools"){: caption="Software architecture diagram for Hybrid HPC with dynamic cloud resource pools" caption-side="bottom"}

#### Execution flow
{: #dyn-execution-flow}

To understand the interactions between the components of the Hybrid HPC with dynamic cloud resource pools architecture, consider how a job is processed within the system. A typical problem that runs on an HPC system will be split into multiple jobs which are then run on the many compute nodes within the system. The way these are executed is:

1. The user invokes a computation through an application, web browser, or command line interface.
2. The job request is sent to the multicluster manager which uses pre-defined rules and policies to determine whether the job should be processed using on-premises HPC resources or in the cloud.
3. The multicluster manager sends the compute job to the cluster manager for the respective cluster (on-premises or cloud). The job is then queued locally for dispatch to the compute node(s).
4. The cloud cluster manager assesses the resource needs for the compute jobs.  If additional compute nodes are required to run the jobs, the cloud cluster manager uses an internal function called a Resource Connector to call the {{site.data.keyword.Bluemix_notm}} Autoscaler component.
5. The autoscaler calls the {{site.data.keyword.Bluemix_notm}} API to provision additional compute nodes into the cluster.  When these compute nodes are provisioned, the autoscaler informs the cloud cluster manager.
6. The cluster manager then sends the job to the HPC Agent. One agent runs on each of the compute nodes. The agent runs the relevant application executables to perform the computation. As part of the computation, the application might access data stored in the shared file system.

When the computational job completes, notification is sent back by the reverse route. The HPC agent informs the local cluster manager. This, in turn, informs the multicluster manager which returns the completion status to the user. Cloud compute nodes that are idle are removed from the cluster by the autoscaler according to a rule or policy.

### Data layer
{: #dyn-data-layer}

For those HPC applications that require data, there needs to be a mechanism to deliver data from on-premises to the cloud environment. There are two potential approaches that might be used to manage the movement of data:

- The data movement is managed by the HPC workload scheduler.
- The data movement is managed by the file system(s) holding the data.

The following is the architecture diagram for the data layer that is used to deliver the Hybrid HPC with dynamic cloud resource pools pattern. This outlines the two data movement approaches.

![Data architecture diagram for Hybrid HPC with dynamic cloud resource pools.](/images/hybrid-hpc-cloud-pool-data.drawio.svg "Data architecture diagram for Hybrid HPC with dynamic cloud resource pools"){: caption="Data architecture diagram for Hybrid HPC with dynamic cloud resource pools" caption-side="bottom"}

#### Execution flow for workload scheduler-managed data movement
{: #dyn-execution-flow-for-workload-scheduler-managed-data-movement}

Applications process data. This data needs to be available on the compute node(s) where the computation on that data will occur. If the data resides on-premises and the computation will occur in the cloud, then the data will need to be moved to the cloud. This data movement can be controlled by the workload scheduler.

1. The user invokes a compute job through an application, web browser or command line interface.
2. The job request is sent to the multicluster manager which uses pre-defined rules and policies to determine whether the job should be processed using on-premises HPC resources or in the cloud.
3. The multicluster manager communicates with the data manager components. These are responsible for moving the data from the on-premises file system to the cloud file system. The data requirement information includes the in-cloud cluster that the job is eligible to be forwarded to.
4. The data manager in the cloud requests that the data file(s) get copied to the local staging area (cache) in the cloud. The data requirement information includes the candidate clusters that the job is eligible to be forwarded to.
5. The required files are copied from the on-premises file system(s) to the in-cloud staging area.
6. When the files are available in the cloud, the data manager informs the cluster manager that the data is in place and that the compute jobs can be run.

Note that this data movement and any autoscaling of the cloud compute nodes can occur in parallel.

#### Execution flow for file system-managed data movement
{: #dyn-execution-flow-for-filesystem-managed-data-movement}

{{site.data.keyword.IBM_notm}} Storage Scale provides a high-performance parallel file system to meet the needs of HPC workloads. Part of Storage Scale is a capability called Active File Management (AFM). Active File Management provides on-demand movement of applications that is transparent to the application using the file system.  This is illustrated by Path A in the diagram.

When a user attempts to access a file in the cloud that is physically located on-premises, the Active File Management capability seamlessly manages the transfer of the file to the in-cloud file system. The in-cloud file system can be configured in different ways depending on whether it is to be used as a read-only "cache" or as a read-write file system. Data changes made to the in-cloud copy of the file are seamlessly transferred back to the primary copy stored on-premises.

In environments where the HPC workload needs a significant volume of data, the time taken to transfer it from the on-premises environment to {{site.data.keyword.Bluemix}}. These environments may benefit from a permanent data storage capability in the cloud with a mechanism to replicate changes to and from the on-premises primary copy. Such a permanent data storage capability will allow the cloud-based HPC environment to be more responsive to business needs and able to process workloads in a shorter time by removing the time taken to transfer data to the cloud.

## Design concepts
{: #dyn-design-concepts}

Review the design considerations and architecture decisions for the following aspects and domains:

- **Compute:** Virtual Servers
- **Storage:** Primary Storage
- **Networking:** Domain Name Services
- **Security:** Data Security, Identity & Access
- **Resiliency:** High Availability
- **Service Management:** Monitoring, Logging, Auditing and Tracking, Management and Orchestration

![Architecture design scope](/images/hybrid-hpc-heat-map.svg "Architecture design scope"){: caption="Architecture design scope" caption-side="bottom"}

### Design choices
{: #dyn-design-choices}

When creating a Hybrid HPC environment using dynamic cloud resource pools the key decision that needs to be taken is which resources to provision dynamically. The following diagram shows the key components of the HPC environment (cluster management, data and compute) and the ways that these can be either permanently or dynamically provisioned.

![Environment choices for Hybrid HPC with dynamic cloud resource pools.](/images/hybrid-hpc-dynamic-choice.drawio.svg "Environment choices for Hybrid HPC with dynamic cloud resource pools"){: caption="Environment choices for Hybrid HPC with dynamic cloud resource pools" caption-side="bottom"}

There is a spectrum of options with increasing cost, but also increasing responsiveness to business demands:

- The lowest cost but least responsive option is where the entire HPC environment (cluster management, data, and compute) is created from scratch as and when required.
- The next option is to permanently provision just the cluster management nodes and to provision both data and compute resources when required. In environments where there is no significant data required for computation to occur, this is likely to offer the best balance between minimized cost and greatest business responsiveness.
- Environments that use significant volumes of data may benefit from a permanent data presence in the cloud. This improves business responsiveness at the cost of that cloud data storage and just the compute nodes are provisioned on demand.
- Finally, there is the case where all HPC resources are permanently provisioned. [Hybrid HPC with Permanent Cloud Resource Pools](/docs/hybrid-hpc-persistent-cloud-pool) describes this option in more detail.

#### HPC cluster management software
{: #dyn-hpc-cluster-mgmt-software}

Different HPC problems require different HPC cluster management software solutions. {{site.data.keyword.Bluemix_notm}} provides two options. IBM Spectrum LSF (Load Sharing Facility) is a batch scheduler. Users submit jobs onto a queue and these are processed in turn according to the policies and rules that have been defined. IBM Spectrum Symphony is a realtime scheduler that's designed to deliver faster response times and aimed specifically at the needs of the Financial Services industry. {{site.data.keyword.Bluemix_notm}} provides tiles that can automatically deploy these software solutions into an HPC cluster.

Other HPC cluster scheduler solutions from the open source community such as Slurm and Condor, or from other commercial organizations are also available. These must be manually deployed. Refer to the websites of the open source projects or the documentation provided by commercial organizations for further information on how these can be used within the Hybrid HPC with persistent cloud resource pools architecture.

#### Storage options
{: #dyn-storage-options}

Most HPC environments consume data stored in file systems. {{site.data.keyword.Bluemix_notm}} provides two shared file systems. VPC File Storage provides a slower performance file system that can be used to store the metadata required by the cluster management software or for low-use data storage for HPC workloads, for example, application binaries. Workloads needing high performance parallel file systems should use IBM Storage Scale. This is best deployed on VPC Bare metal servers.

#### Compute nodes
{: #dyn-compute-nodes}

Computation is performed in Virtual Servers (VSIs). There are many different VSI profiles that can be chosen to best meet the compute and memory requirements of the application(s) being run within the HPC environment. Consider an application that requires 3 vCPUs of compute and 7GB of memory. The needs of this application might be met by the cx2-4x8 VSI profile which provides 4 vCPUs and 8GB of memory. If this profile is chosen, one instance of the application runs on one compute node at any one time. As a comparison, a VSI profile such as cx2-64x128 could be chosen instead. The workload scheduler can be configured to pack multiple jobs into a single VSI.  In this case, 18 instances of the application could run within a single VSI with this profile.

It is recommended that in environments where multiple applications with different resource needs run simultaneously, that the compute nodes be sized to support the application footprint requiring the most CPU and memory. The {{site.data.keyword.IBM_notm}} workload scheduling software is able to run multiple instances of workloads with reduced CPU and memory needs on these VSIs to make optimal use of the compute resources available.

The number of compute nodes will vary over time as the autoscaler in conjunction with the cluster management software, provisions and deprovisions compute capacity to meet the workload needs. Limits on the maximum and minimum numbers of nodes can be defined to ensure that costs can be controlled and that there is always some compute capability available for immediate use if needed.

## Requirements
{: #dyn-requirements}

The following table outlines the requirements used in the architecture for each aspect:

| Aspect | Requirements |
|-----| -----|
| Compute | Provide properly isolated compute resources with adequate compute capacity for the application. Remember to allow for the resource needs of the operating system and any other software needed. |
| Storage | Provide storage that meets the application data volume and performance requirements. |
| Networking | Deploy workloads in an isolated environment and enforce information flow policies.  \n Provide secure and encrypted connectivity to the cloud's private network for management purposes.  \n Distribute resolution to support the use of hostnames instead of IP addresses. |
| Security | Protect the boundaries of the application against the Denial of Service and application-layer attacks.  \n If it's required, encrypt all the application data in transit and at rest to protect from unauthorized disclosure.  \n Encrypt all the security data and operational and audit logs to protect from unauthorized disclosure. |
| Resiliency | Support application availability targets.  \n Ensure availability of the application in the event of a planned and an unplanned outage.  \n Provide highly available compute, storage, network, and other cloud services to handle application load and performance requirements.  \n Provide highly available storage for security data logs and backup data.  \n Automate recovery tasks to minimize down time.  \n The Autoscaler can be used to provision additional compute resources following a failure to ensure capacity is available to process compute jobs. |
| Service management | Monitor system and application health metrics and logs to detect issues that might impact the availability of the application.  \n Generate alerts and notifications about issues that might impact the availability of applications to trigger appropriate responses to minimize down time.  \n Monitor audit logs to track changes and detect potential security problems.  \n Provide a mechanism to identify and send notifications about issues found in the audit logs. |
{: caption="Requirements" caption-side="bottom"}

## Components
{: #dyn-components}

The following table outlines the products or services used in the architecture for each aspect:

| Aspects | Architecture component | How the component is used |
| -------------- | -------------- | -------------- |
| Compute | [Virtual Servers for VPC](/docs/vpc?topic=vpc-about-advanced-virtual-servers&interface=ui) | HPC cluster management nodes and compute nodes |
| Autoscaler | [VPC autoscaling instance groups](/docs/vpc?topic=vpc-creating-auto-scale-instance-group) | To scale numbers of compute nodes up and down |
| Storage | [VPC File Storage](/docs/vpc?topic=vpc-file-storage-vpc-about) | Low performance storage for HPC management metadata and/or lightweight application data access needs |
|  | [Storage Scale](/docs/storage-scale?topic=storage-scale-about-storage-scale) | High performance parallel file system for data-intensive HPC workloads |
| Networking | [Virtual Private Endpoint (VPE)](/docs/vpc?topic=vpc-about-vpe) | For private network access to Cloud Services, e.g., Key Protect, IAM, etc. |
|  | [Public Gateway](/docs/vpc?topic=vpc-about-public-gateways) | For secure client access to the HPC environment over the Internet |
|  | [Direct Link](/docs/dl?topic=dl-dl-about) | For private, dedicated connectivity between on-premises and cloud HPC resources.  If the dynamic HPC resources are used infrequently, it may be more appropriate to consider using [VPN for VPC](/docs/vpc?topic=vpc-vpn-overview) to provide a secure communication connection to the cloud |
|  | [DNS](/docs/dns-svcs?topic=dns-svcs-about-dns-services) | Domain Name Services for the HPC environment |
| Security | [Identity and Access Management](https://cloud.ibm.com/iam/overview){: external} | Identity and Access Management |
|  | [Key protect](/docs/key-protect?topic=key-protect-about) or [Hyper Protect Crypto Services](/docs/hs-crypto?topic=hs-crypto-get-started) | Hardware security module (HSM) and Key Management Service |
|  | [Secrets Manager](/docs/secrets-manager?topic=secrets-manager-getting-started) | Certificate and Secrets Management |
| Resiliency | [Virtual Servers for VPC](/docs/vpc?topic=vpc-about-advanced-virtual-servers&interface=ui) in conjunction with [Spectrum LSF](/docs/ibm-spectrum-lsf?topic=ibm-spectrum-lsf-about-spectrum-lsf) or [Spectrum Symphony](/docs/hpc-spectrum-symphony?topic=hpc-spectrum-symphony-about-spectrum-symphony) | The HPC workload is split across multiple VSIs. The HPC Management software (LSF or Symphony) manages failures of compute nodes by resubmitting failed compute jobs to other VSIs |
| Service Management | [Spectrum LSF](/docs/ibm-spectrum-lsf?topic=ibm-spectrum-lsf-about-spectrum-lsf) or [Spectrum Symphony](/docs/hpc-spectrum-symphony?topic=hpc-spectrum-symphony-about-spectrum-symphony) | HPC cluster management software provides application and performance status & monitoring, resource consumption and utilization |
|  | [IBM Cloud Monitoring](/docs/monitoring?topic=monitoring-about-monitor) | Operational monitoring |
|  | [IBM Cloud Log Analysis](/docs/log-analysis?topic=log-analysis-getting-started) | Operational logs |
|  | [Activity Tracker Event Routing](/docs/activity-tracker?topic=activity-tracker-getting-started) | Audit logs |
{: caption="Components" caption-side="bottom"}
