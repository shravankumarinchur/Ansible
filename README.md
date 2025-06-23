# Ansible
Learnings


Day-1
* Configuration Management Tools Overview
* Chef / Puppet /Ansible
* What Ansible Can Do
* Scripting vs Ansible
* When to Use Python Over Ansible
* Provisioning: Ansible vs Terraform
* Important Ansible Internals
* Ansible Installation

Day 2
* Passwordless SSH Setup
* Ansible Inventory Management
* Grouping and Targeting Hosts in Ansible Inventory
* Understanding Ad-hoc Commands and Their Usage
* Examples of Ad-hoc Commands for System Management Tasks



 Day 3 

* Learned the basics of **YAML** syntax (scalars, lists, dictionaries).
* Understood how Ansible uses YAML to define **playbooks**.
* Explored the structure of an Ansible playbook: `hosts`, `become`, `tasks`.
* Wrote a playbook to **install Apache** and **deploy an HTML page** on AWS EC2 instance.
* Executed **ad-hoc Ansible commands** to manage services (e.g., start/stop Apache).
 

 Day 04 

*  Problem with large monolithic playbooks
*  Introduction to Ansible Roles
*  Understanding the Role Directory Structure:
*  Converting an existing playbook into a role
*  Example: Apache setup using a role
*  Benefits: Modular, Reusable, Sharable (Ansible Galaxy)
*  Idempotency in Ansible

 Day  05
*  Explore prebuilt roles 
*  Setup Docker using existing roles
*  Publish a role to ansible galaxy

  
  Day 6
* Introduced Ansible Collections for cloud and third-party integrations.
* Used amazon.aws collection to provision an EC2 instance via Ansible.
* Secured AWS credentials using Ansible Vault.
* Resolved boto3/botocore dependency issues with Python 3.12.
*  Learned 5 key methods to manage variables in Ansible with proper precedence:

 Day 07
* Created IAM user with EC2 full access and secured keys using ansible-vault.
* Automated EC2 instance provisioning with amazon.aws.ec2_instance module using loops and unique naming.
* Setup passwordless SSH login to EC2 instances for smooth Ansible runs.
* Used Ansible conditionals (when) to shutdown Ubuntu instances selectively.
* Gathered system facts and filtered specific details using the debug module.
* Executed shell commands on remote hosts, captured output, and displayed results.
* Best practice reminders on idempotency and region-specific AMI usage.

 Day 08
* Learned Ansible's default task execution order 
* Used ignore_errors: yes to allow playbook continuation despite individual task failures.
* Used register and when to conditionally run tasks based on command results (e.g., install Docker only if not installed).
* Handled specific error types using failed_when for fine-grained control over task failure logic.











