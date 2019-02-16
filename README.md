# Backup & Restore DC/OS Instructions

## Testing with Mesosphere Legacy Installer

### Prerequisites
 - Terraform
 - AWS Access Keys
 - AWS SSH Keys
 - Mesosphere GitHub Access

### Deploying CoreOS Machines


This instruction allows a user to create an environment to restore the data to confirm that the restored data on another cluster with a different operating system or specifications will work. 

Currently, this is being tracked here: https://github.com/bernadinm/bar-dcos. The logs of the output can be viewed here: https://github.com/bernadinm/bar-dcos/blob/master/replacement_master_logs.out


To create a cluster to test this, you can run this command below:

NOTE: Using the terraform-dcos-enterprise installer

```
ssh-add <key from https://mesosphere.onelogin.com/notes/41130>
git clone git@github.com:mesosphere/terraform-dcos-enterprise
cd terraform-dcos-enterprise
git checkout verizon-upgrade-restore-test

cat desired_cluster_profile.tfvars <<EOF
dcos_version = "1.10.9"
num_of_masters = "3"
num_of_private_agents = "3"
num_of_public_agents = "1"
aws_region = "us-west-2"
aws_bootstrap_instance_type = "m3.large"
aws_master_instance_type = "m4.2xlarge"
aws_agent_instance_type = "m4.2xlarge"
aws_public_agent_instance_type = "m4.2xlarge"
ssh_key_name = "default"
# Inbound Master Access
admin_cidr = "0.0.0.0/0"
os = "coreos_1632.3.0"
owner = "mbernadin"
expiration = "6h"
dcos_exhibitor_storage_backend = "aws_s3"
dcos_exhibitor_explicit_keys = "false"
dcos_master_discovery = "static"
dcos_license_key_contents = "<INSERT_LICENSE_KEY_HERE>”
EOF
terraform apply -var-file desired_cluster_profile.tfvars
```

After deploying your cluster, you will be able to log in and deploy any application to simulate cluster usage.  When you’re ready to do operating system change,  run this command below:

```
git checkout verizon-upgrade-restore-replacement-test
vi master.tf # update master ip on line 137 with your current master ip list of all masters

# Replace First Master with Changed OS RHEL 7.6
terraform apply -var-file desired_cluster_profile.tfvars -var master_select=1 -target null_resource.master[0] -target aws_elb_attachment.internal-master-elb[0] -target aws_elb_attachment.public-master-elb[0]

# Replace Second Master with Changed OS RHEL 7.6
terraform apply -var-file desired_cluster_profile.tfvars -var master_select=2 -target null_resource.master[1] -target aws_elb_attachment.internal-master-elb[1] -target aws_elb_attachment.public-master-elb[1]

# Replace Thrid Master with Changed OS RHEL 7.6
terraform apply -var-file desired_cluster_profile.tfvars -var master_select=3 -target null_resource.master[2] -target aws_elb_attachment.internal-master-elb[2] -target aws_elb_attachment.public-master-elb[2]
```

You can confirm now that DC/OS is still running.

