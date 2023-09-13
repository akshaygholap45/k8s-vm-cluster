# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:

# Define the number of master and worker nodes
# If this number is changed, remember to update setup-hosts.sh script with the new hosts IP details in /etc/hosts of each VM.
NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 2

IP_NW = "192.168.56."
MASTER_IP_START = 110
NODE_IP_START = 200

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # config.vm.box = "base"
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Provision Master Nodes
  (1..NUM_MASTER_NODE).each do |i|
      config.vm.define "kubemaster" do |node|
        # Name shown in the GUI
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubemaster"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubemaster"
        node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
        node.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"

        node.vm.provision "setup-hosts", :type => "shell", :path => "ubuntu/vagrant/setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end

        node.vm.provision "setup-dns", type: "shell", :path => "ubuntu/update-dns.sh"

        # node.vm.provision "shell", inline: <<-SHELL
        #   # Install Samba
        #   apt-get update
        #   apt-get install -y samba

        #   # Create a directory to share
        #   sudo mkdir -p /srv/smbshare
        #   sudo chmod 777 /srv/smbshare

        #   # Configure the Samba share
        #   echo "[smbshare]" | sudo tee -a /etc/samba/smb.conf
        #   echo "   path = /srv/smbshare" | sudo tee -a /etc/samba/smb.conf
        #   echo "   read only = no" | sudo tee -a /etc/samba/smb.conf
        #   echo "   guest ok = yes" | sudo tee -a /etc/samba/smb.conf

        #   # Set a Samba password for the 'ubuntu' user
        #   echo -e "qwedsa@123\nqwedsa@123" | sudo smbpasswd -a -s ubuntu

        #   # Restart the Samba service
        #   sudo systemctl restart smbd
        # SHELL

        node.vm.provision "copy_public_key", type: "file", source: "ubuntu/vagrant/akshay_key.pub", destination: "/tmp/my_key.pub"
        node.vm.provision  "update_public_key", type: "shell", inline: "cat /tmp/my_key.pub > /home/ubuntu/.ssh/authorized_keys"

      end
  end


  # Provision Worker Nodes
  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "kubenode0#{i}" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubenode0#{i}"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubenode0#{i}"
        node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
                node.vm.network "forwarded_port", guest: 22, host: "#{2720 + i}"

        node.vm.provision "setup-hosts", :type => "shell", :path => "ubuntu/vagrant/setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end

        node.vm.provision "setup-dns", type: "shell", :path => "ubuntu/update-dns.sh"

        # node.vm.provision "shell", inline: <<-SHELL
        #   # update repo
        #   sudo apt-get update
        #   sudo apt-get install -y cifs-utils

        #   # Make fstab entrry for smb share
        #   sudo mkdir -p /mnt/smbshare
        #   sudo chmod 777 /mnt/smbshare

        #   echo "//kubemaster/smbshare /mnt/smbshare cifs username=ubuntu,password=qwedsa@123,uid=1000,gid=1000 0 0" | sudo tee -a /etc/fstab
        #   # Mount the share
        #   sudo /bin/mount -a
        # SHELL

        node.vm.provision "copy_public_key", type: "file", source: "ubuntu/vagrant/akshay_key.pub", destination: "/tmp/my_key.pub"
        node.vm.provision  "update_public_key", type: "shell", inline: "cat /tmp/my_key.pub > /home/ubuntu/.ssh/authorized_keys"

    end
  end
end
