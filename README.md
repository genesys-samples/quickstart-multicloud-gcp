# Genesys Multicloud CX Private Edition - Quickstart on GCP

The purpose of this repository is to provide sample platform reference architecture for deploying Genesys Multicloud CX Private Edition containers and Helm charts to Google Kubernetes Engine (GKE).  The content provided in this repository SHOULD NOT be used directly for Production environments as it is not designed to meet typical HA, DR, service sizing, multi-region or Security requirements.  All content is being provided AS-IS without any SLA, warranty or coverage via Genesys product support.

This project is derived from the following two repositories maintained by Genesys Product:
* [MultiCloud Platform](https://github.com/genesys/multicloud-platform)
* [MultiCloud Services](https://github.com/genesys/multicloud-services)

## Repository Structure

This repository is broken up into two sections:
* Platform - Terraform files to provision a new GCP project with the necessary Google API's and services. 
* Services - Helm Charts to provision the MultiCloud Private Edition containers. 

<pre>
multicloud-quickstart
|
├── platform - Terraform scripts that provision GCP with the necessary services, API enablement, etc
│   ├── terraform
|   |   ├── 1-prereqs   -   Terraform files that need to be manually invoked before Cloud Build Ci/CD can take over.
│   │   |   ├── 1-gcp - Sets up the GCP project with the necessary API's and creates a storage bucket for future Terraform state files
|   |   |   |   ├── main.tf
│   │   |   ├── 2-cloudbuild - Sets up the Cloud Build trigger jobs that will be invoked to provision the rest of GCP via Terraform and Private Edition with Helm
|   |   |   |   ├── main.tf
│   │   |   ├── 3-pe-secrets - Sets up the the Google Secrets with chosen passwords for the various MultiCloud services and accounts
|   |   |   |   ├── main.tf
|   |   ├── 2-gcp    -   Terraform files that will provision GCP services such as VPC, GKE, etc. All of these subfolders are executing by a CloudBuild trigger job
│   │   |   ├── 1-network - Provisions the GCP networking components needed such as VPC, DNS, Routers, NATS, etc
|   |   |   |   ├── main.tf
│   │   |   ├── 2-gke-cluster - Provisions the GKE cluster that MultiCloud will use
|   |   |   |   ├── main.tf
│   │   |   ├── 3-k8s-setup - Provisions the Filestore StorageClass for K8's on the newly provisioned GKE cluster
|   |   |   |   ├── main.tf
|   |   ├── 3-gcp-posttasks    -   Terraform files that perform post tasks such as configuring consul, MSSQL, etc
│   │   |   ├── 1-certs - Sets up the Ngnix Ingress for outside connections and Cert Manager
|   |   |   |   ├── main.tf
│   │   |   ├── 2-thirdparty - Sets up thirdparty components such as Consul, Kafka, MSSQL, etc
|   |   |   |   ├── main.tf
│   │   |   ├── 3-consul-mssql - Configures the thirdparty components such as a MSSQL database and related account. 
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
|   |   ├── 4-artifactory-optional  -   Terraform job that will copy the MultiCloud Helm Charts and Containers out of the Genesys JFROG Artifactory into Google Artifacts.
                                        If you already have an established repo to store Helm Charts and Containers, then this job is not needed and you will need to perform your own copy job
│   |   |   ├── main.tf
│   |   |   ├── provision-gcp-repo.sh
│   ├── tfm - The Terraform submodules containing the resource and variable definitions
|   |   ├── 1-prereqs 
│   │   |   ├── 1-gcp - Sets up the GCP project with the necessary API's and creates a storage bucket for future Terraform state files
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
│   │   |   ├── 2-cloudbuild - Sets up the Cloud Build trigger jobs that will be invoked to provision the rest of GCP via Terraform and Private Edition with Helm
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
│   │   |   ├── 3-pe-secrets - Sets up the the Google Secrets with chosen passwords for the various MultiCloud services and accounts
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
|   |   ├── 2-gcp    -   Terraform files that will provision GCP services such as VPC, GKE, etc. All of these subfolders are executing by a CloudBuild trigger job
│   │   |   ├── 1-network - Provisions the GCP networking components needed such as VPC, DNS, Routers, NATS, etc
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
│   │   |   ├── 2-gke-cluster - Provisions the GKE cluster that MultiCloud will use
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
│   │   |   ├── 3-k8s-setup - Provisions the Filestore StorageClass for K8's on the newly provisioned GKE cluster
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
|   |   ├── 3-gcp-posttasks    -   Terraform files that perform post tasks such as configuring consul, MSSQL, etc
│   │   |   ├── 1-certs - Sets up the Ngnix Ingress for outside connections and Cert Manager
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
│   │   |   ├── 2-thirdparty - Sets up thirdparty components such as Consul, Kafka, etc
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
│   |   |   |   └── README.md  -  Details on all inputs that must be modified
│   │   |   ├── 3-consul-mssql 
│   |   |   |   ├── main.tf
│   |   |   |   ├── variables.tf
|   |   ├── 4-artifactory-optional
│   |   |   ├── main.tf
│   |   |   ├── variables.tf
├── services - Helm charts that provisions the MultiCloud Private Edition stack
│   ├── [multicloud_service_helm_chart]  -  like "gauth", "iwd", ...
│   |   ├── XX_chart_product-subsystem
|   |   |   ├── XX_release_product
|   |   |   |   |── override_values.yaml
|   |   |   |   |── pre-release-script.yaml
|   |   |   |   |── post-release-script.yaml
|   |   |   |── chart.ver
|   |   |   |── override_values.yaml
|   └── provision-[product].sh - Shell script responsible for executing the Helm Charts for the specific product
└── cloudbuild-services-[product].yaml - Cloud Build YAML definition file.
</pre>

## Getting Started
The scripts provided in this repository assumes a newly created and dedicated GCP project for MultiCloud Private Edition is created. The Terraform files will provision the newly created Google Project with the necessary Google API's and services (GKE, etc) and prepare the project and GKE cluster for MultiCloud Private Edition. Once the GKE cluster is stood up, then the provided Helm Charts will execute one by one to stand up the MultiCloud Private Edition services (GAuth, Voice, WWE, IWD, etc).

Cloud Build is a serverless CI/CD platform provided by GCP and has been chosen to execute the provided Terraform files and Helm Charts. In one of the first steps, a Terraform job will create and stage the Cloud Build triggers and once these are created, then the rest of the platform can be setup in an automated fasion.

To keep this initial ReadMe short and not cluttered, a dedicated "Manual" folder has been created and is broken down into a few ReadMe's that will guide you through provisioning GCP and the MultiCloud Private Edition stack. 

## License

All content in this repository is released under the [MIT License](LICENSE)
