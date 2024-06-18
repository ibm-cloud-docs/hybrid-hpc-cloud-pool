---

# The YAML header is required. For more information about the YAML header, see
# https://test.cloud.ibm.com/docs/writing?topic=writing-reference-architectures

copyright:
  years: 2024
lastupdated: "2024-06-18"

keywords: # Not typically populated

subcollection: hybrid-hpc-cloud-pool

authors:
  - name: John Easton
    url: https://linkedin.com/in/johnpeaston
  - name: "John Easton"
    url: "linkedIn profile URL"

# The release that the reference architecture describes
version: 1.0

# Use if the reference architecture has deployable code.
# Value is the URL to land the user in the IBM Cloud catalog details page for the deployable architecture.
# See https://test.cloud.ibm.com/docs/get-coding?topic=get-coding-deploy-button
deployment-url: url

docs: https://cloud.ibm.com/docs/solution-guide

image_source: https://github.com/terraform-ibm-modules/module/reference-architectures/xxx.svg

related_links:
  - title: 'Title'
    url: 'https://url.com'
    description: 'Description.'
  - title: 'related or follow-on architectures'
    url: 'https://url'
    description: 'Description'

# use-case from 'code' column in
# https://github.ibm.com/digital/taxonomy/blob/main/topics/topics_flat_list.csv
use-case: HPConIBMCloud

# industry from 'code' column in
# https://github.ibm.com/digital/taxonomy/blob/main/industries/industries_flat_list.csv
industry: FinancialSector, Electronics, Healthcare

# compliance from 'code' column in
# https://github.ibm.com/digital/taxonomy/blob/main/compliance_entities/compliance_entities_flat_list.csv
compliance: ISOIEC27001

content-type: reference-architecture

# Set production to true to publish the reference architecture to IBM Cloud docs.

production: false
---

# Hybrid HPC with persistent cloud resource pools
{: #hybrid-hpc-pers-cloud-pool}
{: toc-content-type="reference-architecture"}
{: toc-industry="FinancialSector, Electronics, Healthcare"}
{: toc-use-case="HPConIBMCloud"}
{: toc-compliance="ISOIEC27001"}
{: toc-version="1.0"}

An organization with an existing HPC on-premises facility might decide to augment this with cloud-based resources.  This reference architecture summarizes the best practices for deploying a Hybrid HPC environment connecting an on-premises HPC environment to a persistent pool of HPC compute on IBM Cloud.

![Hybrid HPC high level architcture](./hybrid-hpc-cloud-pool-hla.drawio.svg "Hybrid HPC high level architcture"){: caption="Figure 1. High level Hybrid HPC architecture" caption-side="bottom"}

In the diagram above, an existing HPC environment on-premises is connected to an HPC environment in IBM Cloud.  Jobs can be submitted to and run on either compute environment.  There is a communications link between the on-premises datacenter and the IBM Cloud datacenter, for example using a Direct Link.  The Hybrid HPC environment is enabled by a "hybrid HPC capability" that hides the complexity of the environment from the user and automates decisions about which compute jobs are processed where through a set of rules and policies. Hybrid HPC environments where data needs to be transferred or kept synchronized between on-premises and the cloud will require a capability to enable this data movement.

## Architecture diagrams
{: #architecture-diagrams}

The architecture for the Hybrid HPC with persistent cloud resource pools has multiple constituent layers:

- Infrastructure layer
- Software layer
- Data layer

Each of these is described in more detail below.

### Infrastructure layer
{: #infrastructure-layer}

The architecture diagram for the Hybrid HPC with persistent cloud resource pools infrastructure is shown below:

![Infrastructure architecture diagram for Hybrid HPC with persistent cloud resource pools.](./hybrid-hpc-cloud-pool-infra.drawio.svg "Infrastructure architecture diagram for Hybrid HPC with persistent cloud resource pools"){: caption="Figure 2. Infrastructure architecture diagram for Hybrid HPC with persistent cloud resource pools" caption-side="bottom"}

From an infrastructure perspective, the HPC environment in IBM Cloud consists of one or more **HPC Cluster Management Nodes**. Multiple nodes are deployed to provide resilience of the environment.  These **HPC Cluster Management Nodes** distribute workload across a pool of **Compute Nodes**.  The number and type of **Compute Nodes** will depend on the characteristics of the workload(s) needing to be run within the environment.  Both **HPC Cluster Management Nodes** and **Compute Nodes** are deployed as Virtual Server Instances (VSIs) within a Virtual Private Cloud (VPC).

Many HPC applications process data.  A number of **Storage Nodes** are deployed to deliver this data to the **Compute Nodes**.  The **Storage Nodes** typically use attached **Block Storage** and present a **Shared Filesystem** within which the data is stored.  The **Compute Nodes** read and write data to the **Shared Filesystem**.  There is also a requirement for a **Shared Filesystem** to hold the metadata used by the **HPC Cluster Management Nodes**.  This is separate to the filesystem used to store the application data.

The HPC environment in the cloud will likely need its own **DNS** service to provide name resolution and **Virtual Private Endpoints** for secure access to other cloud services such as Monitoring, Logging, IAM etc.

Network connectivity between the enterprise datacenter and IBM Cloud is likely to use a **Direct Link** private network.

### Software layer
{: #software-layer}

The following is the architecture diagram for the software that is used to deliver the Hybrid HPC with persistent cloud resource pools pattern:

![Software architecture diagram for Hybrid HPC with persistent cloud resource pools.](./hybrid-hpc-cloud-pool-sw.drawio.svg "Software architecture diagram for Hybrid HPC with persistent cloud resource pools"){: caption="Figure 3. Software architecture diagram for Hybrid HPC with persistent cloud resource pools" caption-side="bottom"}

#### Execution flow
{: #execution-flow}

1. User invokes a compute job through an application, web browser or command line intergace.
2. The job request is sent to the **Multicluster Manager** which uses pre-defined rules and policies to determine whether the job should be processed using on-premises HPC resources or in the cloud.
3. The **Multicluster Manager** sends the compute job to the **Cluster Manager** for the respective cluster (on-premises or cloud). The job is then queued locally for dispatch to the compute node(s).
4. The **Cluster manager** sends the job to the **HPC Agent**.  One Agent runs on each of the compute nodes.  The Agent runs the relevant application executables to perform the computation.  As part of the computation, the application may access data stored in the shared filesystem.

When the computational job completes, notification is sent back by the reverse route.  The HPC Agent informs the local Cluster Manager.  This, in turn, informs the Multicluster Manager which returns the completion status to the user.

### Data layer
{: #data-layer}

For those HPC applications that require data, there needs to be a mechanism to deliver data from on-premises to the cloud environment.  There are two potential approaches that might be used to manage the movement of data:

- Data movement is managed by the HPC workload scheduler
- Data movement is managed by the filesystem(s) holding the data

The following is the architecture diagram for the data layer that is used to deliver the Hybrid HPC with persistent cloud resource pools pattern.  This shows both of the above approaches.

![Data architecture diagram for Hybrid HPC with persistent cloud resource pools.](./hybrid-hpc-cloud-pool-data.drawio.svg "Data architecture diagram for Hybrid HPC with persistent cloud resource pools"){: caption="Figure 4. Data architecture diagram for Hybrid HPC with persistent cloud resource pools" caption-side="bottom"}

#### Execution flow for workload scheduler-managed data movement
{: #execution-flow-for-workload-scheduler-managed-data-movement}

1. User invokes a compute job through an application, web browser or command line interface.
2. The job request is sent to the **Multicluster Manager** which uses pre-defined rules and policies to determine whether the job should be processed using on-premises HPC resources or in the cloud.
3. The **Multicluster Manager** communicates with the **Data Manager** components.  These are responsible for moving the data from the on-premises filesystem to the Cloud filesystem.  The data requirement information includes the in-cloud cluster that the job is eligible to be forwarded to.
4. The **Data Manager** in the cloud requests that the data file(s) get copied to the local **staging area** (cache) in the cloud. The data requirement information includes the candidate clusters that the job is eligible to be forwarded to.
5. The required files are copied from the on-premises filesystem(s) to the in-cloud **staging area**.
6. When the files are available in the cloud, the **Data Manager** informs the  **Cluster Manager** that the data is in place and that the compute jobs can be run.

#### Execution flow for filesystem-managed data movement
{: #execution-flow-for-filesystem-managed-data-movement}

IBM Storage Scale provides a high-performance parallel filesystem to meet the needs of HPC workloads.  Part of Storage Scale is a capability called Active File Management (AFM).  AFM provides on-demand movement of applications that is transparent to the application using the filesystem. (See A in the diagram above).

When a user attempts to access a file in the cloud that is physically located on-premises, the AFM capability seamlessly manages the transfer of the file to the in-cloud filesystem.  The in-cloud filesystem can be configured in different ways depending on whether it is to be used as a read-only "cache" or as a read-write filesystem.  Data changes made to the in-cloud copy of the file are seamlessly transferred back to the primary copy stored on-premises.

## Design concepts
{: #design-concepts}

Below is a map that covers design considerations and architecture decisions for the following aspects and domains:

- **Compute:** Virtual Servers
- **Storage:** Primary Storage
- **Networking:** Domain Name Services
- **Security:** Data Security, Identity & Access
- **Resiliency:** High Availability
- **Service Management:** Monitoring, Logging, Auditing / tracking, Management / orchestration

![Architecture design scope](./hybrid-hpc-heat-map.svg "Architecture design scope"){: caption="Figure 5. Architecture design scope" caption-side="bottom"}

### Design choices
{: #design-choices}

#### HPC cluster management software
{: #hpc-cluster-mgmt-software}

Different HPC problems require different HPC cluster management software solutions.  IBM Cloud provides two options.  **IBM Spectrum LSF** is a batch scheduler.  Users submit jobs onto a queue and these are processed in turn according to the policies and rules that have been defined.  **IBM Spectrum Symphony** is a "realtime scheduler" designed to deliver faster response times and aimed specifically at the needs of the Financial Services industry.  IBM Cloud provides tiles that can automatically deploy these software solutions into an HPC cluster.

Other HPC cluster scheduler solutions from the open source community such as SLURM and Condor, or from other commercial organisations are also available.  These will need to be manually deployed.

#### Storage options
{: #storage-options}

Most HPC environments consume data stored in filesystems.  IBM Cloud provides two shared filesystems.  **VPC File Storage** provides a lower performance filesystem that can be used to store the metadata required by the cluster management software or for low-use data storage for HPC workloads (e.g application binaries).  Workloads needing high performance parallel filesystems should use **IBM Storage Scale**.  This is best deployed on **VPC Bare metal servers**.

#### Compute nodes
{: #compute-nodes}

Computation is performed in **Virtual Servers (VSIs)**.  There are many different VSI profiles that can be chosen to best meet the compute and memory requirements of the application(s) being run within the HPC environment. Consider an application that requires 3 vCPUs of compute and 7GB of memory.  The needs of this application might be met by the cx2-4x8 VSI profile which provides 4 vCPUs and 8GB of memory.  If this profile is chosen, one instance of the application will run on one compute node.  

It is recommended that in environments where multiple applications with different resource needs will run simultaneously, that the compute nodes be sized to support the largest application footprint.  The IBM workload scheduling software will run multiple instances of smaller workload on these larger VSIs to make optimal use of the compute resources available.

## Requirements
{: #requirements}

The following table outlines the requirements that are addressed in this architecture.

| Aspect | Requirements |
| -------------- | -------------- |
| Compute            | Provide properly isolated compute resources with adequate compute capacity for the applications.  Remember to allow for the resource needs of the operating system and any other software needed. |
| Storage            | Provide storage that meets the application data volume and performance requirements. |
| Networking         | Deploy workloads in isolated environment and enforce information flow policies.  \n Provide secure, encrypted connectivity to the cloud’s private network for management purposes.  \n Distribute incoming application requests across available compute resources.  \n Support failover of application to alternate site in the event of planned or unplanned outages.  \n Provide public and private DNS resolution to support use of hostnames instead of IP addresses. |
| Security           | Protect the boundaries of the application against denial-of-service and application-layer attacks.  \n If required, encrypt all application data in transit and at rest to protect from unauthorized disclosure.  \n Encrypt all security data (operational and audit logs) to protect from unauthorized disclosure. |
| Resiliency         | Support application availability targets.  \n Ensure availability of the application in the event of planned and unplanned outages.  \n Provide highly available compute, storage, network, and other cloud services to handle application load and performance requirements.  \n Provide highly available storage for security data (logs) and backup data.  \n Automate recovery tasks to minimize down time |
| Service Management | Monitor system and application health metrics and logs to detect issues that might impact the availability of the application.  \n Generate alerts/notifications about issues that might impact the availability of applications to trigger appropriate responses to minimize down time.  \n Monitor audit logs to track changes and detect potential security problems.  \n Provide a mechanism to identify and send notifications about issues found in audit logs. |
{: caption="Table 1. Requirements" caption-side="bottom"}

## Components
{: #components}

The following table outlines the products or services used in the architecture for each aspect:

| Aspects | Architecture component | How the component is used |
| -------------- | -------------- | -------------- |
| Compute | [Virtual Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-advanced-virtual-servers&interface=ui) | HPC cluster management nodes and compute nodes |
| Storage | [VPC File Storage](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about) | Low performance storage for HPC management metadata and/or lightweight application data access needs |
|  | [Storage Scale](https://cloud.ibm.com/docs/storage-scale?topic=storage-scale-about-storage-scale) | High performance parallel filesystem for data-intensive HPC workloads |
| Networking | [Virtual Private Endpoint (VPE)](https://cloud.ibm.com/docs/vpc?topic=vpc-about-vpe) | For private network access to Cloud Services, e.g., Key Protect, IAM, etc. |
|  | [Public Gateway](https://cloud.ibm.com/docs/vpc?topic=vpc-about-public-gateways) | For secure client access to the HPC environment over the Internet |
|  | [Direct Link](https://cloud.ibm.com/docs/dl?topic=dl-dl-about) | For private, dedicated connectivity between on-premises and cloud HPC resources |
|  | [DNS](https://cloud.ibm.com/docs/dns-svcs?topic=dns-svcs-about-dns-services) | Domain Name Services for the HPC environment |
| Security | [Identity & Access Management](https://cloud.ibm.com/iam/overview) | Identity & Access Management |
|  | [Key protect](https://cloud.ibm.com/docs/key-protect?topic=key-protect-about) or [Hyper Protect Crypto Services](https://cloud.ibm.com/docs/hs-crypto?topic=hs-crypto-get-started) | Hardware security module (HSM) and Key Management Service |
|  | [Secrets Manager](https://cloud.ibm.com/docs/secrets-manager?topic=secrets-manager-getting-started) | Certificate and Secrets Management |
| Resiliency | [Virtual Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-advanced-virtual-servers&interface=ui) in conjunction with [Spectrum LSF](https://cloud.ibm.com/docs/ibm-spectrum-lsf?topic=ibm-spectrum-lsf-about-spectrum-lsf) or [Spectrum Symphony](https://cloud.ibm.com/docs/hpc-spectrum-symphony?topic=hpc-spectrum-symphony-about-spectrum-symphony) | The HPC workload is split across multiple VSIs.  The HPC Management software (LSF or Symphony) manages failures of compute nodes by resubmitting failed compute jobs to other VSIs |
| Service Management | [Spectrum LSF](https://cloud.ibm.com/docs/ibm-spectrum-lsf?topic=ibm-spectrum-lsf-about-spectrum-lsf) or [Spectrum Symphony](https://cloud.ibm.com/docs/hpc-spectrum-symphony?topic=hpc-spectrum-symphony-about-spectrum-symphony) | HPC cluster management software provides application and performance status & monitoring, resource consumption and utilisation |
|  | [IBM Cloud Monitoring](https://cloud.ibm.com/docs/monitoring?topic=monitoring-about-monitor) | Operational monitoring |
|  | [IBM Cloud Log Analysis](https://cloud.ibm.com/docs/log-analysis?topic=log-analysis-getting-started) | Operational logs |
|  | [Activity Tracker Event Routing](https://cloud.ibm.com/docs/activity-tracker?topic=activity-tracker-getting-started) | Audit logs |
{: caption="Table 2. Components" caption-side="bottom"}

