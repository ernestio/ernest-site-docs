---
date: 2016-03-09T20:08:11+01:00
title: Documentation
weight: 20
---

## Using the CLI
### Connecting to Ernest

```sh
#The first step is to specify the IP address of our Ernest Server:

$ ernest target https://<Ernest Server IP>
Target set

#Now we should login to Ernest with the built-in admin account:

ernest login
Username: admin
Password: *******

#The default admin password is ‘w4rmR3d’
#and change the admin password:

$ ernest user change-password
#You're about to change your password, please respond the questions below:
Current password: <old admin password>
New password: <new admin password>
Confirm new password: <new admin password>
Your password has been changed

#Now we can create a user, a group, and add the user to the group:

$ ernest user create <user> <password>
User <user> successfully created

$ ernest group create <group>
Group <group> successfully created, you can add users with 'ernest group add-user username <group>'

$ ernest group add-user <user> <group>
User <user> is now assigned to group <group>

#Now we can login with our new user:

$ ernest logout
Bye.

$ ernest login
Username: <user>
Password: <password>
Welcome back <user>

#We can confirm our Ernest Server IP and user:

$ ernest info
Current target : https://<Ernest Server IP>
Current user : <user>
```

### Datacenters
Ernest uses the concept of a datacenter to specify an infrastructure service provider, the provider API endpoint, credentials to access the provider API, and information specific to certain provider types. An example of creating a datacenter in Ernest for Amazon Web Services is:
```sh
$ ernest datacenter create aws --region region --secret_access_key secret_access_key --access_key_id access_key_id <dc name>
```
### Services
#### What is a Service?
With Ernest a Service is a collection of networking, virtual machines, and configuration options that collectively define an environment, and also the datacenter where the environment should be built. A service is defined in YAML format, and you can see examples of this in the provider-specific documentation.

#### How to create/modify Services
Once we have created a datacenter and defined our service in YAML the process of building the service is:
```sh
$ ernest service apply <yaml file>
```
Once the service is created we can modify it by changing the YAML and then re-applying it. Ernest will calculate the differences and apply only those changes necessary to transition the service from the current state to the requested state.

Finally you can also destroy services if needed:
```sh
$ ernest service destroy <service name>
```
#### View Service State and History
Ernest stores the state of each service, as well as all historical states. You can list the available service with:
```sh
$ ernest service list

#For a given service you can see the history of all builds for that service:

$ ernest service history <service name>

#We can view the YAML for a specific build by:

$ ernest service definition <service name> --build <build id>

#We can view provider-generated information related to our service:

$ ernest service info <service name>
```

## CLI Command Reference
### ernest target

```sh

NAME:
   ernest target - Configure Ernest target instance.

USAGE:
   ernest target <ernest_url>

DESCRIPTION:
   Sets up ernest instance target.

   Example:
    $ ernest target https://myernest.com
```
### ernest info

```sh
NAME:
   ernest info - Display system-wide information.

USAGE:
   ernest info  

DESCRIPTION:
   Displays ernest instance information.

   Example:
    $ ernest info
```

### ernest login

```sh
NAME:
   ernest login - Login with your Ernest credentials.

USAGE:
   ernest login [command options]  

DESCRIPTION:
   Logs an user into Ernest instance.

   Example:
    $ ernest login

   It can also be used without asking the username and password.

   Example:
    $ ernest login --user <user> --password <password>


OPTIONS:
   --user value      User credentials
   --password value  Password credentials
```

### ernest logout

```sh
NAME:
   ernest logout - Clear local authentication credentials.

USAGE:
   ernest logout  

DESCRIPTION:
   Logs out an user from Ernest instance.

   Example:
    $ ernest logout
```

### ernest user list

```sh
NAME:
   ernest user list - List available users.

USAGE:
   ernest user list  

DESCRIPTION:
   List available users.

   Example:
    $ ernest user list
```

### ernest user create

```sh
NAME:
   ernest user create - Create a new user.

USAGE:
   ernest user create [command options] <username> <password>

DESCRIPTION:
   Create a new user on the targeted instance of Ernest.

   Example:
    $ ernest user create <username> <password>

   You can also add an email to the user with the flag --email

   Example:
    $ ernest user create --email username@example.com <username> <password>


OPTIONS:
   --email value  Email for the user
```

### ernest user change-password

```sh
NAME:
   ernest user change-password - Change password of available users.

USAGE:
   ernest user change-password [command options] [arguments...]

DESCRIPTION:
   Change password of available users.

   Example:
    $ ernest user change-password

    or changing a change-password by being admin:

    $ ernest user change-password --user <username> --current-password <current-password> --password <new-password>


OPTIONS:
   --user value              The username of the user to change password
   --password value          The new user password
   --current-password value  The current user password
```

### ernest user disable

```sh
NAME:
   ernest user disable - Disable available users.

USAGE:
   ernest user disable <username>

DESCRIPTION:
   Disable available users.

  Example:
   $ ernest user disable <user-name>
```

### ernest group delete

```sh
NAME:
   ernest group delete - Deletes a group.

USAGE:
   ernest group delete  

DESCRIPTION:
   Deletes a group by name

    Example:
      $ ernest group delete <name>
```

### ernest group list

```sh
NAME:
   ernest group list - List available groups.

USAGE:
   ernest group list  

DESCRIPTION:
   List available groups.

   Example:
    $ ernest group list
```

### ernest group create

```sh
NAME:
   ernest group create - Create a group.

USAGE:
   ernest group create  

DESCRIPTION:
   Create a group.

   Example:
    $ ernest group create <name>
```

### ernest group add-user

```sh
NAME:
   ernest group add-user - Adds a user to a group.

USAGE:
   ernest group add-user  

DESCRIPTION:
   Adds a user to a group.

    Example:
      $ ernest group add-user <user-name> <group-name>
```

### ernest group remove-user

```sh
NAME:
   ernest group remove-user - Removes an user from a group.

USAGE:
   ernest group remove-user  

DESCRIPTION:
   Removes an user from a group.

    Example:
      $ ernest group remove-user <username> <group name>
```

### ernest group add-datacenter

```sh
NAME:
   ernest group add-datacenter - Adds a datacenter to a group.

USAGE:
   ernest group add-datacenter  

DESCRIPTION:
   Adds a datacenter to a group.

    Example:
      $ ernest group add-datacenter <datacenter-name> <group-name>
```

### ernest group remove-datacenter

```sh
NAME:
   ernest group remove-datacenter - Removes a datacenter from a group.

USAGE:
   ernest group remove-datacenter  

DESCRIPTION:
   Removes an datacenter from a group.

    Example:
      $ ernest group remove-datacenter <datacenter name> <group name>
```

### ernest datacenter list

```sh
NAME:
   ernest datacenter list - List available datacenters.

USAGE:
   ernest datacenter list  

DESCRIPTION:
   List available datacenters.

   Example:
    $ ernest datacenter list
```

### ernest datacenter create vcloud

```sh
NAME:
   ernest datacenter create vcloud - Create a new vcloud datacenter.

USAGE:
   ernest datacenter create vcloud [command options] <datacenter-name>

DESCRIPTION:
   Create a new vcloud datacenter on the targeted instance of Ernest.

   Example:
    $ ernest datacenter create vcloud --user username --password xxxx --org MY-ORG-NAME --vse-url http://vse.url --vcloud-url https://myernest.com --public-network MY-PUBLIC-NETWORK mydatacenter

   Template example:
    $ ernest datacenter create vcloud --template mydatacenter.yml mydatacenter
    Where mydatacenter.yaml will look like:
      ---
      fake: true
      org: org
      password: pwd
      public-network: MY-NETWORK
      user: bla
      vcloud-url: "http://ss.com"
      vse-url: "http://ss.com"



OPTIONS:
   --user value            Your VCloud valid user name
   --password value        Your VCloud valid password
   --org value             Your vCloud Organization
   --vse-url value         VSE URL
   --vcloud-url value      VCloud URL
   --public-network value  Public Network
   --template value        Datacenter template
   --fake                  Fake datacenter
```

### ernest datacenter create aws

```sh
NAME:
   ernest datacenter create aws - Create a new aws datacenter.

USAGE:
   ernest datacenter create aws [command options] <datacenter-name>

DESCRIPTION:
   Create a new AWS datacenter on the targeted instance of Ernest.

  Example:
   $ ernest datacenter create aws --region region --secret_access_key secret_access_key --access_key_id access_key_id my_datacenter

   Template example:
    $ ernest datacenter create aws --template mydatacenter.yml mydatacenter
    Where mydatacenter.yaml will look like:
      ---
      fake: true
      secret_access_key: secret_access_key
      access_key_id: access_key_id
      region: region


OPTIONS:
   --region value    Datacenter region
   --secret_access_key value     AWS secret_access_key
   --access_key_id value    AWS access_key_id
   --template value  Datacenter template
   --fake            Fake datacenter
```

### ernest datacenter update vcloud

```sh
NAME:
   ernest datacenter update vcloud - Updates the specified VCloud datacenter.

USAGE:
   ernest datacenter update vcloud [command options] <datacenter-name>

DESCRIPTION:
   Updates the specified VCloud datacenter.

   Example:
    $ ernest datacenter update vcloud --user <me> --org <org> --password <secret> my_datacenter


OPTIONS:
   --user value      Your VCloud valid user name
   --password value  Your VCloud valid password
   --org value       Your vCloud Organization
```

### ernest datacenter update aws

```sh
NAME:
   ernest datacenter update aws - Updates the specified AWS datacenter.

USAGE:
   ernest datacenter update aws [command options] <datacenter-name>

DESCRIPTION:
   Updates the specified AWS datacenter.

   Example:
    $ ernest datacenter update aws --secret_access_key <secret_access_key> --access_key_id <access_key_id> my_datacenter


OPTIONS:
   --secret_access_key value   Your AWS valid secret_access_key
   --access_key_id value  Your AWS valid access_key_id
```

### ernest datacenter delete

```sh
NAME:
   ernest datacenter delete - Deletes the specified datacenter.

USAGE:
   ernest datacenter delete <datacenter-name>

DESCRIPTION:
   Deletes the name specified datacenter.

   Example:
    $ ernest datacenter delete my_datacenter
```

### ernest service list

```sh
NAME:
   ernest service list - List available services.

USAGE:
   ernest service list  

DESCRIPTION:
   List available services and shows its most relevant information.

   Example:
    $ ernest service list
```

### ernest service apply

```sh
NAME:
   ernest service apply - Builds or changes infrastructure.

USAGE:
   ernest service apply <file.yml>

DESCRIPTION:
   Sends a service YAML description file to Ernest to be executed.
   You must be logged in to execute this command.

   If the file is not provided, ernest.yml will be used by default.

   Example:
    $ ernest apply myservice.yml
```

### ernest service destroy

```sh
NAME:
   ernest service destroy - Destroy a service.

USAGE:
   ernest service destroy [command options] <service_name>

DESCRIPTION:
   Destroys a service by its name.

   Example:
    $ ernest destroy myservice


OPTIONS:
   --force, -f  Force destroy command without asking for permission.
```

### ernest service history

```sh
NAME:
   ernest service history - Shows the history of a service, a list of builds

USAGE:
   ernest service history <service_name>

DESCRIPTION:
   Shows the history of a service, a list of builds and its status and basic information.

   Example:
    $ ernest history myservice
```

### ernest service reset

```sh
NAME:
   ernest service reset - Reset an in progress service.

USAGE:
   ernest service reset <service_name>

DESCRIPTION:
   Reseting a service creation may cause problems, please make sure you know what are you doing.

   Example:
    $ ernest reset myservice
```

### ernest service definition

```sh
NAME:
   ernest service definition - Show the current definition of a service by its name

USAGE:
   ernest service definition [command options] <service_name>

DESCRIPTION:
   Show the current definition of a service by its name getting the definition about the build.

   Example:
    $ ernest service definition myservice


OPTIONS:
   --build value  Build ID
```

### ernest service info

```sh
NAME:
   ernest service info - $ ernest service info <my_service> --build <specific build>

USAGE:
   ernest service info [command options] <service_name>

DESCRIPTION:
   Will show detailed information of the last build of a specified service.
  In case you specify --build option you will be able to output the detailed information of specific build of a service.

   Examples:
    $ ernest service definition myservice
    $ ernest service definition myservice --build build1

OPTIONS:
   --build value  Build ID
```

### ernest service monitor

```sh
NAME:
   ernest service monitor - Monitor a service.

USAGE:
   ernest service monitor <service_name>

DESCRIPTION:
   Monitors a service while it is being built by its service name.

   Example:
    $ ernest monitor my_service
```

### ernest preferences logger list

```sh
NAME:
   ernest preferences logger list - Lists active loggers.

USAGE:
   ernest preferences logger list  

DESCRIPTION:
   List active loggers.

   Example:
    $ ernest preferences logger list
```    
### ernest preferences logger add

```sh
NAME:
   ernest preferences logger add - Creates / updates a logger based on its type.

USAGE:
   ernest preferences logger add [command options]  

DESCRIPTION:
   Creates / updates a logger based on its types.

   Example:
    $ ernest preferences logger add basic --logfile /tmp/ernest.log
    $ ernest preferences logger add logstash --hostname 10.50.1.1 --port 5000 --timeout 50000
    $ ernest preferences logger add rollbar --secret_access_key MY_ROLLBAR_TOKEN


OPTIONS:
   --logfile value   Specify the path for the loging file
   --secret_access_key value     Rollbar token
   --env value       Rollbar environment
   --hostname value  Logstash hostname
   --port value      Logstash port (default: 0)
   --timeout value   Logstash timeout (default: 0)
```

### ernest preferences logger delete

```sh
NAME:
   ernest preferences logger delete - Deletes a logger based on its type.

USAGE:
   ernest preferences logger delete  

DESCRIPTION:
   Deletes a logger based on its types.

   Example:
    $ ernest preferences logger delete basic
```

## Bootstrapping with Salt

Once we have created our servers we may want to install and configure software on them (bootstrap them). We can use Ernest to bootstrap the servers for us.

Ernest uses [Salt](https://saltstack.com/) for bootstrapping. When you enable Salt bootstrapping in an environment Ernest deploys a small footprint Salt Master into the environment. Ernest then instructs the Salt Master to install the Salt Minion on each server, and have each server register themselves with the Salt Master.

Once the servers have been registered we can securely communicate with them from Ernest, and execute remote commands on them. These commands can be included in the YAML that defines the environment. You can find an example of bootstrapping [here].

If you wish to use Salt bootstrapping the supported OS versions and OS-specific requirements are shown in the following table:

<table>
<thead>
<tr>
<th align="center">OS</th>
<th align="center">Windows</th>
<th align="center">CentOS</th>
<th align="center">Ubuntu</th>
</tr>
</thead>

<tbody>
<tr>
<td align="center">Supported Versions</td>
<td align="center">2012r2 and newer</td>
<td align="center">6.5 and newer</td>
<td align="center">14.04 and newer</td>
</tr>

<tr>
<td align="center">Installation Method</td>
<td align="center">WinRM</td>
<td align="center">SSH</td>
<td align="center">SSH</td>
</tr>

<tr>
<td align="center">Username</td>
<td align="center">ernest</td>
<td align="center">ernest</td>
<td align="center">ernest</td>
</tr>

<tr>
<td align="center">Password</td>
<td align="center">b00tStr4pp3rR</td>
<td align="center">b00tStr4pp3rR</td>
<td align="center">b00tStr4pp3rR</td>
</tr>

<tr>
<td align="center">Privilege Level</td>
<td align="center">local administrator</td>
<td align="center">passwordless sudo</td>
<td align="center">passwordless sudo</td>
</tr>

<tr>
<td align="center">Notes</td>
<td align="center">-</td>
<td align="center">disable sudo tty req</td>
<td align="center">disable sudo tty req</td>
</tr>
</tbody>
</table>


You can download the Salt Master image from the [Download page](downloads/).   

## Amazon Web Services
### Introduction
Amazon Web Services (AWS) is a secure cloud services platform, offering compute power, database storage, content delivery and other functionality to help businesses scale and grow.

### Supported Services
The following AWS services are supported by Ernest:

- EC2
- VPC
- ELB
- S3
- Route53
- RDS
- EBS

## AWS Examples
### Setup
Before we get started we will need the following information:

- AWS access key
- AWS secret key
- AWS VPC ID
- Ernest IP address
- Ernest username/password

```sh
#The first step is to set the IP address of Ernest:

$ ernest target https://10.50.1.11
Target set

#Next we login to Ernest:

$ ernest login
Username: user1
Password: ******
Welcome back user1

#Once we have logged in to Ernest we can setup the AWS datacenter and credentials that Ernest will use to create our infrastructure:

$ ernest datacenter create aws --region eu-west-1 --secret_access_key XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX --access_key_id YYYYYYYYYYYYYYYYYYYY my-dc
Datacenter 'my-dc' successfully created

#Now that we have our datacenter created in Ernest we can start building stuff.

```

### Creating a Service
We will create a simple environment with one Ubuntu server, a public IP assigned to it, and the ability to ssh to that server from our IP (52.211.19.211).


Our environment is defined in the following YAML:

```sh
---
name: demo
datacenter: my-dc
vpc_id: vpc-abcdef01

networks:
  - name: public
    subnet: 10.0.10.0/24
    public: true

security_groups:
  - name: public-sg
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
```      

Lets apply our definition:

```sh
$ ernest service apply demo.yml
#Environment creation requested
#Ernest will show you all output from your requested service creation
#You can cancel at any moment with Ctrl+C, even the service is still #being created, you won't have any output
Starting environment creation

Creating networks:
 - my-dc-demo-public
   IP     : 10.0.10.0/24
   AWS ID : subnet-defabc01
   Status : completed
Networks successfully created

Creating firewalls:
 - my-dc-demo-public-sg
   Status    : completed
Firewalls created

Creating instances:
 - my-dc-demo-public-1
   IP        : 10.0.10.11
   AWS ID    : i-abcdef01abcdef011
   Status    : completed
Instances successfully created

Updating instances:
 - my-dc-demo-public-1
   IP        : 10.0.10.11
   PUBLIC IP : 52.210.179.96
   AWS ID    : i-abcdef01abcdef011
   Status    : completed
Instances successfully updated
SUCCESS: rules successfully applied
Your environment endpoint is:
```

We can list the services we have built:

```sh
$ ernest service list
NAME  UPDATED         STATUS  ENDPOINT
demo  2016-09-12 15:57:46.195942 +0000 UTC  done
```

We can see detailed provider-generated information related to our service:

```sh
$ ernest service info demo
Name : demo
VPC : vpc-abcdef01

ELBs (empty)

Networks:
+---------------------------+-----------------+
|           NAME            |       ID        |
+---------------------------+-----------------+
| my-dc-demo-public         | subnet-defabc01 |
+---------------------------+-----------------+

Instances:
+-----------------------------+---------------------+---------------+------------+
|            NAME             |         ID          |   PUBLIC IP   | PRIVATE IP |
+-----------------------------+---------------------+---------------+------------+
| my-dc-demo-public-1         | i-abcdef01abcdef011 | 52.210.179.96 | 10.0.10.11 |
+-----------------------------+---------------------+---------------+------------+

NAT gateways (empty)

Security groups:
+------------------------------+-------------+
|             NAME             |  GROUP ID   |
+------------------------------+-------------+
| my-dc-demo-public-sg         | sg-01234567 |
+------------------------------+-------------+
```

We can view the history of applies for our service:

```sh
$ ernest service history demo
NAME  BUILD ID                UPDATED         STATUS
demo  89389b76-cc25-4add-55e5-b7647217b4b1-abf663d6c173d4af98e3ff20bb7e8dde 2016-09-12 15:57:46.195942 +0000 UTC  done
```

For our service we can show the definition applied for a given Build ID:

```sh
$ ernest service definition demo --build 89389b76-cc25-4add-55e5-b7647217b4b1-abf663d6c173d4af98e3ff20bb7e8dde
---
name: demo
datacenter: my-dc
vpc_id: vpc-abcdef01

networks:
  - name: public
    subnet: 10.0.10.0/24
    public: true

security_groups:
  - name: public-sg
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
```

### Modifying a Service

Lets modify the service we create above. We will add a private network with a server on it and a NAT gateway attached to it.


Our modified YAML file is:

```sh
---
name: demo
datacenter: my-dc
vpc_id: vpc-abcdef01

networks:
  - name: public
    subnet: 10.0.10.0/24
    public: true
  - name: private
    subnet: 10.0.11.0/24
    public: false
    nat_gateway: private-nat

nat_gateways:
  - name: private-nat
    public_network: public

security_groups:
  - name: public-sg
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

When we apply this YAML we can see Ernest make the necessary changes:

```sh
$ ernest service apply demo.yml
#Environment creation requested
#Ernest will show you all output from your requested service creation
#You can cancel at any moment with Ctrl+C, even the service is still #being created, you won't have any output
Starting environment creation

Creating networks:
 - my-dc-demo-private
   IP     : 10.0.11.0/24
   AWS ID : subnet-defabc02
   Status : completed
Networks successfully created

Creating firewalls:
 - my-dc-demo-private-sg
   Status    : completed
Firewalls created

Creating instances:
 - my-dc-demo-private-1
   IP        : 10.0.11.11
   AWS ID    : i-abcdef01abcdef012
   Status    : completed
Instances successfully created

Updating instances:
 - my-dc-demo-private-1
   IP        : 10.0.11.11
   PUBLIC IP :
   AWS ID    : i-abcdef01abcdef012
   Status    : completed
Instances successfully updated

Creating nats:
 - my-dc-demo-private-nat
   Status    : completed
Nats created
SUCCESS: rules successfully applied
Your environment endpoint is:
```

We can see our service in the list, with the most recent update time:

```sh
$ ernest service list
NAME  UPDATED         STATUS  ENDPOINT
demo  2016-09-12 16:03:40.195942 +0000 UTC  done
```

The service info has also updated with the new information:

```sh
$ ernest service info demo
Name : demo
VPC : vpc-abcdef01

ELBs (empty)

Networks:
+---------------------------+-----------------+
|           NAME            |       ID        |
+---------------------------+-----------------+
| my-dc-demo-public         | subnet-defabc01 |
| my-dc-demo-private        | subnet-defabc02 |
+---------------------------+-----------------+

Instances:
+-----------------------------+---------------------+---------------+------------+
|            NAME             |         ID          |   PUBLIC IP   | PRIVATE IP |
+-----------------------------+---------------------+---------------+------------+
| my-dc-demo-public-1         | i-abcdef01abcdef011 | 52.210.179.96 | 10.0.10.11 |
| my-dc-demo-private-1        | i-abcdef01abcdef012 |               | 10.0.11.11 |
+-----------------------------+---------------------+---------------+------------+

NAT gateways:
+-------------------------------+-----------------------+
|             NAME              |       GROUP ID        |
+-------------------------------+-----------------------+
| my-dc-demo-private-nat        | nat-abcdef01abcdef013 |
+-------------------------------+-----------------------+

Security groups:
+------------------------------+-------------+
|             NAME             |  GROUP ID   |
+------------------------------+-------------+
| my-dc-demo-public-sg         | sg-01234567 |
| my-dc-demo-private-sg        | sg-89012345 |
+------------------------------+-------------+
```
The history shows both of the applies we have done for this service:

``sh
$ ernest service history demo
NAME  BUILD ID                UPDATED         STATUS
demo  72c1306c-c6ee-4d9b-4230-617d0e969ea9-abf663d6c173d4af98e3ff20bb7e8dde 2016-09-12 16:03:40.235144 +0000 UTC  done
demo  89389b76-cc25-4add-55e5-b7647217b4b1-abf663d6c173d4af98e3ff20bb7e8dde 2016-09-12 15:57:46.195942 +0000 UTC  done
```
The definition for the most recent build can be displayed:

```sh
$ ernest service definition demo --build 72c1306c-c6ee-4d9b-4230-617d0e969ea9-abf663d6c173d4af98e3ff20bb7e8dde
---
name: demo
datacenter: my-dc
vpc_id: vpc-abcdef01

networks:
  - name: public
    subnet: 10.0.10.0/24
    public: true
  - name: private
    subnet: 10.0.11.0/24
    public: false
    nat_gateway: private-nat

nat_gateways:
  - name: private-nat
    public_network: public

security_groups:
  - name: public-sg
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
### Clean-up
After we have finished with our service we can remove it:

```sh
$ ernest service destroy demo
Are you sure? Please type yes or no and then press enter: yes

Deleting nats:
 - my-dc-demo-private-nat
   Status    : completed
Nats deleted

Deleting instances:
 - my-dc-demo-public-1
   IP        : 10.0.10.11
   PUBLIC IP : 52.210.179.96
   AWS ID    : i-abcdef01abcdef011
   Status    : completed
 - my-dc-demo-private-1
   IP        : 10.0.11.11
   PUBLIC IP :
   AWS ID    : i-abcdef01abcdef012
   Status    : completed
Instances deleted

Deleting networks:
 - my-dc-demo-public
   IP     : 10.0.10.0/24
   AWS ID : subnet-defabc01
   Status : completed
 - my-dc-demo-private
   IP     : 10.0.11.0/24
   AWS ID : subnet-defabc02
   Status : completed
Networks deleted

Deleting firewalls:
 - my-dc-demo-public-sg
   Status    : completed
 - my-dc-demo-private-sg
   Status    : completed
Firewalls deleted
SUCCESS: your environment has been successfully deleted
```

## AWS YAML Reference

<p>Environments built and managed with Ernest are defined in YAML format.</p>

### Example

<pre><code>---
name: demo
datacenter: my-dc
vpc_id: vpc-abcdef01
vpc_subnet: 10.0.0.0/16

networks:
  - name: web
    subnet: 10.0.10.0/24
    public: true
  - name: db
    subnet: 10.0.11.0/24
    public: false
    nat_gateway: db-nat
  - name: db-standby
    subnet: 10.0.12.0/24
    public: false
    nat_gateway: db-nat

nat_gateways:
  - name: db-nat
    public_network: web

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
      - ip: 52.211.19.211/32
        protocol: tcp
        from_port: '22'
        to_port: '22'
  - name: db-sg
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
</code></pre>


<h3 id="service-options">Service Options</h3>

<pre><code>name: demo
datacenter: my-dc
vpc_id: vpc-abcdef01
vpc_subnet: 10.0.0.0/16

</code></pre>

<p>Service Options support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the service to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>datacenter</strong></p>

<ul>
<li>String that defines the name of the datacenter where the service is built.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must 50 characters maximum.</li>
</ul></li>

<li><p><strong>vpc_id</strong></p>

<ul>
<li>String that defines the ID of the existing VPC the service will use.</li>
<li>This field is mandatory, unless <strong>vpc_subnet</strong> is present.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>vpc_subnet</strong></p>

<ul>
<li>String that defines the subnet of the VPC that will be created for the service to use.</li>
<li>This field is mandatory, unless <strong>vpc_id</strong> is present.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must 50 characters maximum.</li>
</ul></li>
</ul>

<h3 id="networking">Networking</h3>

<pre><code>networks:
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

</code></pre>

<p>Networking supports the following fields:</p>

<p><strong>networks</strong></p>

<p>A network is a subnet that our instances can connect to. This is what allows us to network/isolate VM’s together.</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the subnet to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>subnet</strong></p>

<ul>
<li>String that defines the IP address range of the subnet.</li>
<li>It must follow CIDR notation as described at: <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing</a></li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique for this user &amp; all the networks on the manifest.</li>
</ul></li>

<li><p><strong>public</strong></p>

<ul>
<li>String that defines if the subnet will be public or private.</li>
<li>This field is optional.</li>
<li>Values can be: “true“ or “false“.</li>
<li>This field will default to “false“.</li>
</ul></li>

<li><p><strong>nat_gateway</strong></p>

<ul>
<li>String that defines the name of the NAT gateway to use for this subnet.</li>
<li>This field is optional.</li>
<li>This field can be empty.</li>
</ul></li>
</ul>

<p><strong>nat_gateways</strong></p>

<p>NAT gateways enable instances in a private subnet to connect to the Internet or other AWS services, but prevent the Internet from initiating a connection with those instances.</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the NAT gateway to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>public_network</strong></p>

<ul>
<li>String that defines the name of the public network the NAT gateway will reside.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>
</ul>

<h3 id="security-groups">Security Groups</h3>

<pre><code>security_groups:
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

</code></pre>

<p>Security Groups support the following fields:</p>

<ul>
<li><strong>name</strong>

<ul>
<li>String that defines the name of the security group to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>
</ul>

<p><strong>egress</strong></p>

<p>Security group egress rules.</p>

<ul>
<li><p><strong>ip</strong></p>

<ul>
<li>String that defines the IP address range for this rule.</li>
<li>It must follow CIDR notation as described at: <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing</a></li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>protocol</strong></p>

<ul>
<li>String that defines the protocol for this rule.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>Values can be: tcp | udp | icmp | any</li>
</ul></li>

<li><p><strong>from_port</strong></p>

<ul>
<li>Starting port-range number.</li>
<li>This field is mandatory.</li>
<li>Values can be: 0 - 65535.</li>
</ul></li>

<li><p><strong>to_port</strong></p>

<ul>
<li>Ending port-range number.</li>
<li>This field is mandatory.</li>
<li>Values can be: 0 - 65535.</li>
</ul></li>
</ul>

<p><strong>ingress</strong></p>

<p>Security group ingress rules.</p>

<ul>
<li><p><strong>ip</strong></p>

<ul>
<li>String that defines the IP address range for this rule.</li>
<li>It must follow CIDR notation as described at: <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing</a></li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>protocol</strong></p>

<ul>
<li>String that defines the protocol for this rule.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>Values can be: tcp | udp | icmp | any</li>
</ul></li>

<li><p><strong>from_port</strong></p>

<ul>
<li>Starting port-range number.</li>
<li>This field is mandatory.</li>
<li>Values can be: 0 - 65535.</li>
</ul></li>

<li><p><strong>to_port</strong></p>

<ul>
<li>Ending port-range number.</li>
<li>This field is mandatory.</li>
<li>Values can be: 0 - 65535.</li>
</ul></li>
</ul>

<h3 id="instances">Instances</h3>

<pre><code>instances:
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

</code></pre>

<p>Instances support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the instance to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>elastic_ip</strong></p>

<ul>
<li>String that defines if the instance will be assigned an elastic IP.</li>
<li>This field is optional.</li>
<li>Values can be: “true“ or “false“.</li>
<li>This field will default to “false“.</li>
</ul></li>

<li><p><strong>type</strong></p>

<ul>
<li>String that defines the instance size.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be a valid AWS EC2 size.</li>
</ul></li>

<li><p><strong>image</strong></p>

<ul>
<li>String that defines the name of an AMI to install.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>network:</strong></p>

<ul>
<li>String that defines the name of a network to attach this instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field should exist on the networks section previously specified.</li>
</ul></li>

<li><p><strong>start_ip:</strong></p>

<ul>
<li>String that defines the starting IP to allocate the VMs to be built.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be a valid IP.</li>
<li>This IP should belong to the network already defined on the instance.</li>
</ul></li>

<li><p><strong>count</strong></p>

<ul>
<li>Integer that defines the number of instances to be created.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be greater or equal that 1</li>
</ul></li>

<li><p><strong>key_pair</strong></p>

<ul>
<li>String that specifies the key pair to be used to login to the instances.</li>
<li>This field is optional.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>security_groups</strong></p>

<ul>
<li>Array that contains security groups that will be applied to the instances.</li>
<li>This field is optional.</li>
<li>This field can be empty.</li>
</ul></li>
</ul>

<p><strong>volumes</strong></p>

<p>EBS volumes to be attached to the vm. Please note that the count of the EBS volume must be greater or equal to the amount of instances specified.</p>

<ul>
<li><strong>volume</strong></li>
<li>String that references the name of an EBS volume specidied on the yaml</li>
<li>This field is mandatory</li>

<li><p>This field cannot be empty or null</p></li>

<li><p><strong>device</strong></p></li>

<li><p>String that defines what device name is exposed to the instance</p></li>

<li><p>This field is mandatory</p></li>

<li><p>This field cannot be empty or null</p></li>
</ul>

<h3 id="load-balancers">Load Balancers</h3>

<pre><code>loadbalancers:
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

</code></pre>

<p>Loadbalancers support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the instance to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>private</strong></p>

<ul>
<li>Boolean that defines if the loadbalancer will be private or public.</li>
<li>This field is optional.</li>
<li>Values can be: “true“ or “false“.</li>

<li><p>This field will default to “false“.</p></li>

<li><p><strong>networks</strong></p></li>

<li><p>Array of String&rsquo;s that defines which public network/subnet to attach to the ELB.</p></li>

<li><p>This field is mandatory if private is set to false.</p></li>

<li><p>This field cannot be null or empty.</p></li>

<li><p>This field must specify a network/subnet that exists on the yaml.</p></li>

<li><p>Only one subnet can be deployed per availablilty zone. It is advised that you specify an AZ when defining a network/subnet.</p></li>
</ul></li>

<li><p><strong>instances</strong></p>

<ul>
<li>Array of String&rsquo;s that defines which group of instances to add to the ELB.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must specify an instance group that exists on the yaml.</li>
</ul></li>

<li><p><strong>security_groups</strong></p>

<ul>
<li>Array of String&rsquo;s that defines which security groups to apply to the ELB.</li>
<li>This field not is mandatory.</li>
<li>This field must specify a security group that exists on the yaml.</li>
</ul></li>
</ul>

<p><strong>listeners</strong></p>

<p>ELB Listeners.</p>

<ul>
<li><p><strong>protocol</strong></p>

<ul>
<li>String that defines the protocol for this rule.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>Values can be: http | https | ssl | tcp</li>
<li>If https or ssl is specified, an ssl_cert must be specified.</li>
</ul></li>

<li><p><strong>from_port</strong></p>

<ul>
<li>Starting port-range number.</li>
<li>This field is mandatory.</li>
<li>Values can be: 1 - 65535.</li>
</ul></li>

<li><p><strong>to_port</strong></p>

<ul>
<li>Ending port-range number.</li>
<li>This field is mandatory.</li>
<li>Values can be: 1 - 65535.</li>
</ul></li>

<li><p><strong>ssl_cert</strong></p>

<ul>
<li>The ssl certificate ID of a valid AWS Certificate Manager (ACM) cert.</li>
<li>This field is mandatory only if protocol is set to https or ssl.</li>
</ul></li>
</ul>

<h3 id="s3-buckets">S3 Buckets</h3>

<pre><code>s3_buckets:
  - name: bucket-1
    acl: private
    bucket_location: eu-west-1
    grantees:
      - id: foo
        type: emailaddress, id, uri
        permissions: full, read, read-acl, write-acl
</code></pre>

<p>S3 support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>(string) name of the bucket.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>acl</strong></p>

<ul>
<li>(string) the canned ACL to apply to the bucket.</li>
<li>The value of this field can be private, public-read, public-read-write</li>
<li>or authenticated-read</li>
</ul></li>

<li><p><strong>bucket_location</strong></p>

<ul>
<li>(string) the location constraint</li>
<li>The value of this field can be EU, eu-west-1, us-west-1, us-west-2,</li>
<li>ap-south-1, ap-southeast-2, ap-northeast-1, sa-east-1, cn-north-1 or</li>
<li>eu-central-1</li>
</ul></li>

<li><p><strong>grantees</strong></p>

<ul>
<li>(collection) List of permissions on the bucket.</li>
</ul></li>

<li><p><strong>id</strong></p>

<ul>
<li>(string) Grantee name</li>
</ul></li>

<li><p><strong>type</strong></p>

<ul>
<li>(string) Grantee type</li>
<li>Possible values : email address, id or uri</li>
</ul></li>

<li><p><strong>permissions</strong></p>

<ul>
<li>(string) Permissions for this grantee on the bucket</li>
<li>Possible values: full, read, read-acl and write-acl</li>
</ul></li>
</ul>

<p>### Route53 Zones</p>

<pre><code> route53_zones:
   - name: example.com
     private: false
     records:
       - entry: one.example.com
         type: A
         ttl: 3600
         values:
           - 8.8.8.8
</code></pre>

<p>S3 support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>(string) The domain name of the zone.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
</ul></li>

<li><p><strong>private</strong></p>

<ul>
<li>(boolean) sets the zone to be private hosted zone.</li>
<li>The value of this field can be true or false. Default is false.</li>
</ul></li>
</ul>

<p><strong>records</strong></p>

<p>The collection of DNS entries you want to host.</p>

<ul>
<li><p><strong>entry</strong></p>

<ul>
<li>(string) the full domain record entry</li>
<li>The value of this field should be an F.Q.D.N (fully qualified domain name)</li>
</ul></li>

<li><p><strong>Type</strong></p></li>

<li><p>(string) the full domain record entry</p></li>

<li><p>The value of this field should be one of &lsquo;A&rsquo;, &lsquo;AAAA&rsquo;, &lsquo;CNAME&rsquo;, &lsquo;MX&rsquo;, &lsquo;PTR&rsquo;, &lsquo;TXT&rsquo;, &lsquo;SRV&rsquo;, &lsquo;SPF&rsquo;, &lsquo;NAPTR&rsquo;, &lsquo;NS&rsquo;, &lsquo;SOA&rsquo;</p></li>

<li><p><strong>TTL</strong></p></li>

<li><p>(int) the expiration period of the entry.</p></li>

<li><p>The value of this field should be greater than 0</p></li>

<li><p><strong>values</strong></p>

<ul>
<li>Array of strings. The values the entry will return</li>
<li>These values must be valid for the type of record you have selected.</li>
</ul></li>

<li><p><strong>Loadbalancers</strong></p>

<ul>
<li>Array of strings. The loadbalancers you wish the entry to resolve to.</li>
<li>The type of record must be set to CNAME</li>
<li>You must specify only loadbalancers or instances, not both.</li>
</ul></li>

<li><p><strong>instances</strong></p>

<ul>
<li>Array of strings. The loadbalancers you wish the entry to resolve to.</li>
<li>The type of record must be set to A</li>
<li>This must target the individual instance, including the instances number, i.e. web-1</li>
<li>You must specify only loadbalancers or instances, not both.</li>
</ul></li>
</ul>

<h3 id="rds-clusters">RDS Clusters</h3>

<pre><code>rds_clusters:
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

</code></pre>

<p>RDS Clusters support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>(string) The name of the rds cluster.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
</ul></li>

<li><p><strong>engine</strong></p>

<ul>
<li>(string) The engine type of the rds cluster.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>Currently only &lsquo;aurora&rsquo; is supported as a value.</li>
</ul></li>

<li><p><strong>engine_version</strong></p>

<ul>
<li>(string) The engine_version of the rds cluster.</li>
<li>This field is not mandatory.</li>
</ul></li>

<li><p><strong>port</strong></p>

<ul>
<li>(int) The port exposed by the rds cluster.</li>
<li>This field not is mandatory.</li>
<li>This field must be between 1150 - 65535 (default: 3306).</li>
</ul></li>

<li><p><strong>public</strong></p>

<ul>
<li>(boolean) Sets the rds cluster to be publicly accessible.</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or alse. Default is false.</li>
</ul></li>

<li><p><strong>availability_zones</strong></p>

<ul>
<li>Array of strings. The availability zones to deploy the rds cluster to.</li>
<li>This field not is mandatory.</li>
<li>The availability zone must be in the same region as the vpc it is deployed to.</li>
<li>If the rds cluster is set to private, there must be networks specified for each availability zone.</li>
</ul></li>

<li><p><strong>security_groups</strong></p>

<ul>
<li>Array of String&rsquo;s that defines which security groups to apply to the rds cluster.</li>
<li>This field not is mandatory.</li>
<li>This field must specify a security group that exists on the yaml.</li>
</ul></li>

<li><p><strong>networks</strong></p>

<ul>
<li>Array of String&rsquo;s that defines which networks to expose the rds cluster to.</li>
<li>This field not is mandatory (if public is set to true).</li>
<li>This field must specify a security group that exists on the yaml.</li>
</ul></li>

<li><p><strong>database_name</strong></p>

<ul>
<li>(string) The database name of the rds cluster.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>database_username</strong></p>

<ul>
<li>(string) The database username of the rds cluster.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>database_username</strong></p>

<ul>
<li>(string) The database password of the rds cluster.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>maintenance_window</strong></p>

<ul>
<li>(string) The time period at which any maintenance tasks will occur.</li>
<li>This field is not mandatory.</li>
<li>Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)</li>
</ul></li>

<li><p><strong>replication_source</strong></p>

<ul>
<li>(string) The target rds cluster or mysql instance to replicate from.</li>
<li>Must be a valid ARN (Amazon Resource Name).</li>
<li><a href="http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html" target="_blank">http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html</a></li>
</ul></li>

<li><p><strong>final_snapshot</strong></p>

<ul>
<li>(boolean) Performs a final snapshot when the rds cluster is deleted.</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or alse. Default is false.</li>
</ul></li>
</ul>

<p><strong>backups</strong></p>

<p>Backup configuration.</p>

<ul>
<li><p><strong>window</strong></p>

<ul>
<li>(string) The time period at which backups will occur.</li>
<li>This field is not mandatory.</li>
<li>Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)</li>
</ul></li>

<li><p><strong>retention</strong></p>

<ul>
<li>(int) The amount of time in days that backups will be retained for.</li>
<li>This field is not mandatory.</li>
<li>Must be between 1 - 35 days</li>
</ul></li>
</ul>

<h3 id="rds-instances">RDS Instances</h3>

<pre><code>rds_instances:
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
</code></pre>

<p>RDS Instances support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>(string) The name of the rds instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
</ul></li>

<li><p><strong>engine</strong></p>

<ul>
<li>(string) The engine type of the rds instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>Currently only &lsquo;aurora&rsquo; is supported as a value.</li>
</ul></li>

<li><p><strong>engine_version</strong></p>

<ul>
<li>(string) The engine_version of the rds instance.</li>
<li>This field is not mandatory.</li>
</ul></li>

<li><p><strong>port</strong></p>

<ul>
<li>(int) The port exposed by the rds instance.</li>
<li>This field not is mandatory.</li>
<li>This field must be between 1150 - 65535 (default: 3306).</li>
</ul></li>

<li><p><strong>public</strong></p>

<ul>
<li>(boolean) Sets the rds instance to be publicly accessible.</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or alse. Default is false.</li>
</ul></li>

<li><p><strong>multi_az</strong></p>

<ul>
<li>(boolean) Deploys a synchronous standby instance in a different availability zone</li>
<li>Cannot be used in conjunction with the &lsquo;availability_zone&rsquo; parameter</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or alse. Default is false.</li>
</ul></li>

<li><p><strong>availability_zones</strong></p>

<ul>
<li>Array of strings. The availability zones to deploy the rds instance to.</li>
<li>This field not is mandatory.</li>
<li>The availability zone must be in the same region as the vpc it is deployed to.</li>
<li>If the rds instance is set to private, there must be networks specified for each availability zone.</li>
</ul></li>

<li><p><strong>security_groups</strong></p>

<ul>
<li>Array of String&rsquo;s that defines which security groups to apply to the rds instance.</li>
<li>This field not is mandatory.</li>
<li>This field must specify a security group that exists on the yaml.</li>
</ul></li>

<li><p><strong>networks</strong></p>

<ul>
<li>Array of String&rsquo;s that defines which networks to expose the rds instance to.</li>
<li>This field not is mandatory (if public is set to true).</li>
<li>This field must specify a security group that exists on the yaml.</li>
</ul></li>

<li><p><strong>database_name</strong></p>

<ul>
<li>(string) The database name of the rds instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>database_username</strong></p>

<ul>
<li>(string) The database username of the rds instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>database_username</strong></p>

<ul>
<li>(string) The database password of the rds instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
</ul></li>

<li><p><strong>maintenance_window</strong></p>

<ul>
<li>(string) The time period at which any maintenance tasks will occur.</li>
<li>This field is not mandatory.</li>
<li>Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)</li>
</ul></li>

<li><p><strong>promotion_tier</strong></p>

<ul>
<li>(int) specifies the order in which an aurora replica is promoted to a primary.</li>
<li>This field is not mandatory.</li>
<li>Must be between 1 - 15.</li>
<li>Default is 1.</li>
</ul></li>

<li><p><strong>auto_upgrade</strong></p>

<ul>
<li>(boolean) Performs automatic upgrades on the database (minor versions only)</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or alse. Default is false.</li>
</ul></li>

<li><p><strong>replication_source</strong></p>

<ul>
<li>(string) The target an rds instance to replicate from.</li>
<li>Must be a valid ARN (Amazon Resource Name) or instance name (if in the same region).</li>
<li><a href="http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html" target="_blank">http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html</a></li>
</ul></li>

<li><p><strong>final_snapshot</strong></p>

<ul>
<li>(boolean) Performs a final snapshot when the rds instance is deleted.</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or alse. Default is false.</li>
</ul></li>

<li><p><strong>license</strong></p>

<ul>
<li>(string) The type of license you want to use with the database</li>
<li>This field is not mandatory.</li>
<li>Must be one of &lsquo;license-included&rsquo;, &lsquo;bring-your-own-license&rsquo; or &lsquo;general-public-license&rsquo;</li>
</ul></li>
</ul>

<p><strong>storage</strong></p>

<p>Storage configuration. Only configurable when using a non-aurora engine.</p>

<ul>
<li><p><strong>type</strong></p>

<ul>
<li>(string) The type of storage you want to use.</li>
<li>This field is not mandatory.</li>
<li>Must be one of &lsquo;gp2&rsquo; (general purpose SSD), &lsquo;io1&rsquo; (performance optimised SSD) or &lsquo;standard&rsquo; (magnetic disk)</li>
<li>If io1 is specified, you must also set the &lsquo;iops&rsquo; parameter</li>
</ul></li>

<li><p><strong>size</strong></p>

<ul>
<li>(int) The amount of storage (GB) that you want to use.</li>
<li>This field is not mandatory.</li>
<li>Must be a value between 5 - 6144.</li>
<li>Minimum size can vary between engine types.</li>
</ul></li>

<li><p><strong>iops</strong></p>

<ul>
<li>(int) The amount of iops that you want the database to be limited to.</li>
<li>This field is mandatory if storage type is &lsquo;io1&rsquo;.</li>
<li>This field must only be used in conjunction with &lsquo;io1&rsquo; as a storage type.</li>
<li>Must be a multiple between 3 and 10 of the storage amount.</li>
<li>Must also be an integer multiple of 1000.</li>
</ul></li>
</ul>

<p><strong>backups</strong></p>

<p>Backup configuration.</p>

<ul>
<li><p><strong>window</strong></p>

<ul>
<li>(string) The time period at which backups will occur.</li>
<li>This field is not mandatory.</li>
<li>Format must be a range in the structure of DDD:HH::MM-DDD:HH::MM (Day:Hour:Minute)</li>
</ul></li>

<li><p><strong>retention</strong></p>

<ul>
<li>(int) The amount of time in days that backups will be retained for.</li>
<li>This field is not mandatory.</li>
<li>Must be between 1 - 35 days</li>
</ul></li>
</ul>

<p>### EBS Volumes</p>

<pre><code> ebs_volumes:
   - name: database-vol
     count: 1
     type: io1
     size: 100
     iops: 10000
     availability_zone: eu-west-1a
     encrypted: true
     encryption_key_id: kms-id
</code></pre>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>(string) The name of the ebs volume.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
</ul></li>

<li><p><strong>count</strong></p>

<ul>
<li>Integer that defines the number of volunes to be created.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>

<li><p>This field must be greater or equal that 1</p></li>

<li><p><strong>type</strong></p></li>

<li><p>(string) The type of storage you want to use.</p></li>

<li><p>This field is not mandatory.</p></li>

<li><p>Must be one of &lsquo;gp2&rsquo; (general purpose SSD), &lsquo;io1&rsquo; (performance optimised SSD) or &lsquo;st1&rsquo; (magnetic disk)</p></li>

<li><p>If io1 is specified, you must also set the &lsquo;iops&rsquo; parameter</p></li>

<li><p><strong>size</strong></p></li>

<li><p>(int) The amount of storage (GB) that you want to use.</p></li>

<li><p>This field is not mandatory.</p></li>

<li><p>Must be a value between 1 - 16384. (may vary for different storage types)</p></li>

<li><p><strong>iops</strong></p></li>

<li><p>(int) The amount of iops that you want the volume to be limited to.</p></li>

<li><p>This field is mandatory if storage type is &lsquo;io1&rsquo;.</p></li>

<li><p>This field must only be used in conjunction with &lsquo;io1&rsquo; as a storage type.</p></li>

<li><p>Must be a multiple between 3 and 10 of the storage amount.</p></li>

<li><p>Must also be an integer multiple of 1000.</p></li>

<li><p>Must be a value between 100 - 20000</p></li>
</ul></li>

<li><p><strong>encrypted</strong></p>

<ul>
<li>(boolean) Specifies whether the volume will be encrypted.</li>
<li>This field not is mandatory.</li>
<li>The value of this field can be true or false. Default is false.</li>
<li>When enabling encryption, a kms key id must be provided</li>
</ul></li>

<li><p><strong>encryption_key_id</strong></p>

<ul>
<li>(string) The kms key id to use when encrypting data on the volume.</li>
<li>This field is mandatory, only if encryption is set to true.</li>
</ul></li>
</ul>

## vCloud Director
vCloud Director (vCD) is cloud management tool from VMWare that acts as an overlay on top of vSphere, providing users with a self-service GUI and API. vCloud Director enables service providers to offer Infrastructure as a Service to their customers by providing users with direct control of virtual machine provisioning, and some aspects of networking.

### vCloud Limitations
With vCD most network functionality is delivered by the VMWare vShield Edge (vSE) virtual appliance. Unfortunately certain key network operations can only be performed by the service provider and are not available to users. They include:

- deploying a vSE
- connecting a vSE to an external network
- sub-allocating external IP addresses to a vSE

All other common network operations are accesible to users, however many of them are limited to users with the [Organization Administrator](https://pubs.vmware.com/vca/index.jsp?topic=%2Fcom.vmware.vcloud.api.doc_56%2FGUID-BC504F6B-3D38-4F25-AACF-ED584063754F.html) role assigned to them.

## vCloud Director Examples

<p>If you would like to try Ernest but do not have access to vCloud you can request a demo vCloud account <a href="/vcloud-account">here</a>.</p>

<p>Before we get started we will need the following information:</p>

<ul>
<li>Ernest IP address (31.210.241.221)</li>
<li>Ernest username/password (user1/xxxxxx)</li>
<li>vCloud URL (myvdc.carrenza.net)</li>
<li>vCloud organisation (r3labs-development)</li>
<li>vCloud datacenter (r3-jreid2)</li>
<li>vCloud network (DVS-VCD-EXT-665)</li>
<li>vCloud username/password (jreid/xxxxxx)</li>
<li>vCloud router name (test1)</li>
<li>vCloud sub-allocated IP (195.3.186.42)</li>
</ul>

<p><em>The values in brackets are what we will use for this example. Your service provider can provide you with the vCloud information.</em></p>

<p>The first step is to set the IP address of Ernest:</p>

<pre><code>$ ernest target https://31.210.241.221
Target set

</code></pre>

<p>Next we login to Ernest:</p>

<pre><code>$ ernest login
Username: user1
Password: ******
Welcome back user1

</code></pre>

<p>Once we have logged in to Ernest we can setup the vCloud datacenter and credentials that Ernest will use to create our infrastructure:</p>

<pre><code>$ ernest datacenter create vcloud --user jreid --password xxxxxx --org r3labs-development --vcloud-url https://myvdc.carrenza.net --public-network DVS-VCD-EXT-665 r3-jreid2
Datacenter 'r3-jreid2' successfully created

</code></pre>

<p>Now that we have our datacenter created in Ernest we can start building stuff. Below we will show examples for:</p>

<ul>
<li>creating infrastructure only</li>
<li>creating infrastructure and configuring the servers</li>
<li>creating the servers only</li>
</ul>

<h3 id="infrastructure-only">Infrastructure Only</h3>

<p>For our example we will deploy a single Ubuntu server from a catalog image (image: r3/ubuntu-1404) and configure our networking to permit SSH access to the server. Our YAML for this example is:</p>

<pre><code>---
name: demo1
datacenter: r3-jreid2
bootstrapping: none
service_ip: 195.3.186.42

routers:
  - name: test1
    rules:
    - name: in_out_any
      source: internal
      from_port: any
      destination: external
      to_port: any
      protocol: any
      action: allow

    - name: out_in_22
      source: any
      from_port: any
      destination: internal
      to_port: '22'
      protocol: tcp
      action: allow

    networks:
      - name: web
        subnet: 10.1.0.0/24
        dns:
          - 8.8.8.8
          - 8.8.4.4

    port_forwarding:
      - source: 195.3.186.42
        from_port: '22'
        to_port: '22'
        destination: 10.1.0.11

instances:
  - name: web
    image: r3/ubuntu-1404
    cpus: 1
    memory: 1GB
    count: 1
    networks:
      name: web
      start_ip: 10.1.0.11

</code></pre>

<p>Now that we have defined our infrastructure we are ready to apply our definition:</p>

<pre><code>$ ernest service apply demo1.yml
#Environment creation requested
#Ernest will show you all output from your requested service creation
#You can cancel at any moment with Ctrl+C, even the service is still #being created, you won't have any output
Starting environment creation
Creating networks:
 - r3-jreid2-demo1-web
   IP     : 10.1.0.0/24
   Status : completed
Networks successfully created
Creating instances:
 - r3-jreid2-demo1-web-1
   IP        : 10.1.0.11
   Status    : completed
Instances successfully created
Updating instances:
 - r3-jreid2-demo1-web-1
   IP        : 10.1.0.11
   Status    : completed
Instances successfully updated
Creating firewalls:
 - r3-jreid2-demo1-test2
   Status    : completed
Firewalls created
Creating nats:
 - r3-jreid2-demo1-test2
   Status    : completed
Nats created
SUCCESS: rules successfully applied
Your environment endpoint is: 195.3.186.42

</code></pre>

<p>Congratulations! You have built your first infrastructure with Ernest. You can now SSH to the server on IP 195.3.186.42 to install and configure your software.</p>

<p>If you wish to change the infrastructure update your YAML to show how you want the infrastructure to look, then re-apply the YAML. Ernest will make the appropriate changes to the infrastructure.</p>

<p>For example if we increase the &lsquo;web&rsquo; instance count from 1 to 2 and re-apply the YAML a new server is created:</p>

<pre><code>$ ernest service apply demo1.yml
#Environment creation requested
#Ernest will show you all output from your requested service creation
#You can cancel at any moment with Ctrl+C, even the service is still #being created, you won't have any output
Starting environment creation
Creating instances:
 - r3-jreid2-demo1-web-2
   IP        : 10.1.0.12
   Status    : completed
Instances successfully created
Updating instances:
 - r3-jreid2-demo1-web-2
   IP        : 10.1.0.12
   Status    : completed
Instances successfully updated
SUCCESS: rules successfully applied
Your environment endpoint is: 195.3.186.42

</code></pre>

<p>We can list the services we created:</p>

<pre><code>$ ernest service list
NAME  UPDATED       STATUS  ENDPOINT
demo1 2016-05-29 14:52:13 +0100 BST done  195.3.186.42

</code></pre>

<p>For a given service we can see the history:</p>

<pre><code>$ ernest service history demo1
NAME  BUILD ID                                                              UPDATED             STATUS
demo1 488249c2-63c2-484e-b6e0-c690ba859dab-8c22343f8f9d3dfb68e41cc033e7b492 2016-05-29 14:52:13 done
demo1 6799e858-0484-42cb-bbb1-c1bc5f61e500-8c22343f8f9d3dfb68e41cc033e7b492 2016-05-29 14:39:56 done

</code></pre>

<p>For a given service and build ID we can see the definition we applied:</p>

<pre><code>$ ernest service definition demo1
---
name: demo1
datacenter: r3-jreid2
bootstrapping: none
service_ip: 195.3.186.42

routers:
  - name: test1
    rules:
    - name: in_out_any
      source: internal
      from_port: any
      destination: external
      to_port: any
      protocol: any
      action: allow

    - name: out_in_22
      source: any
      from_port: any
      destination: internal
      to_port: '22'
      protocol: tcp
      action: allow

    networks:
      - name: web
        subnet: 10.1.0.0/24
        dns:
          - 8.8.8.8
          - 8.8.4.4

    port_forwarding:
      - source: 195.3.186.42
        from_port: '22'
        to_port: '22'
        destination: 10.1.0.11

instances:
  - name: web
    image: r3/ubuntu-1404
    cpus: 1
    memory: 1GB
    count: 2
    networks:
      name: web
      start_ip: 10.1.0.11

</code></pre>

<h3 id="infrastructure-and-server-configuration">Infrastructure and Server Configuration</h3>

<p>In the above example the task of installing and configuring software was left to the user to do. In this example we will bootstrap the servers and install our software directly from the YAML.</p>

<p>The new YAML is shown here:</p>

<pre><code>---
name: demo2
datacenter: r3-jreid2
bootstrapping: salt
service_ip: 195.3.186.42
ernest_ip:
  - 31.210.241.221
  - 31.210.240.161

routers:
  - name: test1
    rules:
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

    port_forwarding:
      - source: 195.3.186.42
        from_port: '80'
        to_port: '80'
        destination: 10.1.0.11

instances:
  - name: web
    image: r3/ubuntu-1404
    cpus: 1
    memory: 1GB
    count: 1
    networks:
      name: web
      start_ip: 10.1.0.11
    provisioner:
      - exec:
        - 'sudo apt-get update'
        - 'sudo apt-get install apache2 -y'

</code></pre>

<p>The first thing to notice is that we have changed &lsquo;bootstrapping&rsquo; from &lsquo;none&rsquo; to &lsquo;salt&rsquo;. This will result in Ernest deploying a SALT instance that we can use to manage our environment. Note that the &lsquo;bootstrapping&rsquo; option must be specified when an environment is created and cannot be changed for that environment.</p>

<p>The next thing we have changed are the firewall and port-forwarding configuration. We have removed the sections needed for SSH access to the server since we are now able to run commands directly from the YAML.</p>

<p>Finally, we have added a provisioner section to the instance that will install the Apache HTTP Server.</p>

<p>Now that we have defined our platform we are ready to create it:</p>

<pre><code>$ ernest service apply demo2.yml
#Environment creation requested
#Ernest will show you all output from your requested service creation
#You can cancel at any moment with Ctrl+C, even the service is still #being created, you won't have any output
Starting environment creation
Creating networks:
 - r3-jreid2-demo2-salt
   IP     : 10.254.254.0/24
   Status : completed
 - r3-jreid2-demo2-web
   IP     : 10.1.0.0/24
   Status : completed
Networks successfully created
Creating instances:
 - r3-jreid2-demo2-salt-master
   IP        : 10.254.254.100
   Status    : completed
 - r3-jreid2-demo2-web-1
   IP        : 10.1.0.11
   Status    : completed
Instances successfully created
Updating instances:
 - r3-jreid2-demo2-salt-master
   IP        : 10.254.254.100
   Status    : completed
 - r3-jreid2-demo2-web-1
   IP        : 10.1.0.11
   Status    : completed
Instances successfully updated
Creating firewalls:
 - r3-jreid2-demo2-test2
   Status    : completed
Firewalls created
Creating nats:
 - r3-jreid2-demo2-test2
   Status    : completed
Nats created
Running bootstraps:
 - Bootstrap r3-jreid2-demo2-web-1
   Status    : completed
Bootstrap ran
Running executions:
 - Execution web 1
   Status    : completed
Executions ran
SUCCESS: rules successfully applied
Your environment endpoint is: 195.3.186.44

</code></pre>

<p>Notice that Ernest has automatically created a SALT instance for us on network 10.254.254.0/24. It has also trigged the bootstrapping process that installs the SALT minion on each of our servers, and then run the commands we specified in the provisioner section of each instance defined in the YAML.</p>

<p>You should be able to browse to <a href="http://195.3.186.42" target="_blank">http://195.3.186.42</a>. Congratulations!</p>

<p>If you wish to change the platform update your YAML to show how you want the platform to look, then re-apply the YAML. Ernest will make the appropriate changes to the platform.</p>

<blockquote>
<p>The SALT image can be downloaded from <a href="/downloads/r3-salt-master.ova">here</a>. For the current version of Ernest (1.0) it is not possible to specify the catalog name that Ernest will get the SALT image from, you will need to place the image in a catalog named &lsquo;r3&rsquo;.</p>
</blockquote>

<h3 id="servers-only">Servers Only</h3>

<p>If your vCloud account does not have the Organization Administrator assigned to it most of the network related configuration in the above examples will be impossible. You will be limited to the creation and modification of virtual machines only. An example YAML for this scenario is:</p>

<pre><code>---
name: demo3
datacenter: r3-jreid2

instances:
  - name: web
    image: r3/ubuntu-1404
    cpus: 1
    memory: 1GB
    count: 1
    networks:
      name: r3-jreid2-test3-web
      start_ip: 10.1.0.11

</code></pre>

<p>And applying it gives:</p>

<pre><code>$ ernest service apply demo3.yml
#Environment creation requested
#Ernest will show you all output from your requested service creation
#You can cancel at any moment with Ctrl+C, even the service is still #being created, you won't have any output
Starting environment creation
Creating instances:
 - r3-jreid2-demo3-web-1
   IP        : 10.1.0.11
   Status    : completed
Instances successfully created
Updating instances:
 - r3-jreid2-demo3-web-1
   IP        : 10.1.0.11
   Status    : completed
Instances successfully updated
SUCCESS: rules successfully applied
Your environment endpoint is:

</code></pre>

## vCloud Director YAML Reference

<p>Environments built and managed with Ernest are defined in YAML format.</p>

<h3 id="example">Example</h2>

<pre><code>---
name: demo
datacenter: r3-jreid2
bootstrapping: salt
service_ip: 195.3.186.44
ernest_ip:
  - 31.210.240.161

routers:
  - name: demo
    rules:
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

    port_forwarding:
      - source: 195.3.186.44
        from_port: '80'
        to_port: '80'
        destination: 10.1.0.11

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
        - if [ x$1 == x&quot;postcustomization&quot; ]; then
        - yum -y install httpd
        - service httpd start
        - fi
</code></pre>

<h3 id="field-reference">Field Reference</h2>

<h3 id="service-options">Service Options</h3>

<pre><code>name: demo
datacenter: r3-jreid2
bootstrapping: salt
service_ip: 195.3.186.44
ernest_ip:
  - 31.210.240.161
</code></pre>

<p>Service Options support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the service to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>datacenter</strong></p>

<ul>
<li>String that defines the name of the datacenter where the service is built.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must 50 characters maximum.</li>
</ul></li>

<li><p><strong>bootstrapping</strong></p>

<ul>
<li>String [salt, none] that defines if you’re going to be able to execute commands on the servers</li>
<li>This field is optional.</li>
<li>This field defaults to none.</li>
<li>The value of this is restricted to “salt” or “none”</li>
</ul></li>

<li><p><strong>service_ip</strong></p>

<ul>
<li>String that defines the public ip of the router already created on vcloud</li>
<li>This field is optional</li>
<li>This field defaults to none.</li>
<li>The value of this field is an IP, internally defined as <a href="https://golang.org/pkg/net/#IP" target="_blank">https://golang.org/pkg/net/#IP</a></li>
</ul></li>

<li><p><strong>ernest_ip</strong></p>

<ul>
<li>Array of IPs that defines the ernest instance public ip, so it will be allowed through the created service firewall rules.</li>
<li>The value of this field is an IP, internally defined as <a href="https://golang.org/pkg/net/#IP" target="_blank">https://golang.org/pkg/net/#IP</a><br /></li>
<li>This field is mandatory only if you’ve defined bootstrapping as “salt”</li>
<li>This field defaults to an empty array.</li>
</ul></li>
</ul>

<h3 id="networking">Networking</h3>

<pre><code>routers:
  - name: demo
    rules:
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

    port_forwarding:
      - source: 195.3.186.44
        from_port: '80'
        to_port: '80'
        destination: 10.1.0.11
</code></pre>

<p>Networking supports the following fields:</p>

<ul>
<li><strong>name</strong>

<ul>
<li>String that defines the name of the service to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
<li>The value of this field can be 50 characters maximum.</li>
</ul></li>
</ul>

<p><strong>rules</strong></p>

<p>A firewall rule controls traffic that is allowed to flow between both internal and external routers and networks.</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the rule.</li>
<li>This field is mandatory.</li>
<li>This field can’t be null or empty.</li>
</ul></li>

<li><p><strong>source</strong></p>

<ul>
<li>String the source of the network where this firewall acts</li>
<li>This field is mandatory.</li>
<li>Values can be: external | internal | any | named networks | CIDR</li>
</ul></li>

<li><p><strong>from_port</strong></p>

<ul>
<li>Source port numbers</li>
<li>This field is mandatory.</li>
<li>Values can be:  an string ( valid port number, 1 - 65535 ) or any</li>
</ul></li>

<li><p><strong>destination</strong></p>

<ul>
<li>String the destination of the network where this firewall acts</li>
<li>This field is mandatory.</li>
<li>Values can be:  external | internal | any | named networks | CIDR</li>
</ul></li>

<li><p><strong>to_port</strong></p>

<ul>
<li>Destination port numbers</li>
<li>This field is mandatory.</li>
<li>Values can be:  an string ( valid port number, 1 - 65535 ) or any</li>
</ul></li>

<li><p><strong>protocol</strong></p>

<ul>
<li>String with the protocol we are firewalling</li>
<li>This field is mandatory.</li>
<li>Values can be: tcp | udp | icmp | any | tcp &amp; udp</li>
</ul></li>
</ul>

<p><strong>networks</strong></p>

<p>A network is a virtual network that attaches to a router (in vcloud, other providers will differ). This is what allows us to network/isolate VM’s together.</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the service to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>subnet</strong></p>

<ul>
<li>String that defines the name of a network to add to this service.</li>
<li>It must follow CIDR notation as described at: <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing</a></li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique for this user &amp; all the networks on the manifest.</li>
</ul></li>

<li><p><strong>dns</strong></p>

<ul>
<li>Array of IPs to use as dns servers on this service</li>
<li>It must follow CIDR notation as described at: <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing</a></li>
<li>This field is optional.</li>
<li>This field can be empty and it will default to [&laquo;8.8.8.8&raquo;, &laquo;8.8.4.4&raquo;].</li>
</ul></li>
</ul>

<p><strong>port_forwarding</strong></p>

<p>A port forward is something that converts translates a request on a port from one IP to another on a different network. Only tcp port forwarding is supported.</p>

<ul>
<li><p><strong>from_port</strong></p>

<ul>
<li>Source port numbers</li>
<li>Values can be:  an string ( valid port number, 1 - 65535 ) or any</li>
</ul></li>

<li><p><strong>destination</strong></p>

<ul>
<li>String the destination of the IPv4 Address to translate to</li>
<li>Values can be: IPv4</li>
</ul></li>

<li><p><strong>to_port</strong></p>

<ul>
<li>Destination port numbers</li>
<li>Values can be:  an string ( valid port number, 1 - 65535 ) or any</li>
</ul></li>

<li><p><strong>source</strong></p>

<ul>
<li>String that defines the IP from the provider-specified sub-allocated list of IPs of the router.</li>
<li>It must follow CIDR notation as described at: <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing</a></li>
<li>This field is optional.</li>
<li>This field can be null or empty, in case a router is created by Ernest it will default to the created one, on the other hand this value will be used.</li>
<li>This field must be unique for this user &amp; all the networks on the manifest.</li>
</ul></li>
</ul>

<h3 id="instances">Instances</h3>

<pre><code>instances:
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
        - if [ x$1 == x&quot;postcustomization&quot; ]; then
        - yum -y install httpd
        - service httpd start
        - fi
</code></pre>

<p>Instances support the following fields:</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of the instance to build.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be unique by user &amp; manifest.</li>
<li>The value of this field must be 50 characters maximum.</li>
</ul></li>

<li><p><strong>image</strong></p>

<ul>
<li>String that defines the name of an image to install on the VM.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field is composed in two parts one for the “catalog” and another one for the “image” separated by a slash “/”</li>
<li>Both “catalog” and “image” are strings and can’t be null.</li>
</ul></li>

<li><p><strong>cpus</strong></p>

<ul>
<li>Integer that defines how many CPUs will have the VM.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be greater or equal than 1</li>
</ul></li>

<li><p><strong>memory</strong></p>

<ul>
<li>String that defines the name of a load balancer to add to this service.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must follow the Binary Prefix format: <a href="https://en.wikipedia.org/wiki/Binary_prefix#Computer_memory" target="_blank">https://en.wikipedia.org/wiki/Binary_prefix#Computer_memory</a></li>
<li>The possible binary prefixes are MB, GB, TB, PB, YB</li>
</ul></li>

<li><p><strong>disks:</strong></p>

<ul>
<li>A list of sizes of hard disks that belongs to the VM</li>
<li>This field can be an empty list</li>
<li>This field is a list of strings</li>
<li>Each element of the string must follow the Binary Prefix format: <a href="https://en.wikipedia.org/wiki/Binary_prefix#Hard_disk_drives" target="_blank">https://en.wikipedia.org/wiki/Binary_prefix#Hard_disk_drives</a></li>
<li>The possible binary prefixes are MB, GB, TB, PB, YB</li>
</ul></li>

<li><p><strong>root_disk</strong></p>

<ul>
<li>String that defines the root disk size of the template the instance is based on.</li>
<li>This field is optional.</li>
<li>This field cannot be null or empty.</li>
<li>The value must be larger than the default root disk size of the template.</li>
<li>Each element of the string must follow the Binary Prefix format: <a href="https://en.wikipedia.org/wiki/Binary_prefix#Hard_disk_drives" target="_blank">https://en.wikipedia.org/wiki/Binary_prefix#Hard_disk_drives</a></li>
<li>The possible binary prefixes are MB, GB, TB, PB, YB</li>
</ul></li>

<li><p><strong>count</strong></p>

<ul>
<li>Integer that defines the number of VM to be created.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be greater or equal that 1</li>
</ul></li>
</ul>

<p><strong>networks</strong>
Networks is a map with two fields</p>

<ul>
<li><p><strong>name</strong></p>

<ul>
<li>String that defines the name of a network to attach this instance.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>The value of this field should exist on the networks section previously specified.</li>
</ul></li>

<li><p><strong>start_ip</strong></p>

<ul>
<li>String that defines the starting IP to allocate the VMs to be built.</li>
<li>This field is mandatory.</li>
<li>This field cannot be null or empty.</li>
<li>This field must be a valid IP.</li>
<li>This IP should belong to the network already defined on the instance.</li>
</ul></li>
</ul>

<p><strong>provisioner</strong></p>

<p>Provisioner defines the available provisioner mechanisms.</p>

<ul>
<li><p><strong>exec</strong></p>

<ul>
<li>Array of strings, any command characters must be escaped with backslashes: (RFC7159, Section 7)</li>
<li>Each command in the exec array is concatenated together and delimited by a semicolon.</li>
<li>This field is optional</li>
<li>If specified as a provisioner type, this array field must contain at least one element</li>
<li>The specified commands will be executed by the SALT master.</li>
</ul></li>

<li><p><strong>shell</strong></p>

<ul>
<li>Array of strings, any command characters must be escaped with backslashes: (RFC7159, Section 7)</li>
<li>Each command in the exec array is concatenated together and delimited by a semicolon.</li>
<li>This field is optional</li>
<li>If specified as a provisioner type, this array field must contain at least one element</li>
<li>The specified commands will be executed by the vCloud guest customization script.</li>
</ul></li>
</ul>
