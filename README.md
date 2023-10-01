
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

Below is the directory structure of the repo:

```bash
.
├── ansible_config
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
├── README.md
├── ubuntu 
│   ├── update-dns.sh
│   └── vagrant
│       ├── install-guest-additions.sh
│       ├── my_key.pub
│       └── setup-hosts.sh
└── Vagrantfile

18 directories, 20 files
```

Detailed explanation for ansible config used in this project

Setup Your Inventory with the all the necessary details of the nodes your going to use in your cluster

Inventory Directory Structure

```
├── ansible_config
│   ├── ansible.cfg
│   ├── inventory
│   │   ├── group_vars
│   │   │   ├── master.yaml
│   │   │   └── worker.yaml
│   │   ├── hosts.yaml
│   │   └── host_vars

```

K8S Cluster Setup is constructed in 3 stages

Stage 1 - Install common packages and dependencies on all nodes

Stage 2 - Prepare Kubernetes Master for cluster setup

Stage 3 - Join worked nodes to K8S cluster

