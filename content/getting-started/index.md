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
$ ernest login
Username: admin
Password: *********
Welcome back admin
```

Create a user:

```
$ ernest user create user1 xxx111YYY
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
$ ernest login
Username: user1
Password: *********
Welcome back user1
```

Configure the datacenter on Ernest using your Access Key, Secret Key, and VPC ID:

```
$ ernest datacenter create aws --region eu-central-1 --secret_access_key XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX --access_key_id YYYYYYYYYYYYYYYYYYYY my-dc
Datacenter 'my-dc' successfully created
```

## Create Your Environment

We will use this YAML (demo.yml) to create our environment:

```
---
name: quick-start
datacenter: my-dc

vpcs:
  - name: demo-vpc
    subnet: 10.0.0.0/16
    auto_remove: true

networks:
  - name: demo-net
    vpc: demo-vpc
    subnet: 10.0.11.0/24
    availability_zone: eu-central-1a
    public: true

security_groups:
  - name: demo-sg
    vpc: demo-vpc
    egress:
      - ip: 0.0.0.0/0
        protocol: any
        from_port: '0'
        to_port: '65535'
    ingress:
      - ip: 80.184.194.73/32
        protocol: tcp
        from_port: '80'
        to_port: '80'
      - ip: 80.184.194.73/32
        protocol: tcp
        from_port: '22'
        to_port: '22'

instances:
  - name: demo-ec2
    type: t2.micro
    image: ami-3f1bd150
    network: demo-net
    start_ip: 10.0.11.11
    count: 1
    key_pair: my-key
    security_groups:
      - demo-sg
```

Now we can apply our YAML:

```
$ ernest service apply demo.yml

Service Name: quick-start
Service ID: 30001e65-362d-4fd9-500f-03ecb416b11c

Firewalls           1/1   Created
Instances           1/1   Created
Internet Gateways   1/1   Created
Networks            1/1   Created
Vpcs                1/1   Created

Status: Applied

================
Platform Details
================

Name : quick-start
Status : done
Date : 2017-07-14 18:21:54.399942 +0000 UTC

VPCs:
+----------+--------------+--------+
|   NAME   |      ID      | SUBNET |
+----------+--------------+--------+
| demo-vpc | vpc-b92f54d1 |        |
+----------+--------------+--------+

Networks:
+----------+-----------------+-------------------+
|   NAME   |       ID        | AVAILABILITY ZONE |
+----------+-----------------+-------------------+
| demo-net | subnet-f8b3b390 | eu-central-1a     |
+----------+-----------------+-------------------+

Instances:
+------------+---------------------+------------+------------+
|    NAME    |         ID          | PUBLIC IP  | PRIVATE IP |
+------------+---------------------+------------+------------+
| demo-ec2-1 | i-0cc3e0b9917f465a9 | 52.58.85.0 | 10.0.11.11 |
+------------+---------------------+------------+------------+

Security groups:
+---------+-------------+
|  NAME   |  GROUP ID   |
+---------+-------------+
| demo-sg | sg-a3e5b5c8 |
+---------+-------------+
```

Congratulations you have built something with Ernest!

### Next Steps
Find out how to use the CLI [here](/documentation/). Then dive into the detailed examples for the supported providers:

- Amazon Web Services
- vCloud Director
