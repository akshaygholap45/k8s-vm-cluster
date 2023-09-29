
# K8S Cluster Automation 

## Vagrant Setup

To create multiple vms locally

1. Update the public key in directory /ubuntu/vagrant with my_key.pub
    This will be copied in Ubuntu user's authorized keys for passwordless authentication

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

    Step 3: "vagrant up" this triggers the script and provisions the VMs as mentioned in Vagrantfile

    Step 4: "vagrant status" - Make sure that all the VMs are in running state

    Step 5: Useful Vagrant Commands

        - vagrant halt - Power off all the VMs
        - vagrant ssh vm_hostname - logs in using vagrant user key
        - vagrant destroy - this will destroy all the VMs

## Ansible Config
To configure K8S setup
