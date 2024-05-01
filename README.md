# Creating-a-Talos-Cluster-with-Equinix-Metal

**What is Talos?**

Talos is a frame designed for Kubernetes. The expectation is that this will be as minimal as possible while preserving practical functionality.

**Here are a few key highlights and viewpoints of Talos:**

**Kubernetes-Native**: Talos is Kubernetes-native, meaning it integrates well with Kubernetes concepts. It removes standard Linux components and offers the Kubernetes experience.

**Immutable Infrastructure:** Talos obeys primary permanent command. Once deployed, the operating system does not change immediately, but it may or may not be installed with the latest version. This increases security and predictability

**Minimalist Plan:** Talos is expected to be more rational, reducing affected area and resource usage. It literally contains the basic components needed to run Kubernetes, in a simple and efficient system

**Security**: Security is a center center of Talos. It uses key features like read-only root, boot protection, and software upgrades to increase security. Again, acquiring a Kubernetes cluster takes a lot of time

**Automatic Updates:** Talos uses automatic updates for Kubernetes tasks and components. This ensures that the cluster is constantly updated with the latest vulnerabilities.
Ease to Manage: Talos provides flight control and control throughout the flight. It includes tools like talosctl for hub control, rendering, and other cluster components.

Let's take look of quickstart before we create a Talos cluster with Equinix Metal.

**Quickstart**:

A short guide on setting up a simple Talos Linux cluster locally with Docker.

**Local Docker Cluster:**

Docker should be installed on the machine for Talos cluster setup, because Talos will run as a docker container.

The simplest method to test Talos is to set up a cluster on a computer running Docker using the CLI (talosctl).

**Prerequisites**

**talosctl**

Download talosctl (macOS or Linux):

```
curl -sL https://talos.dev/install | sh

'''

**kubectl**

Download kubectl and install via one of methods in the documentation.

**Create the Cluster**

Run the below command for cluster creation

```
talosctl cluster create

```

Note: If you are using Docker Desktop on a macOS computer you will need to enable the default Docker socket in your settings.

You can investigate using below Talos API command:

```
talosctl dashboard --nodes 10.5.0.2
```

Validate that you can reach Kubernetes cluster:

```
kubectl get nodes -o wide

NAME                           STATUS   ROLES    AGE    VERSION          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                 KERNEL-VERSION   CONTAINER-RUNTIME

talos-default-controlplane-1   Ready    master   115s   v1.30.0   10.5.0.2      <none>        Talos (v1.7.0)   <host kernel>    containerd://1.5.5

talos-default-worker-1         Ready    <none>   115s   v1.30.0   10.5.0.3      <none>        Talos (v1.7.0)   <host kernel>    containerd://1.5.5

```
**Destroy the Cluster**

When you are all done, remove the cluster:

```
talosctl cluster destroy

```

That's it for the quick and simplest Kubernetes cluster setup for testing purpose.

Let's move on to create a dedicated Talos cluster on Equinix Metal.

**Creating Talos clusters with Equinix Metal:**

We can create the Talos Linux cluster on Equinix Metal in various ways, for example through the EM web interface  or the metal networking tool.

**Below are the process:**

1. Create DNS entries on your Kubernetes endpoint.
2. Create configuration using talosctl.
3. Provision your machines on Equinix Metal.
4. Upload the program to your server (if it is not running as part of the distribution machine).
5. Configure your Kubernetes endpoint to point to the recently made control plane nodes.
6. Bootstrap the cluster.
   
**Define the Kubernetes Endpoint:**

There are several ways to create an HA endpoint for a Kubernetes cluster. Some ways are:

* DNS
* Load Balancer
* BGP

Whichever method is chosen must result in an IP address/DNS mapping session for all control plane operations.

We do not know the control plane IP address of this program, but we need to specify the DNS endpoints that will be used to create the cluster. Once the hubs are assigned, endpoint A can be used to create scripts or send them to the load balancer, etc.

**Create the Machine Setup Files**
**Generating Configurations**

Create a basic script to identify Talos machines using the DNS header of the load balancer shown above:

```
$ talosctl gen config talos-k8s-em-tutorial https://<load balancer IP or DNS>:<port>

created controlplane.yaml

created worker.yaml

created talosconfig
```

The port used above should be 6443 unless your load balancing card is different from port 6443 on the control plane.

**Validate the Arrangement Files**

```
talosctl validate --config controlplane.yaml --mode metal

talosctl validate --config worker.yaml --mode metal
```

**Provision the machines in Equinix Metal**

Talos Linux can install PXE on an Equinix network using Image Factory using the EquinixMetal class: e.g. https://pxe.factory.talos.dev/pxe/376567988ad370138ad8b2698212367b8edcb69b5fd68c80be1f2ec7d603b4ba/v1.7.0/equinixMetal-amd64 (this URL references the default schematic and amd64 architecture).

**Using the Equinix Metal UI**

Simply select the region and machine type from the Equinix Metal web interface. Select 'Custom iPXE' as the task and enter the PXE image URL as the IPXE URL. Then select the number of servers to run and give them a name (in lowercase letters). You can also connect the control plane under normal conditions. use the yaml file created above (make sure to add one at the beginning of the #!talos line).

You can update this by preparing to run different types of flight control machines and custom hubs (even if you pass worker.yaml to the worker node like client data).

If you have not set the machine as client data, you must pass it to each machine with the following command:

```
talosctl apply-config --insecure --nodes <Node IP> --file ./controlplane.yaml
```

**Update the Kubernetes endpoint**

Now that our control planes are created and we know their IP addresses, we can connect them to the Kubernetes endpoint. Configure your load balancer to perform operations on these hubs or finally add records to your DNS servers for each control plane.
host endpoint.mydomain.com

```
endpoint.mydomain.com has address 145.40.90.201

endpoint.mydomain.com has address 147.75.109.71

endpoint.mydomain.com has address 145.40.90.177
```

**Bootstrap Etcd**

Set the endpoints and nodes for talosctl:
```
talosctl --talosconfig talosconfig config endpoint <control plane 1 IP>

talosctl --talosconfig talosconfig config node <control plane 1 IP>
```

**Bootstrap etcd:**
```
talosctl --talosconfig talosconfig bootstrap
```
