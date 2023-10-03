
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
    
    In this case my setup 



## Ansible Config

To configure K8S setup using Ansible

Below is the directory structure of the repo:

```bash
.
├── ansible_playbooks
│   ├── ansible.cfg
│   ├── inventory
│   │   ├── group_vars
│   │   │   ├── master.yaml
│   │   │   └── worker.yaml
│   │   ├── hosts.yaml
│   │   └── host_vars
│   ├── join_cluster.sh
│   ├── roles
│   │   ├── cluster-setup
│   │   │   └── tasks
│   │   │       └── main.yaml
│   │   ├── commons
│   │   │   └── tasks
│   │   │       └── main.yaml
│   │   ├── configure-kernel
│   │   │   └── tasks
│   │   │       └── main.yaml
│   │   ├── containerd
│   │   │   └── tasks
│   │   │       └── main.yaml
│   │   └── worker-setup
│   │       └── tasks
│   │           └── main.yaml
│   └── setup.yaml
├── image.png
├── README.md
├── ubuntu
│   ├── update-dns.sh
│   └── vagrant
│       ├── install-guest-additions.sh
│       ├── my_key.pub
│       └── setup-hosts.sh
├── ubuntu-bionic-18.04-cloudimg-console.log
└── Vagrantfile
```

Detailed explanation for ansible config used in this project

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

In my case I'm using the Vagrant setup mentioned above to bootstrap my VM environments. 

You can refer the screenshot given below with the IP and Host details mentioned in `hosts.yaml` file for our ansible inventory.

```bash
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

> ### Necessary variables

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

## K8S Cluster Setup is constructed in 3 stages

1. `setup.yaml` - Primary ansible playbook that we will be using to trigger our complete setup automation task.

2. Roles - Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure.

    File Structure for roles in our repo:

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


### Stage 1 - Install common packages and dependencies on all nodes

Make sure you have all the necessary Nodes configured in you inventory file with all the necessary variable as mentioned in the above section.

Once done try running ping from ansible to check connectivity.

`ansible -m ping all`

You will get the similar output as mentioned below. If not verify whether the node network reachability and user public key used for this setup.

Output:

```bash
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

Test you ansible playbook before actually applying the configuration

`ansible-playbook setup.yaml -C`

This ensures that you have configured your inventory correctly and helps you dry run your configuration on your setup.



Stage 2 - Prepare Kubernetes Master for cluster setup

Stage 3 - Join worked nodes to K8S cluster

