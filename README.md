# K8S Cluster Automation 

## Vagrant Setup

To create multiple vms locally

1. Update the public key in directory `/ubuntu/vagrant` with `my_key.pub`

    This key will be copied into Ubuntu user's authorized_keys file for password-less authentication

2. Install Vagrant on the local machine

3. Make sure you have the oracle virtual box setup and working fine

4. Steps to create VMs:

    Step 1: Update the VM Configuration details in Vagrantfile

        NUM_MASTER_NODE = 1 (Number of Master nodes)
        NUM_WORKER_NODE = 2 (Number of Worker nodes)

    Step 2: Update the IP details for the nodes

        IP_NW = "192.168.56." (IPv4 Network CIDR needed for VMs)
        MASTER_IP_START = 110 (This is the Start IP of the Master Nodes)
        NODE_IP_START = 200 (This is the Start IP of the Worker Nodes)

    Step 3: `vagrant up` this triggers the script and provisions the VMs as mentioned in Vagrantfile

    Step 4: `vagrant status` - Make sure that all the VMs are in running state

    Step 5: Useful Vagrant Commands

    `vagrant halt` - Power off all the VMs

    `vagrant ssh vm_hostname` - logs in using vagrant user key

    `vagrant destroy` - this will destroy all the VMs
    

## Ansible Config

To configure K8S setup using Ansible

Setup Your Inventory with the all the necessary details of the nodes your going to use in your cluster

> ### Inventory Directory Structure

```bash

├── ansible_playbooks
│   ├── ansible.cfg
│   ├── inventory
│   │   ├── group_vars
│   │   │   ├── master.yaml
│   │   │   └── worker.yaml
│   │   ├── hosts.yaml
│   │   └── host_vars
```

In my case I'm using the Vagrant setup mentioned above to bootstrap my VM environment.

You can refer the IP and Host details mentioned in `hosts.yaml` file for our ansible inventory below.

```yaml
File: ansible_playbooks/inventory/hosts.yaml
---
all:
  hosts:
  children:
    master:
      hosts:
        kubemaster:
          ansible_host: 192.168.56.111

    worker:
      hosts:
        kubenode01:
          ansible_host: 192.168.56.201
        kubenode02:
          ansible_host: 192.168.56.202
```

> ### Overview of Some Necessary Ansible Directories & Files

`group_vars` - This directory contains yaml files which are named after groups mentioned in our main inventory file `hosts.yaml`

`hosts.yaml` - This directory is useful to declare the variables for individual hosts. We are not using any such variables in this setup.

`master.yaml` - This file contains variables for Kubernetes master node

```bash
File: master.yaml
---
ansible_user: ubuntu  # Ubuntu user will be used by ansible for all the setup
pod_nw_cidr: 10.244.0.0/16  # This is necessary IPv4 CIDR for pod-to-pod network

```

`worker.yaml` - This file contians variables for worker nodes

```bash
File: worker.yaml
---
ansible_user: ubuntu # Ubuntu user will be used by ansible for all the setup
```

`setup.yaml` - Primary ansible playbook that we will be using to trigger our complete setup automation task.

`roles` - Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure.

    File Structure for roles directory in our repo:
```bash
    roles
    ├── cluster-setup # K8S Cluster setup on master node
    │   └── tasks
    │       └── main.yaml
    ├── commons # Installs common packages & dependencies
    │   └── tasks
    │       └── main.yaml
    ├── configure-kernel # Updates system level settings for all nodes
    │   └── tasks
    │       └── main.yaml
    ├── containerd # Installs CRI and other necessary packages
    │   └── tasks
    │       └── main.yaml
    └── worker-setup # Helps worker nodes to join the K8S cluster
        └── tasks
            └── main.yaml
```

Make sure you have all the necessary Nodes configured in your inventory file with all the necessary variable as mentioned in the above section.

Once done try running ping from ansible to check connectivity.

`ansible -m ping all`

You will get the similar output as mentioned below. If not verify whether the node network reachability and user public key used for this setup.

Output:

```javascript
kubemaster | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
kubenode02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
kubenode01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

---
> Test your Environment Setup


Testing your ansible playbook code before actually applying the configuration one of the crucial step to tackle any misconfigurations or human errors.

`ansible-playbook setup.yaml -C`

This ensures that you have configured your inventory correctly and helps you dry run your configuration on your setup.

Now you can run same command without check to deploy the Kubernetes cluster environment on running VMs

`ansible-playbook setup.yaml`

Upon successful execution of above playbook you will be able to browse your Kubernetes cluster using `ubuntu` user.

Output: You will receive the output similar to below once you executed the ansible playbook `setup.yaml`

```bash
TASK [worker-setup : Check result for cluster] *********************************************
ok: [kubenode01] => {
    "msg": "Node 192.168.56.201 has joined the cluster. "
}
ok: [kubenode02] => {
    "msg": "Node 192.168.56.202 has joined the cluster. "
}
```

You can refer to the below workflow to understand the basic working concept of Ansible playbooks to construct Kubernetes cluster.

## Workflow of the ansible playbook is divided into 3 stages:

### Stage 1 - Install common packages and dependencies on all nodes

1. commons - Installs common packages & dependencies
    - curl
    - gnupg
    - net-tools
    - apt-transport-https
    - ca-certificates

2. configure-kernel - Updates system level settings and kernel parameter for all nodes

    - Loads module 'br_netfilter' in kernel
    - Enables following kernel parameters
        - net.bridge.bridge-nf-call-ip6tables
        - net.bridge.bridge-nf-call-iptables
        - net.ipv4.ip_forward

3. containerd - Installs CRI and other necessary packages

    Below is the list of oackages that will be installed on the nodes
    1. CRI (Container Runtime Interface) - We will need containerd.io CRI to run containers in our pods for Kubernetes.

    2. kubeadm - Using kubeadm, you can create a minimum viable Kubernetes cluster that conforms to best practices.

    3. kubelet - Kubelet is an agent or program that runs on each node in a Kubernetes cluster.

    4. kubectl - Kubectl is a command-line tool that is used to manage Kubernetes clusters. It provides a way to communicate with the Kubernetes API and carry out HTTP requests to the API. 

This stage makes your nodes compatible with kubernetes environment while resolving all the dependencies and installing all the necessary packages for kubernetes cluster.

### Stage 2 - Prepare Kubernetes Master for cluster setup

- In this stage we will be initializing the kubernetes cluster using single kubeadm command.

`kubeadm init <args>` - To initialize the control-plane node.

- Complete command with arguments should be like this:

`kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.111`

Note that, depending on the variables you set the above values might change for your infrastructure.

### Stage 3 - Join Worker Nodes to K8S Cluster

- Joining Worker Nodes to the cluster

`kubeadm join` - Running this command with necessary token info from controlplane node helps you to join any number of nodes to your kubernetes cluster.

- Refer below for complete command:

`kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

- Creating token on control plane:

`kubeadm token create --print-join-command` - You can generate the token anytime after initiating the cluster using this command.

> ### Test your Kubernetes cluster

- Check the cluster status on Control Plane node

    `kubectl cluster-info` - This command will give you the cluster info with control plane nodes details

    ```bash
    ubuntu@kubemaster:~$ kubectl cluster-info
    Kubernetes control plane is running at https://192.168.56.111:6443
    CoreDNS is running at https://192.168.56.111:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
    ```

- Check the list of nodes in your cluster

    ```bash
    ubuntu@kubemaster:~$ kubectl get nodes
    NAME         STATUS   ROLES           AGE     VERSION
    kubemaster   Ready    control-plane   7m53s   v1.28.2
    kubenode01   Ready    <none>          7m12s   v1.28.2
    kubenode02   Ready    <none>          7m12s   v1.28.2
    ```

- Run your first pod in K8S cluster

    `kubectl run mypod --image=nginx`

    Output:
    ```bash
    ubuntu@kubemaster:~$ kubectl get pods
    NAME    READY   STATUS    RESTARTS   AGE
    mypod   1/1     Running   0          60s
    ```

- Create your first deployment in K8S cluster

    `kubectl create deployment webserver --image=nginx --port=80 --replicas=3`

    Output: You can observe the 3 pods running from this deployment
    ```bash
    ubuntu@kubemaster:~$ kubectl get all
    NAME                             READY   STATUS    RESTARTS   AGE
    pod/mypod                        1/1     Running   0          2m26s
    pod/webserver-5d5c5c44c7-m7l86   1/1     Running   0          45s
    pod/webserver-5d5c5c44c7-xc9vc   1/1     Running   0          45s
    pod/webserver-5d5c5c44c7-zz5h6   1/1     Running   0          45s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   12m

    NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/webserver   3/3     3            3           45s

    NAME                                   DESIRED   CURRENT   READY   AGE
    replicaset.apps/webserver-5d5c5c44c7   3         3         3       45s
    ```

- Expose the pod to external network

    `kubectl expose deployment webserver --port=80 --type=NodePort`

    Output: Check Kubernetes service with same name as that of the deployment.
    ```bash
    ubuntu@kubemaster:~$ kubectl get service
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        13m
    webserver    NodePort    10.111.120.231   <none>        80:31158/TCP   17s
    ```

    You can verify the nginx webpage using curl call as mentioned below:

    Make sure you use NodePort Service IP to access the nginx webpage

    ```bash
    ubuntu@kubemaster:~$ curl 10.111.120.231:80
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>    
    ```

Thanks to make till this end and hope you find this interesting. There are many ways and possibilities for deploying kubernetes cluster. Here we are using kubeadm utility to deploy kubernetes cluster which is easy to setup with minimal operational overhead.

Please do let me know if you have any suggestions or improvements.

