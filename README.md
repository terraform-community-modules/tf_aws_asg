tf_aws_asg
==========
A Terraform module for creating an Auto-Scaling Group and a launch
configuration for it.
This module makes the following assumptions:
* *You do not want an ELB associated with your ASG*
   * If you need ELB support, see [tf_aws_asg_elb](https://github.com/terraform-community-modules/tf_aws_asg_elb)
* You have subnets in a VPC
* You want your instance in N subnets
* You can fully bootstrap your instances using an AMI + user_data
* You're using a single Security Group for all instances in the ASG

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
    - This is usually set to resolve to a security group you make in the
      same template as this module, e.g. "${module.sg_web.security_group_id_web}"
    - It needs to be customized based on the name of your module resource.
   should use.
- `user_data` - The path to the user_data file for the Launch Configuration.
    - Terraform will include the contents of this file in the Launch Configuration.
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
   `EC2`. It defaults to `EC2` in this module.
- `azs` - The list of AZs - comma separated list
- `subnet_azs` - The VPC subnet IDs - comma separated list
  - Subnets must match the Availability Zones in var.azs


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
  source               = "github.com/terraform-community-modules/tf_aws_asg"
  lc_name              = "${var.lc_name}"
  ami_id               = "${var.ami_id}"
  instance_type        = "${var.instance_type}"
  iam_instance_profile = "${var.iam_instance_profile}"
  key_name             = "${var.key_name}"

  //Using a reference to an SG we create in the same template.
  // - It needs to be customized based on the name of your module resource
  security_group = "${module.sg_web.security_group_id_web}"

  user_data                      = "${var.user_data}"
  asg_name                       = "${var.asg_name}"
  asg_number_of_instances        = "${var.asg_number_of_instances}"
  asg_minimum_number_of_instances = "${var.asg_minimum_number_of_instances}"

  // The health_check_type can be EC2 or ELB and defaults to EC2
  health_check_type = "${var.health_check_type}"
  
  azs        = "${var.azs}"
  subnet_azs = "${var.subnet_azs}"

  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
  aws_region     = "${var.aws_region}"
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
- azs
- subnet_azs

Authors
=======

Created and maintained by [Brandon Burton](https://github.com/solarce) (brandon@inatree.org).

License
=======

Apache 2 Licensed. See LICENSE for full details.
