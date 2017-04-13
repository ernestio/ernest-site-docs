---
date: 2016-03-09T00:11:02+01:00
title: Getting started
weight: 10
---

## Setup Ernest
In this quickstart guide we will build a simple environment consisting of a public server and private server on AWS, and allow SSH access from our IP. You will need a working copy of Ernest and the Ernest CLI, your AWS Access Key and Secret Key, and an AWS VPC.

### Get Ernest
Follow the instructions [here](/downloads/) to get Ernest and the Ernest CLI.

### Setup Your User
Target your Ernest instance:

```
$ ernest target https://10.50.1.11
Target set
```

where 10.50.1.11 is the IP address of the host running our Ernest instance.

Login to Ernest as admin:

```
$ ernest login --user admin --password abc123
Welcome back admin
```

Create a user:

```
$ ernest user create user1 abc123
User user1 successfully created
```

Create a group:

```
$ ernest group create group1
Group 'group1' successfully created, you can add users with 'ernest group add-user username group1'
```

Add the user to the group:

```
$ ernest group add-user user1 group1
User 'user1' is now assigned to group 'group1'
```

### Setup Your Datacenter

Login as the user:

```
$ ernest login --user user1 --password abc123
Welcome back user1
```

Configure the datacenter on Ernest using your Access Key, Secret Key, and VPC ID:

```
$ ernest datacenter create aws --region eu-west-1 --secret_access_key XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX --access_key_id YYYYYYYYYYYYYYYYYYYY my-dc
Datacenter 'my-dc' successfully created
```

## Create Your Environment

We will use this YAML (demo.yml) to create our environment:

```
---
name: demo
datacenter: my-dc

vpcs:
  - name: my-vpc
    subnet: 10.0.0.0/16
    auto_remove: true

networks:
  - name: public
    vpc: my-vpc
    subnet: 10.0.10.0/24
    public: true
  - name: private
    vpc: my-vpc
    subnet: 10.0.11.0/24
    public: false
    nat_gateway: private-nat

nat_gateways:
  - name: private-nat
    public_network: public

security_groups:
  - name: public-sg
    vpc: my-vpc
    egress:
      - ip: 0.0.0.0/0
        protocol: any
        from_port: '0'
        to_port: '65535'
    ingress:
      - ip: 52.211.19.211/32
        protocol: tcp
        from_port: '22'
        to_port: '22'
  - name: private-sg
    vpc: my-vpc
    egress:
      - ip: 0.0.0.0/0
        protocol: any
        from_port: '0'
        to_port: '65535'
    ingress:
      - ip: 10.0.0.0/16
        protocol: tcp
        from_port: '22'
        to_port: '22'

instances:
  - name: public
    elastic_ip: true
    type: t2.micro
    image: ami-ed82e39e
    network: public
    start_ip: 10.0.10.11
    count: 1
    key_pair: my-key
    security_groups:
      - public-sg

  - name: private
    type: t2.micro
    image: ami-ed82e39e
    network: private
    start_ip: 10.0.11.11
    count: 1
    key_pair: my-key
    security_groups:
      - private-sg

```

Now we can apply our YAML:

```
$ ernest service apply demo.yml
Environment creation requested
Ernest will show you all output from your requested service creation
You can cancel at any moment with Ctrl+C, even the service is still being created, you won't have any output
Applying you definition
 Created VPC 
   Subnet    : 10.0.0.0/16
   Status    : completed
 Created Internet Gateway my-vpc
   AWS ID : igw-46d2d12f
   Status : completed
 Created Network private
   Subnet : 10.0.11.0/24
   AWS ID : subnet-ec95dc84
   Status : completed
 Created Firewall public-sg
   Status    : completed
 Created Firewall private-sg
   Status    : completed
 Created Network public
   Subnet : 10.0.10.0/24
   AWS ID : subnet-2294dd4a
   Status : completed
 Created Instance private-1
   IP        : 10.0.11.11
   AWS ID    : i-04731288df658cab9
   Status    : completed
 Created Instance public-1
   IP        : 10.0.10.11
   PUBLIC IP : 52.58.250.234
   AWS ID    : i-0a9f811ea1242adc1
   Status    : completed
 Created Nat private-nat
   Status    : completed
SUCCESS: rules successfully applied
```

Congratulations you have built something with Ernest!

### Next Steps
Find out how to use the CLI [here](/documentation/). Then dive into the detailed examples for the supported providers:

- Amazon Web Services
- vCloud Director
