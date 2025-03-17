---
title: "OpenStack All-in-one Kolla Ansible Deployment"
datePublished: Mon Mar 17 2025 17:58:49 GMT+0000 (Coordinated Universal Time)
cuid: cm8ddc4hf000509jphwga1ue5
slug: openstack-all-in-one-kolla-ansible-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1742234221458/408b1330-4947-4ebf-a8ff-982c1f4d282d.jpeg
tags: openstack, openstack-development-company

---

OpenStack is an open-source cloud computing platform that supports all types of cloud environments. It is used by large organizations to manage and control large-scale deployments of virtual machines and help in building private or public clouds.

It is Infrastructure-as-a-service functionality.

# Installation

This section provides information related to deploying OpenStack. It can be installed on a single node or multi-node.

OpenStack can be installed on the following platforms:

* Linux
    
* Mac
    
* Windows using WSL
    

Here, we will discuss the installation of OpenStack on Linux on a single node.

## Prerequisites

* Select the [Deployment Tools](https://www.openstack.org/software/project-navigator/deployment-tools).
    
* [Host Machine Requirements](https://docs.openstack.org/kolla-ansible/2024.2/user/quickstart.html) as per released version.
    
    * 2 network interfaces ( Provide IP address to one of them )
        
    * 8GB main memory
        
    * 40GB disk space
        

**Note: There are many different deployment tools OpenStack provide for single or multinode installation of OpenStack.**

Here, we will use Kolla-Ansible deployment tool to install OpenStack

### Step to install dependencies

Step 1: For Debian or Ubuntu, update the package index.

```bash
sudo apt update
```

Step 2: Install Python build dependencies, For Debian or Ubuntu, run:

```bash
sudo apt install git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev
```

### Step to Install dependencies for the virtual environment

Step 1: Install the virtual environment dependencies. For Debian or Ubuntu, run:

```bash
sudo apt install python3-venv
```

Step 2: Create a virtual environment and activate it:

```bash
python3 -m venv /path/to/venv
source /path/to/venv/bin/activate
```

Step 3: Ensure the latest version of pip is installed:

```bash
sudo pip install -U pip
```

Step 4: Install Ansible. Kolla Ansible requires at least Ansible core 2.16 and supports up to 2.17.

```bash
sudo pip install 'ansible-core>=2.16,<2.17.99'
```

### Step to install Kolla-Ansible

Step 1: Install kolla-ansible and its dependencies using `pip`.

```bash
pip install git+https://opendev.org/openstack/kolla-ansible@stable/2024.2
```

Noted if above URL is not working, check the url *git+https://opendev.org/openstack/kolla-ansible* and see their tag/branch name and use them in place of ‘stable’ in above url.

Step 2: Create the `/etc/kolla` directory.

```bash
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
```

$USER is your username on which you are deploying openstack.

Step 3: Copy `globals.yml` and `passwords.yml` to `/etc/kolla` directory.

```bash
cp -r /path/to/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
```

Step 4: Copy `all-in-one` inventory file to the current directory.

```bash
cp /path/to/venv/share/kolla-ansible/ansible/inventory/all-in-one .
```

Step 5: Install Ansible Galaxy dependencies:

```bash
kolla-ansible install-deps
```

### Step to Configure Kolla-Ansible

Step 1: Add the following option to the Ansible configuration file */etc/ansible/ansible.cfg*:

```bash
[defaults]
host_key_checking=False
pipelining=True
forks=100
```

Step 2: Check whether the configuration of inventory is correct or not, run the below command:

```bash
ansible -i all-in-one all -m ping
```

Step 3: Kolla Password. Passwords used in our deployment are stored in `/etc/kolla/passwords.yml` file. All passwords are blank in this file and have to be filled either manually or by running random password generator:

```bash
kolla-genpwd
```

Step 4: `globals.yml` is the main configuration file for Kolla Ansible and per default stored in */etc/kolla/globals.yml* file. There are a few options that are required to deploy Kolla Ansible:

* ```bash
      kolla_base_distro: "Ubuntu"
    ```
    
    Kolla provides choice of several Linux distributions in containers:
    
    * CentOS Stream (`centos`)
        
    * Debian (`debian`)
        
    * Rocky (`rocky`)
        
    * Ubuntu (`ubuntu`)
        
    
* ```bash
      kolla_install_type: "source"
    ```
    
* ```bash
      network_interface: "ens3"
    ```
    
    This interface should be active with IP address.
    
* ```bash
      neutron_network_interface: "ens8"
    ```
    
    This interface should be active without IP address.
    
* ```bash
      kolla_internal_vip_address: "10.1.0.250"
    ```
    
    We need to provide the floating ip address for managment traffic. This IP address not being used in your internal IP but must be in the same subnet as ip address of *‘ens3’.*
    
* ```bash
      neutron_plugin_agent: "openvswtich"
    ```
    
* ```bash
      enable_neutron_provider_network: "yes"
    ```
    
    Used for external connectivity of OpenStack instances.
    

### Steps for Deployment

Step 1: Bootstrap servers with Kolla deploy dependencies:

```bash
kolla-ansible -i ./all-in-one bootstrap-servers
```

Step 2: Do pre-deployment checks for hosts:

```bash
kolla-ansible -i ./all-in-one prechecks
```

Step 3: Finally proceed to actual OpenStack deployment:

```bash
kolla-ansible -i ./all-in-one deploy
```

Note: In any case deployment is failed due to any reason use the below command

```bash
docker rm $(docker ps -aq)
```

```bash
docker rmi $(docker images -aq)
```

```bash
kolla-ansible destroy -i all-in-one --yes-i-really-really-meant-it
```

### Step to use OpenStack

Step 1: nstall the OpenStack CLI client:

```bash
pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/2024.2
```

Step 2: OpenStack requires a `clouds.yaml` file where credentials for the admin user are set. To generate this file:

```bash
kolla-ansible post-deploy
```

Step 3: Depending on how you installed Kolla Ansible, there is a script that will create example networks, images, and so on.

```bash
/path/to/venv/share/kolla-ansible/init-runonce
```

## Way to Access OpenStack Dashboard

Step 1: source /etc/kolla/admin-openrc.sh

User can check Dasboard ceredential like username and password. login to openstack stack using local ip address of ens3 through browser.