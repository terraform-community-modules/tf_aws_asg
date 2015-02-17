tf_aws_asg
==========
A Terraform module for creating an Auto-Scaling Group and a launch
configuration for it.
This module makes the following assumptions:
* You have subnets in a VPC and that you want your instances
   in two subnets (in two AZs)
* You can fully bootstrap your instances using an AMI + user_data
* You want to associate the ASG with an ELB
* Your instances behind the ELB will be in a VPC
* Your using a single Security Group for all instances in the ASG

Input Variables
---------------

- `lc_name` - The launch configuration name
- `ami_id`
- `instance_type`
- `iam_instance_profile` - The ARN of the Instance Profile the LC should
   launch instances with.
   E.g. arn:aws:iam::XXXXXXXXXXXX:instance-profile/my-instance-profile
- `key_name` - The SSH key name (uploaded to EC2) instances should
   be populated with.
- `security_group` - The Security Group ID that instances in the ASG
   should use.
- `user_data` - The path to the user_data file for the Launch
   Configuration
- `asg_name` - The Auto-Scaling group name.
- `asg_number_of_instances` - The number of instances we want in the ASG
    - This is used to populate the following ASG settings.
    - max_size
    - desired_capacity
- `asg_minimum_number_of_instances` - The minimum number of instances
   the ASG should maintain.
    - This is used for min_size
    - It defaults to 1
    - You can set it to 0 if you want the ASG to do nothing when an
      instances fails
- `health_check_grace_period` - Number of seconds for the health check
   time out. Defaults to 300.
- `health_check_type` - The health check type. Options are `ELB` and
   `EC2`. It defaults to `ELB` in this module.
- `load_balancer_name` - The name of the ELB to associate with the ASG,
   for settings it's backend instances. Ideally this is a reference to
   an ELB you're making in the same template as this ASG.
- `subnet_az1` - The VPC subnet ID for AZ1
- `subnet_az2` - The VPC subnet ID for AZ2

Outputs
-------

- `launch_config_id`
- `asg_id`

Usage
-----

You can use these in your terraform template with the following steps.

1.) Adding a module resource to your template, e.g. `main.tf`

```
module "my_autoscaling_group" {
  source = "github.com/solarce/tf_aws_asg"
  lc_name = "${var.lc_name}"
  ami_id = "${var.ami_id}"
  instance_type = "${var.instance_type}"
  iam_instance_profile = "${var.iam_instance_profile}"
  key_name = "${var.key_name}"
  //Using a reference to an SG we create in the same template.
  security_group = "${module.sg_web.security_group_id_web}"
  user_data = "${var.user_data}"
  asg_name = "${var.asg_name}"
  asg_number_of_instances = "${var.asg_number_of_instances}"
  asg_minimum_number_of_instancs = "${var.asg_minimum_number_of_instances}"
  //Using a reference to an SG we create in the same template
  load_balancer_name = "${module.my_elb.elb_name}"
  subnet_az1 = "${var.subnet_az1}"
  subnet_az2 = "${var.subnet_az2}"

  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
  aws_region = "${var.aws_region}"
}
```

2.) Setting values for the following variables, either through `terraform.tfvars` or `-var` arguments on the CLI

- aws_access_key
- aws_secret_key
- aws_region
- lc_name
- ami_id
- instance_type
- iam_instance_profile
- key_name
- security_group
- user_data
- asg_name
- asg_number_of_instances.
- load_balancer_name
- subnet_az1
- subnet_az2
