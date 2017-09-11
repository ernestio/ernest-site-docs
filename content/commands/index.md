---
date: 2016-03-09T20:08:11+01:00
title: Commands
weight: 15
---

## Using the CLI

### Connecting to Ernest

First specify the IP address of our Ernest Server:

```
$ ernest target https://<Ernest Server IP>
Target set
```

Now we should login to Ernest with the built-in admin account:

```
$ ernest login
Username: admin
Password: *******
```

### Projects

Ernest uses the concept of a project to group together one or more environments. A project can specify the default infrastructure service provider for those projects, although this can be overriden on at the environment level. An example of creating a project in Ernest for Amazon Web Services is:

```
$ ernest project create aws --region <region> --secret_access_key <secret_access_key> --access_key_id <access_key_id> <project_name>
```

The credentials of a project can be changed with the `updated` command. A project that does not contain any environments can be `deleted`.

### Environments

#### What is an Environment?

With Ernest an environment is a collection of networking, virtual machines, and configuration options. An environment is defined in YAML format, and you can see examples of this in the provider-specific documentation.

#### How to create/modify Environments

Once we have created a project and defined our environment in YAML the process of building the environment is:

```
$ ernest env apply <yaml_file>
```

Once the environment is created we can modify it by changing the YAML and then re-applying it. Ernest will calculate the differences and apply only those changes necessary to transition the environment from the current state to the requested state.

Finally you can also delete environments if needed:

```
$ ernest env delete <project_name> <environment_name>
```

#### View Environment State and History

Ernest stores the current state of each environment, as well as the historical states. You can list the available environments with:

```
$ ernest env list
```

For a given environment you can see the history of all builds:

```
$ ernest env history <project_name> <environment_name>
```

We can view the YAML for a specific build by:

```
$ ernest env definition <project_name> <environment_name> --build <build_id>
```

We can view the difference between two builds:

```
$ ernest env diff <project_name> <environment_name> <build_id 1> <build_id 2>
```

We can revert to a previous build:

```
$ ernest env revert <project_name> <environment_name> <build_id>
```

We can view provider-generated information related to our environment:

```
$ ernest service info <service name>
```

### Users and Roles

Users are created by an admin account:

```
$ ernest user create <user> <password>
User <user> successfully created
```

Access to projects and environments is controlled by roles. Ernest supports two different roles:

* `owner` can view, modify, and delete the project or environment
* `reader` can view the project or environment

Anyone with the `owner` role for a project or environment may set or unset roles for other users.

Assigning the `owner` role for a project to a user is done like this:

```
$ ernest role set -u user1 -r owner -p <project_name>
```

Assigning the `owner` role for an environment to a user is done like this:

```
$ ernest role set -u user1 -r owner -p <project_name> <environment_name>
```

### Logs

By default the Ernest server will write logs to a local file `logs/ernest.log`. Ernest can be configured to send logs to an external service such as logstash:

```
$ ernest preferences logger add logstash --hostname 10.50.1.1 --port 5000 --timeout 50000
```

An admin user can also stream the logs real-time via the CLI:

```
$ ernest log
```

### Notifications

Ernest can be configured to send notifications. For example messages can be sent to Slack:

```
$ ernest notify create my_notify slack '{"url":"https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"}'
```

### Usage

The usage command can produce a report showing the virtual machines managed by Ernest during a specified period:

```
$ ernest usage --from 2017-01-01 --to 2017-02-01 --output=report.log
```

## CLI Command Reference

### ernest target

```
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

```
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

```
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

```
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

```
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

```
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

```
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

```
NAME:
   ernest user disable - Disable available users.

USAGE:
   ernest user disable <username>

DESCRIPTION:
   Disable available users.

  Example:
   $ ernest user disable <user-name>
```

### ernest user info

```
NAME:
   ernest user info - Displays information about the specified user (current user by default).

USAGE:
   ernest user info [command options] [arguments...]

DESCRIPTION:


  Example:
   $ ernest user info
   $ ernest user info --user <user-name>


OPTIONS:
   --user value  Username
```

### ernest project list

```
NAME:
   ernest project list - List available projects.

USAGE:
   ernest project list

DESCRIPTION:
   List available projects.

   Example:
    $ ernest project list
```

### ernest project create aws

```
NAME:
   ernest project create aws - Create a new aws project.

USAGE:
   ernest project create aws [command options] <project-name>

DESCRIPTION:
   Create a new AWS project on the targeted instance of Ernest.

  Example:
   $ ernest project create aws --region us-west-2 --access_key_id AKIAIOSFODNN7EXAMPLE --secret_access_key wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY my_project

   Template example:
    $ ernest project create aws --template myproject.yml myproject
    Where myproject.yaml will look like:
      ---
      fake: true
      access_key_id : AKIAIOSFODNN7EXAMPLE
      secret_access_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
      region: us-west-2


OPTIONS:
   --region value, -r value             Project region
   --access_key_id value, -k value      AWS access key id
   --secret_access_key value, -s value  AWS Secret access key
   --template value, -t value           Project template
   --fake, -f                           Fake project
```

### ernest project create azure

```
NAME:
   ernest project create azure - Create a new azure project.

USAGE:
   ernest project create azure [command options] <project-name>

DESCRIPTION:
   Create a new Azure project on the targeted instance of Ernest.

  Example:
   $ ernest project create azure --region westus --subscription_id SUBSCRIPTION --client_id USER --client_secret PASSWORD --tenant_id TENANT --environment public my_project

   Template example:
    $ ernest project create azure --template myproject.yml myproject
    Where myproject.yaml will look like:
      ---
      fake: true
      region: westus
      subscription_id: SUBSCRIPTION
      client_id: USER
      client_secret: PASSWORD
      tenant_id: TENANT
      environment: public


OPTIONS:
   --region value, -r value           Project region
   --subscription_id value, -s value  Azure subscription id
   --client_id value, -c value        Azure client id
   --client_secret value, -p value    Azure client secret
   --tenant_id value, -t value        Azure tenant_id
   --environment value, -e value      Azure environment. Supported values are public(default), usgovernment, german and chine
   --fake, -f                         Fake project
```

### ernest project create vcloud

```
NAME:
   ernest project create vcloud - Create a new vcloud project.

USAGE:
   ernest project create vcloud [command options] <project-name>

DESCRIPTION:
   Create a new vcloud project on the targeted instance of Ernest.

   Example:
    $ ernest project create vcloud --user username --password xxxx --org MY-ORG-NAME --vse-url http://vse.url --vcloud-url https://myernest.com --public-network MY-PUBLIC-NETWORK myproject

   Template example:
    $ ernest project create vcloud --template myproject.yml myproject
    Where myproject.yaml will look like:
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
   --template value        Project template
   --fake                  Fake project
```

### ernest project update aws

```
NAME:
   ernest project update aws - Updates the specified AWS project.

USAGE:
   ernest project update aws [command options] <project-name>

DESCRIPTION:
   Updates the specified AWS project.

   Example:
    $ ernest project update aws --access_key_id AKIAIOSFODNN7EXAMPLE --secret_access_key wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY my_project


OPTIONS:
   --access_key_id value      Your AWS access key id
   --secret_access_key value  Your AWS secret access key
```

### ernest project update azure

```
NAME:
   ernest project update azure - Updates the specified Azure project.

USAGE:
   ernest project update azure [command options] <project-name>

DESCRIPTION:
   Updates the specified Azure project.

   Example:
   $ ernest project update azure --subscription_id SUBSCRIPTION --client_id USER --client_secret PASSWORD --tenant_id TENANT --environment public my_project


OPTIONS:
   --subscription_id value, -s value  Azure subscription id
   --client_id value, -c value        Azure client id
   --client_secret value, -p value    Azure client secret
   --tenant_id value, -t value        Azure tenant_id
   --environment value, -e value      Azure environment. Supported values are public(default), usgovernment, german and chine
```

### ernest project update vcloud

```
NAME:
   ernest project update vcloud - Updates the specified VCloud project.

USAGE:
   ernest project update vcloud [command options] <project-name>

DESCRIPTION:
   Updates the specified VCloud project.

   Example:
    $ ernest project update vcloud --user <me> --org <org> --password <secret> my_project


OPTIONS:
   --user value      Your VCloud valid user name
   --password value  Your VCloud valid password
   --org value       Your vCloud Organization
```

### ernest project delete

```
NAME:
   ernest project delete - Deletes the specified project.

USAGE:
   ernest project delete <project-name>

DESCRIPTION:
   Deletes the name specified project.

   Example:
    $ ernest project delete my_project
```

###ernest project info

```
NAME:
   ernest project info - Project information

USAGE:
   ernest project info

DESCRIPTION:
   Display specific project information.

   Example:
    $ ernest project info <my_project>
```

### ernest environment list

```
NAME:
   ernest environment list - List available environments.

USAGE:
   ernest environment list

DESCRIPTION:
   List available environments and shows its most relevant information.

   Example:
    $ ernest environment list
```

### ernest environment create

```
NAME:
   ernest environment create - Creates an empty environment based on a specific project

USAGE:
   ernest environment create [command options] <project> <environment>

DESCRIPTION:
   You must be logged in to execute this command.

   Examples:
    $ ernest env create my_project my_environment
    $ ernest env create --credentials project.yml my_project my_environment


OPTIONS:
   --credentials value                     will override project information
   --user value                            Your VCloud valid user name
   --password value                        Your VCloud valid password
   --org value                             Your vCloud Organization
   --vse-url value                         VSE URL
   --vcloud-url value                      VCloud URL
   --public-network value                  Public Network
   --vcloud-region value, --reg value      Project region
   --access_key_id value, -k value         AWS access key id
   --secret_access_key value, --sak value  AWS Secret access key
   --region value, -r value                Project region
   --subscription_id value, -s value       Azure subscription id
   --client_id value, -c value             Azure client id
   --client_secret value, -p value         Azure client secret
   --tenant_id value, -t value             Azure tenant_id
   --environment value, -e value           Azure environment. Supported values are public(default), usgovernment, german and chine
```

### ernest environment update

```
NAME:
   ernest environment update - Creates an empty environment based on a specific project

USAGE:
   ernest environment update [command options] <project> <environment>

DESCRIPTION:
   You must be logged in to execute this command.

   Examples:
    $ ernest env update --credentials project.yml my_project my_environment


OPTIONS:
   --user value                            Your VCloud valid user name
   --password value                        Your VCloud valid password
   --org value                             Your vCloud Organization
   --vse-url value                         VSE URL
   --vcloud-url value                      VCloud URL
   --public-network value                  Public Network
   --vcloud-region value, --reg value      Project region
   --access_key_id value, -k value         AWS access key id
   --secret_access_key value, --sak value  AWS Secret access key
   --region value, -r value                Project region
   --subscription_id value, -s value       Azure subscription id
   --client_id value, -c value             Azure client id
   --client_secret value, -p value         Azure client secret
   --tenant_id value, -t value             Azure tenant_id
   --environment value, -e value           Azure environment. Supported values are public(default), usgovernment, german and chine
```

### ernest environment apply

```
NAME:
   ernest environment apply - Builds or changes infrastructure.

USAGE:
   ernest environment apply [command options] <file.yml>

DESCRIPTION:
   Sends an environment YAML description file to Ernest to be executed.
   You must be logged in to execute this command.

   If the file is not provided, ernest.yml will be used by default.

   Examples:
    $ ernest env apply myenvironment.yml
    $ ernest env apply --dry myenvironment.yml


OPTIONS:
   --dry                                   print the changes to be applied on an environment intead of applying them
   --credentials value                     will override project information
   --user value                            Your VCloud valid user name
   --password value                        Your VCloud valid password
   --org value                             Your vCloud Organization
   --vse-url value                         VSE URL
   --vcloud-url value                      VCloud URL
   --public-network value                  Public Network
   --vcloud-region value, --reg value      Project region
   --access_key_id value, -k value         AWS access key id
   --secret_access_key value, --sak value  AWS Secret access key
   --region value, -r value                Project region
   --subscription_id value, -s value       Azure subscription id
   --client_id value, -c value             Azure client id
   --client_secret value, -p value         Azure client secret
   --tenant_id value, -t value             Azure tenant_id
   --environment value, -e value           Azure environment. Supported values are public(default), usgovernment, german and chine
```

### ernest environment delete

```
NAME:
   ernest environment delete - Destroy an environment.

USAGE:
   ernest environment delete [command options] <project> <environment_name>

DESCRIPTION:
   Destroys an environment by name.

   Example:
    $ ernest env delete <my_project> <my_environment>


OPTIONS:
   --force, -f  Hard ernest env removal.
   --yes, -y    Destroy an environment without prompting confirmation.
```

### ernest environment history

```
NAME:
   ernest environment history - Shows the history of an environment, a list of builds

USAGE:
   ernest environment history ernest-cli env history <my_project> <my_env>

DESCRIPTION:
   Shows the history of an environment, a list of builds and its status and basic information.

   Example:
    $ ernest env history <my_project> <my_env>
```

### ernest environment reset

```
NAME:
   ernest environment reset - Reset an in progress environment.

USAGE:
   ernest environment reset <env_name>

DESCRIPTION:
   Reseting an environment creation may cause problems, please make sure you know what are you doing.

   Example:
    $ ernest env reset <my_env>
```

### ernest environment revert

```
NAME:
   ernest environment revert - Reverts an environment to a previous state

USAGE:
   ernest environment revert [command options] <project> <env_name> <build_id>

DESCRIPTION:
   Reverts an environment to a previous known state using a build ID from 'ernest env history'.

   Example:
    $ ernest env revert <project> <env_name> <build_id>
    $ ernest env revert --dry <project> <env_name> <build_id>


OPTIONS:
   --dry  print the changes to be applied on an environment intead of applying them
```

### ernest environment definition

```
NAME:
   ernest environment definition - Show the current definition of an environment by its name

USAGE:
   ernest environment definition [command options] <project_name> <env_name>

DESCRIPTION:
   Show the current definition of an environment by its name getting the definition about the build.

   Example:
    $ ernest env definition <my_project> <my_env>


OPTIONS:
   --build value  Build ID
```

### ernest environment info

```
NAME:
   ernest environment info - $ ernest env info <my_env> --build <specific build>

USAGE:
   ernest environment info [command options] <project_name> <env_name>

DESCRIPTION:
   Will show detailed information of the last build of a specified environment.
  In case you specify --build option you will be able to output the detailed information of specific build.

   Examples:
    $ ernest env definition <my_project> <my_env>
    $ ernest env definition <my_project> <my_env> --build build1


OPTIONS:
   --build value  Build ID
```

### ernest environment monitor

```
NAME:
   ernest environment monitor - Monitor an environment creation.

USAGE:
   ernest environment monitor <project_name> <env_name>

DESCRIPTION:
   Monitors an environment while it is being built by its name.

   Example:
    $ ernest monitor <my_project> <my_env>
```

### ernest environment diff

```
NAME:
   ernest environment diff - $ ernest env diff <project_name> <env_name> <build_a> <build_b>

USAGE:
   ernest environment diff <env_aname> <build_a> <build_b>

DESCRIPTION:
   Will display the diff between two different builds

   Examples:
    $ ernest env diff <my_project> <my_env> 1 2
```

### ernest environment import

```
NAME:
   ernest environment import - $ ernest env import <my_project> <my_env>

USAGE:
   ernest environment import [command options] <env_name>

DESCRIPTION:
   Will import the environment <my_env> from project <project_name>

   Examples:
    $ ernest env import my_project my_env


OPTIONS:
   --project value  Project name
   --filters value  Import filters comma delimited list
```

### ernest preferences logger list

```
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

```
NAME:
   ernest preferences logger add - Creates / updates a logger based on its type.

USAGE:
   ernest preferences logger add [command options]

DESCRIPTION:
   Creates / updates a logger based on its types.

   Example:
    $ ernest preferences logger add basic --logfile /tmp/ernest.log
    $ ernest preferences logger add logstash --hostname 10.50.1.1 --port 5000 --timeout 50000
    $ ernest preferences logger add rollbar --token MY_ROLLBAR_TOKEN


OPTIONS:
   --logfile value   Specify the path for the loging file
   --token value     Rollbar token
   --env value       Rollbar environment
   --hostname value  Logstash hostname
   --port value      Logstash port (default: 0)
   --timeout value   Logstash timeout (default: 0)
```

### ernest preferences logger delete

```
NAME:
   ernest preferences logger delete - Deletes a logger based on its type.

USAGE:
   ernest preferences logger delete

DESCRIPTION:
   Deletes a logger based on its types.

   Example:
    $ ernest preferences logger delete basic
```

### ernest docs

```
NAME:
   ernest docs - Open docs in the default browser.

USAGE:
   ernest docs

DESCRIPTION:
   Open docs in the default browser.

   Example:
    $ ernest docs
```

### ernest setup

```
NAME:
   ernest setup - Use it to setup your ernest instance

USAGE:
   ernest setup [command options] [arguments...]

DESCRIPTION:
   This command will help you to setup your ernest instance by:
- [ ] configure ernest-cli target
- [ ] create a plain user
- [ ] create a group
- [ ] link the user to the group
- [ ] login as the newly created user.
- [ ] create a new project (optional)


OPTIONS:
   --user value, -u value      Admin user
   --password value, -p value  Admin password
   --target value, -t value    Ernest location
```

### ernest component

```
NAME:
   ernest component - Components related subcommands

USAGE:
   ernest component command [command options] [arguments...]

COMMANDS:
     list  List components on your project.

OPTIONS:
   --help, -h  show help
```

### ernest log

```
NAME:
   ernest log - Inline display of ernest logs.

USAGE:
   ernest log [command options]

DESCRIPTION:
   Display ernest server logs inline

   Example:
    $ ernest log
    $ ernest log --raw


OPTIONS:
   --raw  Raw output will be displayed instead of pretty-printed
```

### ernest usage

```
NAME:
   ernest usage - Exports an usage report to the current folder

USAGE:
   ernest usage [command options]

DESCRIPTION:


   Example:
    $ ernest usage --from 2017-01-01 --to 2017-02-01 --output=report.log
      A file named report.log has been exported to the current folder

    Example 2:
    $ ernest usage > myreport.log


OPTIONS:
   --from value    the from date the report will be calculated from. Format YYYY-MM-DD
   --to value      the to date the report will be caluclutated to. Format YYYY-MM-DD
   --output value  the file path to store the report
```

### ernest notify list

```
NAME:
   ernest notify list - List available notifications.

USAGE:
   ernest notify list

DESCRIPTION:
   List available notifications.

   Example:
    $ ernest notification list
```

### ernest notify create

```
NAME:
   ernest notify create - Create a new notify.

USAGE:
   ernest notify create <notify_name> <notify_type> <notify_config>

DESCRIPTION:
   Create a new notify on the targeted instance of Ernest.

   Example:
    $ ernest notify create <notify_name> <provider_type> <provider-details>


   Example:
   $ ernest notify create my_notify slack '{"url":"https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"}'
```

### ernest notify update

```
NAME:
   ernest notify update - Update a new notify.

USAGE:
   ernest notify update <notify_name> <notify_config>

DESCRIPTION:
   Update an existing notify on the targeted instance of Ernest.

   Example:
    $ ernest notify update <notify_name> <provider-details>


   Example:
   $ ernest notify update my_notify '{"url":"https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"}'
```

### ernest notify delete

```
NAME:
   ernest notify delete - Deletes an existing notify.

USAGE:
   ernest notify delete <notify_name>

DESCRIPTION:
   Deletes an existing notify on the targeted instance of Ernest.

   Example:
    $ ernest notify delete <notify_name>


   Example:
   $ ernest notify delete my_notify
```

### ernest notify add

```
NAME:
   ernest notify add - Add service to an existing notify.

USAGE:
   ernest notify add <service_name> <notify_name>

DESCRIPTION:
   Adds a service to an existing notify.

   Example:
    $ ernest notify add <service_name> <notify_name>


   Example:
   $ ernest notify add my_service my_notify
```

### ernest notify remove

```
NAME:
   ernest notify remove - Removes service to an existing notify.

USAGE:
   ernest notify remove <service_name> <notify_name>

DESCRIPTION:
   Removes a service to an existing notify.

   Example:
    $ ernest notify remove <service_name> <notify_name>


   Example:
   $ ernest notify remove my_service my_notify
```

### ernest role set

```
NAME:
   ernest role set - ernest role set -u john -r owner -p project

USAGE:
   ernest role set [command options] [arguments...]

DESCRIPTION:
   Set permissions for a user on a specific resource

   Example:
    $ ernest roles set -u john -r owner -p my_project
    $ ernest roles set -u john -r reader -p my_project -e my_environment


OPTIONS:
   --user value, -u value         User to be authorized over the given resource
   --project value, -p value      Project to authorize
   --role value, -r value         Role type [owner, reader]
   --environment value, -e value  Environment to authorize
```

### ernest role unset

```
NAME:
   ernest role unset - ernest role unset -u john -r owner -p my_project

USAGE:
   ernest role unset [command options] [arguments...]

DESCRIPTION:
   Set permissions for a user on a specific resource

   Example:
    $ ernest roles set -u john -r owner -p my_project
    $ ernest roles set -u john -r reader -p my_project -e my_environment


OPTIONS:
   --user value, -u value         User to be authorized over the given resource
   --project value, -p value      Project to authorize
   --role value, -r value         Role type [owner, reader]
   --environment value, -e value  Environment to authorize
```

