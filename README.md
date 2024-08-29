Amazon-AIO (All in one)
=========

This is an AWS deployment playbook to help get you get familar deploying in AWS with Ansible.  These roles are designed to be run one at a time with the use of tags.  This playbook in its current state (w/ a populated vars file) will deploy the following:

        S3 Bucket(s) - Ansible 2.5
        rds - Ansible 2.6
        VPC - Ansible 2.5
        Public Network(s) - Ansible 2.5
        Private Network(s) - Ansible 2.5
        IGW (Internet Gateway) - Ansible 2.5
        NATG (Nat Gateway) - Ansible 2.5
        EC2-TO-CLW - Ansible 2.5
        Routing Table(s) - Ansible 2.3
        Security Group(s) - Ansible 2.3
        Classic Load Balancer(s) -Ansible 2.3
        EC2 Launch configuration(s) -Ansible 2.3
        EC2 Auto Scaling Group(s) - Ansible 2.3


Requirements
============

Tasks are written to Ansibe 2.3 - Currently updating to 2.6/2.7  You will need an AWS account.  The vars file is encrypted with ansible-vault.  You will need to vault password to edit the vars file as well as running the playbook.  The "pre-flight" role will help you Configure your deployment host assuming centos/redhat/fedora (pre dnf).  I am also using a specific host to run these playbooks.  If you are running local dont forget to update the play/role

###
###    YUM
###
    epel-release
    python-pip
    libselinx-python (used for selinux enforing hosts)

###
###    PIP
###
    awscli
    saws
    boto
    botocore
    boto3

Variables
=========

Variables are set to be called from the var/vars.yml file.  This file is encrypted wih ansible vault.  An unencrypted vars file cloud also included to suit your needs.  The following var's will need to be set as some are called multiple times.  Please change as you see fit.  I tried to break our vars by role.  This is sill a work in progess.

###
# AWS ACCOUNT #
###
    aws_org_name:
    aws_Acct_num:
###
# TASK - VPC #
###
    aws_access_key:
    aws_secret_key:
    aws_region:
    cidr_block:
    vpc_name:
    aws_dns_support:
###
# TASK - Public Subnets #
###
    pub_sub_cidr_a:
    pub_sub_az_a:
    pub_sub_name_a:
    pub_sub_cidr_b:
    pub_sub_az_b:
    pub_sub_name_b:
    pub_sub_cidr_c:
    pub_sub_az_c:
    pub_sub_name_c:

###
# TASK - Private Subnets #
###
    pri_sub_cidr_a:
    pri_sub_az_a:
    pri_sub_name_a:
    pri_sub_cidr_b:
    pri_sub_az_b:
    pri_sub_name_b:
    pri_sub_cidr_c:
    pri_sub_az_c:
    pri_sub_name_c:

###
# Task - ec2-to-clw #
###
    pem_key: /path/to/pem.key


Dependencies
============

To get started with dynamic inventory management, you’ll need to grab the EC2.py script and the EC2.ini config file. The EC2.py script is written using the Boto EC2 library and will query AWS for your running Amazon EC2 instances. The EC2.ini file is the config file for EC2.py, and can be used to limit the scope of Ansible’s reach. You can specify the regions, instance tags, or roles that the EC2.py script will find.
        https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py
        https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini

Or have a quick look at
        https://aws.amazon.com/blogs/apn/getting-started-with-ansible-and-dynamic-amazon-ec2-inventory-management/


Playbook Roles
=================

Pre-Flight:
============
Sets up your host to play with AWS.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t pre-flight -k --ask-vault-pass

RDS: (pending validation)
=====
Sets up one or more RDS instances.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t rds -k --ask-vault-pass

S3:
====
Sets up one or more Private S3 buckets without a bucket policy.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t s3 -k --ask-vault-pass

VPC:
====
Sets up a single VPC in US-West-2 Region.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t vpc -k --ask-vault-pass

Public Subnets:
========
Sets up 3 subnets. one per AZ.  These subnets tagged for public IGW Attached in a later play.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t public-subnets -k --ask-vault-pass

Private Subnets
========
Sets up 3 subnets.  One per AZ.  These subnets are tagged for private use.  NAT Gateway Attached in a later play  
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t private-subnets -k -- ask-vault-pass

IGW: (Internet Gateway)
=========
Sets up an Internet Gateway
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t igw -k --ask-vault-pass

NATG (NAT Gateway)
Sets up a Nat Gateway for every private subnet.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t natg -k --ask-vault-pass

Routing-Public:
===============
Sets up a routing table for the public subnets with the VPC's Internet Gateway.
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t routing-public -k --ask-vault-pass

Routing-Private:
================
Sets up routing tables for each of the private subnets.

ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t routing-private-2a -k --ask-vault-pass
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t routing-private-2b -k --ask-vault-pass
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t routing-private-2c -k --ask-vault-pass

Security Groups:
================
Sets up Basic security groups.

ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t sec-ssh -k --ask-vault-pass
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t sec-http -k --ask-vault-pass

Load Balancers:
================
Sets up one Internet-Facing load balancer and one Internal load balancer.

ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t elb-public -k --ask-vault-pass
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t elb-private -k --ask-vault-pass

Launch configuration:
=====================
Sets up two basic Launch configurations.

ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t lc-ssh -k --ask-vault-pass
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t lc-http -k --ask-vault-pass

Auto Scaling Group:
===================
Sets up two Auto-scaling groups based on the Launch configurations.

ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t asg-ssh -k --ask-vault-pass
ansible-playbook -i inventory/local.yml Amazon-Setup.yml -t asg-http -k --ask-vault-pass

EC2 Custom Metrics to CloudWatch:
=================================
Sets up Centos 6/7 and Ubuntu ec2 hosts to send disk/mem/swap stats to CloudWatch as a custom metric.
The target EC2's will also need a iam perms to complete this task.  you will need the following.

	cloudwatch:PutMetricData
	cloudwatch:GetMetricStatistics
	cloudwatch:ListMetrics
	ec2:DescribeTags

ansible-playbook -i inventory/utm.yml Amazon-Setup.yml -t ec2-to-clw -- ask-vault-pass

License
=======

MIT

Author Information
==================

Adam A. Valenzuela - Brevity is the Soul of Wit.
# Ansible
