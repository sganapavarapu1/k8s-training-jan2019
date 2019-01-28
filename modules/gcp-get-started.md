# Getting Started with GCP

## Module Objectives

Before getting started, you first have to prepare the environment for the workshop.

1. Get a GCP account from the instructor
1. Connect to the Cloud Shell using the GCP account
1. Enable the necessary APIs
1. Set computing zone and project
1. Download the lab source code from GitHub

---

## Google Cloud Platform Overview

- Managed by Google
- Provides basic resources like compute, storage and network
- Also provides services like Cloud SQL and Kubernetes engine
- All operations can be done through the API
- SLAs define reliability guarantees for the APIs
- Three ways of access
  - API calls
  - SDK commands
  - Cloud Console web UI

Google Cloud Computing service groups:

- Compute
- Storage
- Migration
- Networking
- Databases
- Developer Tools
- Management Tools

You will use these services while doing the lab:

- IAM & Admin: Manage users and permissions
- Compute Engine: Run virtual machines for worker nodes
- VPC Network: Connectivity between the nodes
- Load Balancing: Create Ingress of LoadBalancer type

Cloud Console is the admin user interface for Google Cloud. With Cloud Console you can find and manage your resources through a secure administrative interface.

Cloud Console features:

- Resource Management
- Billing
- SSH in Browser
- Activity Stream
- Cloud Shell

Projects

- Managing APIs
- Enabling billing
- Adding and removing collaborators
- Managing permissions for GCP resources

Zonal, Regional, and Global Resources

- Zone: Instances and persistent disks
- Region: Subnets and addresses
- Global: VPC Network and firewall

---

## Google Cloud Platform (GCP) Account

In this workshop you will run Kubernetes in GCP. We have created a separate project for each student. You should receive an email with the credentials to log in.

We recommend using Google's Chrome browser during the workshop.

1. Go to https://console.cloud.google.com/
1. Enter the username
1. Enter the user password

    > Note: Sometimes GCP asks for a verification code when it detects logins from unusual locations. It is a security measure to keep the account protected. If this happens, please ask the instructor for the verification code.

1. In the top left corner select the project "XXXXXXXXXXXXX-yyyyyy", where XXXXXXXXXXXXX matches the name of the e-mail you were given

## Cloud Shell

Console is the UI tool for managing cloud resources. Most of the exercises in this course are done from the command line, so you will need a terminal and an editor.

Click "Activate Cloud Shell" button in the top right corner.

![](img/cloud-shell.png)

Now click the "Start Cloud Shell" button in the lower right of the dialog.

This will start a virtual machine in the cloud and give you access to a terminal and an editor.

## Set Computing Zone and Region

When the shell is open, set your default compute zone and region:

```shell
echo '
## GCP K8S Deep Dive
export PROJECT_ID=$(gcloud config get-value project)
export COMPUTE_REGION=us-west1
gcloud config set compute/region $COMPUTE_REGION
export COMPUTE_ZONE=us-west1-c
gcloud config set compute/zone $COMPUTE_ZONE
' >> ~/.bashrc
```

Lets also add completion for kubectl that we will want later:

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
bash -l    
```

Every time you open a new terminal you will need to input these commands. To avoid this, we place the above commands inside `~/.bashrc` file and they will be executed automatically each time you log in.

> Note: Changing the zone will not change the region automatically.

You can check for additional information with:

```shell
gcloud info
```

## Enable APIs

As a project owner, you control which APIs are accessible for the project. Enable the APIs which are required for the workshop:

View initial setup:

```shell
gcloud services list --enabled
```

Enable services we will use:
```shell
gcloud services enable \
  compute.googleapis.com \
  monitoring.googleapis.com \
  logging.googleapis.com \
  stackdriver.googleapis.com \
  storage-api.googleapis.com
```

The operation will return once complete. Enabling all these apis will take about 5m.

```shell
gcloud services list --enabled
```

```
NAME                        TITLE
compute.googleapis.com      Compute Engine API
logging.googleapis.com      Stackdriver Logging API
monitoring.googleapis.com   Stackdriver Monitoring API
oslogin.googleapis.com      Cloud OS Login API
stackdriver.googleapis.com  Stackdriver API
```

> Note: Notice how some APIs are enabled as dependencies, such as Cloud OS Login.


## Download the Lab Source Code from GitHub

Clone the lab repository in your cloud shell, then `cd` into that directory:

```shell
git clone https://github.com/Altoros/k8s-training-jan2019.git
```
```shell
cd k8s-training-jan2019
```

---

Next: [Containers](containers.md)
