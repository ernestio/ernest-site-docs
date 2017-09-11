---
date: 2016-03-09T20:08:11+01:00
title: Providers
weight: 16
---

## Amazon Web Services

Amazon Web Services (AWS) is a secure cloud services platform, offering compute power, database storage, content delivery and other functionality to help businesses scale and grow.

The following AWS services are supported by Ernest:

- EC2
- VPC
- ELB
- S3
- Route53
- RDS
- EBS
- IAM

### Example YAML

```
---
name: demo
project: my-dc


vpcs:
  - name: my-vpc
    id: vpc-abcdef01
    subnet: 10.0.0.0/16
    auto_remove: true

networks:
  - name: web
    subnet: 10.0.10.0/24
    vpc: my-vpc
    public: true
  - name: db
    subnet: 10.0.11.0/24
    vpc: my-vpc
    public: false
    nat_gateway: db-nat
  - name: db-standby
    subnet: 10.0.12.0/24
    vpc: my-vpc
    public: false
    nat_gateway: db-nat

nat_gateways:
  - name: db-nat
    public_network: web

security_groups:
  - name: web-sg
    vpc: my-vpc
    egress:
      - ip: 0.0.0.0/0
        protocol: any
        from_port: '0'
        to_port: '65535'
    ingress:
      - ip: 10.0.0.0/16
        protocol: any
        from_port: '0'
        to_port: '65535'
      - ip: 52.211.19.211/32
        protocol: tcp
        from_port: '22'
        to_port: '22'
  - name: db-sg
    vpc: my-vpc
    egress:
      - ip: 0.0.0.0/0
        protocol: any
        from_port: '0'
        to_port: '65535'
    ingress:
      - ip: 10.0.0.0/16
        protocol: any
        from_port: '0'
        to_port: '65535'
      - ip: 52.211.19.211/32
        protocol: tcp
        from_port: '22'
        to_port: '22'

instances:
  - name: web
    elastic_ip: true
    type: t2.micro
    image: ami-ed82e39e
    network: web
    start_ip: 10.0.10.11
    count: 1
    key_pair: web-key
    security_groups:
      - web-sg

  - name: db
    type: t2.micro
    image: ami-ed82e39e
    network: db
    start_ip: 10.0.11.11
    count: 1
    key_pair: db-key
    iam_profile: test-iam
    security_groups:
      - db-sg
    volumes:
      - volume: database-vol
        device: /dev/sdx

ebs_volumes:
  - name: database-vol
    type: io1
    size: 100
    iops: 10000
    count: 1
    availability_zone: eu-west-1a

loadbalancers:
  - name: elb-1
    private: false
    networks:
      - web
    instances:
      - web
    listeners:
      - from_port: 80
        to_port: 80
        protocol: http
      - from_port: 443
        to_port: 443
        protocol: https
        ssl_cert: ssl-cert-id
    security_groups:
      - web-sg

s3_buckets:
  - name: bucket-1
    acl: private
    bucket_location: eu-west-1
    grantees:
      - id: foo
        type: emailaddress, id, uri
        permissions: full, read, read-acl, write-acl

route53_zones:
  - name: example.com
    private: false
    records:
      - entry: one.example.com
        type: A
        ttl: 3600
        instances:
          - web-1
      - entry: two.example.com
        type: CNAME
        ttl: 3600
        loadbalancers:
          - elb-1
      - entry: three.example.com
        type: MX
        ttl: 3600
        values:
          - 8.8.8.8
          - 8.8.4.4
      - entry: db.example.com
        type: CNAME
        ttl: 3600
        rds_clusters:
          - rds-aurora

rds_clusters:
  - name: rds-aurora
    engine: aurora
    port: 3306
    networks:
      - db
      - db-standby
    security_groups:
      - db-sg-1
    database_name: test
    database_username: test
    database_password: testpass-2

rds_instances:
  - name: rds-test-1
    cluster: rds-test
    size: db.r3.xlarge

iam_policies:
  - name: test-iam
    path: '/'
    description: 'test policy'
    policy_document_raw: |-
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "Stmt1431095274000",
                  "Effect": "Allow",
                  "Action": [
                      "iam:RemoveRoleFromInstanceProfile",
                      "iam:DeleteInstanceProfile",
                      "cloudformation:DescribeStacks",
                      "cloudformation:DescribeStackResources",
                      "ec2:AllocateAddress",
                      "ec2:AssignPrivateIpAddresses",
                      "ec2:AssociateAddress",
                      "ec2:CreateVolume",
                      "ec2:DescribeNetworkAcls",
                      "ec2:CreateTags",
                      "ec2:DescribeAddresses",
                      "ec2:DescribeAvailabilityZones",
                      "ec2:DescribeInstanceAttribute",
                      "ec2:DescribeInstanceStatus",
                      "ec2:DescribeInstances",
                      "ec2:DescribeInternetGateways",
                      "ec2:DescribeInternetGatewaysResult",
                      "ec2:DescribeKeyPairs",
                      "ec2:DescribeRouteTables",
                      "ec2:DescribeRegions",
                      "ec2:DescribeSecurityGroups",
                      "ec2:DescribeSubnets",
                      "ec2:DescribeTags",
                      "ec2:DescribeVolumeAttribute",
                      "ec2:DescribeVolumeStatus",
                      "ec2:DescribeVolumes",
                      "ec2:DescribeVpcs",
                      "ec2:ReplaceNetworkAclAssociation",
                      "ec2:RunInstances",
                      "ec2:StartInstances",
                      "ec2:StopInstances",
                      "ec2:TerminateInstances",
                      "ec2:UnassignPrivateIpAddresses",
                      "ec2:DescribeImages",
                      "ec2:GetPasswordData",
                      "ec2:GetConsoleOutput"
                  ],
                  "Resource": [
                      "*"
                  ]
              }
          ]
      }

iam_instance_profiles:
  - name: test-iam
    path: '/'
    roles:
      - test-iam

iam_roles:
  - name: test-iam
    path: '/'
    policies:
      - test-iam
    assume_policy_document:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
```

### Field Reference

#### Environment Options

```
name: demo
project: my-dc
```

Environment Options support the following fields:

* **name**
 * String that defines the name of the environment to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

* **project**
 * String that defines the name of the project where the environment is built.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must 50 characters maximum.


#### Networking

```
networks:
  - name: web
    subnet: 10.0.10.0/24
    public: true
  - name: db
    subnet: 10.0.11.0/24
    public: false
    nat_gateway: db-nat

nat_gateways:
  - name: db-nat
    public_network: web

```

Networking supports the following fields:

**networks**

A network is a subnet that our instances can connect to. This is what allows us to network/isolate VM’s together.

* **name**
 * String that defines the name of the subnet to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

* **subnet**
 * String that defines the IP address range of the subnet.
 * It must follow CIDR notation as described at: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique for this user & all the networks on the manifest.

* **public**
 * String that defines if the subnet will be public or private.
 * This field is optional.
 * Values can be: “true“ or “false“.
 * This field will default to “false“.

* **nat_gateway**
 * String that defines the name of the NAT gateway to use for this subnet.
 * This field is optional.
 * This field can be empty.

**nat_gateways**

NAT gateways enable instances in a private subnet to connect to the Internet or other AWS services, but prevent the Internet from initiating a connection with those instances.

* **name**
 * String that defines the name of the NAT gateway to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

* **public_network**
 * String that defines the name of the public network the NAT gateway will reside.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

#### Security Groups

```
security_groups:
  - name: web-sg
    egress:
      - ip: 0.0.0.0/0
        protocol: any
        from_port: '0'
        to_port: '65535'
    ingress:
      - ip: 10.0.0.0/16
        protocol: any
        from_port: '0'
        to_port: '65535'

```

Security Groups support the following fields:

* **name**
 * String that defines the name of the security group to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

**egress**

Security group egress rules.

* **ip**
 * String that defines the IP address range for this rule.
 * It must follow CIDR notation as described at: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 * This field is mandatory.
 * This field cannot be null or empty.

* **protocol**
 * String that defines the protocol for this rule.
 * This field is mandatory.
 * This field cannot be null or empty.
 * Values can be: tcp | udp | icmp | any

* **from_port**
 * Starting port-range number.
 * This field is mandatory.
 * Values can be: 0 - 65535.

* **to_port**
 * Ending port-range number.
 * This field is mandatory.
 * Values can be: 0 - 65535.

**ingress**

Security group ingress rules.

* **ip**
 * String that defines the IP address range for this rule.
 * It must follow CIDR notation as described at: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 * This field is mandatory.
 * This field cannot be null or empty.

* **protocol**
 * String that defines the protocol for this rule.
 * This field is mandatory.
 * This field cannot be null or empty.
 * Values can be: tcp | udp | icmp | any

* **from_port**
 * Starting port-range number.
 * This field is mandatory.
 * Values can be: 0 - 65535.

* **to_port**
 * Ending port-range number.
 * This field is mandatory.
 * Values can be: 0 - 65535.

#### Instances

```
instances:
  - name: web
    elastic_ip: true
    type: t2.micro
    image: ami-ed82e39e
    network: web
    start_ip: 10.0.10.11
    count: 1
    key_pair: web-key
    security_groups:
      - web-sg
    volumes:
      - volume: big-vol
        device: /dev/sdx

```

Instances support the following fields:

* **name**
 * String that defines the name of the instance to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user & manifest.
 * The value of this field must be 50 characters maximum.

* **elastic_ip**
 * String that defines if the instance will be assigned an elastic IP.
 * This field is optional.
 * Values can be: “true“ or “false“.
 * This field will default to “false“.

* **type**
 * String that defines the instance size.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be a valid AWS EC2 size.

* **image**
 * String that defines the name of an AMI to install.
 * This field is mandatory.
 * This field cannot be null or empty.

* **network:**
 * String that defines the name of a network to attach this instance.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field should exist on the networks section previously specified.

* **start_ip:**
 * String that defines the starting IP to allocate the VMs to be built.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be a valid IP.
 * This IP should belong to the network already defined on the instance.

* **count**
 * Integer that defines the number of instances to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be greater or equal that 1

* **key_pair**
 * String that specifies the key pair to be used to login to the instances.
 * This field is optional.
 * This field cannot be null or empty.

* **security_groups**
 * Array that contains security groups that will be applied to the instances.
 * This field is optional.
 * This field can be empty.

* **iam_profile**
 * (string) The iam instance profile that you want to associate to the instance.
 * This must specify an instance profile defined on the same environment yaml.
 * This field is optional.
 * This field can be empty.


  **volumes**

  EBS volumes to be attached to the vm. Please note that the count of the EBS volume must be greater or equal to the amount of instances specified.

  * **volume**
  * String that references the name of an EBS volume specidied on the yaml
  * This field is mandatory
  * This field cannot be empty or null

  * **device**
  * String that defines what device name is exposed to the instance
  * This field is mandatory
  * This field cannot be empty or null


#### Load Balancers

```
loadbalancers:
  - name: elb-1
    private: false
    subnets:
      - web-subnet
    instances:
      - web
    listeners:
      - from_port: 80
        to_port: 80
        protocol: http
      - from_port: 443
        to_port: 443
        protocol: https
        ssl_cert: ssl-cert-id
    security_groups:
      - web-sg

```
Loadbalancers support the following fields:

* **name**
 * String that defines the name of the instance to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.
 * The value of this field must be 50 characters maximum.

* **private**
 * Boolean that defines if the loadbalancer will be private or public.
 * This field is optional.
 * Values can be: “true“ or “false“.
 * This field will default to “false“.

 * **networks**
  * Array of String's that defines which public network/subnet to attach to the ELB.
  * This field is mandatory if private is set to false.
  * This field cannot be null or empty.
  * This field must specify a network/subnet that exists on the yaml.
  * Only one subnet can be deployed per availablilty zone. It is advised that you specify an AZ when defining a network/subnet.

* **instances**
 * Array of String's that defines which group of instances to add to the ELB.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must specify an instance group that exists on the yaml.

* **security_groups**
 * Array of String's that defines which security groups to apply to the ELB.
 * This field not is mandatory.
 * This field must specify a security group that exists on the yaml.

**listeners**

ELB Listeners.

* **protocol**
 * String that defines the protocol for this rule.
 * This field is mandatory.
 * This field cannot be null or empty.
 * Values can be: http | https | ssl | tcp
 * If https or ssl is specified, an ssl_cert must be specified.

* **from_port**
 * Starting port-range number.
 * This field is mandatory.
 * Values can be: 1 - 65535.

* **to_port**
 * Ending port-range number.
 * This field is mandatory.
 * Values can be: 1 - 65535.

* **ssl_cert**
 * The ssl certificate ID of a valid AWS Certificate Manager (ACM) cert.
 * This field is mandatory only if protocol is set to https or ssl.


#### S3 Buckets

```
s3_buckets:
  - name: bucket-1
    acl: private
    bucket_location: eu-west-1
    grantees:
      - id: foo
        type: emailaddress, id, uri
        permissions: full, read, read-acl, write-acl
```

S3 support the following fields:

* **name**
 * (string) name of the bucket.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.
 * The value of this field must be 50 characters maximum.

* **acl**
 * (string) the canned ACL to apply to the bucket.
 * The value of this field can be private, public-read, public-read-write
 * or authenticated-read

* **bucket_location**
 * (string) the location constraint
 * The value of this field can be EU, eu-west-1, us-west-1, us-west-2,
 * ap-south-1, ap-southeast-2, ap-northeast-1, sa-east-1, cn-north-1 or
 * eu-central-1

* **grantees**
 * (collection) List of permissions on the bucket.

* **id**
 * (string) Grantee name

* **type**
 * (string) Grantee type
 * Possible values : email address, id or uri

* **permissions**
 * (string) Permissions for this grantee on the bucket
 * Possible values: full, read, read-acl and write-acl



 #### Route53 Zones

 ```
 route53_zones:
   - name: example.com
     private: false
     records:
       - entry: one.example.com
         type: A
         ttl: 3600
         values:
           - 8.8.8.8
 ```

 S3 support the following fields:

 * **name**
  * (string) The domain name of the zone.
  * This field is mandatory.
  * This field cannot be null or empty.
  * This field must be unique by user &amp; manifest.

 * **private**
  * (boolean) sets the zone to be private hosted zone.
  * The value of this field can be true or false. Default is false.

**records**

The collection of DNS entries you want to host.

 * **entry**
  * (string) the full domain record entry
  * The value of this field should be an F.Q.D.N (fully qualified domain name)

 * **Type**
 * (string) the full domain record entry
 * The value of this field should be one of 'A', 'AAAA', 'CNAME', 'MX', 'PTR', 'TXT', 'SRV', 'SPF', 'NAPTR', 'NS', 'SOA'

 * **TTL**
 * (int) the expiration period of the entry.
 * The value of this field should be greater than 0

 * **values**
  * Array of strings. The values the entry will return
  * These values must be valid for the type of record you have selected.

 * **Loadbalancers**
  * Array of strings. The loadbalancers you wish the entry to resolve to.
  * The type of record must be set to CNAME
  * You must specify only loadbalancers or instances, not both.

 * **instances**
  * Array of strings. The loadbalancers you wish the entry to resolve to.
  * The type of record must be set to A
  * This must target the individual instance, including the instances number, i.e. web-1
  * You must specify only loadbalancers or instances, not both.

#### RDS Clusters

```
rds_clusters:
  - name: aurora-test
    engine: aurora
    engine_version: 1.9
    public: false
    port: 3306
    availablily_zones:
      - eu-west-1a
      - eu-west-1b
      - eu-west-1c
    security_groups:
      - sg-1
    networks:
      - db-nw
    database_name: aurora-test
    database_username: username
    database_password: password
    backups:
      window: Mon:21:30-Mon:22:00
      rentention_period: 1
    maintenance_window: Mon:21:30-Mon:22:00
    final_snapshot: true
```

RDS Clusters support the following fields:

* **name**
 * (string) The name of the rds cluster.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.

* **engine**
 * (string) The engine type of the rds cluster.
 * This field is mandatory.
 * This field cannot be null or empty.
 * Currently only 'aurora' is supported as a value.

* **engine_version**
 * (string) The engine_version of the rds cluster.
 * This field is not mandatory.

* **port**
 * (int) The port exposed by the rds cluster.
 * This field not is mandatory.
 * This field must be between 1150 - 65535 (default: 3306).

* **public**
 * (boolean) Sets the rds cluster to be publicly accessible.
 * This field not is mandatory.
 * The value of this field can be true or alse. Default is false.

* **availability_zones**
 * Array of strings. The availability zones to deploy the rds cluster to.
 * This field not is mandatory.
 * The availability zone must be in the same region as the vpc it is deployed to.
 * If the rds cluster is set to private, there must be networks specified for each availability zone.

* **security_groups**
 * Array of String's that defines which security groups to apply to the rds cluster.
 * This field not is mandatory.
 * This field must specify a security group that exists on the yaml.

* **networks**
 * Array of String's that defines which networks to expose the rds cluster to.
 * This field not is mandatory (if public is set to true).
 * This field must specify a security group that exists on the yaml.

* **database_name**
 * (string) The database name of the rds cluster.
 * This field is mandatory.
 * This field cannot be null or empty.

* **database_username**
 * (string) The database username of the rds cluster.
 * This field is mandatory.
 * This field cannot be null or empty.

* **database_username**
 * (string) The database password of the rds cluster.
 * This field is mandatory.
 * This field cannot be null or empty.

* **maintenance_window**
 * (string) The time period at which any maintenance tasks will occur.
 * This field is not mandatory.
 * Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)

* **replication_source**
 * (string) The target rds cluster or mysql instance to replicate from.
 * Must be a valid ARN (Amazon Resource Name).
 * http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html

* **final_snapshot**
 * (boolean) Performs a final snapshot when the rds cluster is deleted.
 * This field not is mandatory.
 * The value of this field can be true or alse. Default is false.

**backups**

Backup configuration.

* **window**
 * (string) The time period at which backups will occur.
 * This field is not mandatory.
 * Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)

* **retention**
 * (int) The amount of time in days that backups will be retained for.
 * This field is not mandatory.
 * Must be between 1 - 35 days

#### RDS Instances

```
rds_instances:
  - name: rds-test-1
    size: db.r3.xlarge
    engine: mysql
    engine_version: 5.7
    port: 3306
    public: true
    multi_az: true
    storage:
      type: gp2
      size: 100
      iops: 5000
    auto_upgrade: true
    final_snapshot: true
    license: bring-your-own-license
    database_name: aurora-test
    database_username: username
    database_password: password
    backups:
      window: Mon:21:30-Mon:22:00
      rentention_period: 1
    maintenance_window: Mon:21:30-Mon:22:00
    final_snapshot: true
```

RDS Instances support the following fields:

* **name**
 * (string) The name of the rds instance.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.

* **engine**
 * (string) The engine type of the rds instance.
 * This field is mandatory.
 * This field cannot be null or empty.
 * Currently only 'aurora' is supported as a value.

* **engine_version**
 * (string) The engine_version of the rds instance.
 * This field is not mandatory.

* **port**
 * (int) The port exposed by the rds instance.
 * This field not is mandatory.
 * This field must be between 1150 - 65535 (default: 3306).

* **public**
 * (boolean) Sets the rds instance to be publicly accessible.
 * This field not is mandatory.
 * The value of this field can be true or alse. Default is false.

* **multi_az**
 * (boolean) Deploys a synchronous standby instance in a different availability zone
 * Cannot be used in conjunction with the 'availability_zone' parameter
 * This field not is mandatory.
 * The value of this field can be true or alse. Default is false.

* **availability_zones**
 * Array of strings. The availability zones to deploy the rds instance to.
 * This field not is mandatory.
 * The availability zone must be in the same region as the vpc it is deployed to.
 * If the rds instance is set to private, there must be networks specified for each availability zone.

* **security_groups**
 * Array of String's that defines which security groups to apply to the rds instance.
 * This field not is mandatory.
 * This field must specify a security group that exists on the yaml.

* **networks**
 * Array of String's that defines which networks to expose the rds instance to.
 * This field not is mandatory (if public is set to true).
 * This field must specify a security group that exists on the yaml.

* **database_name**
 * (string) The database name of the rds instance.
 * This field is mandatory.
 * This field cannot be null or empty.

* **database_username**
 * (string) The database username of the rds instance.
 * This field is mandatory.
 * This field cannot be null or empty.

* **database_username**
 * (string) The database password of the rds instance.
 * This field is mandatory.
 * This field cannot be null or empty.

* **maintenance_window**
 * (string) The time period at which any maintenance tasks will occur.
 * This field is not mandatory.
 * Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)

* **promotion_tier**
 * (int) specifies the order in which an aurora replica is promoted to a primary.
 * This field is not mandatory.
 * Must be between 1 - 15.
 * Default is 1.

* **auto_upgrade**
 * (boolean) Performs automatic upgrades on the database (minor versions only)
 * This field not is mandatory.
 * The value of this field can be true or alse. Default is false.

* **replication_source**
 * (string) The target an rds instance to replicate from.
 * Must be a valid ARN (Amazon Resource Name) or instance name (if in the same region).
 * http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html

* **final_snapshot**
 * (boolean) Performs a final snapshot when the rds instance is deleted.
 * This field not is mandatory.
 * The value of this field can be true or alse. Default is false.


* **license**
 * (string) The type of license you want to use with the database
 * This field is not mandatory.
 * Must be one of 'license-included', 'bring-your-own-license' or 'general-public-license'

**storage**

Storage configuration. Only configurable when using a non-aurora engine.

* **type**
 * (string) The type of storage you want to use.
 * This field is not mandatory.
 * Must be one of 'gp2' (general purpose SSD), 'io1' (performance optimised SSD) or 'standard' (magnetic disk)
 * If io1 is specified, you must also set the 'iops' parameter

* **size**
 * (int) The amount of storage (GB) that you want to use.
 * This field is not mandatory.
 * Must be a value between 5 - 6144.
 * Minimum size can vary between engine types.

* **iops**
 * (int) The amount of iops that you want the database to be limited to.
 * This field is mandatory if storage type is 'io1'.
 * This field must only be used in conjunction with 'io1' as a storage type.
 * Must be a multiple between 3 and 10 of the storage amount.
 * Must also be an integer multiple of 1000.

**backups**

Backup configuration.

* **window**
 * (string) The time period at which backups will occur.
 * This field is not mandatory.
 * Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)

* **retention**
 * (int) The amount of time in days that backups will be retained for.
 * This field is not mandatory.
 * Must be between 1 - 35 days

#### EBS Volumes

```
 ebs_volumes:
   - name: database-vol
     count: 1
     type: io1
     size: 100
     iops: 10000
     availability_zone: eu-west-1a
     encrypted: true
     encryption_key_id: kms-id
```

* **name**
 * (string) The name of the ebs volume.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.

* **count**
 * Integer that defines the number of volunes to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be greater or equal that 1

 * **type**
  * (string) The type of storage you want to use.
  * This field is not mandatory.
  * Must be one of 'gp2' (general purpose SSD), 'io1' (performance optimised SSD) or 'st1' (magnetic disk)
  * If io1 is specified, you must also set the 'iops' parameter

 * **size**
  * (int) The amount of storage (GB) that you want to use.
  * This field is not mandatory.
  * Must be a value between 1 - 16384. (may vary for different storage types)

 * **iops**
  * (int) The amount of iops that you want the volume to be limited to.
  * This field is mandatory if storage type is 'io1'.
  * This field must only be used in conjunction with 'io1' as a storage type.
  * Must be a multiple between 3 and 10 of the storage amount.
  * Must also be an integer multiple of 1000.
  * Must be a value between 100 - 20000

* **encrypted**
 * (boolean) Specifies whether the volume will be encrypted.
 * This field not is mandatory.
 * The value of this field can be true or false. Default is false.
 * When enabling encryption, a kms key id must be provided

* **encryption_key_id**
 * (string) The kms key id to use when encrypting data on the volume.
 * This field is mandatory, only if encryption is set to true.

#### IAM Roles

```
iam_roles:
  - name: test-role
    path: '/'
    policies:
      - test-policy
    assume_policy_document:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
      Principal:
         Service: ec2.amazonaws.com
      Action: sts:AssumeRole
```

* **name**
 * (string) The name of the iam role.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.

* **description**
 * (string) A description of the iam role.
 * This field is not mandatory.

* **path**
 * (String) Defines the path associated with this role.
 * This field is mandatory.
 * This field cannot be null or empty.

* **policies**
 * Array of (string) The policies that you want to attach to this role.
 * This field is not mandatory.
 * Must be a policy specified on the same environment yaml

* **assume_policy_document**
 * (Hash) The trust policy that is associated with this role.
 * This field is mandatory.

#### IAM Policies

```
iam_policies:
  - name: test-policy
    path: '/'
    description: 'test policy'
    policy_document:
      Version: '2012-10-17'
      Statement:
      - Sid: Stmt1431095274000
        Effect: Allow
        Action:
          - ec2:StartInstances
          - ec2:StopInstances
        Resource:
          - "*"
```

* **name**
 * (string) The name of the iam role.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.

* **description**
 * (string) A description of the iam policy.
 * This field is not mandatory.

* **path**
 * (String) Defines the path associated with this policy.
 * This field is mandatory.
 * This field cannot be null or empty.

* **policy_document**
 * (Hash) The iam policy that is associated with this policy.
 * This field is mandatory.

#### IAM Instance Profiles

```
iam_instance_profiles:
  - name: test-instance-profile
    path: '/'
    roles:
      - test-role
```

* **name**
 * (string) The name of the iam instance profile.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user &amp; manifest.

* **path**
 * (String) Defines the path associated with this iam instance profile.
 * This field is mandatory.
 * This field cannot be null or empty.

* **roles**
 * Array of (string) The roles that you want to attach to this instance profile.
 * This field is not mandatory.
 * Must be a role specified on the same environment yaml

## Microsoft Azure

Microsoft Azure is an open, flexible, enterprise-grade cloud computing platform. Move faster, do more and save money with IaaS + PaaS.

The following Azure services are supported by Ernest:

- Availability sets
- Load balancers
- Network gateways
- Network interfaces
- Public IP
- Resource groups
- Security groups
- SQL servers
- SQL databases
- SQL firewall rules
- Storage accounts
- Storage containers
- Subnets
- Virtual machines
- Virtual networks

### Example YAML

```
name: demo
project: my-dc

resource_groups:
  - name: rg1
    location: eastus

    security_groups:
      - name: sg1
        rules:
          - name: rule1
            description: "awesome rule"
            priority: 101
            direction: Inbound
            access: Allow
            protocol: Tcp
            source_port_range: 100-4096
            destination_port_range: 100-4096
            source_address_prefix: VirtualNetwork
            destination_address_prefix: VirtualNetwork
        tags:
          environment: staging

    virtual_networks:
      - name: vn_test
        address_spaces:
          - 10.0.0.0/16
        subnets:
          - name: sub_test
            address_prefix: 10.0.1.0/24
            security_group: sg1

    virtual_machines:
      - name: vm_test
        count: 2
        size: Standard_A0
        image: Canonical:UbuntuServer:14.04.2-LTS:latest
        network_interfaces:
          - name: ni_test
            ip_configurations:
              - name: config_1
                subnet: vn_test:sub_test
                private_ip_address_allocation: dynamic
        authentication:
          admin_username: testadmin
          admin_password: Password1234!
        storage_os_disk:
          name: myosdiskaOne
          storage_account: safest12354
          storage_container: scfestla
          caching: ReadWrite
          create_option: FromImage
        storage_data_disk:
          name: myosdiskaTwo
          storage_account: safest12354
          storage_container: scfestla
          disk_size_gb: 1023
          create_option: empty
          lun: 0
        os_profile:
          computer_name: test

    storage_accounts:
      - name: safest12354
        account_type: Standard_LRS
        containers:
          - name: scfestla
            access_type: private

    sql_servers:
      - name: ernestserver01
        version: "12.0"
        administrator_login: mradministrator
        administrator_login_password: P4ssw0rd
        databases:
          - name: mydb

    public_ips:
      - name: my_ip

    loadbalancers:
      - name: lb1
        frontend_ip_configurations:
          - name: fic1
            subnet: sub_test
```

### Field Reference

#### Environment Options

```
name: demo
project: my-dc
```

Environment Options support the following fields:

* **name**
 * String that defines the name of the environment to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

* **project**
 * String that defines the name of the project where the environment is built.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must 50 characters maximum.


### Resource groups

Resource groups are supporting the following fields:

* **name**
 * String defining the resource group
 * This field is mandatory.
 * This field cannot be null or empty.

* **location**
 * String : the location where the resource group will be created.
 * This field is mandatory.
 * This field cannot be null or empty.
 * List of valid locations (here)[http://azure.microsoft.com/en-us/regions/]

* **security_groups**

A security_group block is described below

* **virtual_networks**

A virtual_network block is described below

* **virtual_machines**

A virtual_machine block is described below

* **storage_accounts**

A storage_account block is described below

* **sql_servers**

A sql_server block is described below

* **public_ips**

A public_ip block is described below

* **loadbalancers**

A loadbalancer block is described below

### Security Group

Security groups are supporting the following fields:

* **name**
 * String : the name of the security group
 * This field is mandatory.
 * This field cannot be null or empty.

* **rules**
	* **name**
	 * String : the name of the security group rule
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **description**
	 * String : A description for this rule. Restricted to 140 characters.
	 * This field is optional.
	* **priority**
	 * String : Specifies the priority of the rule. The value can be between 100 and 4096. The priority number must be unique for each rule in the collection. The lower the priority number, the higher the priority of the rule.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **direction**
	 * String : The direction specifies if rule will be evaluated on incoming or outgoing traffic. Possible values are “Inbound” and “Outbound”.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **access**
	 * String : Specifies whether network traffic is allowed or denied. Possible values are “Allow” and “Deny”.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **protocol**
	 * String : Network protocol this rule applies to. Can be Tcp, Udp or * to match both.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **source_port_range**
	 * String : Integer or range between 0 and 65535 or * to match any.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **destination_port_range**
	 * String : Integer or range between 0 and 65535 or * to match any.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **source_address_prefix**
	 * String : CIDR or source IP range or * to match any IP. Tags such as ‘VirtualNetwork’, ‘AzureLoadBalancer’ and ‘Internet’ can also be used.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **destination_address_prefix**
	 * String : CIDR or destination IP range or * to match any IP. Tags such as ‘VirtualNetwork’, ‘AzureLoadBalancer’ and ‘Internet’ can also be used.
	 * This field is mandatory.
	 * This field cannot be null or empty.
* **tags**
 * List String : tags to assign to the resource
 * This field is optional.

### Storage Accounts
* **name**
 * String : Specifies the name of the storage account. Changing this forces a new resource to be created. This must be unique across the entire Azure service, not just within the resource group.
 * This field is mandatory.
 * This field cannot be null or empty.
* **account_type**
 * String : Defines the type of storage account to be created. Valid options are **Standard_LRS**, **Standard_ZRS**, **Standard_GRS**, **Standard_RAGRS**, **Premium_LRS**. Changing this is sometimes valid - see the Azure documentation for more information on which types of accounts can be converted into other types.
 * This field is mandatory.
 * This field cannot be null or empty.
* **account_kind**
 * String : Defines the Kind of account. Valid options are **Storage** and **BlobStorage**. Changing this forces a new resource to be created. Defaults to **Storage**.
 * This field is optional.
* **enable_blob_encryption**
 * Boolean : Boolean flag which controls if Encryption Services are enabled for Blob storage, see here for more information.
 * This field is optional.
 * This field cannot be null or empty.
* **containers** A list of Storage Account Containers as described below
	* **name**
	 * String : Specifies the name of the storage account. Changing this forces a new resource to be created. This must be unique across the entire Azure service, not just within the resource group.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **access_type**
	 * String : The 'interface' for access the container provides. Can be either **blob**, **container** or **private**.
	 * This field is mandatory.
	 * This field cannot be null or empty.
* **tags**
 * List String : tags to assign to the resource
 * This field is optional.


### Virtual Networks
* **name**
 * String : The name of the virtual network. Changing this forces a new resource to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
* **address_spaces**
 * String : The address space that is used the virtual network. You can supply more than one address space. Changing this forces a new resource to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
* **dns_servers**
 * String : List of IP addresses of DNS servers
 * This field is optional.
* **subnets**
A list of Subnets as described below

### Subnet
* **name**
 * String : The name of the virtual network. Changing this forces a new resource to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
* **address_prefix**
 * String : The address prefix to use for the subnet.
 * This field is mandatory.
 * This field cannot be null or empty.
* **security_group**
 * String : The name of the Security Group to associate with the subnet.
 * This field is mandatory.
 * This field cannot be null or empty.

### SQL Servers
* **name**
 * String : The name of the sql server. Changing this forces a new resource to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
* **version**
 * String : The version for the new server. Valid values are: 2.0 (for v11 server) and 12.0 (for v12 server).
 * This field is mandatory.
 * This field cannot be null or empty.
* **administrator_login**
 * String : The administrator login name for the new server.
 * This field is mandatory.
 * This field cannot be null or empty.
* **administrator_login_password**
 * String : The password for the new AdministratorLogin. Please following Azures Password Policy
 * This field is mandatory.
 * This field cannot be null or empty.
* **tags**
 * List String : tags to assign to the resource
 * This field is optional.
* **databases**
A list of SQL Databases as described below
	* **name**
	 * String : The name of the database.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **create_mode**
	 * String : Specifies the type of database to create.
	 * This field is optional.
	 * Defaults to Default.
	* **source_database_id**
	 * String : The URI of the source database if create_mode value is not Default.
	 * This field is optional.
	* **restore_point_in_time**
	 * String : The point in time for the restore. Only applies if create_mode is PointInTimeRestore e.g. 2013-11-08T22:00:40Z
	 * This field is optional.
	* **edition**
	 * String : The edition of the database to be created. Applies only if create_mode is Default. Valid values are: Basic, Standard, Premium, or DataWarehouse. Please see Azure SQL Database Service Tiers.
	 * This field is optional.
	* **collation**
	 * String : The name of the collation. Applies only if create_mode is Default. Azure default is SQL_LATIN1_GENERAL_CP1_CI_AS
	 * This field is optional.
	* **max_size_bytes**
	 * String : The maximum size that the database can grow to. Applies only if create_mode is Default. Please see Azure SQL Database Service Tiers.
	 * This field is optional.
	* **requested_service_objective_id**
	 * String : Use requested_service_objective_id or requested_service_objective_name to set the performance level for the database. Valid values are: S0, S1, S2, S3, P1, P2, P4, P6, P11 and ElasticPool. Please see Azure SQL Database Service Tiers.
	 * This field is optional.
	* **requested_service_objective_name**
	 * String : Use requested_service_objective_name or requested_service_objective_id to set the performance level for the database. Please see Azure SQL Database Service Tiers.
	 * This field is optional.
	* **source_database_deletion_date**
	 * String : The deletion date time of the source database. Only applies to deleted databases where create_mode is PointInTimeRestore.
	 * This field is optional.
	* **tags**
	 * List String : tags to assign to the resource
	 * This field is optional.

* **rules**
A list of SQL Firewall Rules as described below
	* **name**
	 * String : The name of the SQL Firewall Rule.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **start_ip_address**
	 * String :  The starting IP address to allow through the firewall for this rule.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **end_ip_address**
	 * String : The ending IP address to allow through the firewall for this rule.
	 * This field is mandatory.
	 * This field cannot be null or empty.

### Load Balancers

* **name**
 * String : Specifies the name of the LoadBalancer.
 * This field is mandatory.
 * This field cannot be null or empty.
* **location**
 * String : Specifies the supported Azure location where the resource exists.
 * This field is mandatory.
 * This field cannot be null or empty.
* **backend_address_pools**
 * List of strings : With names of the backend address pools
 * This field is mandatory.
 * This field cannot be null or empty.
* **frontend_ip_configurations**
A list of Frontend IP Configurations as described below
	* **name**
	 * String : Specifies the name of the frontend ip configuration.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **subnet**
	 * String : Reference to subnet associated with the IP Configuration.
	 * This field is optional.
	* **public_ip_address_allocation**
	 * String : Defines how a public IP address is assigned. Options are Static or Dynamic.
	 * This field is optional.
	* **private_ip_address**
	 * String : Private IP Address to assign to the Load Balancer. The last one and first four IPs in any range are reserved and cannot be manually assigned.
	 * This field is optional.
	* **private_ip_address_allocation**
	 * String : Defines how a private IP address is assigned. Options are Static or Dynamic.
	 * This field is optional.
	* **rules**
	A list of Load balancer Rules as described below
		* **name**
		 * String : Specifies the name of the Rule.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **protocol**
		 * String : The transport protocol for the external endpoint. Possible values are Udp or Tcp.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **frontend_port**
		 * String :  The port for the external endpoint. Port numbers for each Rule must be unique within the Load Balancer. Possible values range between 1 and 65534, inclusive.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **backend_port**
		 * String : The port used for internal connections on the endpoint. Possible values range between 1 and 65535, inclusive.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **backend_address_pool**
		 * String :  A reference to a Backend Address Pool over which this Load Balancing Rule operates.
		 * This field is optional.
		* **probe**
		 * String : A reference to a Probe used by this Load Balancing Rule.
		 * This field is optional.
		* **floating_ip**
		 * String : Floating IP is pertinent to failover scenarios: a "floating” IP is reassigned to a secondary server in case the primary server fails. Floating IP is required for SQL AlwaysOn.
		 * This field is optional.
		* **idle_timeout**
		 * String : Specifies the timeout for the Tcp idle connection. The value can be set between 4 and 30 minutes. The default value is 4 minutes. This element is only used when the protocol is set to Tcp.
		 * This field is optional.
		* **load_distribution**
		 * String : Specifies the load balancing distribution type to be used by the Load Balancer. Possible values are: Default – The load balancer is configured to use a 5 tuple hash to map traffic to available servers. SourceIP – The load balancer is configured to use a 2 tuple hash to map traffic to available servers. SourceIPProtocol – The load balancer is configured to use a 3 tuple hash to map traffic to available servers.
		 * This field is optional.

* **probes**
A list of Load Balancer Probe as described below
	* **name**
	 * String : Specifies the name of the Probe.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **port**
	 * String : Port on which the Probe queries the backend endpoint. Possible values range from 1 to 65535, inclusive.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **protocol**
	 * String : Specifies the protocol of the end point. Possible values are Http or Tcp. If Tcp is specified, a received ACK is required for the probe to be successful. If Http is specified, a 200 OK response from the specified URI is required for the probe to be successful.
	 * This field is optional.
	* **request_path**
	 * String : The URI used for requesting health status from the backend endpoint. Required if protocol is set to Http. Otherwise, it is not allowed.
	 * This field is optional.
	* **interval**
	 * String : The interval, in seconds between probes to the backend endpoint for health status. The default value is 15, the minimum value is 5.
	 * This field is optional.
	* **max_failures**
	 * String : The number of failed probe attempts after which the backend endpoint is removed from rotation. The default value is 2. NumberOfProbes multiplied by intervalInSeconds value must be greater or equal to 10.Endpoints are returned to rotation when at least one probe is successful.
	 * This field is optional.
* **tags**
 * List String : tags to assign to the resource
 * This field is optional.


### Virtual Machines
* **name**
 * String : Specifies the name of the virtual machine resource. Changing this forces a new resource to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
* **count**
 * Integer : Number of virtual machines to be created
 * This field is optional.
 * Defaults to 1
* **size**
 * String : Specifies the size of the virtual machine.
 * This field is mandatory.
 * This field cannot be null or empty.
* **image**
 * String : String like `Canonical:UbuntuServer:14.04.2-LTS:latest` composed by:
	* `Canonical` publisher : Specifies the publisher of the image used to create the virtual machine.
	* `UbuntuServer` offer : Specifies the offer of the image used to create the virtual machine.
	* `14.04.2-LTS` sku : Specifies the SKU of the image used to create the virtual machine.
	* `latest` version : Specifies the version of the image used to create the virtual machine.
 * This field is optional.
 * Changing this forces a new resource to be created.
* **availability_set**
 * String : Availability set name
 * This field is optional.
* **authentication**
 	* **disable_password_authentication**
	 * Boolean : Specifies whether password authentication should be disabled.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **admin_username**
	 * String : Admin username to be set up on the vm
	 * This field is optional, depending on disable_password_authentication.
	* **admin_password**
	 * String : Admin password to be set up on the vm
	 * This field is optional, depending on disable_password_authentication.
	* **ssh_keys**
	 * String : A key/value list of ssh keys to be set on the vm
	 * This field is optional.
* **storage_os_disk**
	* **name**
	 * String : Name of the storage OS disk to be used
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **storage_account**
	 * String : Related storage account name
	 * This field is optional.
	* **storage_container**
	 * String : Related storage container name.
	 * This field is optional.
	* **create_option**
	 * String : Specifies how the virtual machine should be created. Possible values are attach and FromImage.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **caching**
	 * String : Specifies the caching requirements.
	 * This field is optional.
	* **image_uri**
	 * String : Specifies the image_uri in the form publisherName:offer:skus:version. image_uri can also specify the VHD uri of a custom VM image to clone. When cloning a custom disk image the os_type documented below becomes required.
	 * This field is optional.
	* **os_type**
	 * String : Specifies the operating system Type, valid values are windows, linux.
	 * This field is optional.
	* **disk_size_gb**
	 * String : Specifies the size of the data disk in gigabytes.
	 * This field is optional.
	* **managed_disk_type**
	 * String : When provided it creates an attached managed disk.
	 * This field is optional.
	 * Allowable values are Standard_LRS or Premium_LRS.
* **os_profile**
	* **computer_name**
	 * String : Specifies the name of the virtual machine.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **custom_data**
	 * String : Specifies a base-64 encoded string of custom data. The base-64 encoded string is decoded to a binary array that is saved as a file on the Virtual Machine. The maximum length of the binary array is 65535 bytes.
	 * This field is optional.
* **os_profile_windows_config**
	* **provision_vm_agent**
	 * String :
	 * This field is optional.
	* **enable_automatic_upgrades**
	 * Boolean : enables automatic upgrades
	 * This field is optional.
	* **winrm** (Optional) A collection of WinRM configuration blocks.
		* **protocol**
		 * String : Specifies the protocol of listener
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **certificate_url**
		 * String : Specifies URL of the certificate with which new Virtual Machines is provisioned.
		 * This field is optional.
	* **additional_unattend_config** (Optional)
		* **pass**
		 * String : Specifies the name of the pass that the content applies to. The only allowable value is oobeSystem.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **component**
		 * String : Specifies the name of the component to configure with the added content. The only allowable value is Microsoft-Windows-Shell-Setup.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **setting_name**
		 * String : Specifies the name of the setting to which the content applies. Possible values are: FirstLogonCommands and AutoLogon.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **content**
		 * String : Specifies the base-64 encoded XML formatted content that is added to the unattend.xml file for the specified path and component.
		 * This field is optional.

* **network_interfaces**
	* **name**
		 * String : The name of the network interface. Changing this forces a new resource to be created.
		 * This field is mandatory.
		 * This field cannot be null or empty.
	* **security_group**
		 * String : The name of the Network Security Group to associate with the network interface.
		 * This field is optional.
	* **internal_dns_name_label**
		 * String : Relative DNS name for this NIC used for internal communications between VMs in the same VNet
		 * This field is optional.
	* **enable_ip_forwarding**
		 * String : Enables IP Forwarding on the NIC. Defaults to false.
		 * This field is optional.
	* **dns_servers**
		 * String List : List of DNS servers IP addresses to use for this NIC, overrides the VNet-level server list
		 * This field is optional.
	* **ip_configurations** (Optional) Collection of ipConfigurations associated with this NIC.
		* **name**
		 * String : User-defined name of the IP.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **subnet**
		 * String : Reference to a subnet in which this NIC has been created.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **public_ip_address_allocation**
		 * String : Defines how a public IP address is assigned. Options are Static or Dynamic.
		 * This field is required.
		 * This field cannot be null or empty.
		* **private_ip_address_allocation**
		 * String : Defines how a private IP address is assigned. Options are Static or Dynamic.
		 * This field is mandatory.
		 * This field cannot be null or empty.
		* **private_ip_address**
		 * String : Reference to a Public IP Address to associate with this NIC
		 * This field is optional.
		* **load_balancer_backend_address_pools**
		 * String :
		 * This field is mandatory.
		 * This field cannot be null or empty.
	* **tags**
		 * String List : List of Load Balancer Backend Address Pool names references to which this NIC belongs
		 * This field is mandatory.
		 * This field cannot be null or empty.
* **plan** (Optional)
	* **name**
	 * String : Specifies the name of the image from the marketplace.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **publisher**
	 * String : pecifies the publisher of the image.
	 * This field is optional.
	* **product**
	 * String : Specifies the product of the image from the marketplace.
	 * This field is optional.
* **boot_diagnostics** (Optional)
	* **enabled**
	 * String : Whether to enable boot diagnostics for the virtual machine.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **storage_uri**
	 * String : Blob endpoint for the storage account to hold the virtual machine's diagnostic files. This must be the root of a storage account, and not a storage container.
	 * This field is mandatory.
	 * This field cannot be null or empty.
* **storage_data_disk** (Optional)
	* **name**
	 * String : Specifies the name of the data disk.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **storage_account**
	 * String : Related storage account name
	 * This field is optional.
	* **storage_container**
	 * String : Related storage container name.
	 * This field is optional.
	* **managed_disk_type**
	 * String : When provided it creates an attached managed disk.
	 * This field is optional.
	* **create_option**
	 * String : Specifies how the virtual machine should be created. Possible values are attach and FromImage.
	 * This field is mandatory.
	 * This field cannot be null or empty.
	* **caching**
	 * String : Specifies the caching requirements.
	 * This field is optional.
	* **image_uri**
	 * String : Specifies the image_uri in the form publisherName:offer:skus:version. image_uri can also specify the VHD uri of a custom VM image to clone. When cloning a custom disk image the os_type documented below becomes required.
	 * This field is optional.
	* **os_type**
	 * String : Specifies the operating system Type, valid values are windows, linux.
	 * This field is optional.
* **delete_os_disk_on_termination**
 * String : Flag to enable deletion of the OS Disk VHD blob when the VM is deleted, defaults to false
 * This field is optional.
 * Defaults to false
* **delete_data_disks_on_termination**
 * String : Flag to enable deletion of Storage Disk VHD blobs when the VM is deleted, defaults to false
 * This field is optional.
 * Defaults to false
* **license_type**
 * String : Specifies the Windows OS license type. The only allowable value, if supplied, is Windows_Server.
 * This field is optional, when a windows machine.
* **tags**
 * List String : tags to assign to the resource
 * This field is optional.

## VMware vCloud Director

vCloud Director (vCD) is cloud management tool from VMWare that acts as an overlay on top of vSphere, providing users with a self-service GUI and API. vCloud Director enables service providers to offer Infrastructure as a Service to their customers by providing users with direct control of virtual machine provisioning, and some aspects of networking.

With vCD most network functionality is delivered by the VMWare vShield Edge (vSE) virtual appliance. Unfortunately certain key network operations can only be performed by the service provider and are not available to users. They include:

- deploying a vSE
- connecting a vSE to an external network
- sub-allocating external IP addresses to a vSE

All other common network operations are accesible to users, however many of them are limited to users with the [Organization Administrator](https://pubs.vmware.com/vca/index.jsp?topic=%2Fcom.vmware.vcloud.api.doc_56%2FGUID-BC504F6B-3D38-4F25-AACF-ED584063754F.html) role assigned to them.

### Example YAML

```
---
name: demo
project: r3-jreid2

routers:
  - name: demo
    firewall_rules:
    - name: in_out_any
      source: internal
      from_port: any
      destination: external
      to_port: any
      protocol: any
      action: allow

    - name: out_in_80
      source: any
      from_port: any
      destination: internal
      to_port: '80'
      protocol: tcp
      action: allow

    networks:
      - name: web
        subnet: 10.1.0.0/24
        dns:
          - 8.8.8.8
          - 8.8.4.4

    nat_rules:
      - source: 195.3.186.44
        from_port: '80'
        to_port: '80'
        destination: 10.1.0.11
        protocol: tcp
        type: dnat
      - source: 10.0.0.0/8
        from_port: any
        to_port: any
        destination: 195.3.186.44
        protocol: any
        type: snat

instances:
  - name: web
    image: r3/ubuntu-1404
    cpus: 1
    memory: 1GB
    disks:
     - 10GB
    root_disk: 32GB
    count: 1
    networks:
      name: web
      start_ip: 10.1.0.11
    provisioner:
      - exec:
        - 'sudo apt-get update'
        - 'sudo apt-get install apache2 -y'
      - shell:
        - #!/bin/sh
        - if [ x$1 == x"postcustomization" ]; then
        - yum -y install httpd
        - service httpd start
        - fi
```

### Field Reference

#### Environment Options

```
name: demo
project: r3-jreid2
```

Environment Options support the following fields:

* **name**
 * String that defines the name of the environment to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

* **project**
 * String that defines the name of the project where the environment is built.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must 50 characters maximum.

#### Networking

```
routers:
  - name: demo
    firewall_rules:
    - name: in_out_any
      source: internal
      from_port: any
      destination: external
      to_port: any
      protocol: any
      action: allow

    - name: out_in_80
      source: any
      from_port: any
      destination: internal
      to_port: '80'
      protocol: tcp
      action: allow

    networks:
      - name: web
        subnet: 10.1.0.0/24
        dns:
          - 8.8.8.8
          - 8.8.4.4

    nat_rules:
      - source: 195.3.186.44
        from_port: '80'
        to_port: '80'
        destination: 10.1.0.11
        protocol: tcp
        type: dnat
      - source: 10.0.0.0/8
        from_port: any
        to_port: any
        destination: 195.3.186.44
        protocol: any
        type: snat
```

Networking supports the following fields:

* **name**
 * String that defines the name of the vSE to manage.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user & manifest.
 * The value of this field can be 50 characters maximum.

**firewall_rules**

A firewall rule controls traffic that is allowed to flow between both internal and external routers and networks.

* **name**
 * String that defines the name of the rule.
 * This field is mandatory.
 * This field can’t be null or empty.

* **source**
 * String the source of the network where this firewall acts
 * This field is mandatory.
 * Values can be: external | internal | any | named networks | CIDR

* **from_port**
 * Source port numbers
 * This field is mandatory.
 * Values can be:  an string ( valid port number, 1 - 65535 ) or any

* **destination**
 * String the destination of the network where this firewall acts
 * This field is mandatory.
 * Values can be:  external | internal | any | named networks | CIDR

* **to_port**
 * Destination port numbers
 * This field is mandatory.
 * Values can be:  an string ( valid port number, 1 - 65535 ) or any

* **protocol**
 * String with the protocol we are firewalling
 * This field is mandatory.
 * Values can be: tcp | udp | icmp | any | tcp & udp

**networks**

A network is a virtual network that attaches to a router (vSE). This is what allows us to network/isolate VM’s together.

* **name**
 * String that defines the name of the network to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field must be 50 characters maximum.

* **subnet**
 * String that defines the subnet of the network.
 * It must follow CIDR notation as described at: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique for this user & all the networks on the manifest.

* **dns**
 * Array of IPs to use as dns servers on this environment
 * It must follow CIDR notation as described at: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 * This field is optional.
 * This field can be empty and it will default to ["8.8.8.8", "8.8.4.4"].

**nat_rules**

A nat rule is something that converts translates a request on a port from one IP to another on a different network.

* **from_port**
 * Source port numbers
 * Values can be:  an string ( valid port number, 1 - 65535 ) or any

* **destination**
 * String the destination of the IPv4 Address to translate to
 * Values can be: IPv4

* **to_port**
 * Destination port numbers
 * Values can be:  an string ( valid port number, 1 - 65535 ) or any

* **source**
 * String that defines the IP from the provider-specified sub-allocated list of IPs of the router.
 * It must follow CIDR notation as described at: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing

* **protocol**
 * String that defines the protocols to be translated.
 * Supported values are TCP, UDP, ICMP, and ANY.

* **type**
 * String that defines the type of translation to configure.
 * `dnat` is for in-bound translations.
 * `snat` is for out-bound translations.

#### Instances

```
instances:
  - name: web
    image: r3/ubuntu-1404
    cpus: 1
    memory: 1GB
    disks:
     - 10GB
    root_disk: 32GB
    count: 1
    networks:
      name: web
      start_ip: 10.1.0.11
    provisioner:
      - exec:
        - 'sudo apt-get update'
        - 'sudo apt-get install apache2 -y'
      - shell:
        - #!/bin/sh
        - if [ x$1 == x"postcustomization" ]; then
        - yum -y install httpd
        - service httpd start
        - fi
```

Instances support the following fields:

* **name**
 * String that defines the name of the instance to build.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be unique by user & manifest.
 * The value of this field must be 50 characters maximum.

* **image**
 * String that defines the name of an image to install on the VM.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field is composed in two parts one for the “catalog” and another one for the “image” separated by a slash “/”
 * Both “catalog” and “image” are strings and can’t be null.

* **cpus**
 * Integer that defines how many CPUs will have the VM.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be greater or equal than 1

* **memory**
 * String that defines the name of a load balancer to add to this environment.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must follow the Binary Prefix format: https://en.wikipedia.org/wiki/Binary_prefix#Computer_memory
 * The possible binary prefixes are MB, GB, TB, PB, YB

* **disks:**
 * A list of sizes of hard disks that belongs to the VM
 * This field can be an empty list
 * This field is a list of strings
 * Each element of the string must follow the Binary Prefix format: https://en.wikipedia.org/wiki/Binary_prefix#Hard_disk_drives
 * The possible binary prefixes are MB, GB, TB, PB, YB

* **root_disk**
 * String that defines the root disk size of the template the instance is based on.
 * This field is optional.
 * This field cannot be null or empty.
 * The value must be larger than the default root disk size of the template.
 * Each element of the string must follow the Binary Prefix format: https://en.wikipedia.org/wiki/Binary_prefix#Hard_disk_drives
 * The possible binary prefixes are MB, GB, TB, PB, YB

* **count**
 * Integer that defines the number of VM to be created.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be greater or equal that 1

**networks**

Networks is a map with two fields

* **name**
 * String that defines the name of a network to attach this instance.
 * This field is mandatory.
 * This field cannot be null or empty.
 * The value of this field should exist on the networks section previously specified.

* **start_ip**
 * String that defines the starting IP to allocate the VMs to be built.
 * This field is mandatory.
 * This field cannot be null or empty.
 * This field must be a valid IP.
 * This IP should belong to the network already defined on the instance.

**provisioner**

Provisioner defines the available provisioner mechanisms.

* **exec**
 * Array of strings, any command characters must be escaped with backslashes: (RFC7159, Section 7)
 * Each command in the exec array is concatenated together and delimited by a semicolon.
 * This field is optional
 * If specified as a provisioner type, this array field must contain at least one element
 * The specified commands will be executed by the SALT master.

* **shell**
 * Array of strings, any command characters must be escaped with backslashes: (RFC7159, Section 7)
 * Each command in the exec array is concatenated together and delimited by a semicolon.
 * This field is optional
 * If specified as a provisioner type, this array field must contain at least one element
 * The specified commands will be executed by the vCloud guest customization script.