# CloudStack Automation

CloudStack installation, setup, deployment and management can be primarily be
performed via CloudMonkey, the official CLI and Ansible.

## CloudMonkey

CloudMonkey is the official CLI for Apache CloudStack.

Install the old Python based CloudMonkey, follow this guide:
https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+cloudmonkey+CLI

Also, install and test the new Golang based CloudMonkey. Get the binary from:
https://github.com/apache/cloudstack-cloudmonkey/releases

Documentation for the modern CloudMonkey:
https://github.com/apache/cloudstack-cloudmonkey/wiki

Things to hack and learn:
- Understand how to configure CloudMonkey
- How to create and use CloudMonkey server profiles
- How to make API requests in interpreter and command line modes
- How to get API response outputs in various display format
- How to use sed/awk and/or jq to process API response

Tools to use:
- cloudmonkey (legacy)
- cmk (modern)
- bash and jq

**Challenges**:
- Easy/Medium: Write a bash script to deploy few VMs using CloudMonkey and using
  jq capture and print list of running VMs. The bash script should take this
  number as a command line argument, for example to deploy 10 VMs:

      bash deploy-script.sh 10

- Hard: Write a bash script that can deploy an advanced CloudStack zone using
  CloudMonkey (you may either use KVM+MonkeyBox or simulator based environment)

## Ansible and CloudStack

Ansible is a popular Python based configuration management tool. You can learn
more about it here: https://www.ansible.com/resources/get-started

Recommended reading:
http://docs.cloudstack.apache.org/projects/archived-cloudstack-getting-started/en/latest/ansible.html
https://docs.ansible.com/ansible/2.6/scenario_guides/guide_cloudstack.html

Example projects using Ansible for CloudStack automation:
- Trillian, ACS environment automation: https://github.com/shapeblue/trillian
- All in a box setup example: https://github.com/shapeblue/cloudstack-ansible
- KVM automation example: https://github.com/rhtyd/peppercorn

Things to hack and learn:
- How to use Ansible for any kind of automation
- How to configure ansible and use the `cloudstack` ansible module

Tools to use:
- ansible
- bash

**Challenges**:
- Deploy a VM in a VPC with two tiers
- Deploy a basic zone (KVM+Monkeybox or simulator based environment)
