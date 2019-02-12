# Backup & Restore DC/OS Instructions

## Testing with Universal Installer

### Prerequisites
 - Terraform
 - AWS Access Keys
 - AWS SSH Keys

### Deploying CoreOS Machines

We assume a person is already familiar with the Mesosphere's Universal Installer documentation here: https://docs.mesosphere.com/1.12/installing/evaluation/aws/.

We will be using a customized version of the `main.tf` and will be using it to install a specific version of CoreOS

```bash
provider "aws" {
  # Change your default region here
  region = "us-west-2"
}

module "dcos" {
  source  = "dcos-terraform/dcos/aws"
  version = "~> 0.1"

  dcos_instance_os    = "coreos_1632.3.0"
  cluster_name        = "vz-os-upgrade-test"
  ssh_public_key_file = "<path-to-public-key-file>"
  admin_ips           = ["${data.http.whatismyip.body}/32"]

  num_masters        = "3"
  num_private_agents = "3"
  num_public_agents  = "1"

  dcos_version              = "1.10.9"
  custom_dcos_download_path = "<ASK_MESOSPHERE>" 

  providers = {
    aws = "aws"
  }

  dcos_install_mode = "${var.dcos_install_mode}"
}

variable "dcos_install_mode" {
  description = "specifies which type of command to execute. Options: install or upgrade"
  default     = "install"
}

# Used to determine your public IP for forwarding rules
data "http" "whatismyip" {
  url = "http://whatismyip.akamai.com/"
}

output "masters-ips" {
  value = "${module.dcos.masters-ips}"
}

output "cluster-address" {
  value = "${module.dcos.masters-loadbalancer}"
}

output "public-agents-loadbalancer" {
  value = "${module.dcos.public-agents-loadbalancer}"
}
```

### Manual Backup and Restore Procedure

#### Manual Backup Solution

```bash
# Backup ZK directory of each master
sudo tar -cvf exhibitor-server1.tar /var/lib/dcos/exhibitor

# For each master perform this command below
# Copy Exhibitor File to Server
scp exhibitor-server1.tar core@213.0.113.101:~/
scp exhibitor-server2.tar core@213.0.113.102:~/
scp exhibitor-server3.tar core@213.0.113.103:~/
```

### Deploying RHEL 7.6 Machines

To deploy a RHEL machine, one 
```bash
provider "aws" {
  # Change your default region here
  region = "us-west-2"
}

module "dcos" {
  source  = "dcos-terraform/dcos/aws"
  version = "~> 0.1"

  cluster_name        = "vz-os-upgrade-test"
  ssh_public_key_file = "<path-to-public-key-file>"
  admin_ips           = ["${data.http.whatismyip.body}/32"]

  aws_ami             = "ami-036affea69a1101c9" # RHEL 7.6 in US-WEST-2 for DC/OS 1.10.9

  num_masters        = "3"
  num_private_agents = "3"
  num_public_agents  = "1"

  dcos_version              = "1.10.9"
  custom_dcos_download_path = "<ASK_MESOSPHERE>" 

  providers = {
    aws = "aws"
  }

  dcos_install_mode = "${var.dcos_install_mode}"
}

variable "dcos_install_mode" {
  description = "specifies which type of command to execute. Options: install or upgrade"
  default     = "install"
}

# Used to determine your public IP for forwarding rules
data "http" "whatismyip" {
  url = "http://whatismyip.akamai.com/"
}

output "masters-ips" {
  value = "${module.dcos.masters-ips}"
}

output "cluster-address" {
  value = "${module.dcos.masters-loadbalancer}"
}

output "public-agents-loadbalancer" {
  value = "${module.dcos.public-agents-loadbalancer}"
}
```

#### Manual Restore Solution

```bash
#RESTORE COMMENCE
sudo systemctl stop dcos-exhibitor
sudo mv /var/lib/dcos/exhibitor/ /var/lib/dcos/exhibitor-previous.bak
sudo tar -xvf old-zkstate.tar
sudo cp -fr var/* /var/
sudo chown -R dcos_exhibitor:root /var/lib/dcos/exhibitor/
sudo systemctl reboot
#DONE

# Reboot the all agents
sudo systemctl reboot
```


<!---
## Testing

```bash
Vagrant.configure("2") do |config|
  config.vm.box = "kaarolch/coreos-stable"
  config.vm.box_version = "1632.3.0"
end
```
-->
