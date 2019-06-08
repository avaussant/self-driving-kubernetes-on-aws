# Self-Driving Infrastructure using Kubernetes on AWS with EKS

This workshop will provide hands on experience on setting up and running an AWS Kubernetes cluster using EKS. We will use gitops, and explore kubernetes tools to make the cluster self-driving, with automated management and remedy of common cluster level problems. To achieve this, we will use eksctl, cluster-autoscaler, kube-prometheus (prometheus operator), node-problem-detector, draino, and node-local-dns-cache.


## Exercise 1: Write your own microservice in one line that provides:
- [ ] High Availability with Health Checks and Automated Recovery
- [ ] Scalability: Automatic horizontal and vertical scaling
- [ ] Security + Policy
- [ ] Service discovery
- [ ] Observabilitiy: Metrics, Tracing, Logs, Alerts
- [ ] Traffic Management 
- [ ] Retries
- [ ] Timeouts
- [ ] Load balancing
- [ ] Rate limiting
- [ ] Bulkheading
- [ ] Circuit breaking
- [ ] Actually useful business purpose �those things were for!

*Hint:* Kubernetes and Istio (or Linkerd) Service Mesh

You either need a rich execution environment allow your microservice to laser focus on business value, or you need to add so much overhead to your microservice that it winds up not very micro.  

If you can build a straightforward monolithic app and never think about all this asynchronous stuff, go for it! If your system is big enough that you need to refactor into microservices for sanity’s sake, or you need to scale components independently to manage load, or you need to make temporary outages survivable, then microservices with a rich execution environment are a great way to go. 

## Ok, Fine. Microservices. But Kubernetes on AWS?

Many of the most powerful architectures involve combining diverse workloads like a stateless web application with a persistent, stateful database, along with periodically running to completion a finite task. Even if these pieces are packed in containers, they still need to be coordinated. We need to deploy, manage, and scale the disparate pieces in different ways.  We also wish to span some pieces over many servers while looking like one single unit to other pieces. On top of this, managing persistent storage is a distinct problem from managing other computational resources.

There are many disparate technical solutions for managing each of the concerns of applications, computational resources, and storage resources. Kubernetes provides a single, common solution to these common problems.

As Paul Ingles said, one of Kubernetes’ greatest strengths is providing a ubiquitous language that connects applications teams and infrastructure teams. And, because it’s extensible, this can grow beyond the core concepts to more domain and business specific concepts.

We also found Kubernetes attractive because it allowed us to iterate quickly for a proof of concept, while giving us built-in resilience and an easy path to scale it in production. 

Kubernetes extensible nature, first class support for healthchecks, detailed metrics, and automated recovery for applications make it very simple to automate maintenance and recovery of the Kubernetes platform and it's components.

This workshop will provide hands on experience on running an AWS Kubernetes cluster using EKS. We will use gitops, and explore kubernetes tools to make the cluster self-driving, with automated management and remedy of cluster level problems. To achieve this, we will use eksctl, cluster-autoscaler, external-dns, kube-prometheus (prometheus operator), node-problem-detector, draino, node-local-dns-cache, and dashboard.

While we won't focus on it, we will provide a continuous delivery pipeline on top of kubernetes.

Also [Why do I need kubernetes and what can it do](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-do-i-need-kubernetes-and-what-can-it-do) and ([see why containers](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-containers))


## Intended audience
This workshop is intended to appeal primarily to four types of people:

1. Application developers looking to get an AWS kubernetes cluster to experiment without a lot of infrastructure knowledge
2. AWS DevOps people without a lot of kubernetes experience
3. Kubernetes DevOps people without a lot of AWS experience
4. Full-stack, [Full-cycle](https://medium.com/netflix-techblog/full-cycle-developers-at-netflix-a08c31f83249) developers in small or large teams. 

## Prerequisites
1. [An AWS Account ($25 credit code will be given on day of the workshop)](#create-aws-account)
2. [aws cli installed and configured to access account](#install-aws-cli)
3. [kubectl installed](#install-kubectl)
4. [aws-iam-authenticator installed](#install-aws-iam-authenticator)
5. [eksctl installed](#install-eksctl)
6. [docker installed](#install-docker)

### <a name="create-aws-account">Create AWS Account</a>
This workshop expects you to [create your own AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), but participants will be given a $25 cost code to cover any costs incurred during the workshop. A pre-existing VPC is not required. Participants should create their accounts ASAP.  A small percentage of account may be pulled into a manual verification workflow.  Also if any users have accounts that have been deactivated for non-payment it will take some time for them to reactive once a credit card is added.

<table><tr><td>:bulb: Tip: Your account must have the ability to create new IAM roles and scope other IAM permissions.</td></tr></table>

1. If you don't already have an AWS account with Administrator access: [create
one now by clicking here](https://portal.aws.amazon.com/billing/signup#/start)

2. [Create a billing alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) - Super important!

### <a name="install-aws-cli">Install the AWS CLI</a>
<table><tr><td>:bulb: Tip: After you have the AWS CLI installed (as below), You will need to have AWS API credentials configured. You can use <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html"><code>~/.aws/credentials</code> file</a>
or <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html">environment variables</a>. For more information read <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html">AWS documentation</a>. </td></tr></table>

MacOS users can use [Homebrew](https://brew.sh):
```
brew install awscli
```

and Windows users can use [chocolatey](https://chocolatey.org):
```
chocolatey install awscli
```

If you already have pip and a supported version of Python, and [ideally you know how to set up a virtual environment](https://docs.aws.amazon.com/cli/latest/userguide/install-virtualenv.html). You can install the AWS CLI by using the following command. If you have Python version 3+ installed, we recommend that you use the pip3 command. 
```
pip3 install awscli --upgrade --user
```

Although it might provide an outdated version, Linux users can also use the default package managers for installing AWS CLI, e.g.:
```
$ sudo apt-get update awscli                       
$ sudo yum install awscli
```

### <a name="install-kubectl">Install kubectl</a>
Linux or Mac:
```
sudo curl --silent --location -o /usr/local/bin/kubectl curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl
```
Or Download the [Windows executable](https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/windows/amd64/kubectl.exe)

### <a name="install-aws-iam-authenticator">Install AWS IAM Authenticator</a>
If you have golang installed and your `$PATH` includes `$GOPATH/bin`:
```
go get -u -v github.com/kubernetes-sigs/aws-iam-authenticator/cmd/aws-iam-authenticator
```
Otherwise, download the Amazon EKS-vended aws-iam-authenticator binary from Github Releases:
+ [Linux](https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.4.0/aws-iam-authenticator_0.4.0_linux_amd64)

+ [MacOS](https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.4.0/aws-iam-authenticator_0.4.0_darwin_amd64)

+ [Windows](https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.4.0/aws-iam-authenticator_0.4.0_windows_amd64.exe)

### <a name="install-eksctl">Install eksctl</a>
To download the latest release, run:

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

Alternatively, macOS users can use [Homebrew](https://brew.sh):
```
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
```

and Windows users can use [chocolatey](https://chocolatey.org):
```
chocolatey install eksctl
```
### <a name="install-docker">Install Docker</a>
#### MacOS:
> :bulb: Tip: Avoid _Docker Toolbox_ and _boot2docker_. These are older packages that have been ceded by _Docker for Mac_.
```
brew cask install docker       # Install Docker
open /Applications/Docker.app  # Start Docker
```
#### Ubuntu

`docker.io` is available from the Ubuntu repositories (as of Xenial).

```bash
# Install Docker
sudo apt install docker.io
sudo apt install docker-compose

# Start it
sudo systemctl start docker
```

<table><tr><td> :bulb: Tip: If the `docker.io` package isn't available for you, see [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for an alternative.</td></tr></table>

#### Windows

Install [Windows Subsystem for Linux][wsl] and choose _Ubuntu_ as your guest OS. Install Docker as you normally would on Ubuntu (see above). After that, [see these instructions](https://github.com/Microsoft/WSL/issues/2291#issuecomment-383698720) for info on how to get it running.

<table><tr><td> :bulb: Tip: Avoid _Docker for Windows_. While it works in most cases, you'll still face NTFS limitations without WSL (eg, lack of symlinks, which is needed for Yarn/npm to work). </td></tr></table>

[wsl]: https://docs.microsoft.com/en-us/windows/wsl/install-win10

#### Other OS's

For other operating systems, see: <https://www.docker.com/community-edition#download>


### Verify the binaries are in the path and executable
```
for command in docker kubectl aws-iam-authenticator eksctl
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done
```

## How to make Kubernetes Self-Driving

Amazon EKS works by provisioning (starting) and managing the Kubernetes control plane for you. At a high level, Kubernetes consists of two major components – a cluster of 'worker nodes' that run your containers and the 'control plane' that manages when and where containers are started on your cluster and monitors their status.

Without Amazon EKS, you have to run both the Kubernetes control plane and the cluster of worker nodes yourself. With Amazon EKS, you provision your cluster of worker nodes and AWS handles provisioning, scaling, and managing the Kubernetes control plane in a highly available and secure configuration. This removes a significant operational burden for running Kubernetes and allows you to focus on building your application instead of managing AWS infrastructure.

At present the experience of creating a VPC, EKS cluster, and worker nodes using the web console or using the provided Amazon Machine Image (AMI) and AWS CloudFormation scripts leaves you with something that can be difficult to see how to operationalize or provides a frictionless developer experience.

Fortunately, both the Kubernetes Ecosystem and that of AWS are vast and dynamic, and tools have filled this gap.

### `eksctl` - a CLI for Amazon EKS

While it started as a simple CLI for EKS, it's clear ambition is to serve both developer use-cases and operational best practices like GitOps. At present, it already does a pretty good job of both.

GitOps takes full advantage of the move towards immutable infrastructure and declarative container orchestration. In order to minimize the risk of change after a deployment, whether intended or by accident via "configuration drift" it is essential that we maintain a reproducible and reliable deployment process.   

Our whole system’s desired state (aka "the source of truth") is described in Git. We use containers for immutability as well as different cloud native tools like Cloudformation and Terraform to automate and manage our configuration. These tools together with containers and declarative nature of Kubernetes provide what we need for a complete recovery in the case of an entire meltdown.

Meanwhile, Developers want a quick and easy way to spin up a flexible, friendly, and frictionless continuous delivery pipeline so they can focus on delivering business value (or just doing cool stuff) without getting bogged down in yak-shaving (i.e. details, details).

The `eksctl` tool lets us spin up and manage a fully operational cluster with sensible defaults and a broad array of addons and configuration settings. You can choose to pass it flags on the cli (dev mode), or specify detailed configuration via yaml, and checked into git (ops mode). As it improves, it continues to better meet the use cases of both audiences.

### horizontal pod autoscaling with kube-prometheus (prometheus operator)

The kube-prometheus stack is meant for cluster monitoring, so it is pre-configured to collect metrics from all Kubernetes components. The kube-prometheus stack includes a resource metrics API server for horizontal pod autoscaling, like the metrics-server does. In addition to that it delivers a default set of dashboards and alerting rules. 

It provides Kubernetes manifests, Grafana dashboards, and Prometheus rules combined with documentation and scripts to provide easy to operate end-to-end Kubernetes cluster monitoring with Prometheus using the Prometheus Operator. 

It includes:

* The [Prometheus Operator](https://github.com/coreos/prometheus-operator)
* Highly available [Prometheus](https://prometheus.io/)
* Highly available [Alertmanager](https://github.com/prometheus/alertmanager)
* [Prometheus node-exporter](https://github.com/prometheus/node_exporter)
* [Prometheus Adapter for Kubernetes Metrics APIs](https://github.com/DirectXMan12/k8s-prometheus-adapter)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [Grafana](https://grafana.com/)


### node-problem-detector and draino with the cluster autoscaler 

[Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/) is a standalone program that adjusts the size of a Kubernetes cluster to meet the current needs.

A node is a worker machine in Kubernetes. There are tons of node problems could possibly affect the pods running on the
node such as:
* Infrastructure daemon issues: ntp service down;
* Hardware issues: Bad cpu, memory or disk, ntp service down;
* Kernel issues: Kernel deadlock, corrupted file system;
* Container runtime issues: Unresponsive runtime daemon;

If problems are invisible to the upstream layers in cluster management
stack, Kubernetes will continue scheduling pods to the bad nodes. The daemonset **[Node Problem Detector](https://github.com/kubernetes/node-problem-detector)** 
collects node problems from various daemons and make them visible to the upstream
layers. It is running as a Kubernetes Addon enabled by default in GCE clusters, but we need to manually install it in EKS.

[Draino](https://github.com/planetlabs/draino/) is intended for use alongside the Kubernetes [Node Problem Detector](https://github.com/kubernetes/node-problem-detector) and [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/). The Node Problem Detector can set a node condition when it detects something wrong with a node - for instance by watching node logs or running a script. The Cluster Autoscaler can be configured to delete nodes that are underutilised. Adding Draino to the mix enables autoremediation:

+ The Node Problem Detector detects a permanent node problem and sets the corresponding node condition.
+ Draino notices the node condition. It immediately cordons the node to prevent new pods being scheduled there, and schedules a drain of the node.
+ Once the node has been drained the Cluster Autoscaler will consider it underutilised. It will be eligible for scale down (i.e. termination) by the Autoscaler after a configurable period of time.


### What is an operator?

From the [operator documentation](https://coreos.com/operators/):
> An Operator is an application-specific controller that extends the Kubernetes API to create, configure and manage instances of complex stateful applications on behalf of a Kubernetes user. It builds upon the basic Kubernetes resource and controller concepts, but also includes domain or application-specific knowledge to automate common tasks better managed by computers.

From Kubernetes official documentation, [Kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager/)
> In applications of robotics and automation, a control loop is a non-terminating loop that regulates the state of the system. In Kubernetes, a controller is a control loop that watches the shared state of the cluster through the API server and makes changes attempting to move the current state towards the desired state. Examples of controllers that ship with Kubernetes today are the replication controller, endpoints controller, namespace controller, and serviceaccounts controller.

An operator is a combination of custom resource types and the controllers that take care of the reconciliation process.